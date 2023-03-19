---
layout: post
title: "[Rails] 在 validates_presence_of 中使用 :allow_nil => true"
date: 2015-02-04 21:45
---

今天在实现一个当用户 update profile 时必须上传头像的功能，被 CTO review 代码后要求在 `validates_presence_of :avatar, on: :update` 语句后添加 `:allow_nil => true`。当时十分不解，问了 CTO 原因，CTO 解释的大意是如果不使用 `:allow_nil => true`，当用户已经上传了头像，在 edit profile 时就每次都上传头像了。

查阅了 API 文档后，摘抄其说明：

   >:allow_nil - If set to true, skips this validation if the attribute is nil

这样做之后就只在 :photo 是 blank 的情况不通过验证。

这里的知识点就是 Rails 的 ActiveModel::Validator 有 allow_nil 和 allow_blank 重要选项。附源码：

``` ruby
# File activemodel/lib/active_model/validator.rb, line 149
def validate(record)
  attributes.each do |attribute|
    value = record.read_attribute_for_validation(attribute)
    next if (value.nil? && options[:allow_nil]) || (value.blank? && options[:allow_blank])
    validate_each(record, attribute, value)
  end
end
```

另外就是查阅 API 时还留意到：

> If you want to validate the presence of a boolean field (where the real values are true and false), you will want to use validates_inclusion_of :field_name, :in => [true, false].

简单翻译下就是：如果想验证是否提供了布尔值，应该使用 `validates_inclusion_of :field_name, :in => [true, false]`。

原因是 Rails 中 `false.blank?` 返回 true。如果是使用 `validates_presence_of :field_name`，那么当 `field_name` 被设为 fasle 时，验证就会总是不通过。

-EOF-
