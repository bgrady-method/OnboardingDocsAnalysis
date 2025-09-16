# Visual Studio NuGet Issues

## Common NuGet Problems in Method Development

NuGet package management issues can significantly impact Method development. This guide covers common problems and solutions.

## Prerequisites

- Visual Studio 2019/2022 with NuGet Package Manager
- Access to Method's internal NuGet feeds
- VPN connection for internal package access

## NuGet Configuration

### Package Sources Configuration
```xml
<!-- NuGet.Config typically located at %AppData%\NuGet\ -->
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <add key="nuget.org" value="https://api.nuget.org/v3/index.json" protocolVersion="3" />
    <add key="Method Internal" value="https://method-nuget.methodintegration.com/nuget" />
    <add key="Visual Studio Offline Packages" value="C:\Program Files (x86)\Microsoft SDKs\NuGetPackages\" />
  </packageSources>
  <packageSourceCredentials>
    <Method_x0020_Internal>
      <add key="Username" value="your-username" />
      <add key="ClearTextPassword" value="your-password" />
    </Method_x0020_Internal>
  </packageSourceCredentials>
</configuration>
```

### Global NuGet Settings
```powershell
# Set NuGet sources via command line
nuget sources add -Name "Method Internal" -Source "https://method-nuget.methodintegration.com/nuget"

# Set credentials for internal feed
nuget sources update -Name "Method Internal" -UserName "your-username" -Password "your-password"

# List configured sources
nuget sources list
```

## Common Issues and Solutions

### Package Restore Failures

#### Symptoms
- Build errors: "Package not found"
- Missing references in Solution Explorer
- Red X icons on NuGet package references

#### Solutions
```powershell
# Clear NuGet caches
nuget locals all -clear
dotnet nuget locals all --clear

# Restore packages
nuget restore
dotnet restore

# Force package reinstallation
Update-Package -reinstall
```

### Authentication Issues with Internal Feeds

#### Symptoms
- 401 Unauthorized errors
- "Unable to load the service index" errors
- Timeouts when accessing Method's internal NuGet feed

#### Solutions
1. **Verify VPN Connection**
   ```powershell
   # Test connectivity to internal feed
   Test-NetConnection -ComputerName method-nuget.methodintegration.com -Port 443
   ```

2. **Update Credentials**
   ```powershell
   # Clear stored credentials
   nuget sources remove -Name "Method Internal"
   
   # Re-add with fresh credentials
   nuget sources add -Name "Method Internal" -Source "https://method-nuget.methodintegration.com/nuget" -UserName "your-username" -Password "your-password"
   ```

3. **Use Personal Access Token**
   ```xml
   <!-- In NuGet.Config -->
   <packageSourceCredentials>
     <Method_x0020_Internal>
       <add key="Username" value="username" />
       <add key="ClearTextPassword" value="personal-access-token" />
     </Method_x0020_Internal>
   </packageSourceCredentials>
   ```

### Package Version Conflicts

#### Symptoms
- Build warnings about version conflicts
- Runtime exceptions due to assembly version mismatches
- "Could not load file or assembly" errors

#### Solutions
```xml
<!-- Add binding redirects to web.config or app.config -->
<runtime>
  <assemblyBinding xmlns="urn:schemas-microsoft-com:asm.v1">
    <dependentAssembly>
      <assemblyIdentity name="Newtonsoft.Json" publicKeyToken="30ad4fe6b2a6aeed" culture="neutral" />
      <bindingRedirect oldVersion="0.0.0.0-13.0.0.0" newVersion="13.0.3.0" />
    </dependentAssembly>
  </assemblyBinding>
</runtime>
```

```powershell
# Consolidate package versions across solution
Update-Package -ProjectName YourProject -Reinstall

# Check for package updates
Get-Package -Updates

# Update specific package
Update-Package Newtonsoft.Json
```

### Slow Package Operations

#### Symptoms
- Long delays during package restore
- Timeouts during package installation
- Visual Studio freezing during NuGet operations

#### Solutions
1. **Optimize Package Sources**
   ```xml
   <!-- Prioritize faster sources -->
   <packageSources>
     <clear />
     <add key="Method Internal" value="https://method-nuget.methodintegration.com/nuget" />
     <add key="nuget.org" value="https://api.nuget.org/v3/index.json" />
   </packageSources>
   ```

2. **Use Local Package Cache**
   ```powershell
   # Create local package cache
   mkdir C:\LocalNuGetCache
   
   # Copy frequently used packages
   robocopy "%userprofile%\.nuget\packages" "C:\LocalNuGetCache" /MIR
   ```

3. **Configure HTTP Timeout**
   ```xml
   <config>
     <add key="http_proxy.user" value="username" />
     <add key="http_proxy.password" value="password" />
     <add key="http_proxy" value="proxy-server:port" />
   </config>
   ```

### Package Manager Console Issues

#### Symptoms
- Commands not recognized
- PowerShell execution policy errors
- Package Manager Console not loading

