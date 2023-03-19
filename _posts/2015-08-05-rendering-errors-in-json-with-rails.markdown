---
layout: post
title: "[Rails] 在 Ajax 中渲染错误信息"
date: 2015-08-05 23:23
---

在最近的开发中，越来越多地使用 Ajax 来提交表单输入。一涉及到表单输入，肯定要处理错误情况，并给用户反馈适当的纠正信息。之前一直不知道如何正确处理 Ajax 错误请求。经过一番搜索后，整理下自己认为较为正确的做法。

先给出常见的错误做法：

JavaScript:

```javascript
$.ajax({
  data: {},
  url: '/xxxx',
  type: 'POST',
  dataType: 'json',
  success: function(data){
    if(data.errors) {
      // show error messages
    } else {
     // show successful messages
    }
  }
});
```

Controller:

```ruby
def create
  @model = Model.new model_params
  if @model.save
    render json: @model
  else
    render json: { errors: @model.errors.full_messages }
  end
end
```

这种做法的问题所在是没有使用 HTTP 的错误状态码来表示数据有问题，这样就需要把错误信息嵌在成功的响应中返回，导致 JS 代码多了一个 if 分支来判断是否有错误信息。

**正确的做法如下**：

JavaScript:

```javascript
$.ajax({
  data: {},
  url: '/xxxx',
  type: 'POST',
  dataType: 'json',
  success: function(data){
    // show successful messages
  },
  error: function(xhr){
    var errors = $.parseJSON(xhr.responseText).errors
    // show error messages
  }
});
```

Controller:

```ruby
def create
  @model = Model.new model_params
  if @model.save
    render json: @model
  else
    render json: { errors: @model.errors.full_messages }, status: 4xx // 使用符合错误信息的状态码
  end
end
```

-EOF-
