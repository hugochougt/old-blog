---
layout: post
title: "写给周佳琦小朋友的多密码教程"
date: 2016-01-31 18:28
comments: true
---

### 确定 root key

选择任何一个你能记住的英文单词或者拼音作为 root key。当然你记不住单词的话，root key 可以用 `zhoujiaqi`。

现在就假设 root key 是 `zhoujiaqi` 吧。

### 加上每个帐号的 special key

除了 root key 外，对于每个网站的密码，还需要加上唯一的 special key。可以是跟该网站相关的信息，例如网站域名（去掉 .com、.net 等后缀）或者创始人姓名拼音。然后这个 special key 加在 root key 的前面或者后面可以自己决定。

例如，对于淘宝，可以选域名 `taobao` 作为 special key，加在 root key 后面就是 `zhoujiaqitaobao`。

### 密码转换规则

元音字母 `aeiou` 根据以下规则进行转换，括号里是助记说明。

  1. a => 4 (A 的变体)
  2. e => 3 (E 的左右反过来)
  3. i => ! (英文的叹号，i 上下翻过来)
  4. o => 0 (数字零)
  5. u => 1_1 (两个数字1中间加个下划线，其实就是字母 u 的图画化)
  6. 第一个以及最后一个非元音字母大写

对前两步生成的 key，应用以上规则，淘宝网的密码就是 `Zh01_1j!4q!t40B40`。

**-EOF-**
