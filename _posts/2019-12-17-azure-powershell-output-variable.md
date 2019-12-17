---
  title: Output a variable from Powershell to an Azure DevOps Pipeline
  categories:
    - devops
  tags:
    - azure-devops
    - devops
  permalink: /azure-devops/output-variables/
---

To set the value of a variable in Azure DevOps, you have a couple of options.

Note that the variable values will be set *for the scope of the job only*. To set a variable for the scope of the Stage, you can use [this method](https://github.com/paxdev/SetReleaseVariable)

1. To set the value of a variable that you have already defined in the `Pipeline Variables`

   In a PowerShell script set 

   ```powershell
       Write-Host "##vso[task.setvariable variable=MyVariableName;]My Value"  
   ``` 

   You can now reference the variable in the normal way as `$(MyVariableName)` 

1. To set an output variable that you have not defined elsewhere

   In a PowerShell script set 

   ```powershell
       Write-Host "##vso[task.setvariable variable=MyVariableName;isOutput=true]My Value"  
   ```

   Make sure to expand `Output Variables` and set a `Reference name`

   You can now reference your variable as `$(MyReferenceName.MyVariableName)`

   Note that you won't get any intellisense when referring to the variable using this method.
