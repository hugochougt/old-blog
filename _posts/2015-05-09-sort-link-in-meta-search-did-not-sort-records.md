---
layout: post
title: "meta_search 的 sort_link 方法没有对数据进行排序"
date: 2015-05-09 15:36
tags: [Rails, meta_search]
---

## 功能需求

管理员在后台可以对用户的 Last Active Time 按顺序或倒序排序。

## [meta_search](https://github.com/activerecord-hackery/meta_search)

meta_search 的用法很简单：

```ruby
# controller
def index
  @search = User.search(params[:search])
  @users = @search.relation # Retrieve the relation, to lazy-load in view
end
```

```ruby
# view
<%= sort_link @search, :last_active_at, "Last Active Time" %>
```

## "Bug"

在页面不管怎么点击 sort link，出来的结果都是一样，并没有按预期的按照 `last_active_at` 的先后进行排序。

## Debug

经过 CTO 的指点，在 `@search` 变量上调用了 `to_sql` 方法，得到了 SQL 语句的结尾有 "ORDER BY id DESC"。原来没有按照 `last_active_at` 字段进行排序的原因在于 User model 里添加了 `default_scope { order('id DESC') }` 语句。

## 解决方法

Rails 提供了 `unscoped()` 来取消 default_scope 的副作用。

> ActiveRecord::Scoping::Default::ClassMethods.unscoped():
>
> Returns a scope for the model without the previously set scopes.

```ruby
# solution
def index
  @search = User.unscoped.search(params[:search])
  @users = @search.relation # Retrieve the relation, to lazy-load in view
end
```

-EOF-
