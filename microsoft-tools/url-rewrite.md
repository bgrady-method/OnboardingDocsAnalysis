# URL Rewrite Installation

This guide covers installing and configuring the IIS URL Rewrite module for Method web applications.

## Overview

The IIS URL Rewrite module enables powerful URL manipulation capabilities for Method web applications hosted in IIS. It provides features like URL redirects, canonical URLs, SEO-friendly URLs, and request routing that are essential for Method's web applications.

## Prerequisites

- IIS installed and configured
- Administrator access for IIS module installation
- Visual Studio with IIS development features
- Method web applications requiring URL rewriting

## Download and Installation

### Direct Download from Microsoft

#### Download URL Rewrite Module

1. **Navigate to Microsoft Download Center:**
   - URL: https://www.iis.net/downloads/microsoft/url-rewrite
   - Or search for "IIS URL Rewrite Module 2.1"

2. **Download Options:**
   - **x64 (64-bit):** `rewrite_amd64_en-US.msi` (recommended for most systems)
   - **x86 (32-bit):** `rewrite_x86_en-US.msi` (for older systems)

#### PowerShell Download

```powershell
# Download URL Rewrite module via PowerShell
$downloadUrl = "https://download.microsoft.com/download/1/2/8/128E2E22-C1B9-44A4-BE2A-5859ED1D4592/rewrite_amd64_en-US.msi"
$downloadPath = "C:\temp\rewrite_amd64_en-US.msi"

# Create temp directory if it doesn't exist
if (!(Test-Path "C:\temp")) {
    New-Item -ItemType Directory -Path "C:\temp" -Force
}

# Download the installer
Write-Host "Downloading IIS URL Rewrite Module..." -ForegroundColor Green
Invoke-WebRequest -Uri $downloadUrl -OutFile $downloadPath

# Verify download
if (Test-Path $downloadPath) {
    Write-Host "✓ Downloaded successfully: $downloadPath" -ForegroundColor Green
    $fileInfo = Get-Item $downloadPath
    Write-Host "File size: $([math]::Round($fileInfo.Length / 1MB, 2)) MB" -ForegroundColor Gray
} else {
    Write-Host "✗ Download failed" -ForegroundColor Red
}
```

### Installation Process

#### Silent Installation

```powershell
# Install URL Rewrite module silently
$installerPath = "C:\temp\rewrite_amd64_en-US.msi"

Write-Host "Installing IIS URL Rewrite Module..." -ForegroundColor Green

# Run installer with administrator privileges
$process = Start-Process -FilePath "msiexec.exe" -ArgumentList "/i `"$installerPath`" /quiet /norestart" -Wait -PassThru -Verb RunAs

if ($process.ExitCode -eq 0) {
    Write-Host "✓ URL Rewrite module installed successfully" -ForegroundColor Green
} else {
    Write-Host "✗ Installation failed with exit code: $($process.ExitCode)" -ForegroundColor Red
}
```

#### Interactive Installation

```powershell
# Launch installer interactively
$installerPath = "C:\temp\rewrite_amd64_en-US.msi"
Start-Process -FilePath $installerPath -Verb RunAs -Wait

Write-Host "Please follow the installation wizard to complete setup" -ForegroundColor Yellow
```

### Alternative Installation Methods

#### Through Web Platform Installer

```powershell
# Install via Web Platform Installer (if available)
# Note: Web Platform Installer is deprecated but may still be available

# Check if WebPI is installed
$webpiPath = "${env:ProgramFiles}\Microsoft\Web Platform Installer\WebPlatformInstaller.exe"
if (Test-Path $webpiPath) {
    Write-Host "Installing via Web Platform Installer..." -ForegroundColor Green
    Start-Process -FilePath $webpiPath -ArgumentList "/install", "/products:UrlRewrite2" -Wait
} else {
    Write-Host "Web Platform Installer not found, using direct installer" -ForegroundColor Yellow
}
```

#### Through PowerShell Gallery (Alternative)

```powershell
# Note: This is not the official method, but alternative tools exist
# Install-Module IISAdministration # Core IIS management (already included in Windows)

