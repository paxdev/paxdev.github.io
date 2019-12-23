---
  title: Powershell Strings
  categories:
    - powershell
  tags:
    - powershell
---

A single-quote delimited string will not expand variables:

```powershell
$SomeVar="SomeVal"
Write-Host 'Value is $SomeVar'
#Outputs Value is $SomeVar
```

A double-quoted string will expand variables:

```powershell
$SomeVar="SomeVal"
Write-Host "Value is $SomeVar"
#Outputs Value is SomeVal
```

To expand an expression use `$(expression)` within a double-quoted string

```powershell
$SomeVar="SomeVal"
Write-Host "Value is $($SomeVar.ToUpper())"
#Outputs Value is SOMEVAL
```

To show double-quotes in a double-quoted string use backticks 
_(Also used to create escape sequences, e.g. \`r, \`n)_

```powershell
$SomeVar="SomeVal"
Write-Host "Value is `"$SomeVar`""
#Outputs Value is "SomeVal"
```

To preserve formatting and use multiple lines (e.g. to create JSON) use a here string.
Here strings must start and end with `@"` and `"@` on individual lines respectively:

```powershell
$SomeVar="SomeVal"
Write-Host @"
Value is 
    "$SomeVar"
"@
#Outputs 
#Value is 
#    "SomeVal"
```
