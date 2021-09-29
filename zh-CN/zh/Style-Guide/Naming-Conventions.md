### 命名规范

一般而言，更倾向于对命令和参数使用完整的显式名称，而不是使用别名或简短形式。 有一些工具，例如 [PSScriptAnalyzer](https://github.com/PowerShell/PSScriptAnalyzer)'s `Invoke-Formater` 以及类似于 [Expand-Alias](https://github.com/PoshCode/ModuleBuilder/blob/master/PotentialContribution/ResolveAlias.psm1) 的脚本来修复许多, 但并非所有这些问题。

#### 使用每个命令的全名。

每个 PowerShell 脚本编写者都会学习实际的命令名称，但不同的人会学习和使用不同的别名（例如：对于 Linux 用户是 ls，对于 DOS 用户是 dir，gci ...）。  在您共享的脚本中，您应该使用更通用的完整命令名。 作为额外奖励，当您使用完整的Verb-Noun 名称时，GitHub 等站点将突出显示命令：

```PowerShell
# Do not write:
gps -Name Explorer

# Instead write:
Get-Process -Name Explorer
```

#### 使用完整的参数名称。

因为在 PowerShell 中有很多的命令，每个脚本编写者不可能知道每个命令。 所以读者可能不熟悉你正在使用的命令，因此，明确你的参数名是有用的。 如果将来的命令更改参数集，这也将有助于您避免出现错误。

```PowerShell
# Do not write:
Get-Process Explorer

# Instead write:
Get-Process -Name Explorer
```

#### 尽可能使用完整、明确的路径。

写脚本时，只有先前已经明确设置位置(在当前函数或脚本内)，才能安全使用 `...` 或 `.` 。 即使您明确设置了路径，在调用 .Net 方法或遗留/本机应用程序时也必须注意使用相对路径，因为它们将使用 `[Environment]::CurrentDirectory`, 它不会自动更新到 PowerShell 的当前工作目录 (`$PWD` ）。

因为对这些类型的错误进行故障排除很乏味（而且它们很容易被忽略），所以最好完全避免使用相对路径，而是在必要时将路径基于 $PSScriptRoot（脚本所在的文件夹）。

```PowerShell
# Do not write:
Get-Content .\README.md

# Especially do not write:
[System.IO.File]::ReadAllText(".\README.md")


# Although you can write:
Push-Location $PSScriptRoot
Get-Content README.md

# It would be better to write:
Get-Content -Path (Join-Path -Path $PSScriptRoot -ChildPath README.md)
# Or to use string concatenation:
Get-Content "$PSScriptRoot\README.md"

# For calling .net methods, pass full paths:
[System.IO.File]::ReadAllText("$PSScriptRoot\README.md")

# Optionally by calling Convert-Path
Push-Location $PSScriptRoot
[System.IO.File]::ReadAllText((Convert-Path README.md))

```

##### 避免使用 `~` 来代表主文件夹。

不幸的是，~ 的含义取决于执行时的“当前”提供者。 这并不是真正的风格问题，但对于您打算共享的代码来说，这是一个重要规则。 相反，使用 `${Env:UserProfile}` 或 `(Get-PSProvider -PSProvider FileSystem).Home`...

```PowerShell
PS C:\Windows\system32> cd ~
PS C:\Users\Name> cd HKCU:\Software
PS HKCU:\Software> cd ~
cd : Home location for this provider is not set. To set the home location, call "(Get-PSProvider 'Registry').Home = 'path'".
At line:1 char:1
+ cd ~
+ ~~~~
    + CategoryInfo          : InvalidOperation: (:) [Set-Location], PSInvalidOperationException
    + FullyQualifiedErrorId : InvalidOperation,Microsoft.PowerShell.Commands.SetLocationCommand
```


#### 另见《大小写惯例》

在代码布局和格式章节中，有一个关于 [大小写惯例](\PowerShellPracticeAndStyle\Style-Guide\Code-Layout-and-Formatting.md#Capitalization-Conventions) 的板块。