# Verify IIS features are enabled
$iisFeatures = Get-WindowsOptionalFeature -Online | Where-Object {$_.FeatureName -like "*IIS*" -and $_.State -eq "Enabled"}
if ($iisFeatures) {
    Write-Host "✓ IIS features are enabled" -ForegroundColor Green
} else {
    Write-Host "⚠ IIS may not be properly configured" -ForegroundColor Yellow
}
```

## Verification

### Check Installation

```powershell
# Verify URL Rewrite module is installed
function Test-URLRewriteInstallation {
    Write-Host "Verifying IIS URL Rewrite installation..." -ForegroundColor Green
    
    # Check registry for installation
    $rewriteRegPath = "HKLM:\SOFTWARE\Microsoft\IIS Extensions\URL Rewrite"
    if (Test-Path $rewriteRegPath) {
        $version = Get-ItemProperty -Path $rewriteRegPath -Name "Version" -ErrorAction SilentlyContinue
        if ($version) {
            Write-Host "✓ URL Rewrite module installed - Version: $($version.Version)" -ForegroundColor Green
        } else {
            Write-Host "⚠ URL Rewrite registry key found but version unclear" -ForegroundColor Yellow
        }
    } else {
        Write-Host "✗ URL Rewrite module not found in registry" -ForegroundColor Red
    }
    
    # Check IIS modules
    try {
        Import-Module WebAdministration -ErrorAction Stop
        $modules = Get-WebGlobalModule | Where-Object {$_.Name -like "*rewrite*"}
        if ($modules) {
            Write-Host "✓ URL Rewrite module found in IIS:" -ForegroundColor Green
            foreach ($module in $modules) {
                Write-Host "  - $($module.Name): $($module.Image)" -ForegroundColor Gray
            }
        } else {
            Write-Host "✗ URL Rewrite module not found in IIS modules" -ForegroundColor Red
        }
    } catch {
        Write-Host "⚠ Could not check IIS modules: $($_.Exception.Message)" -ForegroundColor Yellow
    }
    
    # Check DLL files
    $rewriteDll = "${env:SystemRoot}\System32\inetsrv\rewrite.dll"
    if (Test-Path $rewriteDll) {
        $dllInfo = Get-Item $rewriteDll
        Write-Host "✓ URL Rewrite DLL found: $rewriteDll" -ForegroundColor Green
        Write-Host "  Version: $($dllInfo.VersionInfo.FileVersion)" -ForegroundColor Gray
        Write-Host "  Size: $([math]::Round($dllInfo.Length / 1KB, 2)) KB" -ForegroundColor Gray
    } else {
        Write-Host "✗ URL Rewrite DLL not found at expected location" -ForegroundColor Red
    }
}

# Run verification
Test-URLRewriteInstallation
```

### Test IIS Manager Integration

```powershell
# Check if URL Rewrite appears in IIS Manager
function Test-IISManagerIntegration {
    Write-Host "Testing IIS Manager integration..." -ForegroundColor Green
    
    try {
        # Import WebAdministration module
        Import-Module WebAdministration
        
        # Get default website configuration
        $siteName = "Default Web Site"
        $config = Get-WebConfiguration -Filter "system.webServer/rewrite" -PSPath "IIS:\Sites\$siteName" -ErrorAction SilentlyContinue
        
        if ($config) {
            Write-Host "✓ URL Rewrite configuration accessible via PowerShell" -ForegroundColor Green
        } else {
            Write-Host "⚠ URL Rewrite configuration not accessible (may be normal if no rules configured)" -ForegroundColor Yellow
        }
        
        # Check if module is loaded
        $loadedModules = Get-WebGlobalModule
        $rewriteModule = $loadedModules | Where-Object {$_.Name -eq "RewriteModule"}
        if ($rewriteModule) {
            Write-Host "✓ RewriteModule is loaded globally" -ForegroundColor Green
        } else {
            Write-Host "✗ RewriteModule not found in loaded modules" -ForegroundColor Red
        }
        
    } catch {
        Write-Host "✗ Error testing IIS Manager integration: $($_.Exception.Message)" -ForegroundColor Red
    }
}

