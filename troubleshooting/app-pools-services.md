# App Pools & Services

## IIS Application Pool Management

Managing IIS application pools is critical for Method's web applications to function properly.

## Application Pool Overview

Method typically uses these application pools:

### Core Application Pools
- **DefaultAppPool** - Default IIS application pool
- **Method-UI** - Method user interface applications
- **Method-API** - Method API services
- **Method-Portal** - Customer portal applications
- **Method-Legacy** - Legacy ASP.NET applications

## Application Pool Configuration

### Recommended Settings
```powershell
# PowerShell commands for app pool configuration
Import-Module WebAdministration

# Set .NET Framework version
Set-ItemProperty -Path "IIS:\AppPools\Method-UI" -Name processModel.identityType -Value ApplicationPoolIdentity
Set-ItemProperty -Path "IIS:\AppPools\Method-UI" -Name managedRuntimeVersion -Value "v4.0"

# Set pipeline mode
Set-ItemProperty -Path "IIS:\AppPools\Method-UI" -Name managedPipelineMode -Value Integrated

# Configure recycling
Set-ItemProperty -Path "IIS:\AppPools\Method-UI" -Name recycling.periodicRestart.time -Value "00:00:00"
Set-ItemProperty -Path "IIS:\AppPools\Method-UI" -Name recycling.periodicRestart.memory -Value 1048576
```

### Identity Configuration
```powershell
# Set to Network Service for SQL Server access
Set-ItemProperty -Path "IIS:\AppPools\Method-UI" -Name processModel.identityType -Value NetworkService

# Or use specific service account
Set-ItemProperty -Path "IIS:\AppPools\Method-UI" -Name processModel.identityType -Value SpecificUser
Set-ItemProperty -Path "IIS:\AppPools\Method-UI" -Name processModel.userName -Value "DOMAIN\ServiceAccount"
Set-ItemProperty -Path "IIS:\AppPools\Method-UI" -Name processModel.password -Value "Password123"
```

## Managing Application Pools

### PowerShell Commands
```powershell
# List all application pools
Get-IISAppPool | Select-Object Name, State

# Start an application pool
Start-WebAppPool -Name "Method-UI"

# Stop an application pool
Stop-WebAppPool -Name "Method-UI"

# Restart an application pool
Restart-WebAppPool -Name "Method-UI"

# Check application pool status
Get-WebAppPoolState -Name "Method-UI"

# Recycle application pool
Restart-WebAppPool -Name "Method-UI"
```

### IIS Manager GUI
1. Open **IIS Manager** (inetmgr.exe)
2. Expand server node
3. Click **Application Pools**
4. Right-click desired app pool
5. Select action: **Start**, **Stop**, **Recycle**, **Advanced Settings**

## Windows Services Management

### Critical Method Services

#### SQL Server Services
```powershell
# Check SQL Server status
Get-Service -Name "MSSQLSERVER"
Get-Service -Name "SQLSERVERAGENT"

# Start SQL Server services
Start-Service -Name "MSSQLSERVER"
Start-Service -Name "SQLSERVERAGENT"

# Stop SQL Server services (careful!)
Stop-Service -Name "SQLSERVERAGENT"
Stop-Service -Name "MSSQLSERVER"
```

#### Docker Services
```powershell
# Check Docker service
Get-Service -Name "com.docker.service"

# Start Docker service
Start-Service -Name "com.docker.service"

# Check Docker containers
docker ps -a
docker start container_name
docker stop container_name
```

#### IIS Services
```powershell
# World Wide Web Publishing Service
Get-Service -Name "W3SVC"
Start-Service -Name "W3SVC"

# IIS Admin Service
Get-Service -Name "IISADMIN"
Start-Service -Name "IISADMIN"
```

## Troubleshooting Application Pool Issues

### Common Problems and Solutions

#### App Pool Automatically Stopping
**Symptoms**: 503 Service Unavailable errors
**Causes**:
- Rapid failures triggering shutdown
- Insufficient permissions
- Memory limits exceeded
- Startup time exceeded

**Solutions**:
```powershell
# Disable rapid failure protection temporarily
Set-ItemProperty -Path "IIS:\AppPools\Method-UI" -Name failure.rapidFailProtection -Value $false

# Increase startup time limit
Set-ItemProperty -Path "IIS:\AppPools\Method-UI" -Name processModel.startupTimeLimit -Value "00:02:00"

# Increase memory limit
Set-ItemProperty -Path "IIS:\AppPools\Method-UI" -Name recycling.periodicRestart.memory -Value 2097152
```

#### Permission Issues
**Symptoms**: Access denied errors, authentication failures
**Solutions**:
```powershell
# Grant IIS_IUSRS permissions to application folder
icacls "C:\inetpub\wwwroot\Method-UI" /grant "IIS_IUSRS:(OI)(CI)M"

# Grant Network Service access to temp folders
icacls "C:\Windows\Temp" /grant "NETWORK SERVICE:(OI)(CI)F"
icacls "C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Temporary ASP.NET Files" /grant "NETWORK SERVICE:(OI)(CI)F"
```

