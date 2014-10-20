---
layout: post
title: "Setup Capybara in Rails"
date: 2014-08-26 22:51
comments: true
categories: [Rails, Capybara]
---

今天在新的 Rails 项目中首次使用 Capybara 进行集成测试时，报了「undefined method `visit' for ...」的 NoMethodError 错误。查看了 Capybara 的 GitHub [主页](https://github.com/jnicklas/capybara)，在 *Setup* 一节写着：

If the application that you are testing is a Rails app, add this line to your test helper file:

    require 'capybara/rails'


但是我不知道上述的 **test helper file** 指的是 spec/ 目录下的 `rails_helper.rb` 还是 `spec_helper.rb`。阅读了 SO 上的这个[帖子](http://stackoverflow.com/questions/15148585/undefined-method-visit-when-using-rspec-and-capybara-in-rails)排名第一的答案后，执行了下两步就可以使用 Capybara 的集成测试了：

 1. 在 rails_helper.rb 文件的 `require 'rspec/rails'` 代码下添加一行 `require 'capybara/rails'` 代码
 2. 在 rails_helper.rb 文件的 *RSpec.configure* 代码块中添加一行 `config.include Capybara::DSL` 代码



```ruby
# This file is copied to spec/ when you run 'rails generate rspec:install'
ENV["RAILS_ENV"] ||= 'test'
require 'spec_helper'
require File.expand_path("../../config/environment", __FILE__)
require 'rspec/rails'
require 'capybara/rails'
.
.
.
RSpec.configure do |config|
  .
  .
  .

  config.include Capybara::DSL
end
```

**-EOF-**

版权声明：自由转载-非商用-非衍生-保持署名 | [Creative Commons BY-NC-ND 3.0](http://creativecommons.org/licenses/by-nc-nd/3.0/deed.zh "CC 3.0")