# Run IIS Manager integration test
Test-IISManagerIntegration
```

## Method Web Application Configuration

### Basic URL Rewrite Rules

#### Create web.config with Rewrite Rules

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <system.webServer>
    <rewrite>
      <rules>
        
        <!-- Force HTTPS Redirect -->
        <rule name="Force HTTPS" enabled="true" stopProcessing="true">
          <match url="(.*)" />
          <conditions>
            <add input="{HTTPS}" pattern="off" ignoreCase="true" />
            <add input="{HTTP_HOST}" pattern="localhost" negate="true" />
          </conditions>
          <action type="Redirect" url="https://{HTTP_HOST}/{R:1}" 
                  appendQueryString="true" redirectType="Permanent" />
        </rule>
        
        <!-- Remove trailing slash -->
        <rule name="Remove Trailing Slash" enabled="true" stopProcessing="true">
          <match url="(.*)/$" />
          <conditions>
            <add input="{REQUEST_FILENAME}" matchType="IsFile" negate="true" />
            <add input="{REQUEST_FILENAME}" matchType="IsDirectory" negate="true" />
          </conditions>
          <action type="Redirect" url="{R:1}" redirectType="Permanent" />
        </rule>
        
        <!-- Method API versioning -->
        <rule name="API Versioning" enabled="true" stopProcessing="false">
          <match url="^api/v([0-9]+)/(.*)$" />
          <action type="Rewrite" url="api/{R:2}?version={R:1}" 
                  appendQueryString="true" />
        </rule>
        
        <!-- Method legacy URL support -->
        <rule name="Legacy Method URLs" enabled="true" stopProcessing="true">
          <match url="^legacy/(.*)$" />
          <action type="Redirect" url="/app/{R:1}" 
                  appendQueryString="true" redirectType="Permanent" />
        </rule>
        
        <!-- SPA fallback for Method UI -->
        <rule name="SPA Fallback" enabled="true" stopProcessing="true">
          <match url=".*" />
          <conditions logicalGrouping="MatchAll">
            <add input="{REQUEST_FILENAME}" matchType="IsFile" negate="true" />
            <add input="{REQUEST_FILENAME}" matchType="IsDirectory" negate="true" />
            <add input="{REQUEST_URI}" pattern="^/api/" negate="true" />
          </conditions>
          <action type="Rewrite" url="/index.html" />
        </rule>
        
      </rules>
      
      <!-- Outbound rules for response modification -->
      <outboundRules>
        <rule name="Add Security Headers" enabled="true">
          <match serverVariable="RESPONSE_Cache_Control" pattern=".*" />
          <action type="Rewrite" value="no-cache, no-store, must-revalidate" />
        </rule>
      </outboundRules>
      
      <!-- URL Rewrite maps for complex routing -->
      <rewriteMaps>
        <rewriteMap name="MethodRoutes">
          <add key="dashboard" value="/app/dashboard" />
          <add key="reports" value="/app/reports" />
          <add key="settings" value="/app/configuration" />
        </rewriteMap>
      </rewriteMaps>
      
    </rewrite>
    
    <!-- Additional IIS configuration -->
    <defaultDocument>
      <files>
        <clear />
        <add value="index.html" />
        <add value="default.html" />
      </files>
    </defaultDocument>
    
  </system.webServer>
</configuration>
```

### Method-Specific Rewrite Patterns

#### PowerShell Script to Generate Method Rules

```powershell
# Generate Method-specific URL rewrite rules
function New-MethodRewriteRules {
    param(
        [string]$OutputPath = "C:\temp\method-rewrite-rules.xml"
    )
    
    $rewriteRules = @"
<rewrite>
  <rules>
    
    <!-- Method Environment Detection -->
    <rule name="Method Environment Headers" enabled="true" stopProcessing="false">
      <match url=".*" />
      <serverVariables>
        <set name="HTTP_X_Method_Environment" value="Development" />
        <set name="HTTP_X_Method_Version" value="3.0" />
      </serverVariables>
      <action type="None" />
    </rule>
    
    <!-- Method API Gateway Routing -->
    <rule name="Method API Gateway" enabled="true" stopProcessing="false">
      <match url="^method-api/(.*)$" />
      <action type="Rewrite" url="/api/{R:1}" appendQueryString="true" />
    </rule>
    
    <!-- Method Static Assets -->
    <rule name="Method Assets" enabled="true" stopProcessing="true">
      <match url="^assets/(.*)$" />
      <action type="Rewrite" url="/Content/{R:1}" />
    </rule>
    
    <!-- Method Authentication Redirect -->
    <rule name="Method Auth Redirect" enabled="true" stopProcessing="true">
      <match url="^login$" />
      <action type="Redirect" url="/Account/Login" redirectType="Temporary" />
    </rule>
    
    <!-- Method Health Check -->
    <rule name="Method Health Check" enabled="true" stopProcessing="true">
      <match url="^health$" />
      <action type="Rewrite" url="/api/health" />
    </rule>
    
  </rules>
</rewrite>
"@

    Set-Content -Path $OutputPath -Value $rewriteRules
    Write-Host "✓ Method rewrite rules generated: $OutputPath" -ForegroundColor Green
}

# Generate rules
New-MethodRewriteRules
```

## Advanced Configuration

### Performance Optimization

#### Configure Rewrite Module Performance

