---
  title: Register Azure Deployment Agents
  header:
    subheading: Register Azure Deployment Agents Like a Boss
  categories:
    - azure-devops
  tags:
    - azure-devops
    - powershell
---

The PowerShell script provided on the Deployment Pool/Deployment Group Forms in Azure DevOps is very useful, 
but does not show all the command line switches to allow you to fully automated scripted deployment of Azure DevOps Agents

I find the following two snippets cover most bases

## Register as part of a Deployment Group with tags

```powershell
.\config.cmd --deploymentgroup --deploymentGroupName '{{your-deployment-group}}' `
			--agent $env:COMPUTERNAME `
			--runasservice `
			--windowsLogonAccount "NT AUTHORITY\SYSTEM" `
			--work '_work' `
			--url '{{your-organisation-url}}' `
			--auth pat --token {{your-token}} `
			--projectname '{{your-project}}' `
			--addDeploymentGroupTags --deploymentGroupTags "{{comma-separated-tags}}" 
```

## Register as part of a Deployment Pool

```powershell
.\config.cmd --deploymentpool --deploymentPoolName '{{your-deployment-pool}}' `
			--agent $env:COMPUTERNAME `
			--runasservice `
			--windowsLogonAccount "NT AUTHORITY\SYSTEM" `
			--work '_work' `
			--url '{{your-organisation-url}}' `
			--auth pat --token {{your-token}}
```

