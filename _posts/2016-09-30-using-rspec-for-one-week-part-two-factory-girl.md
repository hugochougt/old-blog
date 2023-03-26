---
layout: post
title: "RSpec 使用一周小结（下）——使用 FactoryGirl 准备测试数据"
date: 2016-09-30 20:10
---

八月下旬发布了 [RSpec 使用一周小结（上篇）](http://www.beansmile.com/blog/posts/using-rspec-for-one-week-part-one)，文末预告了会有下篇介绍使用 FactoryGirl 准备测试数据，现在来了。

>在自动化测试中，准备测试数据是最重要也是最麻烦的，因此我们需要一个好的管理工具来辅助生成测试数据。Rails 中的默认测试数据构件是 fixture，就是一堆 yml 文件，使用简单但不方便组织复杂的测试用例数据。
>-- Rain

在这个前提下，我们使用了 `factory_girl` 这个gem，来做测试数据管理。

## 在 Rails 中安装 FactoryGirl

``` ruby
group :development, :test do
  # Provides Rails integration for factory_girl
  # https://github.com/thoughtbot/factory_girl_rails
  gem 'factory_girl_rails', '~> 4.7'
end
```

在我们 Beansmile 里，所有 Rails 项目的 `Gemfile` 都要求在 gem 前加上一句简介、源码地址，然后指定 gem version 。一句简介、源码地址可以方便其他同事了解 gem 的基本用途，以及快速找到 gem 的更多信息，指定 gem version 可以避免版本冲突。

默认放置 factories 的文件夹是 `test/factories`。如果你使用了 RSpec 作为测试框架，那默认的 factories 文件夹则是 `spec/factories`。后面我们讨论使用 FactoryGirl 都是在测试框架是 RSpec 的前提下。

如果你愿意，可以将所有的 factories 都定义在 `spec/factories.rb` 文件中。但是在 Beansmile 我们都不会这样做，而是将每个 factories 以模型名复数的形式，单独保存一个文件在 `spec/factories` 文件夹下。例如有 User 和 Article 模型，那对应的 facotry 则分别是 `spec/factories/users.rb` 和 `spec/factories/articles.rb`。

## 用法

### 定义 factories

``` ruby
# file: spec/factories/users.rb
FactoryGirl.define do
  # 根据 :user symbol 猜测使用 User model
  factory :user do
    first_name "John"
    last_name  "Doe"
    sequence(:email) { |n| "user#{n}@example.com" } # 只要模型中有唯一性验证，就可以使用序列。
    admin false
  end

  # 指明使用 User model
  factory :admin, class: User do
    first_name "Admin"
    last_name  "User"
    sequence(:email) { |n| "admin#{n}@example.com" }
    admin true
  end
end
```

### 创建 factories

``` ruby
# 创建一个未保存到数据库的实例
user = FactoryGirl.build(:user) # not saved to db

# 创建一个已保存到数据库的实例
user = FactoryGirl.create(:user) # saved to db

# 返回一个包含用于创建 User 实例的相关属性的 hash
user_attrs = FactoryGirl.attributes_for(:user)
# => {:first_name=>"John", :last_name=>"Doe", :email=>"user1@example.com", :admin=>false}

# 覆盖 factory 字段的默认值
user = FactoryGirl.build(:user, first_name: "Joe")
user.first_name
# => "Joe"
```

### 关联关系

假设有以下模型及关联关系：

```ruby
# File: app/model/user.rb
class User < ActiveRecord::Base
  has_many :articles
end

# File: app/model/article.rb
class Post < ActiveRecord::Base
  belongs_to :user
end
```

那么在已经有 user factory 的前提下，可以使用以下简写方式创建 post 的 factory：

```ruby
FactoryGirl.define do
  factory :post do
    sequence(:title) { |n| "Post title#{n}" }
    user
  end
end
```

也可以使用完整的写法关联对应的 factory，同时覆盖一些默认属性：

```ruby
FactoryGirl.define do
  factory :post do
    sequence(:title) { |n| "Post title#{n}" }
    association :user, factory: :user, last_name: "Writely"
  end
end

post = FactoryGirl.build(:post)
post.user.last_name # => "Writely"
```

### Transients

`transient` 可用来定义临时/假*属性*。用 `transient` 定义的是属性名字。

```ruby
FactoryGirl.define do
  factory :user do
    first_name "John"
    last_name  "Doe"
  end

  transient do
    with_posts false
  end

  after :create do |user, options|
    create_list(:post, 3, user: user) if options.with_posts
  end
end

# Usage:
create(:user, with_posts: true)
```

### Traits

`trait` 是用来定义特征*数据样例*的。用 `trait` 定义的是数据样例名字，命名一般是使用形容词或名词，如“未确认的(用户)”。

`trait` 是非常实用的、用于 DRY 测试用例数据的方法，在测试过程中必定会用到，其用法有以下几种：

定义特殊用例，例如 `注册了没有但验证邮箱的用户` 、 `被锁定的用户` 。有些测试场景需要用到这些特殊特征的用户时，就可以使用 `trait` 来定义：

  ``` ruby
  FactoryGirl.define do
    factory :user do
      # 默认用户是已经校验过的，confirmed_at 不为空
      confirmed_at { Time.now }

      # 注册了没有但验证邮箱的用户
      trait :unconfirmed do
        confirmed_at nil
      end

      # 被锁定的用户
      trait :locked do
        locked_at { Time.now }
      end
    end
  end

  # Usage:
  create :user, :unconfirmed
  create :user, :locked
  ```

可组合使用：

```ruby
create :user, traits [:unconfirmed, :locked]
```

`trait` 可嵌套使用：

  ``` ruby
  FactoryGirl.define do
    # 正常用户用 `create :user`
    factory :user do
      confirmed_at { Time.now }

      # 没验证邮箱的用户
      trait :unconfirmed do
        confirmed_at nil
      end

      # 没验证邮箱且注册了1个月以上的认为是“僵尸用户”
      trait :zombie do
        unconfirmed
        created_at { 31.days.ago }
      end
    end
  end

  # Usage:
  create :user, :zombie
  ```

接受 `transient` 参数，例如有些用例指定“僵尸用户”的注册时间：

  ``` ruby
  FactoryGirl.define do
    # 正常用户用 `create :user`
    factory :user do
      confirmed_at { Time.now }

      # 没验证邮箱的用户
      trait :unconfirmed do
        confirmed_at nil
      end

      # 没验证邮箱 且 注册了1个月以上的"僵尸"用户
      trait :zombie do
        unconfirmed
        created_at { 31.days.ago }

        before(:create) do |user, evaluator|
          # 如果有registered_at有值，就用来设置created_at
          self.created_at = evaluator.registered_at if evaluator.registered_at
        end
      end
    end
  end

  # Usage:
  create :user, :zombie, registered_at: 2.months.ago
  ```

以上这些是我们最近在一个项目写测试时用到的 FactoryGirl 特性简介，对于其更多详细的用法，可以参考 [FactoryGirl Getting Started](https://github.com/thoughtbot/factory_girl/blob/master/GETTING_STARTED.md)。

**20160923 Edited**:

Rain 提供了 `Transients` 和 `Traits` 两部分的说明及代码示例。

P.S. 首发于 [Beansmile 官方博客](http://beansmile.com/blog)，一周后转载于自己的博客。

## 参考资料

1. [FactoryGirl Getting Started](https://github.com/thoughtbot/factory_girl/blob/master/GETTING_STARTED.md)
2. [Cheatsheet for FactoryGirl](http://ricostacruz.com/cheatsheets/factory_girl.html)

-EOF-
