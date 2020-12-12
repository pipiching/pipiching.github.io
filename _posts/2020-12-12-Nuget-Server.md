---
layout: post
title:  "Nuget Server"
date:   2020-12-13 00:00:30 +0800
categories: web
---

## Library

### Create a nuget package

#### Nuget.exe

Download the [nuget.exe](https://www.nuget.org/downloads).

Create a folder called "Nuget" and copy Nuget.exe there.

```
.
├── bin
├── obj
├── Properties
├── projectName.csproj
├── Nuget
|	└── nuget.exe
└── packages.config
```

Open CLI and Switch path to your project.

Create a `.nuspec` file.

```shell
Nuget\nuget.exe spec projectName.csproj
```

Right-click the file and choose "Include in Project". 

Edit in VS2019.

The most important part is the `files` tag after closed metadata tag.

Define `.dll` path and framework version.

```xml
<?xml version="1.0" encoding="utf-8"?>
<package >
    <metadata>
        <!-- required tag -->
        <id>your package name</id>
        <version>1.0.0</version>
        <description>description</description>
        <authors>author</authors>
        <!-- optional tag -->
        <requireLicenseAcceptance>false</requireLicenseAcceptance>
        <license type="expression">MIT</license>
        <projectUrl>projectUrl</projectUrl>
        <iconUrl>iconUrl</iconUrl>
        <releaseNotes>Summary of changes made in this release of the package.</releaseNotes>
        <copyright>copyright</copyright>
        <tags>myTag</tags>
    </metadata>
    <files>
        <!-- dll path, framework version-->
        <file src=".\" target="lib"></file>
        <file src="bin\Release\*.dll" target="lib\net472"></file>
    </files>
</package>
```

Create a `.nupkg` file.

```shell
Nuget\nuget.exe pack {your nuspec file name}
```



#### Nuget Package Explorer

Install NuGet Package Explorer in Microsoft Store.

Create a new package.

Create a lib folder at Package.

Pull in your `.dll` file.

Edit Package metadata.

Save the file and get a `.nupkg` file.



## LiGet Server

```
docker pull tomzo/liget
```

```
docker run -d -p 9011:9011 --name nuget-server tomzo/liget
```



## BaGet Server

```shell
docker pull loicsharma/baget
```

Create a `baget.env` file.

```
ApiKey=<Your_ultra_secret_API_key>
Storage__Type=FileSystem
Storage__Path=/var/baget/packages
Database__Type=Sqlite
Database__ConnectionString=Data Source=/var/baget/baget.db
Search__Type=Database
```

Create a `/baget-data folder`.

```
mybaget
├── baget-data
|	└── baget.db
└── baget.env
```

Run `Baget`.

```shell
docker run -d --name nuget-server -p 5555:80 --env-file baget.env -v "$(pwd)/baget-data:/var/baget" loicsharma/baget:latest
```




## Publish packages

Push the package to the Nuget-server.

```shell
dotnet nuget push package.1.0.0.nupkg -s http://{root}:{your port number}/api/v3/index.json -k NUGET-SERVER-API-KEY
```

## Update version

Update `version` tag in `.nuspec` file and push again.

```shell
dotnet nuget push package.1.0.1.nupkg -s http://{root}:{your port number}/api/v3/index.json -k NUGET-SERVER-API-KEY
```

## Clear Nuget cache

(VS2019) Tools->Options->Nuget->General and clearing the cache