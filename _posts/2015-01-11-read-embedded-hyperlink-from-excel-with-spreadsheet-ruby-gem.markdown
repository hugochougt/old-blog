---
layout: post
title: "使用 spreadsheet gem 读取 Excel 单元格中内嵌的 hyperlink"
date: 2015-01-11 10:44
comments: true
---

### Task

读取 xls 后缀的 MS Excel 文件中单元格的内容及内嵌其中的 hyperlink。

### Solution

``` ruby
require 'spreadsheet'

excel = Spreadsheet.open "/path/to/excel.xls"
sheet = excel.worksheet "sheet_name"

sheet.each do |row|
  if row[0].respong_to?(:href)
    puts "#{row[0]}: #{row[0].href}"
  end
end
```

（假设第一列的单元格都内嵌了 hyperlink。）

[spreadsheet](https://github.com/zdavatz/spreadsheet) 文档中有写到：

    To access the values stored in a Row, treat the Row like an Array.

      row[0]

    -> this will return a String, a Float, an Integer, a Formula, a Link or a Date or DateTime object - or nil if the cell is empty.

如果某个单元格内嵌了 hyperlink，`row[i]` 就会直接返回 [Spreadsheet::Link](http://www.rubydoc.info/gems/ruby-spreadsheet/Spreadsheet/Link) class，然后可以通过 `#url` 或 `#href` (url with fragment appended if present) 来获取内嵌的 hyperlink。

### === tl;dr ===

上面是进过简化的内容，只专注标题的意思。下面就啰嗦一下，讲讲整个过程。

Task本来是要读取一个客户的 xlsx 格式的 Excel，里面是一列几千条的学校名字数据，学校名字内嵌了该校主页的链接（其中一些没有链接），最终目的是读取学校名和主页，存到数据库里以备后用。

一开始先是使用 spreadsheet gem 的，试了一下就发现其不能打开 xlsx 格式的 Excel。将源文件转换成 xls 格式后，能读取数据。然后看了文档很久也没有发现怎么在读取单元格的同时，读取其内嵌的 hyperlink。只是发现有 Spreadsheet::Link 这个类，但是文档里写的却是关于写单元格的说明，没有读取的说明。

后来通过使用 pry 来 debug，才发现原来当单元格内嵌了 hyperlink，`row[i]` 就会返回 Spreadsheet::Link class（通过运行 `row[i].class` 命令知道的）。虽然调用 `row[i].href` 能拿到链接了，但是返回值却有乱码（类似 "http://homepage.com/烫烫\u1234\u5678" 的字符串，前面纯 ASCII 字符部分是正确的，只是后面多了一些汉字和 unicode）。

发现读取的值不太对后，就转而尝试 [Roo](https://github.com/roo-rb/roo) 这个可以直接读取 xlsx 格式的 gem。看了一圈文档并写了一些实验性代码后，没有发现读取单元格内嵌链接的接口。

最终就又切换回使用 spreadsheet 这个 gem，使用 regex (最终实现的 regex: `/[:\.\/\w\d]+/`) 来只匹配 hyperlink 中的 ASCII 字符的方法来处理乱码。

**-EOF-**
