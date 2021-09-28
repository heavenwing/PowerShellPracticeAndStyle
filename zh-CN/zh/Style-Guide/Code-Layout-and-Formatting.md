### 代码布局 & 格式

这些准则关于代码可读性。 其中一些规则是武断的，但它们是以几十年的方案拟订传统为基础的。 如果你可能不同意某些规则(而且应该始终遵循个别项目的规则)， 当我们要求您在关闭函数之后留下一条空行，或者在函数之前留下两条线， 是因为它使有经验的开发者更容易扫描你的代码。

#### 保持布局一致性

关于缩进、行长度和大小写的规则涉及基础代码之间的一致性。 长久的实践表明，当你看起来很熟悉并且注意力不会被细节分散时，阅读和理解代码就比较容易。 这意味着每个人都应该遵循一套规则。

我们并不期望每个人都遵守这些准则，个别项目的规则总是超越这些准则。 无论是由于遗留的原因，还是为了在一个单一项目中匹配多种语言的指南，不同的项目都可能有不同的风格指南。 由于目标是一致性，您应该始终遵守您所贡献的项目上任何的样式规则。

如果你确实有一个源码控制的旧项目，你决定修改代码来采用这些规则。 尝试在单个提交中做所有空白的更改，这些更改只能编辑空白处。 您不应该在文件上的空白处进行格式化，因为它是内容变化的 _一部分_ ，会使得更改变得难以发现。

#### 大小写惯例

