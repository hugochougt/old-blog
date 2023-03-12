---
layout: post
title: "[Ruby] 从文件中读取 JSON 数据并转为 Hash"
date: 2015-11-14 12:17
---

最近开发的时候需要调用另一服务的 API 来获取报表数据，但是网络请求比较慢，就想先把 JSON 数据保存一份在文件中，实现页面的时候就先读文件里的数据显示好了。

Google 后，找到 [Ruby: Reading a .json File to Hash](https://hackhands.com/ruby-read-json-file-hash/) 这篇很简明的教程，然后评论里指出了原文说的 `File.read` 创建 file handle 的错误。这个错误我觉得还是很普遍的，所以写本文记录一下。

数据示例：

```
{
  "name": "zhaqiang",
  "job": "Web Developer"
}
```

代码示例：

``` ruby
require 'json'

json_string = File.read('/path/to/data.json') # 这里返回的不是 file handle，只是 string
me = JSON.parse(json_string)

me.is_a? Hash
=> true
me["name"]
=> "zhaqiang"
me["job"]
=> "Web Developer"
```

**-EOF-**
