### 记录和注释

与该代码相矛盾的注释比没有注释更加严重。 当代码更改时，总是优先考虑及时更新注释。

注释应使用英文，并应是完整的句子。 如果注释很短，则可省略末尾的句号。

请记住，注释应该有助于你的推理和决策，而不是试图解释命令的作用。 除了正则表达式之外，编写良好的 PowerShell 可以一目了然。

```PowerShell
# Do not write:
# Increment Margin by 2
$Margin = $Margin + 2

# Maybe write:
# The rendering box obscures a couple of pixels.
$Margin = $Margin + 2
```

#### 块注释

不要过分注释。 除非你的代码特别模糊， 不要在每行前面加上一条注释 — — 这样做会破坏代码并使它更难阅读。  相反，写一个块注释。

块注释一般适用于跟随它们的部分或全部代码，并且缩进与该代码相同的级别。 每一行应以# 和单个空格开头。

如果块特别长(例如文档文本的情况)，建议使用 `<# ... #>` 语法，但你应该将注释放在自己的行上，并缩进：

```PowerShell
# Requiring a space makes things legible and prevents confusion.
# Writing comments one-per line makes them stand out more in the console.

<#
    .SYNOPSIS
        Really long comment blocks are tedious to keep commented in single-line mode.
    .DESCRIPTION
        Particularly when the comment must be frequently edited,
        as with the help and documentation for a function or script.
#>
```

#### 行内注释

与语句在同一行上的注释可能会让人分心，但是当它们没有说明显而易见的事情时，尤其是当您有几行短代码需要解释时，它们会很有用。

They should be separated from the code statement by at least two spaces, and ideally, they should line up with any other inline comments in the same block of code.

```PowerShell
$Options = @{
    Margin = 2          # The rendering box obscures a couple of pixels.
    Padding = 2         # We need space between the border and the text.
    FontSize = 24       # Keep this above 16 so it's readable in presentations.
}
```

#### 记录和注释

基于注释的帮助应该用简单的语言编写。

你没有为你的大学技术课写论文，写一些描述一个函数如何工作的东西。 避免不必要的大字，保持您的解释简短。 你不是想给任何人留下深刻印象，唯一会读到这篇文章的人只是想弄清楚如何使用这个功能。

如果您用外语写作，更简单的单词和更简单的句子结构更好，并且更有可能对母语读者有意义。

注释要完整，但简洁。

##### 位置

为了确保文档与函数保持一致，文档注释应该放在函数内部，而不是上面。 为了在更改函数时更难忘记更新它们，您应该将它们放在函数的顶部，而不是底部。

当然，这并不是说把它们放在别处是错误的——但这比较容易做，而且更难忘记更新。

##### 在Notes中显示细节

如果您想要提供关于您的工具如何工作的详细解释，请为此使用 `Notes` 部分。

##### 描述解决方案

每个脚本函数命令都应该至少有一个描述其函数的简短说明。 这是 `Synopsis`。

##### 记录每个参数

每个参数都应该被记录下来。 为了更容易地使注释与参数更改保持同步，参数文档注释的首选位置位于 `param` 块内，每个参数正上方。 例子可以在ISE代码片段中找到：

```powershell
param (
    # Param1 help description
    [Parameter(Mandatory = $true,
                ValueFromPipelineByPropertyName = $true,
                Position = 0)]
    $Param1,

    # Param2 help description
    [int]
    $Param2
)
```

也可以用其余的文档注释编写`.PARAMETER` 语句，但经验表明，如果您将它们放在更接近它们记录的代码的位置，它们更有可能保持最新。

##### 提供使用实例

您的帮助应该始终为每个主要使用案例提供一个示例。 一个“用法示例”仅仅是一个示例，说明您将输入到 Powershell 来运行脚本 - 在测试您的功能时，您甚至可以从命令行中剪切并粘贴一个。

```PowerShell
function Test-Help {
    <#
        .SYNOPSIS
            An example function to display how help should be written.

        .EXAMPLE
            Get-Help -Name Test-Help

            This shows the help for the example function.
    #>
    [CmdletBinding()]
    param (
        # This parameter doesn't do anything.
        # Aliases: MP
        [Parameter(Mandatory = $true)]
        [Alias("MP")]
        [String]$MandatoryParameter
    )

    <# code here ... #>
}
```

### DOC-01 编写基于注释的帮助

您应该始终在脚本和函数中写入基于注释的帮助。

基于注释的帮助格式如下：

```PowerShell
function Get-Example {
    <#
    .SYNOPSIS
        A brief description of the function or script.

    .DESCRIPTION
        A longer description.

    .PARAMETER FirstParameter
        Description of each of the parameters.
        Note:
        To make it easier to keep the comments synchronized with changes to the parameters,
        the preferred location for parameter documentation comments is not here,
        but within the param block, directly above each parameter.

    .PARAMETER SecondParameter
        Description of each of the parameters.

    .INPUTS
        Description of objects that can be piped to the script.

    .OUTPUTS
        Description of objects that are output by the script.

    .EXAMPLE
        Example of how to run the script.

    .LINK
        Links to further documentation.

    .NOTES
        Detail on what the script does, if this is needed.

    #>
```

当用户键入 `help Get-Example` 或 `Get-Example -?` 等时，会显示基于注释的帮助。

您的帮助应该是有帮助的。 也就是说，如果您已经编写了一个名为 `Get-LOBAppUser`的工具，不要帮助只说会"获取LOB App用户"。

**更多信息：** 您可以在 Powershell 中键入 `help about_Comment_Based_Help` 获得更多关于基于注释的帮助。
