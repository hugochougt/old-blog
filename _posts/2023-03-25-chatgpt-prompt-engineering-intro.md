---
layout: post
title: ChatGPT Prompt 咒语吟唱入门
date: 2023-03-25 21:24
---

网上有很多人分享 ChatGPT 的 prompt 模板，也有很多人通过视频来介绍 ChatGPT。但是如果不从根本上理解如何清晰、有效地给 ChatGPT 提 prompt，就不能高效地修改、应用这些模板，来满足自己的特殊需求。

要理解 prompt 咒语吟唱，首选的起点是查阅官方的 3 个在线文档：

1. [Text completion](https://platform.openai.com/docs/guides/completion/introduction)
2. [Best practices for prompt engineering with OpenAI API](https://help.openai.com/en/articles/6654000-best-practices-for-prompt-engineering-with-openai-api)
3. [Examples](https://platform.openai.com/examples)

以下内容是我根据官方的文档，结合自己的实际操作做的一些简单入门总结。

## 吟唱咒语（Prompt）的两个基本准则

1. 清晰地表达您的需求，通过指令、示例或两者结合向 ChatGPT 展示您想要的结果；
2. 提供高质量的数据和准确的文字（不要有错别字），确保有足够的示例，并对示例进行校对。

## 建议和示例

### 1. 先在咒语（Prompt）的开头描述指令，然后用 `###` 或 `"""` 隔开指令和上下文

```text
总结以下内容的重点

Text: ###
{多段文本内容}
###
```

### 2. 尽可能具体、详细地描述期望的内容，包括结果、长度、格式、文风等

不建议❌：

```text
以《登高》为标写一首诗。
```

建议✅：

```text
以哀愁的口吻写一首七言唐诗，标题为《登高》，风格参考唐代诗人杜甫。
```

两个咒语的输出对比：

![promtp-with-comparison](/images/posts/chatgpt-prompt/promtp-with-comparison.png)

## 3. 通过示例阐述期望的输出格式

不建议❌：

```text
提取以下内容的名字和门店名称

Text: {text}
```

建议✅：

```text
提取以下内容的名字和门店名称，以 Excel 表格输出

Excel 表头格式：序号、门店名称、名字

Text: ###
{多行文本}
###
```

下面截图示例的数据是我的一个朋友让我帮忙处理 100 多条不规则的文件名数据。

![prompt-with-output-format](/images/posts/chatgpt-prompt/prompt-with-output-format.png)

## 4. 减少“空洞”和不准确的描述

不建议❌：

```text
产品的描述应该相当简短，仅需几句话，不要太多文字。
```

建议✅：

```text
用 3 到 5 句话来描述产品。
```

更多咒语示例，可以查阅以下链接：

- [FlowGPT](https://flowgpt.com/)
- [ChatGPT 提示语](https://prompts.fresns.cn/)
- [awesome-chatgpt-prompts](https://github.com/f/awesome-chatgpt-prompts)
