# General Troubleshooting

## Common Development Environment Issues

This guide provides systematic approaches to resolving common issues in the Method development environment.

## Step-by-Step Troubleshooting Process

### 1. Identify the Problem
- **Note Error Messages** - Copy exact error text and stack traces
- **Document Symptoms** - What was working? What changed?
- **Check Timing** - When did the issue first appear?

### 2. Check Service Status
```powershell
# SQL Server Services
Get-Service -Name "MSSQL*" | Select-Object Name, Status
Get-Service -Name "SQLAgent*" | Select-Object Name, Status

# Docker Containers
docker ps -a

# IIS Application Pools
Get-IISAppPool | Select-Object Name, State
```

### 3. Review Log Files
- **Application Logs**: `D:\logs\`
- **IIS Logs**: `C:\inetpub\logs\LogFiles\`
- **Windows Event Logs**: Event Viewer → Windows Logs → Application
- **SQL Server Logs**: SSMS → Management → SQL Server Logs

### 4. Verify Network Connectivity
```powershell
# Test database connections
Test-NetConnection -ComputerName methodlocaldb -Port 1433

# Check MongoDB
Test-NetConnection -ComputerName localhost -Port 27017

# Check Redis
Test-NetConnection -ComputerName localhost -Port 6379
```

## Most Common Issues & Solutions

### SQL Server Connection Issues
**Symptoms**: Cannot connect to database, timeout errors
**Solutions**:
- Verify SQL Server service is running
- Check SQL aliases configuration
- Confirm methodlocaldb in host file
- Test with SSMS first

### IIS Application Pool Failures
**Symptoms**: 503 Service Unavailable, application won't start
**Solutions**:
- Check application pool identity and permissions
- Verify .NET runtime versions
- Review IIS logs for specific errors
- Restart application pools

### Docker Container Issues
**Symptoms**: Services unavailable, connection refused
**Solutions**:
- Verify Docker Desktop is running
- Check container status with `docker ps`
- Restart containers: `docker restart <container_name>`
- Check port conflicts

### Build and Compilation Errors
**Symptoms**: MSBuild failures, missing packages
**Solutions**:
- Verify VPN connection for NuGet feeds
- Clear NuGet cache: `dotnet nuget locals all --clear`
- Check MSBuild path in scripts
- Verify Visual Studio installation

### SSL Certificate Errors
**Symptoms**: HTTPS sites not loading, certificate warnings
**Solutions**:
- Verify certificate installation in Windows certificate store
- Check IIS bindings for HTTPS sites
- Confirm certificate expiration dates
- Test with different browsers

## Health Check Validation

Run the health check application to identify issues:
```powershell
cd C:\MethodDev\DeveloperTools\Method-HealthChecks\publish\
.\HealthCheck.App.exe
```

Review results and focus on failed components first.

## Getting Help

### Internal Resources
- **Slack Channels**: #dev-help, #platform-team
- **Documentation**: This troubleshooting section
- **Health Check Logs**: `D:\logs\HealthCheckApp\`

### Escalation Process
1. Try documented solutions first
2. Search Slack history for similar issues
3. Post in #dev-help with error details and steps attempted
4. Contact platform team for infrastructure issues

**Next**: [Log Files](./log-files.md)
