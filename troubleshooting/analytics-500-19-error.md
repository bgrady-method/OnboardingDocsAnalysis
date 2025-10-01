# Troubleshooting HTTP 500.19 Error - Analytics Microservice

## Error Analysis

**Error Details:**
- **HTTP Error**: 500.19 - Internal Server Error
- **Error Code**: 0x80070003 
- **Config Error**: Cannot read configuration file
- **Config File**: `\\?\C:\MethodDev\ms-analytics-api\analytics\API\web.config`
- **URL**: `http://microservices.methodlocal.int/analytics/health/check`
- **Physical Path**: `C:\MethodDev\ms-analytics-api\analytics\API\health\check`

## Root Cause Analysis

The HTTP 500.19 error with code 0x80070003 indicates that IIS cannot access or read the web.config file. This is typically caused by:

1. **Missing web.config file** at the specified path
2. **File permission issues** - IIS application pool cannot read the file
3. **Corrupt or malformed web.config** file
4. **Path access issues** - UNC path notation suggests potential file system problems
5. **Application pool identity issues**

## Immediate Troubleshooting Steps

### 1. Verify File Existence
```powershell
# Check if the web.config file exists
$configPath = "C:\MethodDev\ms-analytics-api\analytics\API\web.config"
if (Test-Path $configPath) {
    Write-Host "✓ web.config exists at: $configPath" -ForegroundColor Green
    
    # Check file properties
    Get-ItemProperty $configPath | Select-Object Name, Length, CreationTime, LastWriteTime
} else {
    Write-Host "✗ web.config NOT found at: $configPath" -ForegroundColor Red
}
```

### 2. Check File Permissions
```powershell
# Check file permissions
$configPath = "C:\MethodDev\ms-analytics-api\analytics\API\web.config"
if (Test-Path $configPath) {
    Get-Acl $configPath | Select-Object -ExpandProperty Access | 
    Format-Table IdentityReference, FileSystemRights, AccessControlType -AutoSize
}
```

### 3. Verify Application Pool Status
```powershell
# Check analytics application pool
Import-Module WebAdministration
$appPoolName = "Method-Analytics" # Adjust if different

$appPool = Get-IISAppPool -Name $appPoolName -ErrorAction SilentlyContinue
if ($appPool) {
    Write-Host "App Pool '$appPoolName' State: $($appPool.State)" -ForegroundColor Yellow
    
    # Check app pool identity
    $identity = Get-WebConfiguration -Filter "/system.webServer/security/authentication/anonymousAuthentication" -PSPath "IIS:\AppPools\$appPoolName"
    Write-Host "App Pool Identity: $($appPool.ProcessModel.IdentityType)" -ForegroundColor Yellow
} else {
    Write-Host "✗ App Pool '$appPoolName' not found" -ForegroundColor Red
}
```

### 4. Validate Web.config Content
```powershell
# Test web.config XML validity
$configPath = "C:\MethodDev\ms-analytics-api\analytics\API\web.config"
if (Test-Path $configPath) {
    try {
        [xml]$webConfig = Get-Content $configPath
        Write-Host "✓ web.config XML is valid" -ForegroundColor Green
    }
    catch {
        Write-Host "✗ web.config XML is malformed: $($_.Exception.Message)" -ForegroundColor Red
    }
}
```

## Resolution Steps

### Step 1: Fix Missing web.config
If the web.config file is missing, create a basic one:

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <system.web>
    <compilation targetFramework="4.8" />
    <httpRuntime targetFramework="4.8" maxRequestLength="51200" />
    <authentication mode="None" />
  </system.web>
  
  <system.webServer>
    <defaultDocument>
      <files>
        <clear />
      </files>
    </defaultDocument>
    
    <handlers>
      <remove name="ExtensionlessUrlHandler-Integrated-4.0" />
      <remove name="OPTIONSVerbHandler" />
      <remove name="TRACEVerbHandler" />
      <add name="ExtensionlessUrlHandler-Integrated-4.0" path="*." verb="*" type="System.Web.Handlers.TransferRequestHandler" preCondition="integratedMode,runtimeVersionv4.0" />
    </handlers>
  </system.webServer>
