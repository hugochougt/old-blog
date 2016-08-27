---
layout: post
title: "RSpec 使用一周小结"
date: 2016-08-20 15:30
comments: true
---

过去一个星期，在开发中使用 RSpec 对系统的 API 进行了功能测试。在这个过程中现学现用，Rain 和 MC 对我写的测试提出了很多评审建议，现在记录下来，跟大家分享。

## API 功能测试套路

1. setup data & assert original state
2. do something
3. assert response
4. assert new state

例如要测试用户注册，那就要先断言系统中没有用户，用户注册成功后，再断言系统中多了一个新用户。

## 测试代码 code review 评论

### 1. 不要 hack 原生方法

在测试 App API 的时候，有时候需要在请求头（或者其他地方）带上校验信息，为了不在每次请求的时候都重复准备校验信息，一开始我参考 [ruby-china](https://github.com/ruby-china/ruby-china) 的做法：

``` ruby
# File: https://github.com/ruby-china/ruby-china/blob/master/spec/support/api_v3_support.rb#L27

[:get, :post, :put, :delete, :head].each do |method|
  class_eval <<-EOV
    def #{method}(path, parameters = nil, headers = nil)
      # override empty params and headers with default
      parameters = combine_parameters(parameters, default_parameters)
      headers = combine_parameters(headers, default_headers)
      super(path, params: parameters, headers: headers)
    end
  EOV
end
```

然后就收到 Rain 的评审批注：

> 这是hacking， 不要直接hack原生的helper方法，而是使用扩展方式，例如这样 `api_post` 或者

> ApiConsumer.config(app_config)

> ApiConsumer.post()

> 建议这样，更方便测试不同的app_config，也可以为将来要做API的SDK做铺垫。

后来 Rain 还为此写了个 mr：

``` ruby
module AppApiV1Support
  extend ActiveSupport::Concern
  included do
    # usage example:
    #   let(:app_client) { api_consumer.config({id: '123', secret: '12345678'}) }
    #   app_client.post '/app_api/v1/consumers', consumer
    let(:api_consumer) { ApiConsumer.new(self) }
  end

  class ApiConsumer
    attr_accessor :api_config

    def initialize(example)
      @example = example
      self.api_config = {}
    end

    def config(new_config)
      self.api_config.merge!(new_config)
      self
    end

    # define request methods like get/post to send default headers
    [:get, :post, :put, :delete, :head].each do |method|
      class_eval <<-EOV
        def #{method}(path, parameters = nil, headers = nil)
          # merge headers with default
          headers = (headers || {}).merge(default_headers)
          @example.#{method}(path, parameters, headers)
        end
      EOV
    end

    private
      def default_headers
        {
          'App-Id': api_config[:id],
          'App-Secret': api_config[:secret],
          'App-Platform': api_config[:platform] || 'iOS',
          'App-Os-Version': api_config[:os_version] || '10.4',
          'App-Version': api_config[:version] || '1.0'
        }
    end
  end
end

```

### 2. 发送请求后，记得 reload 内存中的对象再做断言

项目有以下关联关系：

``` ruby
class Device < ActiveRecord::Base
  has_many :consumer_device_relations, dependent: :destroy
  has_many :consumers, through: :consumer_device_relations
end

class Consumer < ActiveRecord::Base
  has_many :consumer_device_relations, dependent: :destroy
  has_many :devices, through: :consumer_device_relations
end
```

然后有以下测试：

``` ruby
it 'binds device to a consumer' do
  expect(device.consumers).not_to include(consumer)

  # request to bind device to a consumer

  expect(response.status).to eq 200
  expect(device.consumers).to include(consumer)
end
```

如果不对 `device.consumers` reload 的话，最后一行的断言是会失败的。因为测试请求不会影响到内存中的对象。

正确的写法应该是：

``` ruby
expect(device.consumers.reload).to include(consumer)
```

也有另外的写法：`device.reload.consumers`。区别就是 `device.consumers.reload` 主要是测试 device.consumers 的变化，而 `device.reload.consumers` 主要测试 device 的变化。

### 3. context 写到 it 的同级之后

``` ruby
# Bad
describe 'POST /api/v1/users' do
  context 'when email is missing' do
    # test code
  end

  context 'when password does not match password_confirmation' do
    # test code
  end

  it 'creates a new user' do
    # test code
  end
end

# Good
describe 'POST /api/v1/users' do
  it 'creates a new user' do
    # test code
  end

  context 'when email is missing' do
    # test code
  end

  context 'when password does not match password_confirmation' do
    # test code
  end
end
```

### 4. “一个 it 对应多个 expects” VS “每个 expect 对应一个 it”

在测试用户授权设备的断言时，我是这样写的：

``` ruby
it 'authorizes the device' do
  # test code

  expect(response.status).to eq 200
  expect(JSON.parse(response.body)).to include({ 'authorized' => true })
  expect(device.reload.authorized).to eq true
end
```

然后 MC 评论：

> 这个it我一般分成3个it来写

> 几年前看书是说分3个写的，这样有多少个错误就能真实反映出来了，不然可能出现修好了第一个断言后，继续跑测试第二个断言又出错了

> 3个断言都有问题，如果写在一起，只会报一个错误

> 而且输出的内容更直观，如 it ‘should get 200 code’,  it ’should include authorized params'

Rain 回复：

> 是有推荐分3个写，但3个写会跑三次测试，有个性能问题。只要任何一个错，这个表示功能就是失败的，我们主要目标是让这个功能成功实现。如果3个测试1个出错，2个通过，会有这个功能2/3是正常的“错觉”，但实际这个功能就是不可用的

最后因为测试比较简单，三个断言就写在了一个 it block 里。

当然有复杂的情况是分开写的：

``` ruby
# File: spec/requests/web_api/v1/users_spec.rb

describe 'API V1', 'users', type: :request do
  let(:enterprise) { create(:enterprise) }
  let(:user)       { enterprise.owner }

  before do
    login_as user
  end

  describe 'GET /web_api/v1/users' do
    let!(:user_b) { create(:enterprise).owner }

    before do
      get '/web_api/v1/users'
    end

    it 'responses 200 status code' do
      expect(response.status).to eq 200
    end

    it "gets enterprise's all users" do
      user_ids = JSON.parse(response.body)['data'].map { |user| user['id'] }

      expect(user_ids.size).to eq 1
      expect(user_ids).to include user.id

      expect(user_ids).not_to include user_b.id
    end
  end
end
```

这里有几个需要注意的地方：

1. let 语句后的 block 最好能对齐。
2. 需要用 `let!` 强制创建一个不在测试企业内的用户，以确保断言的时候不会因为 `user_b` 还没创建，使得测试不像预期那样通过。

## 其他用到的测试相关 gem

1. [factory_girl_rails](https://github.com/thoughtbot/factory_girl_rails): 基本上使用 RSpec 测试的，都会同时使用 FactoryGirl 来作为准备测试数据的 fixture 了。
2. [webmock](https://github.com/bblimke/webmock): 如果你测试的功能涉及到请求第三方的网络服务，那可以使用这个 gem。

## 一些参考资料

1. [Structure of RSpec tests](http://jakegoulding.com/presentations/rspec-structure/)
2. [Better Specs { rspec guidelines with ruby }](http://betterspecs.org/)
3. [The RSpec Style Guide](https://github.com/reachlocal/rspec-style-guide)

**-EOF-**