```xml
<!-- In applicationHost.config or web.config -->
<system.webServer>
  <rewrite>
    <allowedServerVariables>
      <add name="HTTP_X_Method_Environment" />
      <add name="HTTP_X_Method_Version" />
      <add name="HTTP_X_Forwarded_For" />
    </allowedServerVariables>
    
    <!-- Global rules (in applicationHost.config) -->
    <globalRules>
      <rule name="Method Global Security" enabled="true" stopProcessing="false">
        <match url=".*" />
        <conditions>
          <add input="{HTTP_USER_AGENT}" pattern="bot|crawler|spider" />
        </conditions>
        <action type="CustomResponse" 
                statusCode="403" 
                statusReason="Forbidden" 
                statusDescription="Bot access not allowed" />
      </rule>
    </globalRules>
  </rewrite>
</system.webServer>
```

### Security Rules

#### Method Security Rewrite Rules

```xml
<rewrite>
  <rules>
    
    <!-- Block malicious requests -->
    <rule name="Block Malicious Requests" enabled="true" stopProcessing="true">
      <match url=".*" />
      <conditions logicalGrouping="MatchAny">
        <add input="{QUERY_STRING}" pattern="[;&lt;&gt;]" />
        <add input="{REQUEST_URI}" pattern="(\.\./|\.\.\\)" />
        <add input="{HTTP_USER_AGENT}" pattern="^$" />
      </conditions>
      <action type="CustomResponse" statusCode="400" statusReason="Bad Request" />
    </rule>
    
    <!-- Method admin area protection -->
    <rule name="Method Admin Protection" enabled="true" stopProcessing="true">
      <match url="^admin/(.*)$" />
      <conditions>
        <add input="{REMOTE_ADDR}" pattern="^(?!192\.168\.|10\.|172\.1[6-9]\.|172\.2[0-9]\.|172\.3[0-1]\.)" />
      </conditions>
      <action type="CustomResponse" statusCode="403" statusReason="Forbidden" />
    </rule>
    
    <!-- Rate limiting simulation -->
    <rule name="Rate Limiting Headers" enabled="true" stopProcessing="false">
      <match url="^api/(.*)$" />
      <serverVariables>
        <set name="HTTP_X_RateLimit_Limit" value="1000" />
        <set name="HTTP_X_RateLimit_Remaining" value="999" />
      </serverVariables>
      <action type="None" />
    </rule>
    
  </rules>
</rewrite>
```

## Testing and Debugging

### Test URL Rewrite Rules

```powershell
# Test URL Rewrite rules
function Test-MethodUrlRewrite {
    param(
        [string]$SiteName = "Default Web Site",
        [string[]]$TestUrls = @("/api/users", "/dashboard", "/legacy/reports")
    )
    
    Write-Host "Testing Method URL Rewrite rules..." -ForegroundColor Green
    
    foreach ($url in $TestUrls) {
        Write-Host "`nTesting URL: $url" -ForegroundColor Yellow
        
        try {
            # Simulate request to test rewrite rules
            # Note: This is a simplified test - actual testing requires IIS
            $testResult = Invoke-WebRequest -Uri "http://localhost$url" -Method Head -ErrorAction SilentlyContinue
            
            if ($testResult) {
                Write-Host "✓ Response: $($testResult.StatusCode) $($testResult.StatusDescription)" -ForegroundColor Green
            } else {
                Write-Host "⚠ No response received" -ForegroundColor Yellow
            }
        } catch {
            Write-Host "✗ Error: $($_.Exception.Message)" -ForegroundColor Red
        }
    }
}

# Run URL rewrite tests
Test-MethodUrlRewrite
```

### Enable Failed Request Tracing

```powershell
# Enable failed request tracing for URL Rewrite debugging
function Enable-URLRewriteTracing {
    param([string]$SiteName = "Default Web Site")
    
    try {
        Import-Module WebAdministration
        
        # Enable failed request tracing
        Set-WebConfigurationProperty -Filter "system.webServer/tracing/traceFailedRequests" -Name "enabled" -Value $true -PSPath "IIS:\Sites\$SiteName"
        
        # Add tracing rule for rewrite module
        Add-WebConfigurationProperty -Filter "system.webServer/tracing/traceFailedRequests" -Name "." -Value @{
            path = "*"
            statusCodes = "200-999"
            providers = "WWW Server"
            areas = "Rewrite"
            verbosity = "Verbose"
        } -PSPath "IIS:\Sites\$SiteName"
        
        Write-Host "✓ Failed request tracing enabled for URL Rewrite" -ForegroundColor Green
        Write-Host "Trace files will be saved to: %SystemDrive%\inetpub\logs\FailedReqLogFiles" -ForegroundColor Gray
        
    } catch {
        Write-Host "✗ Failed to enable tracing: $($_.Exception.Message)" -ForegroundColor Red
    }
}