</configuration>
```

### Step 2: Fix File Permissions
```powershell
# Grant proper permissions to IIS application pool
$configPath = "C:\MethodDev\ms-analytics-api\analytics\API\web.config"
$appPoolName = "Method-Analytics" # Adjust if different

# Grant read permissions to IIS AppPool identity
$acl = Get-Acl $configPath
$accessRule = New-Object System.Security.AccessControl.FileSystemAccessRule("IIS AppPool\$appPoolName", "Read", "Allow")
$acl.SetAccessRule($accessRule)
Set-Acl $configPath $acl

Write-Host "✓ Granted read permissions to IIS AppPool\$appPoolName" -ForegroundColor Green
```

### Step 3: Reset Application Pool
```powershell
# Reset the application pool
Import-Module WebAdministration
$appPoolName = "Method-Analytics" # Adjust if different

if (Get-IISAppPool -Name $appPoolName -ErrorAction SilentlyContinue) {
    Restart-WebAppPool -Name $appPoolName
    Write-Host "✓ Restarted application pool: $appPoolName" -ForegroundColor Green
} else {
    Write-Host "✗ Application pool '$appPoolName' not found" -ForegroundColor Red
}
```

### Step 4: Verify IIS Site Configuration
```powershell
# Check IIS site configuration for analytics
$siteName = "microservices.methodlocal.int" # Main microservices site

$site = Get-WebSite -Name $siteName -ErrorAction SilentlyContinue
if ($site) {
    Write-Host "Site '$siteName' State: $($site.State)" -ForegroundColor Yellow
    
    # Check applications under the site
    Get-WebApplication -Site $siteName | Where-Object { $_.Path -like "*analytics*" } | 
    Format-Table Path, PhysicalPath, ApplicationPool -AutoSize
} else {
    Write-Host "✗ Site '$siteName' not found" -ForegroundColor Red
}
```

## Advanced Troubleshooting

### Check UNC Path Issues
The error shows a UNC path format (`\\?\`) which can indicate file system issues:

```powershell
# Check for long path issues or file system corruption
$analyticsPath = "C:\MethodDev\ms-analytics-api\analytics\API\"

# Test directory access
if (Test-Path $analyticsPath) {
    Write-Host "✓ Directory accessible: $analyticsPath" -ForegroundColor Green
    
    # List directory contents
    Get-ChildItem $analyticsPath | Select-Object Name, Length, LastWriteTime
} else {
    Write-Host "✗ Cannot access directory: $analyticsPath" -ForegroundColor Red
}
```

### Validate Application Pool Configuration
```powershell
# Detailed app pool configuration check
$appPoolName = "Method-Analytics"
$appPool = Get-IISAppPool -Name $appPoolName -ErrorAction SilentlyContinue

if ($appPool) {
    Write-Host "=== App Pool Configuration ===" -ForegroundColor Yellow
    Write-Host "Name: $($appPool.Name)"
    Write-Host "State: $($appPool.State)"
    Write-Host "Runtime Version: $($appPool.ManagedRuntimeVersion)"
    Write-Host "Pipeline Mode: $($appPool.ManagedPipelineMode)"
    Write-Host "Identity: $($appPool.ProcessModel.IdentityType)"
    Write-Host "Auto Start: $($appPool.ProcessModel.IdleTimeout)"
} else {
    Write-Host "Creating missing application pool..." -ForegroundColor Yellow
    
    New-WebAppPool -Name $appPoolName
    Set-ItemProperty -Path "IIS:\AppPools\$appPoolName" -Name managedRuntimeVersion -Value "v4.0"
    Set-ItemProperty -Path "IIS:\AppPools\$appPoolName" -Name processModel.identityType -Value NetworkService
    
    Write-Host "✓ Created application pool: $appPoolName" -ForegroundColor Green
}
```

### Test Health Check Endpoint
```powershell
# Test the analytics health check after fixes
$healthCheckUrl = "http://microservices.methodlocal.int/analytics/health/check"

