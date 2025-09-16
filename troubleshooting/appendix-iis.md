# Appendix - IIS Features

## Required IIS Features for Method Development

This appendix lists all IIS features that must be enabled for Method applications to function properly.

## Core IIS Features

### Web Server (IIS) Role
The following features must be enabled through Windows Features or Server Manager:

### HTTP Features
```
☑ Web Server (IIS)
  ☑ Web Server
    ☑ Common HTTP Features
      ☑ Default Document
      ☑ Directory Browsing
      ☑ HTTP Errors
      ☑ HTTP Redirection
      ☑ Static Content
    ☑ Health and Diagnostics
      ☑ HTTP Logging
      ☑ Custom Logging
      ☑ Logging Tools
      ☑ Request Monitor
      ☑ Tracing
    ☑ Performance Features
      ☑ Static Content Compression
      ☑ Dynamic Content Compression
    ☑ Security
      ☑ Request Filtering
      ☑ Basic Authentication
      ☑ Windows Authentication
      ☑ URL Authorization
      ☑ IP and Domain Restrictions
```

### Application Development Features
```
☑ Application Development Features
  ☑ .NET Extensibility 3.5
  ☑ .NET Extensibility 4.8
  ☑ Application Initialization
  ☑ ASP.NET 3.5
  ☑ ASP.NET 4.8
  ☑ CGI
  ☑ ISAPI Extensions
  ☑ ISAPI Filters
  ☑ Server Side Includes
  ☑ WebSocket Protocol
```

### Management Tools
```
☑ Management Tools
  ☑ IIS Management Console
  ☑ IIS 6 Management Compatibility
    ☑ IIS 6 Metabase Compatibility
    ☑ IIS 6 Management Console
    ☑ IIS 6 Scripting Tools
    ☑ IIS 6 WMI Compatibility
  ☑ IIS Management Scripts and Tools
  ☑ Management Service
```

## PowerShell Installation Script

### Automated Feature Installation
```powershell
# Enable IIS and required features
function Install-MethodIISFeatures {
    Write-Host "Installing IIS features for Method development..." -ForegroundColor Green
    
    # Core IIS features
    $features = @(
        "IIS-WebServerRole",
        "IIS-WebServer",
        "IIS-CommonHttpFeatures",
        "IIS-DefaultDocument", 
        "IIS-DirectoryBrowsing",
        "IIS-HTTPAPI",
        "IIS-HttpErrors",
        "IIS-HttpLogging",
        "IIS-HttpRedirect",
        "IIS-HttpCompressionStatic",
        "IIS-HttpCompressionDynamic",
        "IIS-RequestFiltering",
        "IIS-StaticContent"
    )
    
    # Security features
    $securityFeatures = @(
        "IIS-BasicAuthentication",
        "IIS-WindowsAuthentication", 
        "IIS-URLAuthorization",
        "IIS-IPSecurity"
    )
    
    # Application development features
    $appDevFeatures = @(
        "IIS-NetFxExtensibility",
        "IIS-NetFxExtensibility45",
        "IIS-NetFxExtensibility48",
        "IIS-ApplicationInit",
        "IIS-ASPNET",
        "IIS-ASPNET45",
        "IIS-CGI",
        "IIS-ISAPIExtensions", 
        "IIS-ISAPIFilter",
        "IIS-ServerSideIncludes",
        "IIS-WebSockets"
    )
    
    # Management features
    $managementFeatures = @(
        "IIS-ManagementConsole",
        "IIS-IIS6ManagementCompatibility",
        "IIS-Metabase",
        "IIS-LegacySnapIn",
        "IIS-LegacyScripts",
        "IIS-WMICompatibility",
        "IIS-ManagementScriptingTools",
        "IIS-ManagementService"
    )
    
    # Combine all features
    $allFeatures = $features + $securityFeatures + $appDevFeatures + $managementFeatures
    
    foreach ($feature in $allFeatures) {
        try {
            Enable-WindowsOptionalFeature -Online -FeatureName $feature -All -NoRestart
            Write-Host "✓ Enabled: $feature" -ForegroundColor Green
        }
        catch {
            Write-Host "✗ Failed to enable: $feature" -ForegroundColor Red
            Write-Host "  Error: $($_.Exception.Message)" -ForegroundColor Red
        }
    }
    
    Write-Host "`nIIS feature installation complete!" -ForegroundColor Green
    Write-Host "A restart may be required for all features to be available." -ForegroundColor Yellow
}

