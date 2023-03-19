---
layout: post
title: "Difference between attr_accessor and attr_accessible"
date: 2015-05-10 11:06
tags: [Rails, Ruby, attr_accessor, attr_accessible, code review]
---

某次 code review，CTO 指出某个 field 不能放到 `attr_accessible` 里，因为它是支持 mass-assigment 的，会导致安全问题。直到那个时候，我才留意到原来 attr\_accessible 是不同于 attr\_accessor 的。

`attr_accessible` 是 Rails 3 里才有的方法（Rails 1, 2 里我不知道有没有），Rails 4 之后去掉了该方法，使用 [strong parameters](http://api.rubyonrails.org/classes/ActionController/StrongParameters.html)。

## Ruby's attr_accessor

### 方法签名

> Module#attr_accessor(symbol, ...) -> nil
>
> Module#attr_accessor(string, ...) -> nil
>
> Defines a named attribute for this module, where the name is *symbol.id2name*, creating an instance variable (@name) and a corresponding access method to read it. Also creates a method called name= to set the attribute. String arguments are converted to symbols.

简单翻译一下就是：`attr_accessor` 会创建实例变量及其对应的 getter/setter 方法。类似 attr_accessor 的方法叫做 **Class Macro**。在类似 Java 的静态语言中，当你定义了一个实例变量的时候，就需要为其写类似 getXXX() 和 setXXX() 的方法，虽然会有 Eclipse 帮你生成，但当有一大串的 getter/setter 方法在源码中的时候，还是会令人烦操的。Ruby 使用 Class Macro，很好地解决写一大段 getter/setter 方法的问题。

### 代码示例

```ruby
class Article
  attr_accessor(:title, :body)
end

Article.instance_methods(false).sort
# => [:body, :body=, :title, :title=]

article = Article.new
article.instance_variables # => []
article.title = "Life is short, you need Ruby"
article.instance_variables # => [:@title]

```

## Rails 3's attr_accessible

### 方法签名

> attr_accessible(*args)
>
> Specifies a white list of model attributes that can be set via mass-assignment.

[Mass-Assignment](http://guides.rubyonrails.org/v3.2.13/security.html#mass-assignment)

Mass-assignment 来龙去脉的中文文章：[從 Github 被 hack，談 Rails 的安全性（ mass-assignment ）](http://blog.xdite.net/posts/2012/03/05/github-hacked-rails-security/)

## 为什么我会把字段的 symbol 加到 attr_accessible 里？

因为我最开始以为 Rails 的 attr\_accessible 是 Ruby 的 attr\_accessor，看到代码里的 attr_accessible 后面带了一串属性，我就以为那是为属性生成 getter/setter 的宏，然后想都没有想就把新的字段加到最后里。囧...

*Rails 基本功还不扎实，是以为戒。*

**-EOF-**