#### Database Connection Failures
**Symptoms**: Cannot connect to SQL Server
**Solutions**:
1. Verify SQL Server service is running
2. Check application pool identity has database access
3. Test connection string in web.config
4. Verify SQL aliases configuration

### Application Pool Monitoring

#### Health Check Script
```powershell
# Monitor app pool health
function Check-AppPoolHealth {
    param([string]$AppPoolName)
    
    $appPool = Get-WebAppPoolState -Name $AppPoolName
    $lastHour = (Get-Date).AddHours(-1)
    
    # Check if running
    if ($appPool.Value -ne "Started") {
        Write-Warning "App pool $AppPoolName is not running: $($appPool.Value)"
        return $false
    }
    
    # Check for recent failures
    $failures = Get-WinEvent -FilterHashtable @{
        LogName = 'System'
        Id = 5002, 5021, 5023
        StartTime = $lastHour
    } -ErrorAction SilentlyContinue | Where-Object { $_.Message -like "*$AppPoolName*" }
    
    if ($failures) {
        Write-Warning "Recent failures detected for $AppPoolName"
        return $false
    }
    
    Write-Host "App pool $AppPoolName is healthy"
    return $true
}

# Check all Method app pools
$methodAppPools = @("Method-UI", "Method-API", "Method-Portal")
foreach ($pool in $methodAppPools) {
    Check-AppPoolHealth -AppPoolName $pool
}
```

## Service Startup Scripts

### Automated Service Startup
```powershell
# Start all Method services script
function Start-MethodServices {
    Write-Host "Starting Method development services..."
    
    # Start SQL Server services
    Write-Host "Starting SQL Server..."
    Start-Service -Name "MSSQLSERVER" -ErrorAction SilentlyContinue
    Start-Service -Name "SQLSERVERAGENT" -ErrorAction SilentlyContinue
    
    # Start Docker service
    Write-Host "Starting Docker..."
    Start-Service -Name "com.docker.service" -ErrorAction SilentlyContinue
    
    # Wait for Docker to be ready
    Start-Sleep -Seconds 10
    
    # Start Docker containers
    Write-Host "Starting Docker containers..."
    docker start mongodb redis elasticsearch rabbitmq
    
    # Start IIS services
    Write-Host "Starting IIS..."
    Start-Service -Name "W3SVC" -ErrorAction SilentlyContinue
    
    # Start Method app pools
    Write-Host "Starting Method application pools..."
    $methodAppPools = @("Method-UI", "Method-API", "Method-Portal")
    foreach ($pool in $methodAppPools) {
        Start-WebAppPool -Name $pool -ErrorAction SilentlyContinue
    }
    
    Write-Host "Method services startup complete!"
}

# Run the startup function
Start-MethodServices
```

### Service Status Check
```powershell
function Get-MethodServiceStatus {
    Write-Host "=== Method Development Environment Status ===" -ForegroundColor Green
    
    # SQL Server
    Write-Host "`nSQL Server Services:" -ForegroundColor Yellow
    Get-Service -Name "MSSQLSERVER", "SQLSERVERAGENT" | Format-Table Name, Status
    
    # Docker
    Write-Host "Docker Service:" -ForegroundColor Yellow
    Get-Service -Name "com.docker.service" | Format-Table Name, Status
    
    # Docker containers
    Write-Host "Docker Containers:" -ForegroundColor Yellow
    docker ps -a --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
    
    # IIS Application Pools
    Write-Host "`nIIS Application Pools:" -ForegroundColor Yellow
    Get-IISAppPool | Where-Object { $_.Name -like "*Method*" } | Format-Table Name, State
    
    # IIS Services
    Write-Host "IIS Services:" -ForegroundColor Yellow
    Get-Service -Name "W3SVC", "IISADMIN" | Format-Table Name, Status
}

# Run status check
Get-MethodServiceStatus
```

## Log Locations for Services

### Application Pool Logs
- **IIS Logs**: `C:\inetpub\logs\LogFiles\W3SVC1\`
- **Application Logs**: `D:\logs\<ApplicationName>\`
- **Windows Event Logs**: Event Viewer → Windows Logs → System

### Service Logs
- **SQL Server**: SSMS → Management → SQL Server Logs
- **Docker**: `docker logs <container_name>`
- **Windows Services**: Event Viewer → Windows Logs → System

## Performance Monitoring

### App Pool Performance Counters
```powershell
# Monitor app pool performance
Get-Counter -Counter "\APP_POOL_WAS(*)\Current Application Pool State" -Continuous

# Monitor worker process memory
Get-Counter -Counter "\Process(w3wp*)\Working Set" -Continuous

# Monitor CPU usage
Get-Counter -Counter "\Process(w3wp*)\% Processor Time" -Continuous
```

**Next**: [Visual Studio NuGet Issues](./nuget-issues.md)
