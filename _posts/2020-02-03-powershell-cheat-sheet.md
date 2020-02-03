---
  title: Powershell Cheat Sheet
  categories:
    - powershell
  tags:
    - powershell
---

* `$_` - Current PowerShell object
* `%` - ForEach-Object
* `& "SomeString (or var)"` - Execute what is in the string or var, e.g. 
   ```powershell
   $MsBuild = "Path//To//MsBuild"
   & $MsBuild My.csproj /p:SomeOtions
   ```
* `$var = Read-Host "Prompt"` - Get Input
* `>` - Redirect operator
   ```powershell
   Some-Command > out.txt # redirect ouput of Some-Command to file
   2>&1 # redirect error to success stream 
   ```
* `$null` - Null
* `$true/$false` - Boolean
* `-eq, -ne, -gt, -lt, -le, -ge` - comparison operators Case-insensitive by default.
* `-and, -or, -xor, -not, !` - logical operators
* `-in, -notin, -contains, -notcontains` - containment
* `$( )` - subexpression - evaluate the contents of the brackets
* `1..5` - range operator
