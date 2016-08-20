---
layout: post
title: "RSpec 使用一周小结"
date: 2016-08-20 15:30
comments: true
---

过去一个星期，在开发中使用 RSpec 对使用 Grape 编写的 API 进行了功能测试。在这个过程中现学现用，CTO 和同事的对我写的测试提出了很多评审建议，现在记录下来，以备日后查看。

## API 功能测试套路

  setup data & assert original state
  do something
  assert response
  assert new state

例如要测试用户注册，那就要先断言系统中没有用户，用户注册成功后，再断言系统中多了一个新用户。

## 测试代码 code review 评论

### 1. 不要 hack 原生方法

在测试 App API 的时候，都需要在请求头（或者其他地方）带上校验信息，为了不在每次请求的时候都重复准备校验信息，一开始我参考 [ruby-china](https://github.com/ruby-china/ruby-china) 的做法：

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

然后就收到 CTO 的评审批注：

  这是 hacking，不要直接 hack 原生的 helper 方法，而是使用扩展方式，例如这样 `api_post` 或者：
  AppClient.config(app_config)
  AppClinet.post(path, params, headers)

P.S. 我对批注内容略做了修改。

### 2. 发送请求后，记得 reload 内存中的对象在做断言

假设有以下关联关系：

``` ruby
class Customer < ActiveRecord::Base
  has_many :orders, dependent: :destroy
end

class Order < ActiveRecord::Base
  belongs_to :customer
end
```

然后有以下测试：

``` ruby
let(:customer) { FactoryGirl.create(:suctomer) }

it 'creates new order of customer' do
  expect(customer.orders.size).to eq 0

  post '/URL/for/new/order', params, headers

  expect(response.status).to eq 200
  expect(customer.orders.size).to eq 1
end
```

如果不对 customer reload 的话，最后一行的断言应该是会失败的。因为测试请求理论上不会影响到内存中的 customer。

正确的写法应该是：

``` ruby
expect(customer.orders.reload.size).to eq 1
```

### 3. context 写到 it 的同级之后

```ruby
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

还有一些在 slack 中的讨论，比较分散，而且还要分情况看，就不再展开了。

## 其他用到的测试相关 gem

1. [factory_girl_rails](https://github.com/thoughtbot/factory_girl_rails): 我猜基本上使用 RSpec 测试的，都会同时使用 FactoryGirl 了。
2. [webmock](https://github.com/bblimke/webmock): 如果你测试的功能涉及到请求第三方的网络服务，那可以使用这个 gem。

## 一些参考资料

1. [Structure of RSpec tests](http://jakegoulding.com/presentations/rspec-structure/)
2. [Better Specs { rspec guidelines with ruby }](http://betterspecs.org/)
3. [The RSpec Style Guide](https://github.com/reachlocal/rspec-style-guide)

**-EOF-**
