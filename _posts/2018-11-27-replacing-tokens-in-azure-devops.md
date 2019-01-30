---
  title: Replacing Tokens in Azure DevOps
  categories: 
    - link
  tags:
    - azure-devops
    - devops
---

It seems that out of the box Azure DevOps Pipelines are really only configured to do XDT Transforms of XML config files and substitution in JSON
files in certain situations, typically Web App deployments.

If you need more flexibility (e.g. token substitution in yaml files) then I recommend the 
[Replace Tokens Task](https://marketplace.visualstudio.com/items?itemName=qetza.replacetokens) from [Guillaume Rouchon](https://github.com/qetza/).

It's simple to use but has enough flexibility for all the tasks I've thrown at it.
