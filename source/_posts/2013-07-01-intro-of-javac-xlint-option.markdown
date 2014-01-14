---
layout: post
title: "javac -Xlint 编译选项简介"
date: 2013-07-01 23:07
comments: true
categories: [Java, javac]
---

在看《JAVA 核心技术 卷I：基础知识》第11章《异常、日志、断言和调试》的时候，看到了有关 javac -Xlint 的内容，以前初学 Java 的时候，在命令行编译时经常会看到使用 javac -Xlint 重新编译的提示，但是当时又不理解，只是根据提示照做而已。现在看到相关介绍，就整理摘录下，以免日后又忘记了。

在终端执行`javac -Xlint`会出现以下提示信息：
    -Xlint                     Enable recommended warnings
    -Xlint:{all,cast,deprecation,divzero,empty,unchecked,fallthrough,path,serial,finally,overrides,-cast,-deprecation,-divzero,-empty,-unchecked,-fallthrough,-path,-serial,-finally,-overrides,none}Enable or disable specific warnings

"Enable recommended warnings"就是让编译器对一些普遍容易出现的代码问题进行检查。例如使用`javac -Xlint:fallthrough`这条编译命令的效果就是，当代码中的 switch 代码块缺少 break 时，编译器给出警告报告。

{% blockquote JAVA 核心技术 卷I：基础知识 %}
术语“lint”最初用来描述一种定位 C 程序中潜在问题的工具，现在通常用于描述查找可疑的、但不违背语法规则的代码问题的工具。
{% endblockquote %}

[lint wikipedia](http://en.wikipedia.org/wiki/Lint_\(software\))

-Xlint 各个选项的详细作用，请使用`man javac`查阅。

**-EOF-**

版权声明：自由转载-非商用-非衍生-保持署名 | [Creative Commons BY-NC-ND 3.0](http://creativecommons.org/licenses/by-nc-nd/3.0/deed.zh "CC 3.0")

如果觉得本文对你有帮助，可以请作者喝[咖啡](http://me.alipay.com/zhaqiang)