# Run installation
Install-MethodIISFeatures
```

### DISM Command Line Installation
```cmd
REM Enable IIS via DISM (run as administrator)

REM Core IIS
dism /online /enable-feature /featurename:IIS-WebServerRole
dism /online /enable-feature /featurename:IIS-WebServer
dism /online /enable-feature /featurename:IIS-CommonHttpFeatures
dism /online /enable-feature /featurename:IIS-HttpErrors
dism /online /enable-feature /featurename:IIS-HttpRedirect
dism /online /enable-feature /featurename:IIS-ApplicationDevelopment

REM Authentication
dism /online /enable-feature /featurename:IIS-BasicAuthentication
dism /online /enable-feature /featurename:IIS-WindowsAuthentication

REM ASP.NET
dism /online /enable-feature /featurename:IIS-NetFxExtensibility45
dism /online /enable-feature /featurename:IIS-ASPNET45

REM Management Tools
dism /online /enable-feature /featurename:IIS-ManagementConsole
dism /online /enable-feature /featurename:IIS-IIS6ManagementCompatibility
```

## Feature Verification

### Check Installed Features
```powershell
function Test-MethodIISFeatures {
    Write-Host "=== IIS Features Status Check ===" -ForegroundColor Green
    
    $requiredFeatures = @(
        "IIS-WebServerRole",
        "IIS-ASPNET45", 
        "IIS-NetFxExtensibility45",
        "IIS-ManagementConsole",
        "IIS-WindowsAuthentication",
        "IIS-BasicAuthentication",
        "IIS-HttpCompressionStatic"
    )
    
    foreach ($feature in $requiredFeatures) {
        $featureState = Get-WindowsOptionalFeature -Online -FeatureName $feature
        
        if ($featureState.State -eq "Enabled") {
            Write-Host "✓ $feature - Enabled" -ForegroundColor Green
        } else {
            Write-Host "✗ $feature - $($featureState.State)" -ForegroundColor Red
        }
    }
    
    # Check IIS service status
    Write-Host "`nIIS Services:" -ForegroundColor Yellow
    $services = @("W3SVC", "WAS", "IISADMIN")
    
    foreach ($service in $services) {
        $serviceObj = Get-Service -Name $service -ErrorAction SilentlyContinue
        if ($serviceObj) {
            if ($serviceObj.Status -eq "Running") {
                Write-Host "✓ $service - Running" -ForegroundColor Green
            } else {
                Write-Host "✗ $service - $($serviceObj.Status)" -ForegroundColor Red
            }
        } else {
            Write-Host "✗ $service - Not found" -ForegroundColor Red
        }
    }
}

