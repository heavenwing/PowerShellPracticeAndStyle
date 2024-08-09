### 函数

在函数中避免使用 `return` 关键字。 只需将对象变量单独放置即可。

声明简单函数时，在函数名和参数之间留一个空格。

```PowerShell
function MyFunction ($param1, $param2) {
    ...  
}
```

### 高级函数

对于高级函数和脚本，使用 **\<verb\>-\<noun\>** 的格式命名。 对于已批准动词的列表，cmdlet `Get-Verb` 将会列出他们。 在名词方面，它可以由多个使用 Pascal Case 的词连接组成，并且只能由单数名词组成。

高级函数不使用关键字 `return` 返回一个对象。

在高级函数中，您返回 `Process {}` 块内的对象，而不是 `Begin {}` 或 `End {}` 中的对象，因为它破坏了pipeline的优势

```PowerShell
# Bad
function Get-USCitizenCapability {
    [CmdletBinding()]
    [OutputType([psobject])]
    param (
        [Parameter(Mandatory = $true,
                   ValueFromPipelineByPropertyName = $true,
                   Position = 0)]
        [int16]
        $Age
    )
    process {
        $Capabilities = @{
            MilitaryService = $false
            DrinkAlcohol = $false
            Vote = $false
        }
        if ($Age -ge 18) {
            $Capabilities['MilitaryService'] = $true
            $Capabilities['Vote'] = $true
        }

        $Obj = New-Object -Property $Capabilities -TypeName psobject
    }
    end { return $Obj }
}

# Good
function Get-USCitizenCapability {
    [CmdletBinding()]
    [OutputType([psobject])]
    param (
        [Parameter(Mandatory = $true,
                   ValueFromPipelineByPropertyName = $true,
                   Position = 0)]
        [int16]
        $Age
    )
    process {
        $Capabilities = @{
            MilitaryService = $false
            DrinkAlcohol = $false
            Vote = $false
        }

        if ($Age -ge 18) {
            $Capabilities['MilitaryService'] = $true
            $Capabilities['Vote'] = $true
        }

        New-Object -Property $Capabilities -TypeName psobject
    }
}
```

#### 总是使用 CmdletBinding 属性。

#### 如果任何参数从pipeline取值，总是有至少有一个 `process{}` 代码块。

#### 如果高级函数返回对象或对象集合，则指定 OutputType 属性。

如果函数根据参数集返回不同的对象类型，则为每个参数集提供一个OutputType。

```PowerShell
[OutputType([<TypeLiteral>], ParameterSetName = "<Name>")]
[OutputType("<TypeNameString>", ParameterSetName = "<Name>")]
```

#### 当ParameterSetName被用于任何参数时，总是在 CmdletBinding 属性中提供一个ParameterSetName。

```PowerShell
function Get-User {
    [CmdletBinding(DefaultParameterSetName = "ID")]
    [OutputType("System.Int32", ParameterSetName = "ID")]
    [OutputType([String], ParameterSetName = "Name")]
    param (
        [parameter(Mandatory = $true, ParameterSetName = "ID")]
        [Int[]]
        $UserID,

        [parameter(Mandatory = $true, ParameterSetName = "Name")]
        [String[]]
        $UserName
    )
    <# function body #>
}
```

#### 当使用 CmdletBinding 属性的高级函数或脚本时，可能时避免验证脚本正文中的参数，尽量使用参数验证属性。

* **AllowNull** 验证属性

  AllowNull 属性允许强制参数的值为空 ($null)。

  ```PowerShell
  param (
      [Parameter(Mandatory = $true)]
      [AllowNull()]
      [String]
      $ComputerName
  )
  ```

* **AllowEmptyString** 验证属性

  AllowEmptyString 属性允许强制参数的值为空字符串 ("")。

  ```PowerShell
  param (
      [Parameter(Mandatory = $true)]
      [AllowEmptyString()]
      [String]
      $ComputerName
  )
  ```

