---
layout: post
title: Scala开发教程（一）【HelloWord】
category: Scala开发教程
date: 2014-11-26

---

Scala开发教程（一）【HelloWord】

标签： Scala 开发教程

##Scala是什么
Scala是一门现代的多范式编程语言，志在以简练、优雅及类型安全的方式来表达常用编程模式。它平滑地集成了面向对象和函数语言的特性。

<!-- more -->

>
- Scala是面向对象的：Scala是一个纯面向对象语言，在某种意义上来讲所有数值都是对象。对象的类型和行为是由class和trait来描述
的。Class的抽象可由子类化和一种灵活的基于mixin的组合机制（它可作为多重继承的简单替代方案）来扩展。
- Scala是函数式的： Scala还是一个函数式语言，在某种意义上来讲所有函数都是数值。Scala为定义匿名函数提供了一种轻量级的语法
，它支持高阶（higher- order）函数、允许函数嵌套、支持局部套用（currying）。Scala的case类及其内置支持的模式匹配模型代数类
型在许多函数式编程语言中 都被使用。
- Scala是静态类型的：Scala配备了一套富有表现力的类型系统，该抽象概念以一种安全的和一致的方式被使用。
- Scala是可扩展的：Scala的设计承认了实践事实，领域特定应用开发通常需要领域特定语言扩展。Scala提供了一个独特的语言组合机
制，这可以更加容易地以类库的形式增加新的语言结构：
  任何方式可以被用作中缀（infix）或后缀（postfix）操作符
  闭包按照所期望的类型（目标类型）自动地被构造
  两者结合使用可方便地定义新语句，无需扩展语法，也无需使用类似宏的元编程工具。
- Scala可与Java和.NET进行互操作：Scala 设计时就考虑了与流行编程环境良好交互，如Java 2运行时环境（JRE）和 .NET框架（CLR）
。特别是与主流面向对象语言，如Java和C#尽量无缝交互。Scala有像Java和C#一样的编译模型（独立编译，动态装载 类），允许访问成
千上万的高质量类库。

##HelloWord

{% highlight scala %}
object HelloWorld {
    def main(args: Array[String]) {
        println("Hello,World!")
    }
}
{% endhighlight %}

















