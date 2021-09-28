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

  The ValidateLength attribute specifies the minimum and maximum number of characters in a parameter or variable value. Windows PowerShell generates an error if the length of a value specified for a parameter or a variable is outside of the range.

  ```PowerShell
  param (
      [Parameter(Mandatory = $true)]
      [ValidateLength(1,10)]
      [String[]]
      $ComputerName
  )
  ```

* **ValidatePattern** Validation Attribute

  The ValidatePattern attribute specifies a regular expression that is compared to the parameter or variable value. Windows PowerShell generates an error if the value does not match the regular expression pattern.

  ```PowerShell
  param (
      [Parameter(Mandatory = $true)]
      [ValidatePattern("[0-9][0-9][0-9][0-9]")]
      [String[]]
      $ComputerName
  )
  ```

* **ValidateRange** Validation Attribute

  The ValidateRange attribute specifies a numeric range for each parameter or variable value. Windows PowerShell generates an error if any value is outside that range.

  ```PowerShell
  param (
      [Parameter(Mandatory = $true)]
      [ValidateRange(0,10)]
      [Int]
      $Attempts
  )
  ```

* **ValidateScript** Validation Attribute

  The ValidateScript attribute specifies a script that is used to validate a parameter or variable value. Windows PowerShell pipes the value to the script, and generates an error if the script returns "false" or if the script throws an exception.

  When you use the ValidateScript attribute, the value that is being validated is mapped to the $_ variable. You can use the $_ variable to refer to the value in the script.

  ```PowerShell
  param (
      [Parameter()]
      [ValidateScript({$_ -ge (get-date)})]
      [DateTime]
      $EventDate
  )
  ```

* **ValidateSet** Attribute

  The ValidateSet attribute specifies a set of valid values for a parameter or variable. Windows PowerShell generates an error if a parameter or variable value does not match a value in the set. In the following example, the value of the Detail parameter can only be "Low," "Average," or "High."

  ```PowerShell
  param (
      [Parameter(Mandatory = $true)]
      [ValidateSet("Low", "Average", "High")]
      [String[]]
      $Detail
  )
  ```

* **ValidateNotNull** Validation Attribute

  The ValidateNotNull attribute specifies that the parameter value cannot be null ($null). Windows PowerShell generates an error if the parameter value is null.

  The ValidateNotNull attribute is designed to be used when the type of the parameter value is not specified or when the specified type will accept a value of Null. (If you specify a type that will not accept a null value, such as a string, the null value will be rejected without the ValidateNotNull attribute, because it does not match the specified type.)

  ```PowerShell
  param (
      [Parameter(Mandatory = $true)]
      [ValidateNotNull()]
      $ID
  )
  ```

* **ValidateNotNullOrEmpty** Validation Attribute

  The ValidateNotNullOrEmpty attribute specifies that the parameter value cannot be null ($null) and cannot be an empty string (""). Windows PowerShell generates an error if the parameter is used in a function call, but its value is null, an empty string, or an empty array.

  ```PowerShell
  param (
      [Parameter(Mandatory = $true)]
      [ValidateNotNullOrEmpty()]
      [String[]]
      $UserName
  )
  ```
