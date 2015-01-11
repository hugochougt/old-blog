---
layout: post
title: "使用 spreadsheet gem 读取 Excel 单元格中内嵌的 hyperlink"
date: 2014-01-11 10:44
comments: true
---

### Task

读取 xls 后缀的 MS Excel 文件中单元格的内容及内嵌其中的 hyperlink（假设第一列的单元格都内嵌了 hyperlink）。

### Solution

```
require 'spreadsheet'

excel = Spreadsheet.open "/path/to/excel.xls"
sheet = excel.worksheet "sheet_name" # or 0, 1, 3... access them with index or name

sheet.each do |row|
  if row[0].respong_to?(:href)
    puts "#{row[0]}: #{row[0].href}"
  end
end
```

[spreadsheet](https://github.com/zdavatz/spreadsheet) 文档中有写到：

  To access the values stored in a Row, treat the Row like an Array.

    row[0]

  -> this will return a String, a Float, an Integer, a Formula, a Link or a Date or DateTime object - or nil if the cell is empty.
。
如果某个单元格内嵌了 hyperlink，`row[i]` 就会直接返回 `(Spreadsheet::Link)[http://www.rubydoc.info/gems/ruby-spreadsheet/Spreadsheet/Link]` class，然后可以通过 `#url` 或 `#href`(url with fragment appended if present) 来获取内嵌的 hyperlink。

### === TL;DR ===

**-EOF-**