# Enable tracing for debugging
Enable-URLRewriteTracing
```

## Troubleshooting

### Common Issues

#### Module Not Loading

```powershell
# Troubleshoot URL Rewrite module loading issues
function Repair-URLRewriteModule {
    Write-Host "Diagnosing URL Rewrite module issues..." -ForegroundColor Green
    
    # Check if module files exist
    $rewriteDll = "${env:SystemRoot}\System32\inetsrv\rewrite.dll"
    if (!(Test-Path $rewriteDll)) {
        Write-Host "✗ Rewrite.dll not found - reinstalling module" -ForegroundColor Red
        # Reinstall logic here
        return
    }
    
    # Check module registration
    try {
        Import-Module WebAdministration
        $globalModules = Get-WebGlobalModule
        $rewriteModule = $globalModules | Where-Object {$_.Name -eq "RewriteModule"}
        
        if (!$rewriteModule) {
            Write-Host "⚠ RewriteModule not registered - adding module" -ForegroundColor Yellow
            New-WebGlobalModule -Name "RewriteModule" -Image "%SystemRoot%\System32\inetsrv\rewrite.dll"
            Write-Host "✓ RewriteModule registered" -ForegroundColor Green
        } else {
            Write-Host "✓ RewriteModule is properly registered" -ForegroundColor Green
        }
        
    } catch {
        Write-Host "✗ Error checking module registration: $($_.Exception.Message)" -ForegroundColor Red
    }
    
    # Check IIS restart requirement
    Write-Host "⚠ You may need to restart IIS for changes to take effect" -ForegroundColor Yellow
    $restart = Read-Host "Restart IIS now? (y/n)"
    if ($restart -eq 'y' -or $restart -eq 'Y') {
        iisreset
    }
}

# Run repair if needed
# Repair-URLRewriteModule
```

#### Configuration Errors

```powershell
# Validate URL Rewrite configuration
function Test-URLRewriteConfig {
    param([string]$ConfigPath = "web.config")
    
    if (!(Test-Path $ConfigPath)) {
        Write-Host "✗ Configuration file not found: $ConfigPath" -ForegroundColor Red
        return
    }
    
    try {
        [xml]$config = Get-Content $ConfigPath
        $rewriteSection = $config.configuration.'system.webServer'.rewrite
        
        if ($rewriteSection) {
            Write-Host "✓ URL Rewrite section found in configuration" -ForegroundColor Green
            
            # Check for common issues
            $rules = $rewriteSection.rules.rule
            if ($rules) {
                Write-Host "✓ Found $($rules.Count) rewrite rule(s)" -ForegroundColor Green
                
                foreach ($rule in $rules) {
                    if (!$rule.name) {
                        Write-Host "⚠ Rule without name found" -ForegroundColor Yellow
                    }
                    if (!$rule.match) {
                        Write-Host "⚠ Rule '$($rule.name)' has no match pattern" -ForegroundColor Yellow
                    }
                }
            } else {
                Write-Host "⚠ No rewrite rules configured" -ForegroundColor Yellow
            }
        } else {
            Write-Host "⚠ No URL Rewrite configuration found" -ForegroundColor Yellow
        }
        
    } catch {
        Write-Host "✗ Error parsing configuration: $($_.Exception.Message)" -ForegroundColor Red
    }
}

# Test configuration
Test-URLRewriteConfig
```

### Method-Specific Troubleshooting

#### Debug Method Rewrite Rules

```xml
<!-- Add debug rule to test Method routing -->
<rule name="Method Debug" enabled="true" stopProcessing="false">
  <match url="^debug/rewrite/(.*)$" />
  <serverVariables>
    <set name="HTTP_X_Debug_OriginalUrl" value="{R:0}" />
    <set name="HTTP_X_Debug_MatchedPattern" value="{R:1}" />
    <set name="HTTP_X_Debug_QueryString" value="{QUERY_STRING}" />
  </serverVariables>
  <action type="Rewrite" url="/api/debug?original={R:0}&amp;pattern={R:1}" appendQueryString="true" />