#### Solutions
```powershell
# Set execution policy
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser

# Reload Package Manager Console
# Tools → NuGet Package Manager → Package Manager Console

# Clear console
Clear-Host

# Check NuGet version
Get-Package -ListAvailable NuGet

# Update NuGet Package Manager
Update-Package -Id NuGet.CommandLine
```

## Method-Specific Package Issues

### Internal Package Dependencies

#### Method.Core Packages
```xml
<!-- Common Method packages -->
<PackageReference Include="Method.Core" Version="2.1.0" />
<PackageReference Include="Method.Data" Version="2.1.0" />
<PackageReference Include="Method.Business" Version="2.1.0" />
<PackageReference Include="Method.Utilities" Version="2.1.0" />
```

#### Version Synchronization
```powershell
# Ensure all Method packages use same version
$methodVersion = "2.1.0"
Update-Package Method.Core -Version $methodVersion
Update-Package Method.Data -Version $methodVersion
Update-Package Method.Business -Version $methodVersion
Update-Package Method.Utilities -Version $methodVersion
```

### Legacy Package Migration

#### .NET Framework to .NET Core/5+
```xml
<!-- Old packages.config -->
<packages>
  <package id="EntityFramework" version="6.4.4" targetFramework="net48" />
</packages>

<!-- New PackageReference -->
<PackageReference Include="Microsoft.EntityFrameworkCore" Version="6.0.0" />
<PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="6.0.0" />
```

## Debugging NuGet Issues

### Verbose Logging
```powershell
# Enable verbose NuGet logging
nuget restore -Verbosity detailed

# For dotnet CLI
dotnet restore --verbosity diagnostic

# MSBuild with detailed logging
msbuild /p:RestorePackagesConfig=true /verbosity:diagnostic
```

### Package Manager Log Analysis
1. **Open Package Manager Console**
2. **Tools → Options → NuGet Package Manager → General**
3. **Set logging level to "Verbose"**
4. **Review logs in Output window**

### Network Diagnostics
```powershell
# Test NuGet endpoints
Test-NetConnection -ComputerName api.nuget.org -Port 443
Test-NetConnection -ComputerName method-nuget.methodintegration.com -Port 443

# Check proxy settings
netsh winhttp show proxy

# Test with curl
curl -I https://api.nuget.org/v3/index.json
```

## PowerShell Scripts for NuGet Management

### Package Health Check
```powershell
function Test-NuGetHealth {
    Write-Host "=== NuGet Health Check ===" -ForegroundColor Green
    
    # Check package sources
    Write-Host "`nPackage Sources:" -ForegroundColor Yellow
    nuget sources list
    
    # Test connectivity
    Write-Host "`nConnectivity Tests:" -ForegroundColor Yellow
    $sources = @(
        "api.nuget.org",
        "method-nuget.methodintegration.com"
    )
    
    foreach ($source in $sources) {
        $result = Test-NetConnection -ComputerName $source -Port 443 -WarningAction SilentlyContinue
        if ($result.TcpTestSucceeded) {
            Write-Host "✓ $source - Reachable" -ForegroundColor Green
        } else {
            Write-Host "✗ $source - Not reachable" -ForegroundColor Red
        }
    }
    
    # Check cache size
    Write-Host "`nCache Information:" -ForegroundColor Yellow
    $cacheSize = (Get-ChildItem "$env:USERPROFILE\.nuget\packages" -Recurse | Measure-Object -Property Length -Sum).Sum / 1MB
    Write-Host "Global packages cache size: $([math]::Round($cacheSize, 2)) MB"
}

Test-NuGetHealth
```

### Package Update Script
```powershell
function Update-MethodPackages {
    param(
        [string]$TargetVersion = "latest"
    )
    
    $methodPackages = @(
        "Method.Core",
        "Method.Data", 
        "Method.Business",
        "Method.Utilities"
    )
    
    foreach ($package in $methodPackages) {
        Write-Host "Updating $package..." -ForegroundColor Yellow
        
        if ($TargetVersion -eq "latest") {
            Update-Package $package
        } else {
            Update-Package $package -Version $TargetVersion
        }
    }
    
    Write-Host "Method packages updated successfully!" -ForegroundColor Green
}

# Usage: Update-MethodPackages -TargetVersion "2.1.0"
```

## Best Practices

### Package Management
1. **Use PackageReference format** instead of packages.config
2. **Pin critical package versions** to avoid unexpected updates
3. **Regularly update packages** but test thoroughly
4. **Use central package management** for solution-wide version control

### Security
1. **Verify package sources** before adding them
2. **Use authenticated feeds** for internal packages
3. **Review package dependencies** for security vulnerabilities
4. **Keep credentials secure** - use personal access tokens

### Performance
1. **Use package caching** to speed up builds
2. **Minimize package sources** to reduce lookup time
3. **Use offline packages** when possible
4. **Configure appropriate timeouts** for network operations

**Next**: [DNS Setup (Optional)](./dns-setup.md)