* **AllowEmptyCollection** 验证属性

  AllowEmptyCollection 属性允许强制参数的值为空集合 (@())。

  ```PowerShell
  param (
      [Parameter(Mandatory = $true)]
      [AllowEmptyCollection()]
      [String[]]
      $ComputerName
  )
  ```

* **ValidateCount** 验证属性

  ValidateCount 属性指定参数接受的参数值的最小和最大数量。 如果调用函数命令中的参数值数量超出该范围，Windows PowerShell 会生成错误。

  ```PowerShell
  param (
      [Parameter(Mandatory = $true)]
      [ValidateCount(1,5)]
      [String[]]
      $ComputerName
  )
  ```

* **ValidateLength** 验证属性

  ValidateLength 属性指定参数或变量值中的最小和最大字符数。 如果为参数或变量指定的值的长度超出范围，Windows PowerShell 会生成错误。

  ```PowerShell
  param (
      [Parameter(Mandatory = $true)]
      [ValidateLength(1,10)]
      [String[]]
      $ComputerName
  )
  ```

* **ValidatePattern** 验证属性

  ValidatePattern 属性指定与参数或变量值进行比较的正则表达式。 如果值与正则表达式模式不匹配，Windows PowerShell 会生成错误。

  ```PowerShell
  param (
      [Parameter(Mandatory = $true)]
      [ValidatePattern("[0-9][0-9][0-9][0-9]")]
      [String[]]
      $ComputerName
  )
  ```

* **ValidateRange** 验证属性

  ValidateRange 属性为每个参数或变量值指定一个数字范围。 如果任何值超出该范围，Windows PowerShell 会生成错误。

  ```PowerShell
  param (
      [Parameter(Mandatory = $true)]
      [ValidateRange(0,10)]
      [Int]
      $Attempts
  )
  ```

* **ValidateScript** 验证属性

  ValidateScript 属性指定用于验证参数或变量值的脚本。 Windows PowerShell 将值通过pipeline传递给脚本，如果脚本返回“false”或脚本引发异常，则会生成错误。

  当您使用 ValidateScript 属性时，正在验证的值被映射到 $_ 变量。 您可以使用使用 $_ 变量来引用脚本中的值。

  ```PowerShell
  param (
      [Parameter()]
      [ValidateScript({$_ -ge (get-date)})]
      [DateTime]
      $EventDate
  )
  ```

* **ValidateSet** 验证属性

  ValidateSet 属性为参数或变量指定一组有效值。 如果参数或变量值与集合中的值不匹配，Windows PowerShell 会生成错误。 在以下示例中，Detail 参数的值只能是“Low”、“Average”或“High”。

  ```PowerShell
  param (
      [Parameter(Mandatory = $true)]
      [ValidateSet("Low", "Average", "High")]
      [String[]]
      $Detail
  )
  ```

* **ValidateNotNull** 验证属性

  ValidateNotNull 属性指定参数值不能为空 ($null)。 如果参数值为空，Windows PowerShell 会生成错误。

  ValidateNotNull 属性设计为在未指定参数值的类型或指定的类型将接受 Null 值时使用。 （如果您指定一个不接受空值的类型，例如字符串，则空值将在没有 ValidateNotNull 属性的情况下被拒绝，因为它与指定的类型不匹配。)

  ```PowerShell
  param (
      [Parameter(Mandatory = $true)]
      [ValidateNotNull()]
      $ID
  )
  ```

* **ValidateNotNullOrEmpty** 验证属性

  ValidateNotNullOrEmpty 属性指定参数值不能为空 ($null) 且不能为空字符串 ("")。 如果在函数调用中使用了该参数，但其值为 null、空字符串或空数组，则 Windows PowerShell 会生成错误。

  ```PowerShell
  param (
      [Parameter(Mandatory = $true)]
      [ValidateNotNullOrEmpty()]
      [String[]]
      $UserName
  )
  ```
