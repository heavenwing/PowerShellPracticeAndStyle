待办：本节可能应合并到 [代码布局和格式](Code-Layout-and-Formatting.md)， 且基于 [#15](https://github.com/PoshCode/PowerShellPracticeAndStyle/issues/15)。我们应该删除或重写背景部分。

# READ-01 缩进您的代码

考虑如下代码示例：

```PowerShell
if ($this -gt $that) {
    Do-Something -with $that
}
```

现在考虑这个：

```PowerShell
if ($this -gt $that)
{
    Do-Something -with $that
}
```

两者都不比对方好。 问100个人他们喜欢的代码，会有大约一半喜欢其中一个。 现在，当你开始处理接受脚本块作为参数的命令时，由于PowerShell 解析语法的方式，事情可能变得更加复杂。 "错误"是错误的。 与上面的两个例子一样，脚本构建没有功能上的差别。

继续按照这种思路理解，以下基本上是大众共识的指导方针；它们不是硬性规定。 也就是说，有一些支持这些观点的论据，您应该在驳回这些想法之前考虑这些论点。

首先，正确格式化你的代码。 惯例是在构造内缩进，以使读者其更清楚什么“属于”一个构造。

```PowerShell
foreach ($computer in $computers) {
    Do-This
    Get-Those
}
```

如果您不仔细格式化，可能会受到指责。

# READ-02 避免反引号

请考虑：

```PowerShell
Get-WmiObject -Class Win32_LogicalDisk `
              -Filter "DriveType=3" `
              -ComputerName SERVER2
```

一般来说，社区认为您应该尽可能避免将这些反引号用作“行继续符”。 它们难以阅读，容易错过，也容易打错。 另外，如果您在上述示例中的背面后添加额外的空格，则命令将无法工作。 由此产生的错误很难与实际问题相关联，从而使调试问题变得更加困难。

下面是一个替代方案：

```PowerShell
$GetWmiObjectParams = @{
    Class = "Win32_LogicalDisk"
    Filter = "DriveType=3"
    ComputerName = "SERVER2"
}
Get-WmiObject @GetWmiObjectParams
```

这种技术叫做 _splatting_。 它可以让您在不使用反引号的情况下获得同样漂亮的、中断的格式。 您还可以在几乎任何逗号、pipe字符或分号之后换行而不使用反引号。

反引号并不是普遍讨厌的 - 但它可能不方便。 如果您必须将它用于换行，那么请使用它。 但尽量不要这样做。