</rule>
```

## Best Practices

### Rule Organization

- **Order matters** - place more specific rules before general ones
- **Use descriptive names** for all rules
- **Group related rules** logically
- **Document complex patterns** with comments

### Performance

- **Minimize regex complexity** in match patterns
- **Use stopProcessing="true"** when appropriate
- **Limit server variable usage** to necessary cases
- **Cache rewrite maps** for better performance

### Security

- **Validate all inputs** in rewrite conditions
- **Escape special characters** properly
- **Limit access** to admin areas via IP restrictions
- **Log security-related rewrites** for monitoring

## Verification and Testing

### Complete Installation Test

```powershell
# Comprehensive URL Rewrite installation and configuration test
function Test-MethodURLRewriteSetup {
    Write-Host "Method URL Rewrite Setup Verification" -ForegroundColor Green
    Write-Host "=====================================" -ForegroundColor Green
    
    $results = @()
    
    # Test 1: Module Installation
    $rewriteDll = "${env:SystemRoot}\System32\inetsrv\rewrite.dll"
    $moduleInstalled = Test-Path $rewriteDll
    $results += [PSCustomObject]@{
        Test = "Module Installation"
        Status = if ($moduleInstalled) { "✓ PASS" } else { "✗ FAIL" }
        Details = if ($moduleInstalled) { "rewrite.dll found" } else { "rewrite.dll missing" }
    }
    
    # Test 2: IIS Registration
    try {
        Import-Module WebAdministration -ErrorAction Stop
        $globalModule = Get-WebGlobalModule | Where-Object {$_.Name -eq "RewriteModule"}
        $moduleRegistered = $null -ne $globalModule
        $results += [PSCustomObject]@{
            Test = "IIS Registration"
            Status = if ($moduleRegistered) { "✓ PASS" } else { "✗ FAIL" }
            Details = if ($moduleRegistered) { "RewriteModule registered" } else { "RewriteModule not registered" }
        }
    } catch {
        $results += [PSCustomObject]@{
            Test = "IIS Registration"
            Status = "✗ ERROR"
            Details = "Cannot access WebAdministration: $($_.Exception.Message)"
        }
    }
    
    # Test 3: Configuration Access
    try {
        $config = Get-WebConfiguration -Filter "system.webServer/rewrite" -PSPath "IIS:\" -ErrorAction Stop
        $configAccessible = $null -ne $config
        $results += [PSCustomObject]@{
            Test = "Configuration Access"
            Status = if ($configAccessible) { "✓ PASS" } else { "⚠ WARN" }
            Details = if ($configAccessible) { "Configuration accessible" } else { "No rewrite config found" }
        }
    } catch {
        $results += [PSCustomObject]@{
            Test = "Configuration Access"
            Status = "✗ ERROR"
            Details = "Configuration error: $($_.Exception.Message)"
        }
    }
    
    # Display results
    $results | Format-Table -AutoSize
    
    # Summary
    $passed = ($results | Where-Object {$_.Status -like "*PASS*"}).Count
    $total = $results.Count
    Write-Host "`nSummary: $passed/$total tests passed" -ForegroundColor $(if ($passed -eq $total) { "Green" } else { "Yellow" })
    
    if ($passed -lt $total) {
        Write-Host "`nTroubleshooting steps:" -ForegroundColor Yellow
        Write-Host "1. Ensure IIS is installed and running" -ForegroundColor Gray
        Write-Host "2. Run installer as administrator" -ForegroundColor Gray
        Write-Host "3. Restart IIS after installation" -ForegroundColor Gray
        Write-Host "4. Check Windows Event Log for errors" -ForegroundColor Gray
    }
}

# Run comprehensive test
Test-MethodURLRewriteSetup
```

## Next Steps

After installing URL Rewrite:

1. **Configure Environment:** [Environment Setup](../environment-setup/README.md)
2. **Set up IIS:** [IIS Configuration](../iis-configuration/README.md)
3. **Test Rewrite Rules:** Create and test Method-specific URL rewrite patterns
4. **Enable SSL:** Configure HTTPS redirects and SSL bindings

## Additional Resources

- **IIS URL Rewrite Documentation:** https://docs.microsoft.com/en-us/iis/extensions/url-rewrite-module/
- **URL Rewrite Rule Templates:** https://www.iis.net/learn/extensions/url-rewrite-module/url-rewrite-module-configuration-reference
- **Method URL Patterns:** Internal documentation
- **Regular Expression Reference:** https://docs.microsoft.com/en-us/dotnet/standard/base-types/regular-expression-language-quick-reference

**Back to:** [Microsoft Tools](./README.md)
