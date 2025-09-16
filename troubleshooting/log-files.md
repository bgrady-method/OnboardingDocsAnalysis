# Log Files

## Method Application Logging

Method applications generate extensive logs to help with troubleshooting and monitoring.

## Primary Log Locations

### Application Logs: D:\logs\
```
D:\logs\
├── AccountManagement\          # Account creation and management
├── AuthenticationService\      # User authentication logs
├── GatewayAPI\                # API gateway and routing
├── HealthCheckApp\            # Health check application results
├── Method-UI\                 # Front-end application logs
├── RuntimeCore\               # Business logic engine
└── Various microservices\     # Individual service logs
```

### IIS Logs: C:\inetpub\logs\LogFiles\
```
C:\inetpub\logs\LogFiles\
├── W3SVC1\                    # Default website logs
├── W3SVC2\                    # Additional sites
└── FTPSVC1\                   # FTP service logs (if enabled)
```

### Windows Event Logs
- **Application Log** - Windows application events
- **System Log** - Operating system events
- **Security Log** - Authentication and security events

## Log Analysis Tools

### PowerShell Log Analysis
```powershell
# Search for errors in recent logs
Get-ChildItem "D:\logs\" -Recurse -Filter "*.log" | 
  ForEach-Object { 
    Select-String -Path $_.FullName -Pattern "ERROR|EXCEPTION|FATAL" -SimpleMatch 
  }

# Monitor real-time logs
Get-Content "D:\logs\RuntimeCore\latest.log" -Wait -Tail 10

# Find logs from specific time period
Get-ChildItem "D:\logs\" -Recurse | 
  Where-Object { $_.LastWriteTime -gt (Get-Date).AddHours(-1) }
```

### Event Viewer Analysis
1. Open Event Viewer (eventvwr.msc)
2. Navigate to Windows Logs → Application
3. Filter by:
   - **Source**: Look for Method-related sources
   - **Level**: Error, Warning, Critical
   - **Time**: Recent time periods

## Common Log Patterns

### Error Identification
Look for these patterns in logs:
- `ERROR` - Application errors
- `EXCEPTION` - Unhandled exceptions
- `FATAL` - Critical system failures
- `TIMEOUT` - Database or service timeouts
- `CONNECTION` - Network connectivity issues

### Performance Issues
Monitor for:
- `SLOW QUERY` - Database performance
- `HIGH CPU` - Resource utilization
- `MEMORY` - Memory pressure indicators
- `QUEUE` - Message queue backups

## Log Rotation and Cleanup

### Automatic Cleanup
Method applications typically:
- Rotate logs daily
- Keep 30 days of history
- Compress older logs
- Remove logs older than retention period

### Manual Cleanup
```powershell
# Remove logs older than 30 days
Get-ChildItem "D:\logs\" -Recurse -Filter "*.log" | 
  Where-Object { $_.LastWriteTime -lt (Get-Date).AddDays(-30) } | 
  Remove-Item -Force

# Check disk space usage
Get-ChildItem "D:\logs\" -Recurse | 
  Measure-Object -Property Length -Sum | 
  Select-Object @{Name="Size(MB)";Expression={[math]::round($_.Sum/1MB,2)}}
```

## Troubleshooting by Service

### RuntimeCore Engine
- **Location**: `D:\logs\RuntimeCore\`
- **Common Issues**: Business logic errors, database connection failures
- **Key Files**: `RuntimeCore.log`, `BusinessLogic.log`

### Authentication Service
- **Location**: `D:\logs\AuthenticationService\`
- **Common Issues**: Login failures, token validation errors
- **Key Files**: `Auth.log`, `OAuth.log`

### Gateway API
- **Location**: `D:\logs\GatewayAPI\`
- **Common Issues**: Routing failures, service discovery problems
- **Key Files**: `Gateway.log`, `Routing.log`

### Account Management
- **Location**: `D:\logs\AccountManagement\`
- **Common Issues**: Account creation failures, subscription issues
- **Key Files**: `AccountMgmt.log`, `Subscription.log`

## Log Configuration

### Adjusting Log Levels
Most services use appsettings.json configuration:
```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "System": "Warning"
    }
  }
}
```

### Enabling Debug Logging
For detailed troubleshooting, temporarily set to Debug:
```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Debug"
    }
  }
}
```

**Warning**: Debug logging generates large volumes of data.

## Integration with Health Checks

The health check application monitors log files for:
- Recent error patterns
- Service availability indicators
- Performance metrics
- System health indicators

Results are logged to: `D:\logs\HealthCheckApp\`

**Next**: [Debugging .NET Core Projects](./debug-netcore.md)