try {
    $response = Invoke-WebRequest -Uri $healthCheckUrl -UseBasicParsing
    Write-Host "✓ Health check successful - Status: $($response.StatusCode)" -ForegroundColor Green
    Write-Host "Response: $($response.Content)" -ForegroundColor Cyan
}
catch {
    Write-Host "✗ Health check failed: $($_.Exception.Message)" -ForegroundColor Red
    
    if ($_.Exception.Response) {
        Write-Host "HTTP Status: $($_.Exception.Response.StatusCode)" -ForegroundColor Red
    }
}
```

## Complete Fix Script

```powershell
function Fix-AnalyticsConfigError {
    Write-Host "=== Fixing Analytics HTTP 500.19 Error ===" -ForegroundColor Green
    
    $configPath = "C:\MethodDev\ms-analytics-api\analytics\API\web.config"
    $appPoolName = "Method-Analytics"
    $siteName = "microservices.methodlocal.int"
    
    # Step 1: Check if web.config exists
    if (-not (Test-Path $configPath)) {
        Write-Host "Creating missing web.config..." -ForegroundColor Yellow
        
        $webConfigContent = @"
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <system.web>
    <compilation targetFramework="4.8" />
    <httpRuntime targetFramework="4.8" maxRequestLength="51200" />
    <authentication mode="None" />
  </system.web>
  <system.webServer>
    <defaultDocument>
      <files>
        <clear />
      </files>
    </defaultDocument>
    <handlers>
      <remove name="ExtensionlessUrlHandler-Integrated-4.0" />
      <remove name="OPTIONSVerbHandler" />
      <remove name="TRACEVerbHandler" />
      <add name="ExtensionlessUrlHandler-Integrated-4.0" path="*." verb="*" type="System.Web.Handlers.TransferRequestHandler" preCondition="integratedMode,runtimeVersionv4.0" />
    </handlers>
  </system.webServer>
</configuration>
"@
        
        New-Item -Path (Split-Path $configPath) -ItemType Directory -Force -ErrorAction SilentlyContinue
        $webConfigContent | Out-File -FilePath $configPath -Encoding UTF8
        
        Write-Host "✓ Created web.config" -ForegroundColor Green
    }
    
    # Step 2: Fix permissions
    $acl = Get-Acl $configPath
    $accessRule = New-Object System.Security.AccessControl.FileSystemAccessRule("IIS AppPool\$appPoolName", "Read", "Allow")
    $acl.SetAccessRule($accessRule)
    Set-Acl $configPath $acl
    
    Write-Host "✓ Fixed file permissions" -ForegroundColor Green
    
    # Step 3: Ensure app pool exists and is configured
    Import-Module WebAdministration
    
    if (-not (Get-IISAppPool -Name $appPoolName -ErrorAction SilentlyContinue)) {
        New-WebAppPool -Name $appPoolName
        Set-ItemProperty -Path "IIS:\AppPools\$appPoolName" -Name managedRuntimeVersion -Value "v4.0"
        Set-ItemProperty -Path "IIS:\AppPools\$appPoolName" -Name processModel.identityType -Value NetworkService
        Write-Host "✓ Created and configured app pool" -ForegroundColor Green
    }
    
    # Step 4: Restart app pool
    Restart-WebAppPool -Name $appPoolName
    Write-Host "✓ Restarted application pool" -ForegroundColor Green
    
    # Step 5: Test health check
    Start-Sleep -Seconds 3
    try {
        $response = Invoke-WebRequest -Uri "http://microservices.methodlocal.int/analytics/health/check" -UseBasicParsing
        Write-Host "✓ Health check successful!" -ForegroundColor Green
    }
    catch {
        Write-Host "⚠ Health check still failing - check logs for additional issues" -ForegroundColor Yellow
    }
}

# Run the fix
Fix-AnalyticsConfigError
```

## Prevention

To prevent this issue in the future:
1. Ensure all microservice web.config files are committed to source control
2. Add file permission checks to deployment scripts
3. Monitor application pool health regularly
4. Include web.config validation in build processes

## Related Documentation
- [IIS Configuration](../iis-configuration/README.md)
- [Application Pools](./app-pools-services.md)
- [Architecture Overview](../onboarding/architecture-overview.md)