Test-MethodIISFeatures
```

## Method-Specific IIS Configuration

### Application Pool Settings
```powershell
# Configure application pools for Method applications
function Set-MethodAppPoolSettings {
    Import-Module WebAdministration
    
    $appPools = @("Method-UI", "Method-API", "Method-Portal")
    
    foreach ($poolName in $appPools) {
        if (Get-IISAppPool -Name $poolName -ErrorAction SilentlyContinue) {
            # Set .NET version
            Set-ItemProperty -Path "IIS:\AppPools\$poolName" -Name managedRuntimeVersion -Value "v4.0"
            
            # Set identity
            Set-ItemProperty -Path "IIS:\AppPools\$poolName" -Name processModel.identityType -Value NetworkService
            
            # Set pipeline mode
            Set-ItemProperty -Path "IIS:\AppPools\$poolName" -Name managedPipelineMode -Value Integrated
            
            # Configure recycling
            Set-ItemProperty -Path "IIS:\AppPools\$poolName" -Name recycling.periodicRestart.time -Value "00:00:00"
            
            # Set memory limits
            Set-ItemProperty -Path "IIS:\AppPools\$poolName" -Name recycling.periodicRestart.memory -Value 1048576
            
            Write-Host "✓ Configured app pool: $poolName" -ForegroundColor Green
        }
    }
}
```

### SSL Configuration
```powershell
# Configure SSL settings for Method sites
function Set-MethodSSLSettings {
    # Disable weak protocols
    New-Item -Path "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\SSL 2.0\Server" -Force
    New-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\SSL 2.0\Server" -Name "Enabled" -Value 0 -PropertyType DWORD -Force
    
    New-Item -Path "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\SSL 3.0\Server" -Force  
    New-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\SSL 3.0\Server" -Name "Enabled" -Value 0 -PropertyType DWORD -Force
    
    # Enable TLS 1.2
    New-Item -Path "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server" -Force
    New-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server" -Name "Enabled" -Value 1 -PropertyType DWORD -Force
    New-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server" -Name "DisabledByDefault" -Value 0 -PropertyType DWORD -Force
    
    Write-Host "SSL/TLS configuration updated" -ForegroundColor Green
}
```

## Handler Mappings

### Required Handler Mappings
```powershell
# Verify critical handler mappings
function Test-MethodHandlerMappings {
    $site = "Method-UI"
    
    $requiredHandlers = @(
        @{Name="PageHandlerFactory-Integrated-4.0"; Path="*.aspx"; Verb="GET,HEAD,POST,DEBUG"},
        @{Name="SimpleHandlerFactory-Integrated-4.0"; Path="*.ashx"; Verb="GET,HEAD,POST,DEBUG"},
        @{Name="WebServiceHandlerFactory-Integrated-4.0"; Path="*.asmx"; Verb="GET,HEAD,POST,DEBUG"},
        @{Name="DefaultDocumentModule"; Path="*"; Verb="*"},
        @{Name="StaticFileModule"; Path="*"; Verb="*"}
    )
    
    Write-Host "Checking handler mappings for $site..." -ForegroundColor Yellow
    
    $handlers = Get-WebHandler -PSPath "IIS:\Sites\$site"
    
    foreach ($required in $requiredHandlers) {
        $found = $handlers | Where-Object { $_.Name -eq $required.Name }
        
        if ($found) {
            Write-Host "✓ $($required.Name) - Present" -ForegroundColor Green
        } else {
            Write-Host "✗ $($required.Name) - Missing" -ForegroundColor Red
        }
    }
}
```

## MIME Types

### Required MIME Types for Method
```powershell
# Add Method-specific MIME types
function Add-MethodMimeTypes {
    $mimeTypes = @(
        @{Extension=".woff"; MimeType="application/font-woff"},
        @{Extension=".woff2"; MimeType="application/font-woff2"}, 
        @{Extension=".json"; MimeType="application/json"},
        @{Extension=".map"; MimeType="application/json"},
        @{Extension=".svg"; MimeType="image/svg+xml"}
    )
    
    foreach ($mime in $mimeTypes) {
        try {
            Remove-WebConfigurationProperty -pspath 'MACHINE/WEBROOT/APPHOST' -filter "system.webServer/staticContent" -name "." -AtElement @{fileExtension=$mime.Extension} -ErrorAction SilentlyContinue
            Add-WebConfigurationProperty -pspath 'MACHINE/WEBROOT/APPHOST' -filter "system.webServer/staticContent" -name "." -value @{fileExtension=$mime.Extension;mimeType=$mime.MimeType}
            Write-Host "✓ Added MIME type: $($mime.Extension) -> $($mime.MimeType)" -ForegroundColor Green
        }
        catch {
            Write-Host "✗ Failed to add MIME type: $($mime.Extension)" -ForegroundColor Red
        }
    }
}
```

## Request Filtering

### Configure Request Limits
```powershell
# Set appropriate request limits for Method applications
function Set-MethodRequestLimits {
    $sites = @("Method-UI", "Method-API", "Method-Portal")
    
    foreach ($site in $sites) {
        if (Get-WebSite -Name $site -ErrorAction SilentlyContinue) {
            # Set max content length (50MB)
            Set-WebConfigurationProperty -pspath "IIS:\Sites\$site" -filter "system.webServer/security/requestFiltering/requestLimits" -name "maxAllowedContentLength" -value 52428800
            
            # Set max URL length
            Set-WebConfigurationProperty -pspath "IIS:\Sites\$site" -filter "system.webServer/security/requestFiltering/requestLimits" -name "maxUrl" -value 4096
            
            # Set max query string length
            Set-WebConfigurationProperty -pspath "IIS:\Sites\$site" -filter "system.webServer/security/requestFiltering/requestLimits" -name "maxQueryString" -value 2048
            
            Write-Host "✓ Request limits configured for: $site" -ForegroundColor Green
        }
    }
}
```

## URL Rewrite Module

### Install URL Rewrite Module
```powershell
# Download and install URL Rewrite module
function Install-URLRewriteModule {
    $downloadUrl = "https://download.microsoft.com/download/1/2/8/128E2E22-C1B9-44A4-BE2A-5859ED1D4592/rewrite_amd64_en-US.msi"
    $tempFile = "$env:TEMP\rewrite_amd64_en-US.msi"
    
    try {
        Write-Host "Downloading URL Rewrite module..." -ForegroundColor Yellow
        Invoke-WebRequest -Uri $downloadUrl -OutFile $tempFile
        
        Write-Host "Installing URL Rewrite module..." -ForegroundColor Yellow
        Start-Process msiexec.exe -Wait -ArgumentList "/i `"$tempFile`" /quiet"
        
        Remove-Item $tempFile -Force
        
        Write-Host "✓ URL Rewrite module installed successfully" -ForegroundColor Green
    }
    catch {
        Write-Host "✗ Failed to install URL Rewrite module: $($_.Exception.Message)" -ForegroundColor Red
    }
}
```

## Troubleshooting IIS Configuration

### Common IIS Issues
```powershell
function Test-MethodIISHealth {
    Write-Host "=== Method IIS Health Check ===" -ForegroundColor Green
    
    # Check IIS service
    $w3svc = Get-Service -Name "W3SVC"
    if ($w3svc.Status -eq "Running") {
        Write-Host "✓ IIS service is running" -ForegroundColor Green
    } else {
        Write-Host "✗ IIS service is not running" -ForegroundColor Red
    }
    
    # Check application pools
    $pools = Get-IISAppPool
    foreach ($pool in $pools) {
        if ($pool.Name -like "*Method*") {
            if ($pool.State -eq "Started") {
                Write-Host "✓ App pool $($pool.Name) is started" -ForegroundColor Green
            } else {
                Write-Host "✗ App pool $($pool.Name) is $($pool.State)" -ForegroundColor Red
            }
        }
    }
    
    # Check sites
    $sites = Get-WebSite
    foreach ($site in $sites) {
        if ($site.Name -like "*Method*") {
            if ($site.State -eq "Started") {
                Write-Host "✓ Site $($site.Name) is started" -ForegroundColor Green
            } else {
                Write-Host "✗ Site $($site.Name) is $($site.State)" -ForegroundColor Red
            }
        }
    }
    
    # Check bindings
    Write-Host "`nSite Bindings:" -ForegroundColor Yellow
    Get-WebBinding | Where-Object { $_.ItemXPath -like "*Method*" } | ForEach-Object {
        Write-Host "  $($_.ItemXPath): $($_.protocol)://$($_.bindingInformation)" -ForegroundColor Cyan
    }
}

Test-MethodIISHealth
```

### Reset IIS Configuration
```powershell
# Reset IIS to default configuration (use with caution)
function Reset-IISConfiguration {
    param([switch]$Confirm = $false)
    
    if (-not $Confirm) {
        Write-Host "This will reset IIS configuration to defaults!" -ForegroundColor Red
        Write-Host "Use -Confirm switch to proceed" -ForegroundColor Yellow
        return
    }
    
    Write-Host "Stopping IIS..." -ForegroundColor Yellow
    iisreset /stop
    
    Write-Host "Resetting IIS configuration..." -ForegroundColor Yellow
    
    # Reset application pools
    Remove-WebAppPool -Name "*" -Confirm:$false -ErrorAction SilentlyContinue
    
    # Reset sites
    Remove-WebSite -Name "*" -Confirm:$false -ErrorAction SilentlyContinue
    
    # Recreate default app pool and site
    New-WebAppPool -Name "DefaultAppPool"
    New-WebSite -Name "Default Web Site" -Port 80 -PhysicalPath "C:\inetpub\wwwroot"
    
    Write-Host "Starting IIS..." -ForegroundColor Yellow
    iisreset /start
    
    Write-Host "IIS configuration reset complete!" -ForegroundColor Green
}
```

## Performance Optimization

### IIS Performance Settings
```powershell
# Optimize IIS for Method applications
function Optimize-MethodIIS {
    # Increase connection limits
    Set-WebConfigurationProperty -pspath 'MACHINE/WEBROOT/APPHOST' -filter "system.webServer/serverRuntime" -name "maxConcurrentRequestsPerCPU" -value 5000
    Set-WebConfigurationProperty -pspath 'MACHINE/WEBROOT/APPHOST' -filter "system.webServer/serverRuntime" -name "maxConcurrentThreadsPerCPU" -value 0
    
    # Configure output caching
    Set-WebConfigurationProperty -pspath 'MACHINE/WEBROOT/APPHOST' -filter "system.webServer/caching" -name "enabled" -value "True"
    Set-WebConfigurationProperty -pspath 'MACHINE/WEBROOT/APPHOST' -filter "system.webServer/caching" -name "enableKernelCache" -value "True"
    
    # Enable compression
    Set-WebConfigurationProperty -pspath 'MACHINE/WEBROOT/APPHOST' -filter "system.webServer/httpCompression" -name "doDynamicCompression" -value "True"
    Set-WebConfigurationProperty -pspath 'MACHINE/WEBROOT/APPHOST' -filter "system.webServer/httpCompression" -name "doStaticCompression" -value "True"
    
    Write-Host "IIS performance optimization complete!" -ForegroundColor Green
}
```

## Complete IIS Setup Script

### All-in-One Setup
```powershell
function Initialize-MethodIIS {
    Write-Host "=== Method IIS Complete Setup ===" -ForegroundColor Green
    
    # Install features
    Install-MethodIISFeatures
    
    # Configure application pools
    Set-MethodAppPoolSettings
    
    # Configure SSL
    Set-MethodSSLSettings
    
    # Add MIME types
    Add-MethodMimeTypes
    
    # Set request limits
    Set-MethodRequestLimits
    
    # Optimize performance
    Optimize-MethodIIS
    
    # Restart IIS
    Write-Host "Restarting IIS..." -ForegroundColor Yellow
    iisreset
    
    # Verify setup
    Test-MethodIISHealth
    
    Write-Host "`nMethod IIS setup complete!" -ForegroundColor Green
    Write-Host "You may need to restart your computer for all changes to take effect." -ForegroundColor Yellow
}

# Run complete setup
Initialize-MethodIIS
```

This completes the IIS features appendix with comprehensive configuration options for Method development environments.

**Back to:** [Troubleshooting Home](./README.md)
