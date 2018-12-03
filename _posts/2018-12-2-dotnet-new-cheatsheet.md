---
  title: dotnet new Cheatsheet
  
  header:
    subheading: Cheatsheet for creating dotnet new templates
---

## Install a template from a file:
```shell
dotnet new -i ../{{path-to-project-folder}}
# Where path-to-project-folder is the folder containing the .template.config folder, which itself contains a template.json
```

## Remove *all* templates:
```shell
dotnet new --debug:reinit
```

## Remove a folder template:
```shell
dotnet new -u {{path-to-project}}
# *Note that the above must be fully qualified whereas you can dotnet new -i from a relative path*
# To find the correct path you can use 
dotnet new -u -h
# This will list all installed templates
```

## template.json
```json
{
  "author": "{{Self-explanatory}}",
  "classifications": ["list of tags"],
  "name": "{{The full name}}",
  "shortName": "{{what you'll dotnet new with}}",
   "identity": "{{Unique.Identifier}}",
  "tags": {
    "language": "C#"
  },
  "sourceName": "My.Solution/Project.Name", 
        /* dotnet will find any .sln or .csproj files
         * or any folders with this name and rename them
         * to the project name you specify in dotnet new */
  "preferNameDirectory": "true", 
        /* if no name is specified in dotnet new
         * use the name of the folder as the project name */
  "symbols":{
    "applicationName": {
      "type": "parameter",
      "defaultValue": "default-value",
      "replaces": "some-var-to-replace", // replaces in files
      "fileRename": "folder-or-filename-to-replace", 
        // renames files and folders     
      "isRequired": "true" // self-explanatory    
    }
  }
}
```

## .nuspec folder
### Must be at the same level as the `.template.config` folder
```xml
<?xml version="1.0" encoding="utf-8"?>
<package xmlns="http://schemas.microsoft.com/packaging/2012/06/nuspec.xsd">
  <metadata>
    <id>{{Nuget.Package.Id}}</id>
    <version>1.0.0</version>
    <description>
      {{description}}
    </description>
    <authors>{{author}}</authors>
    <packageTypes>
      <packageType name="Template" />
    </packageTypes>
  </metadata>
</package>
```

## Now when you Package and Push you can 
```shell
dotnet new -i {{Nuget.Package.Id}}
```
