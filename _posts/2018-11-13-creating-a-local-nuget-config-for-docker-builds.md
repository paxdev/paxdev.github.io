---
  title: Accessing private NuGet feeds in a local Docker build
---

A Docker build does not necessarily run in a context that is authorised to access a private NuGet feed, 
so we have a challenge to access our packages on a `dotnet restore`.

We can pass credentials via a NuGet.config file, but we don't really want to store these in Source Control.

First of all, we add `NuGet.config` to our `.gitignore` file (or whatever your source control uses to exclude files

Next we head on over to Azure DevOps to [create a personal access token](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate)

The only scope you need is "Packaging (Read)", which only allows access to pull NuGet packages.

Now we need to create a relevant entry in our root `NuGet.config` file. On Windows, this is located at `%appdata%\NuGet\NuGet.Config`

```xml
<packageSources>
  <add key="((myfeedname))" value="((myfeedurl))" />
</packageSource>

...

<packageSourceCredentials>
  <((myfeedname))>
    <add key="Username" value="anything" />
    <add key="ClearTextPassword" value="((my-personal-access-token))" />
  </((myfeedname))>
</packageSourceCredentials>
```

We can now write a PowerShell script to pull this in to our working directory before each Docker build (I'm using Docker Compose here)

```powershell
copy $env:APPDATA\NuGet\NuGet.Config NuGet.Config
docker-compose build
```

I save this next to my docker-compose.yml which is also next to my `.sln` solution file.

The final step is to use the `dockerfile` to tell the Docker build to pull in our generated configuration

```dockerfile
COPY NuGet.config ((folder containing my .csproj or .sln))/
```

Now when I run the PowerShell script I will get a fresh copy of my NuGet.config from a single source, 
and it is never checked into source control.

When building on the Build server, I recommend using a *different* access token, storing it as a secret and generating the NuGet.config 
on the fly as a build step, e.g.

```powershell
echo '<?xml version="1.0" encoding="utf-8"?><configuration><packageSources><add key="MyFeed" value="$(NuGet-Feed)" /></packageSources><packageSourceCredentials><MyFeed><add key="Username" value="Anything" /><add key="ClearTextPassword" value="$(NuGet-PAT)" /></MyFeed></packageSourceCredentials></configuration>' > NuGet.config
```
