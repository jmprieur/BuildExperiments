# BuildExperiments

This repo contains experiments with MsBuild aiming at streamlining the builds of our libraries.

It contains:
- a couple of Build variables, which are computed to provide a version (PreviewVersionSuffix) and (VersionSuffix).
- A build task to display build variables which are related to versions.


## By default, you get a preview build

### Experience
If you run the following command on Dec 9th 2023 at 4h pm 14min 59s:

```sh
dotnet build
```

The build will display the following, and build the project with the following versions, package version and informational version:

```txt
Version = 1.0.0-preview-41209161459
VersionPrefix = 1.0.0
VersionSuffix = preview-41209161459
PackageVersion = 1.0.0-preview-41209161459
AssemblyVersion = 1.0.0.0
FileVersion = 1.0.0.0
InformationalVersion = 1.0.0-preview-41209161459+52828ea44cbcd5009fd41b606563b127e2cdff04
```

### How is it achieved?
This is achieved by the following build task:

```xml
  <Target Name="TestMessage" AfterTargets="Build" >
    <Message Text="Version = $(Version)" Importance="high"/>
    <Message Text="VersionPrefix = $(VersionPrefix)" Importance="high"/>
    <Message Text="VersionSuffix = $(VersionSuffix)" Importance="high"/>
    <Message Text="PackageVersion = $(PackageVersion)" Importance="high"/>
    <Message Text="AssemblyVersion = $(AssemblyVersion)" Importance="high"/>
    <Message Text="FileVersion = $(FileVersion)" Importance="high"/>
    <Message Text="InformationalVersion = $(InformationalVersion)" Importance="high"/>
```


## How to control the version of preview or release packages

### Experience
This all depends on the value of `$(VersionPrefix)` and `$(VersionsSuffix)`. By default `$(VersionSuffix)` is 1.0.0, and `$(Version)` is computed based on `$(VersionPrefix)` and `$(VersionSuffix)`

| Command                                  | Produced Nuget package                                 |
| ---------------------------------------- | ------------------------------------------------------ |
| dotnet pack                              | BuildExperiments.1.0.0-preview-41209152623.nupkg       |
| dotnet pack /p:Version=2.3.4             | BuildExperiments.2.3.4.nupkg                           |
| dotnet pack /p:Version=2.3.4-beta        | BuildExperiments.2.3.4-beta.nupkg                      |
| dotnet pack /p:VersionPrefix=2.0.0       | BuildExperiments.2.0.0-preview-41209152922.nupkg       |
| dotnet pack /p:VersionSuffix=preview2    | BuildExperiments.1.0.0-preview2.nupkg                  |

The versions properties of AssemblyInfo are generated automatically from $(Version).
For details, see https://learn.microsoft.com/dotnet/core/project-sdk/msbuild-props#generateassemblyinfo

### How is it achieved?

```xml
 <PropertyGroup>   
    <PreviewVersionSuffix>preview-$([System.DateTime]::Now.AddYears(-2019).Year)$([System.DateTime]::Now.ToString("MMddHHmmss"))</PreviewVersionSuffix>
    <VersionSuffix Condition="'$(Version)' == ''">$(PreviewVersionSuffix)</VersionSuffix>
  </PropertyGroup>
```
