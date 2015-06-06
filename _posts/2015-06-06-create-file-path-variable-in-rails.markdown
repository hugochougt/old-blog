---
layout: post
title: "Create File Path Variable in Rails"
date: 2015-06-06 14:32
comments: true
---

最近偶尔需要在 Rails 项目里读取数据库的数据并导出成 csv 文件，code review 的时候 CTO 给我示范了创建文件路径变量的示例，现记录之。

```ruby
# Good :)
file_name = "xyz-#{foo}-#{bar}-#{Date.today}.csv"
file_path = Rails.root.join("tmp", file_name)
```

以下是被 code review 前我的写法：

```ruby
# Bad :(
file_path = [Rails.root, "tmp", "xyz-#{foo}-#{bar}-#{Date.today.to_s}.csv"].join('/')
```

前者的可读性明显比后者好。

**-EOF-**