PowerShell不是大小写敏感的，但我们遵循大小写惯例使得代码可读。 它们基于微软为.NET框架创建的 [大写协议](https://msdn.microsoft.com/en-us/library/ms229043) ，因为PowerShell 是一个 .NET脚本语言，并且 PowerShell cmdlet主要是按照这些准则用.NET语言撰写的。

##### 术语

* lowercase - 所有小写，没有单词分隔
* UPPERCASE - 所有大写，没有单词分隔
* PascalCase - 大写每个单词的第一个字母
* camelCase - 大写每个单词的第一个字母，但不包括第一个单词。

PowerShell 将 PascalCase 用于所有公共标识符：模块名称、函数或 cmdlet 名称、类、枚举和属性名称、公共字段或属性、全局变量和常量等。 事实上，由于PowerShell 命令的参数实际上是 .Net 类的属性，参数也使用 PascalCase 而不是camelCase。

PowerShell 关键词是用小写的(是，甚至包括 `foreach` 和 `dynamicparam`)， 以及诸如操作符 `-eq` 和 `-match` 等。 以评注为基础的帮助中的关键词是用UPPERCASE写成的，以便于关键词可以在繁复的文件被发现。

```powershell
function Write-Host {
    <#
    .SYNOPSIS
        Writes customized output to a host.
    .DESCRIPTION
        The Write-Host cmdlet customizes output. You can specify the color of text by using
        the ForegroundColor parameter, and you can specify the background color by using the
        BackgroundColor parameter. The Separator parameter lets you specify a string to use to
        separate displayed objects. The particular result depends on the program that is
        hosting Windows PowerShell.
    #>
    [CmdletBinding()]
    param(
        [Parameter(Position = 0, ValueFromPipeline = $true, ValueFromRemainingArguments = $true)]
        [psobject]$Object,

        [switch]$NoNewline,

        [psobject]$Separator,

        [System.ConsoleColor]$ForegroundColor,

        [System.ConsoleColor]$BackgroundColor
    )
    begin {
    ...
```

如前所述，PowerShell 使用 PascalCase 来处理 _所有的_ 公开标识符。 函数名称应该遵循PowerShell的 `Verb-Noun` 命名协议，在Verb 和 Noun中同时使用PascalCase 。

特殊情况是英文缩写为两字母，两字母都采用大写。 如变量 `$PSBoundParameters` 或命令 `Get-PSDrive`。 请注意([在 . NET准则中指定的](https://msdn.microsoft.com/en-us/library/ms229043#Anchor_1))这并不影响常用的大写(但不影响缩略语) "OK" 和 "ID" 。 您也不应将其延伸至复合缩略语， 例如：Azure's Resource Manager (RM) 在 `Start-AzureRmVM...` 中遇到Virutal Machine (VM) 时。

> 我们知道，由于各种原因，**许多**地方没有正确遵循这些约定——您应该考虑这些_例外_（例如 COM 互操作）或_错误_（例如`System.Data.SqlClient.SQLDebugging`），但这些不是您无视约定的理由。

如果您愿意，您可以对函数（或模块）中的变量使用camelCase，以区分_私有_变量和参数，但这是个人喜好问题。 共享变量应该通过使用其范围名称来区分，如 `$Script:PSBoundParameters` 或 `$Global:DebugPrefion`。 如果您正在使用camelCase作为一个变量，以双字母缩写开头(两字母均为大写)， 两个字母都应该设置为小写(例如 `adComputer`)。

#### One True Brace 风格

本指南向 K&R</a> 推荐所谓的“One True Brace Style” 变体，它要求每个可支撑的 _ 语句_ 都应在 _ 结尾处 _有左大括号，以及_行首 _有右大括号。</p> 

将小脚本块传递给参数时有一个值得注意的例外（其中 K&R 允许完全省略大括号），我们允许将整个语句放在一行中。



```powershell
enum Color {
    Black,
    White
}

function Test-Code {
    [CmdletBinding()]
    param (
        [int]$ParameterOne
    )
    end {
        if (10 -gt $ParameterOne) {
            "Greater"
        } else {
            "Lesser"
        }
    }
}

# An Exception case:
Get-ChildItem | Where-Object { $_.Length -gt 10mb }
```


这个建议的主要原因是实用的：遵循这个规则没有必要的例外，当代码按照这种风格编写时，_新行_代码可以插入任何两行之间，没有将大括号与其语句块分开来不小心破坏了代码的风险。 因此更容易追踪，并使错误变得不那么可能。

因为这种选择有些争议（约三分之一反对）， 它值得在此添加一些添加理由：第一：在一些历史控制台中， 有必要以这种方式撰写，很多早期的 PowerShell 代码仍然沿用这种风格。 第二：接受脚本模块的PowerShell 函数(例如 `ForEach-Object` and `Where-Object`) 是常见的， 并且是重要的基于 PowerShell 的重要域名语法如DSC 的_固有_部分 。 由于在这些情况下**需要**将左大括号放在行尾，因此唯一_一致_的选择是遵循 OTBS。



#### 总是从 CmdletBinding 开始

你所有的脚本或函数都应该以这个代码片段开始：



```powershell
[CmdletBinding()]
param ()
process {
}
end {
}
```


您始终可以删除或忽略其中一个块（或添加 `begin` 块）、添加参数和必要的验证等，但您应该**避免**编写脚本或函数没有 `[CmdletBinding()]`，并且您应该始终至少_考虑_使其接受pipeline输入。



#### 首选：param (), begin, process, end

以执行顺序写脚本使意图更加清楚。 由于没有功能上的原因让这些块打乱顺序（它们仍将按正常顺序执行），乱序编写它们可能会令人困惑，并使代码更难以维护和调试。

更明确的代码更易维护。 虽然PowerShell的允许舍弃end block的明确的名称(甚至有 `过滤器` 关键字将匿名块转换为 `进程` 块)， 我们建议不要使用这些功能，因为它导致代码不那么明确。



#### 缩进



##### 每个缩进级别使用四个 *空格*

通常您会按 `[Tab]` 键缩进，但大多数编辑器可以配置为插入空格而不是实际tag字符。 对于大多数编程语言和编辑器(包括PowerShell ISE)，默认是四个空格，这是我们建议的。 不同的团队和项目可能有不同的标准，在为某个项目工作时，你当然应该遵守这个项目的主导风格。



```powershell
function Test-Code {
    foreach ($exponent in 1..10) {
        [Math]::Pow(2, $exponent)
    }
}
```


持续行可以缩进超过 4 个空格(当你换行过长时)。 在这种情况下，您可能会缩进多个级别，甚至缩进奇数个空格以与前一行的方法调用或参数块对齐。



```powershell
function Test-Code {
    foreach ($base in 1,2,4,8,16) {
        foreach ($exponent in 1..10) {
            [System.Math]::Pow($base,
                               $exponent)
    }
}
```




#### 最大行长度

可能时，行限制为 115 个字符。

将行保持在较小的宽度，允许脚本在 _一个_ 方向上(从上到下) 读取，而无需水平滚动。 究竟是什么， 这个宽度应该是互联网上开发者最喜欢的争辩点之一(比emacs vs vi 或 gnu GPL vs MIT更多的分割) 。

在本指南中，最大行宽度使用两个特定来源：

默认情况下，PowerShell 控制台的宽度为 120 个字符，但它只允许在输出行上包含 119 个字符，并且在输入多行文本时，PowerShell 使用行继续提示： `>>>`，因此无论如何都将行长度限制为 116。

Github当前的最大行宽度根据您的浏览器和操作系统(即字体) 介于121至126之间。 但是，PowerShell 建议的 115 行长度甚至足以允许显示并排差异，而无需在当前“标准”1080p 显示器上滚动或换行。

这也是一项特别灵活的规则，当你为其他人的项目作出贡献时，你应该始终遵循项目的指导方针。  虽然我们大多数人都在用大屏显示器，但不是每个人都能看到大规模的或极大的字体。

避免长行的首选方法是在括号、方括号和大括号内使用 splatting(见 [Get-Help about_Splating](https://technet.microsoft.com/en-us/library/jj672955.aspx)) 和 PowerShell 在括号内的隐含行延续 - 方括号和方括号—— 应 **始终**使用这些方括号和方括号，并在适用时，这些应始终优先于反引号用于行延续，即使是字符串：



```powershell
Write-Host -Object ("This is an incredibly important, and extremely long message. " +
                         "We cannot afford to leave any part of it out, " +
                         "nor do we want line-breaks in the output. " +
                         "Using string concatenation lets us use short lines here, " +
                         "and still get a long line in the output")
```




#### 空白行和空格

用两个空行包围函数和类定义。

一个类内的方法定义被一个空白行包围。

一堆相关的单行代码之间可以省略空行（例如空函数）。

可以谨慎地使用额外的空行来分隔相关函数组，或在函数内指示逻辑部分（例如，在块注释之前）。

以单个空白行结束每个文件。



#### 尾随空格

行中不应有尾随的空格。 额外的空格会导致将来的编辑，其中唯一的更改是添加或删除空格，从而无缘无故地使更改分析变得更加困难。



#### 参数和运算符前后的空格

您应该在参数名称和运算符周围使用单个空格，包括比较运算符以及数学和赋值运算符，即使这些空格对于 PowerShell 正确解析代码不是必需的。

一个值得注意的例外是使用冒号传递值来切换参数时：



```PowerShell
# Do not write:
$variable=Get-Content $FilePath -Wait:($ReadCount-gt0) -First($ReadCount*5)

# Instead write:
$variable = Get-Content -Path $FilePath -Wait:($ReadCount -gt 0) -TotalCount ($ReadCount * 5)
```


另一个例外是使用 [Unary Operators](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_operators#unary-operators):



```PowerShell
# Do not write:
$yesterdaysDate = (Get-Date).AddDays( - 1)

$i = 0
$i ++

# Instead write:
$yesterdaysDate = (Get-Date).AddDays(-1)

$i = 0
$i++

# Same principle should be applied when using a variable.
$yesterdaysDate = (Get-Date).AddDays(-$i)
```




#### 特殊符号前后的空格

White-space is (mostly) irrelevant to PowerShell, but its proper use is key to writing easily readable code.

Use a single space after commas and semicolons, and around pairs of curly braces.

Subexpressions `$( ... )` and scriptblocks `{ ... }` should have a single space on the _inside_ of the braces or parentheses to improve readability by making code blocks stand out -- and to further distinguish scriptblocks from variable delimiter braces `${...}`

Avoid unnecessary spaces inside parenthesis or square braces.



```powershell
$Var = 1
"This is a string with one (${Var}) delimited variable."

"There are $( (Get-ChildItem).Count ) files."
```


Obviously, these rules should not be applied in such a way as to affect output.



#### Avoid Using Semicolons (`;`) as Line Terminators

PowerShell will not complain about extra semicolons, but they are unnecessary, and can get in the way when code is being edited or copy-pasted. They also result in extra do-nothing edits in source control when someone finally decides to delete them.

They are also unnecessary when declaring hashtables if you are already putting each element on its own line:



```PowerShell
# This is the preferred way to declare a hashtable if it extends past one line:
$Options = @{
    Margin   = 2
    Padding  = 2
    FontSize = 24
}
```
