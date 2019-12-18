---
  title: Multiline variables in Azure DevOps
  categories:
    - azure-devops
  tags:
    - azure-devops
---

I was surprised to discover that you cannot write a multiline value to an Azure DevOps Pipeline Variable from a Powershell script.

```powershell
Write-Host "##vso[task.setvariable variable=MyVariable]$MultilineValue"
```

You will only get the first line of your variable value. I suspect that somewhere something has been rendered and interpreted and the line breaks have essentially cut off the assignment command at the first line.

In my case I was saving some output for later display and having the linebreaks as HTML entities was an acceptable solution, so I was able to use

```powershell
$MultilineValue = $MultilineValue.replace("`n", "%0A").replace("`r", "%0D")
Write-Host "##vso[task.setvariable variable=MyVariable]$MultilineValue"
```
