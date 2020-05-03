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
   & $MsBuild My.csproj /p:SomeOptions
   ```
* `$var = Read-Host "Prompt"` - Get Input
* `>` - Redirect operator
   ```powershell
   Some-Command > out.txt # redirect output of Some-Command to file
   2>&1 # redirect error to success stream 
   ```
* `$null` - Null
* `$true/$false` - Boolean
* `-eq, -ne, -gt, -lt, -le, -ge` - comparison operators Case-insensitive by default.
* `-and, -or, -xor, -not, !` - logical operators
* `-in, -notin, -contains, -notcontains` - containment
* `$( )` - subexpression - evaluate the contents of the brackets
* `1..5` - range operator
* `Get-Childitem -Path Env:* | Sort-Object Name` - list out env vars
* `. some\script.ps1` - execute the contents of the script (the script path could contain variables)
* `$PSScriptRoot` - the folder of the currently executing script
* `$MyInvocation` - get the command that caused this invocation
  * `$MyInvocation.PSCommandPath` - the full path to the command for the current invocation
  * `$MyInvocation.Line` - gets the full text that caused the invocation
  * `$MyInvocation.InvocationName` - gets the command name to invoke the script. If executing as a script would be the script name
  * ```Powershell
  If($MyInvocation.Line.Length -gt $MyInvocationName.Length){
    $params = $MyInvocation.Line.Substring($MyInvocation.InvocationName.Length)
  }
  ```
   \- get the full text after the script name if invoking script from command line
* `| Tee-Object` - splits out the pipeline so you can "tap" the pipeline to pull out a variable or file and continue processing
  If it is the last part of the pipeline it will output to console.
  * `| Tee-Object -Variable SomeVar` - will split out the pipeline to `$SomeVar`.

