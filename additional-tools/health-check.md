# Health Check Application

## Method Health Check System

This guide covers implementing and using comprehensive health checks for Method's development environment and applications.

## Overview

Method's health check system provides:

- **Service Health Monitoring** - Real-time status of critical services
- **Dependency Checks** - Database, cache, and external service availability
- **Performance Metrics** - Response times and resource utilization
- **Alert Integration** - Proactive notification of service degradation
- **Dashboard Views** - Visual health status overview

## Health Check Architecture

### Core Components

```
Method Health Check System
├── Health Check API       # ASP.NET Core health endpoints
├── Check Implementations  # Individual service checks
├── Health Dashboard       # Web-based monitoring interface
├── Alert Engine          # Notification and escalation
└── Metrics Collection    # Performance data gathering
```

### Check Categories

**Infrastructure Checks:**
- Database connectivity (SQL Server, MongoDB)
- Cache services (Redis, MemoryCache)
- File system access and disk space
- Network connectivity

**Application Checks:**
- Service endpoint availability
- Authentication system status
- Background service health
- Configuration validation

**Business Checks:**
- Critical workflow availability
- Integration endpoint status
- License validation
- Data consistency checks

## ASP.NET Core Health Checks

### Basic Setup

#### Startup Configuration
```csharp
// Startup.cs or Program.cs
public void ConfigureServices(IServiceCollection services)
{
    // Add health checks
    services.AddHealthChecks()
        // Database checks
        .AddSqlServer(
            connectionString: Configuration.GetConnectionString("DefaultConnection"),
            name: "sqlserver",
            timeout: TimeSpan.FromSeconds(30),
            tags: new[] { "database", "critical" })
        
        // MongoDB check
        .AddMongoDb(
            mongodbConnectionString: Configuration.GetConnectionString("MongoDB"),
            name: "mongodb",
            timeout: TimeSpan.FromSeconds(15),
            tags: new[] { "database", "nosql" })
        
        // HTTP endpoint checks
        .AddUrlGroup(
            uri: new Uri("https://api.method.com/health"),
            name: "method-api",
            timeout: TimeSpan.FromSeconds(10),
            tags: new[] { "api", "external" })
        
        // Custom checks
        .AddCheck<FileSystemHealthCheck>("filesystem")
        .AddCheck<LicenseHealthCheck>("license")
        .AddCheck<WorkflowEngineHealthCheck>("workflow-engine");
    
    // Add health check UI
    services.AddHealthChecksUI(setup =>
    {
        setup.AddHealthCheckEndpoint("Method Local", "/health");
        setup.SetEvaluationTimeInSeconds(30);
        setup.SetMinimumSecondsBetweenFailureNotifications(60);
    }).AddInMemoryStorage();
}

public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    // Health check endpoints
    app.UseHealthChecks("/health", new HealthCheckOptions
    {
        ResponseWriter = UIResponseWriter.WriteHealthCheckUIResponse,
        ResultStatusCodes =
        {
            [HealthStatus.Healthy] = StatusCodes.Status200OK,
            [HealthStatus.Degraded] = StatusCodes.Status200OK,
            [HealthStatus.Unhealthy] = StatusCodes.Status503ServiceUnavailable
        }
    });
    
    // Detailed health check with filters
    app.UseHealthChecks("/health/critical", new HealthCheckOptions
    {
        Predicate = check => check.Tags.Contains("critical"),
        ResponseWriter = UIResponseWriter.WriteHealthCheckUIResponse
    });
    
    // Health check UI
    app.UseHealthChecksUI(setup =>
    {
        setup.UIPath = "/health-ui";
        setup.ApiPath = "/health-api";
    });
}
```

### Custom Health Checks

#### File System Health Check
```csharp
// FileSystemHealthCheck.cs
public class FileSystemHealthCheck : IHealthCheck
{
    private readonly ILogger<FileSystemHealthCheck> _logger;
    private readonly IConfiguration _configuration;
    
    public FileSystemHealthCheck(ILogger<FileSystemHealthCheck> logger, IConfiguration configuration)
    {
        _logger = logger;
        _configuration = configuration;
    }
    
    public async Task<HealthCheckResult> CheckHealthAsync(HealthCheckContext context, CancellationToken cancellationToken = default)
    {
        try
        {
            var checks = new List<(string Path, string Description, bool Critical)>
            {
                ("D:\\logs", "Log directory access", true),
                ("D:\\MethodData", "Method data directory", true),
                ("C:\\temp", "Temporary directory", false)
            };
            
            var results = new Dictionary<string, object>();
            var hasErrors = false;
            var hasCriticalErrors = false;
            
            foreach (var (path, description, critical) in checks)
            {
                try
                {
                    // Check directory accessibility
                    var dirInfo = new DirectoryInfo(path);
                    var accessible = dirInfo.Exists;
                    
                    if (accessible)
                    {
                        // Check available space
                        var drive = new DriveInfo(dirInfo.Root.FullName);
                        var freeSpaceGB = drive.AvailableFreeSpace / (1024 * 1024 * 1024);
                        var totalSpaceGB = drive.TotalSize / (1024 * 1024 * 1024);
                        var freeSpacePercent = (double)drive.AvailableFreeSpace / drive.TotalSize * 100;
                        
                        results[path] = new
                        {
                            accessible = true,
                            freeSpaceGB = Math.Round(freeSpaceGB, 2),
                            totalSpaceGB = Math.Round(totalSpaceGB, 2),
                            freeSpacePercent = Math.Round(freeSpacePercent, 1),
                            status = freeSpacePercent < 10 ? "Low Space" : "OK"
                        };
                        
                        if (freeSpacePercent < 5)
                        {
                            hasErrors = true;
                            if (critical) hasCriticalErrors = true;
                        }
                    }
                    else
                    {
                        results[path] = new { accessible = false, error = "Directory not accessible" };
                        hasErrors = true;
                        if (critical) hasCriticalErrors = true;
                    }
                }
                catch (Exception ex)
                {
                    results[path] = new { accessible = false, error = ex.Message };
                    hasErrors = true;
                    if (critical) hasCriticalErrors = true;
                }
            }
            
            if (hasCriticalErrors)
            {
                return HealthCheckResult.Unhealthy("Critical file system issues detected", null, results);
            }
            else if (hasErrors)
            {
                return HealthCheckResult.Degraded("File system warnings detected", null, results);
            }
            else
            {
                return HealthCheckResult.Healthy("All file system checks passed", results);
            }
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error during file system health check");
            return HealthCheckResult.Unhealthy("File system health check failed", ex);
        }
    }
}
```

#### Method License Health Check
```csharp
// LicenseHealthCheck.cs
public class LicenseHealthCheck : IHealthCheck
{
    private readonly ILicenseService _licenseService;
    private readonly ILogger<LicenseHealthCheck> _logger;
    
    public LicenseHealthCheck(ILicenseService licenseService, ILogger<LicenseHealthCheck> logger)
    {
        _licenseService = licenseService;
        _logger = logger;
    }
    
    public async Task<HealthCheckResult> CheckHealthAsync(HealthCheckContext context, CancellationToken cancellationToken = default)
    {
        try
        {
            var licenseInfo = await _licenseService.GetLicenseInfoAsync();
            
            if (licenseInfo == null)
            {
                return HealthCheckResult.Unhealthy("No license information found");
            }
            
            var now = DateTime.UtcNow;
            var daysUntilExpiry = (licenseInfo.ExpiryDate - now).TotalDays;
            
            var data = new Dictionary<string, object>
            {
                ["license_type"] = licenseInfo.LicenseType,
                ["expiry_date"] = licenseInfo.ExpiryDate,
                ["days_until_expiry"] = Math.Round(daysUntilExpiry, 1),
                ["user_limit"] = licenseInfo.UserLimit,
                ["current_users"] = licenseInfo.CurrentUsers,
                ["features"] = licenseInfo.EnabledFeatures
            };
            
            if (licenseInfo.ExpiryDate <= now)
            {
                return HealthCheckResult.Unhealthy("License has expired", null, data);
            }
            else if (daysUntilExpiry <= 30)
            {
                return HealthCheckResult.Degraded($"License expires in {Math.Round(daysUntilExpiry)} days", null, data);
            }
            else if (licenseInfo.CurrentUsers >= licenseInfo.UserLimit * 0.9)
            {
                return HealthCheckResult.Degraded("Approaching user limit", null, data);
            }
            else
            {
                return HealthCheckResult.Healthy("License is valid and healthy", data);
            }
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error checking license health");
            return HealthCheckResult.Unhealthy("License health check failed", ex);
        }
    }
}
```

#### Workflow Engine Health Check
```csharp
// WorkflowEngineHealthCheck.cs
public class WorkflowEngineHealthCheck : IHealthCheck
{
    private readonly IWorkflowService _workflowService;
    private readonly ILogger<WorkflowEngineHealthCheck> _logger;
    
    public WorkflowEngineHealthCheck(IWorkflowService workflowService, ILogger<WorkflowEngineHealthCheck> logger)
    {
        _workflowService = workflowService;
        _logger = logger;
    }
    
    public async Task<HealthCheckResult> CheckHealthAsync(HealthCheckContext context, CancellationToken cancellationToken = default)
    {
        try
        {
            var startTime = DateTime.UtcNow;
            
            // Test workflow engine responsiveness
            var engineStatus = await _workflowService.GetEngineStatusAsync();
            
            // Get workflow statistics
            var stats = await _workflowService.GetWorkflowStatsAsync();
            
            var responseTime = (DateTime.UtcNow - startTime).TotalMilliseconds;
            
            var data = new Dictionary<string, object>
            {
                ["engine_status"] = engineStatus.Status,
                ["active_workflows"] = stats.ActiveWorkflows,
                ["pending_workflows"] = stats.PendingWorkflows,
                ["failed_workflows"] = stats.FailedWorkflows,
                ["response_time_ms"] = Math.Round(responseTime, 2),
                ["worker_threads"] = engineStatus.WorkerThreads,
                ["queue_length"] = stats.QueueLength
            };
            
            if (engineStatus.Status != "Running")
            {
                return HealthCheckResult.Unhealthy("Workflow engine is not running", null, data);
            }
            else if (stats.FailedWorkflows > 10 || stats.QueueLength > 100)
            {
                return HealthCheckResult.Degraded("Workflow engine experiencing high load or failures", null, data);
            }
            else if (responseTime > 5000)
            {
                return HealthCheckResult.Degraded("Workflow engine response time is slow", null, data);
            }
            else
            {
                return HealthCheckResult.Healthy("Workflow engine is healthy", data);
            }
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error checking workflow engine health");
            return HealthCheckResult.Unhealthy("Workflow engine health check failed", ex);
        }
    }
}
```

## Health Check Dashboard

### React Health Dashboard
```javascript
// HealthDashboard.js
import React, { useState, useEffect } from 'react';
import { Card, Alert, Badge, Progress, Table, Button } from 'antd';
import { CheckCircleOutlined, CloseCircleOutlined, WarningOutlined, LoadingOutlined } from '@ant-design/icons';

const HealthDashboard = () => {
    const [healthData, setHealthData] = useState(null);
    const [loading, setLoading] = useState(true);
    const [lastUpdated, setLastUpdated] = useState(null);
    
    useEffect(() => {
        loadHealthData();
        const interval = setInterval(loadHealthData, 30000); // Refresh every 30 seconds
        return () => clearInterval(interval);
    }, []);
    
    const loadHealthData = async () => {
        try {
            const response = await fetch('/health');
            const data = await response.json();
            setHealthData(data);
            setLastUpdated(new Date());
            setLoading(false);
        } catch (error) {
            console.error('Failed to load health data:', error);
            setLoading(false);
        }
    };
    
    const getStatusIcon = (status) => {
        switch (status) {
            case 'Healthy':
                return <CheckCircleOutlined style={{ color: '#52c41a' }} />;
            case 'Degraded':
                return <WarningOutlined style={{ color: '#faad14' }} />;
            case 'Unhealthy':
                return <CloseCircleOutlined style={{ color: '#ff4d4f' }} />;
            default:
                return <LoadingOutlined />;
        }
    };
    
    const getStatusColor = (status) => {
        switch (status) {
            case 'Healthy': return 'success';
            case 'Degraded': return 'warning';
            case 'Unhealthy': return 'error';
            default: return 'default';
        }
    };
    
    const columns = [
        {
            title: 'Service',
            dataIndex: 'name',
            key: 'name',
            render: (text, record) => (
                <div>
                    <strong>{text}</strong>
                    {record.tags && (
                        <div>
                            {record.tags.map(tag => (
                                <Badge key={tag} color="blue" text={tag} style={{ marginRight: 4 }} />
                            ))}
                        </div>
                    )}
                </div>
            )
        },
        {
            title: 'Status',
            dataIndex: 'status',
            key: 'status',
            render: (status) => (
                <span>
                    {getStatusIcon(status)}
                    <Badge color={getStatusColor(status)} text={status} style={{ marginLeft: 8 }} />
                </span>
            )
        },
        {
            title: 'Response Time',
            dataIndex: 'duration',
            key: 'duration',
            render: (duration) => duration ? `${Math.round(duration)}ms` : 'N/A'
        },
        {
            title: 'Description',
            dataIndex: 'description',
            key: 'description'
        },
        {
            title: 'Details',
            dataIndex: 'data',
            key: 'data',
            render: (data) => data ? (
                <details>
                    <summary>View Details</summary>
                    <pre style={{ fontSize: '12px', marginTop: '8px' }}>
                        {JSON.stringify(data, null, 2)}
                    </pre>
                </details>
            ) : null
        }
    ];
    
    if (loading) {
        return (
            <div style={{ textAlign: 'center', padding: '50px' }}>
                <LoadingOutlined style={{ fontSize: '24px' }} />
                <div style={{ marginTop: '16px' }}>Loading health status...</div>
            </div>
        );
    }
    
    if (!healthData) {
        return (
            <Alert
                message="Health Check Unavailable"
                description="Unable to load health check data. The health check service may be down."
                type="error"
                showIcon
            />
        );
    }
    
    const overallStatus = healthData.status;
    const healthyCount = healthData.entries ? Object.values(healthData.entries).filter(e => e.status === 'Healthy').length : 0;
    const totalCount = healthData.entries ? Object.keys(healthData.entries).length : 0;
    const healthPercent = totalCount > 0 ? (healthyCount / totalCount) * 100 : 0;
    
    const tableData = healthData.entries ? Object.entries(healthData.entries).map(([name, entry]) => ({
        key: name,
        name,
        ...entry
    })) : [];
    
    return (
        <div className="health-dashboard">
            <div style={{ marginBottom: '24px' }}>
                <h1>Method System Health Dashboard</h1>
                {lastUpdated && (
                    <div style={{ color: '#666', marginBottom: '16px' }}>
                        Last updated: {lastUpdated.toLocaleString()}
                        <Button 
                            type="link" 
                            onClick={loadHealthData}
                            style={{ marginLeft: '16px' }}
                        >
                            Refresh
                        </Button>
                    </div>
                )}
            </div>
            
            <div style={{ marginBottom: '24px' }}>
                <Card>
                    <div style={{ textAlign: 'center' }}>
                        <div style={{ fontSize: '48px', marginBottom: '16px' }}>
                            {getStatusIcon(overallStatus)}
                        </div>
                        <h2>Overall Status: {overallStatus}</h2>
                        <div style={{ marginTop: '16px' }}>
                            <Progress
                                percent={Math.round(healthPercent)}
                                status={healthPercent === 100 ? 'success' : healthPercent >= 75 ? 'normal' : 'exception'}
                                format={() => `${healthyCount}/${totalCount} Services Healthy`}
                            />
                        </div>
                    </div>
                </Card>
            </div>
            
            <Card title="Service Health Details">
                <Table
                    columns={columns}
                    dataSource={tableData}
                    pagination={false}
                    size="middle"
                />
            </Card>
            
            {healthData.totalDuration && (
                <Card title="Performance Summary" style={{ marginTop: '16px' }}>
                    <div>
                        <strong>Total Check Duration:</strong> {Math.round(healthData.totalDuration)}ms
                    </div>
                </Card>
            )}
        </div>
    );
};

export default HealthDashboard;
```

### PowerShell Health Monitoring
```powershell
# MethodHealthMonitor.ps1
function Test-MethodSystemHealth {
    param(
        [string]$HealthEndpoint = "http://localhost:5000/health",
        [switch]$Detailed = $false,
        [switch]$AlertOnFailure = $false
    )
    
    Write-Host "=== Method System Health Check ===" -ForegroundColor Green
    Write-Host "Endpoint: $HealthEndpoint" -ForegroundColor Gray
    Write-Host "Timestamp: $(Get-Date)" -ForegroundColor Gray
    Write-Host ""
    
    try {
        $response = Invoke-RestMethod -Uri $HealthEndpoint -Method Get -TimeoutSec 30
        
        $overallStatus = $response.status
        $statusColor = switch ($overallStatus) {
            "Healthy" { "Green" }
            "Degraded" { "Yellow" }
            "Unhealthy" { "Red" }
            default { "Gray" }
        }
        
        Write-Host "Overall Status: $overallStatus" -ForegroundColor $statusColor
        Write-Host ""
        
        if ($response.entries) {
            Write-Host "Service Details:" -ForegroundColor Yellow
            
            foreach ($serviceName in $response.entries.PSObject.Properties.Name) {
                $service = $response.entries.$serviceName
                $serviceColor = switch ($service.status) {
                    "Healthy" { "Green" }
                    "Degraded" { "Yellow" }
                    "Unhealthy" { "Red" }
                    default { "Gray" }
                }
                
                $duration = if ($service.duration) { " ($([math]::Round($service.duration))ms)" } else { "" }
                Write-Host "  [$($service.status)] $serviceName$duration" -ForegroundColor $serviceColor
                
                if ($service.description) {
                    Write-Host "    $($service.description)" -ForegroundColor Gray
                }
                
                if ($Detailed -and $service.data) {
                    Write-Host "    Data: $($service.data | ConvertTo-Json -Compress)" -ForegroundColor Gray
                }
                
                if ($service.exception) {
                    Write-Host "    Error: $($service.exception)" -ForegroundColor Red
                }
            }
        }
        
        if ($response.totalDuration) {
            Write-Host ""
            Write-Host "Total Duration: $([math]::Round($response.totalDuration))ms" -ForegroundColor Gray
        }
        
        # Alert on failure if requested
        if ($AlertOnFailure -and $overallStatus -ne "Healthy") {
            Send-MethodHealthAlert -Status $overallStatus -Details $response
        }
        
        return $response
    }
    catch {
        Write-Host "Health check failed: $($_.Exception.Message)" -ForegroundColor Red
        
        if ($AlertOnFailure) {
            Send-MethodHealthAlert -Status "Failed" -Error $_.Exception.Message
        }
        
        return $null
    }
}

function Send-MethodHealthAlert {
    param(
        [string]$Status,
        [object]$Details = $null,
        [string]$Error = $null
    )
    
    $alertMessage = @"
Method System Health Alert

Status: $Status
Timestamp: $(Get-Date)
Server: $env:COMPUTERNAME

$(if ($Error) { "Error: $Error" })
$(if ($Details) { "Details: $($Details | ConvertTo-Json -Depth 3)" })
"@
    
    # Send email alert (configure SMTP settings)
    try {
        $smtpParams = @{
            SmtpServer = "smtp.company.com"
            From = "health-monitor@method.com"
            To = "devops@method.com"
            Subject = "Method Health Alert - $Status"
            Body = $alertMessage
        }
        
        Send-MailMessage @smtpParams
        Write-Host "Health alert sent via email" -ForegroundColor Yellow
    }
    catch {
        Write-Host "Failed to send email alert: $($_.Exception.Message)" -ForegroundColor Red
    }
    
    # Write to Windows Event Log
    try {
        $eventId = switch ($Status) {
            "Unhealthy" { 1001 }
            "Degraded" { 1002 }
            "Failed" { 1003 }
            default { 1000 }
        }
        
        $entryType = switch ($Status) {
            "Unhealthy" { "Error" }
            "Degraded" { "Warning" }
            "Failed" { "Error" }
            default { "Information" }
        }
        
        Write-EventLog -LogName "Application" -Source "Method Health Monitor" -EventId $eventId -EntryType $entryType -Message $alertMessage
    }
    catch {
        Write-Host "Failed to write to event log: $($_.Exception.Message)" -ForegroundColor Red
    }
}

# Continuous monitoring function
function Start-MethodHealthMonitoring {
    param(
        [int]$IntervalSeconds = 300, # 5 minutes
        [string]$HealthEndpoint = "http://localhost:5000/health"
    )
    
    Write-Host "Starting Method health monitoring..." -ForegroundColor Green
    Write-Host "Interval: $IntervalSeconds seconds" -ForegroundColor Gray
    Write-Host "Press Ctrl+C to stop" -ForegroundColor Gray
    Write-Host ""
    
    while ($true) {
        try {
            $health = Test-MethodSystemHealth -HealthEndpoint $HealthEndpoint -AlertOnFailure
            
            if ($health) {
                # Log to file
                $logEntry = @{
                    timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
                    status = $health.status
                    duration = $health.totalDuration
                    services = @{}
                }
                
                if ($health.entries) {
                    foreach ($serviceName in $health.entries.PSObject.Properties.Name) {
                        $service = $health.entries.$serviceName
                        $logEntry.services[$serviceName] = @{
                            status = $service.status
                            duration = $service.duration
                        }
                    }
                }
                
                $logFile = "C:\Logs\MethodHealthCheck\health-$(Get-Date -Format 'yyyy-MM-dd').log"
                $logDir = Split-Path $logFile -Parent
                if (!(Test-Path $logDir)) {
                    New-Item -ItemType Directory -Path $logDir -Force | Out-Null
                }
                
                Add-Content -Path $logFile -Value ($logEntry | ConvertTo-Json -Compress)
            }
        }
        catch {
            Write-Host "Monitoring error: $($_.Exception.Message)" -ForegroundColor Red
        }
        
        Start-Sleep -Seconds $IntervalSeconds
    }
}

# Create Windows service for health monitoring
function Install-MethodHealthService {
    $serviceName = "MethodHealthMonitor"
    $serviceDisplayName = "Method Health Monitor"
    $serviceDescription = "Monitors Method system health and sends alerts"
    
    $scriptPath = "C:\Scripts\MethodHealthService.ps1"
    $serviceScript = @"
# Method Health Service Script
Add-Type -AssemblyName System.ServiceProcess

while (`$true) {
    try {
        & "C:\Scripts\MethodHealthMonitor.ps1" -IntervalSeconds 300
    }
    catch {
        Write-EventLog -LogName Application -Source "Method Health Monitor" -EntryType Error -EventId 9999 -Message "Health monitoring error: `$(`$_.Exception.Message)"
    }
    
    Start-Sleep -Seconds 60
}
"@
    
    # Create script file
    $scriptDir = Split-Path $scriptPath -Parent
    if (!(Test-Path $scriptDir)) {
        New-Item -ItemType Directory -Path $scriptDir -Force
    }
    
    Set-Content -Path $scriptPath -Value $serviceScript
    
    # Create and install service
    $serviceParams = @{
        Name = $serviceName
        DisplayName = $serviceDisplayName
        Description = $serviceDescription
        BinaryPathName = "PowerShell.exe -ExecutionPolicy Bypass -File `"$scriptPath`""
        StartupType = "Automatic"
    }
    
    try {
        New-Service @serviceParams
        Write-Host "Method Health Service installed successfully" -ForegroundColor Green
        Write-Host "Use 'Start-Service $serviceName' to start the service" -ForegroundColor Yellow
    }
    catch {
        Write-Host "Failed to install service: $($_.Exception.Message)" -ForegroundColor Red
    }
}

# Example usage functions
function Get-MethodHealthSummary {
    $healthData = Test-MethodSystemHealth -Detailed
    
    if ($healthData) {
        $summary = @{
            OverallStatus = $healthData.status
            Timestamp = Get-Date
            TotalServices = 0
            HealthyServices = 0
            DegradedServices = 0
            UnhealthyServices = 0
            TotalDuration = $healthData.totalDuration
        }
        
        if ($healthData.entries) {
            foreach ($serviceName in $healthData.entries.PSObject.Properties.Name) {
                $service = $healthData.entries.$serviceName
                $summary.TotalServices++
                
                switch ($service.status) {
                    "Healthy" { $summary.HealthyServices++ }
                    "Degraded" { $summary.DegradedServices++ }
                    "Unhealthy" { $summary.UnhealthyServices++ }
                }
            }
        }
        
        return $summary
    }
    
    return $null
}

# Export functions for module use
Export-ModuleMember -Function Test-MethodSystemHealth, Send-MethodHealthAlert, Start-MethodHealthMonitoring, Install-MethodHealthService, Get-MethodHealthSummary
```

## Integration with Monitoring Systems

### Prometheus Metrics Export
```csharp
// PrometheusHealthMetrics.cs
public class PrometheusHealthMetrics : IHostedService
{
    private readonly IHealthCheckService _healthCheckService;
    private readonly ILogger<PrometheusHealthMetrics> _logger;
    private Timer _timer;
    
    // Prometheus metrics
    private static readonly Gauge HealthStatus = Metrics
        .CreateGauge("method_health_status", "Health check status (1=Healthy, 0.5=Degraded, 0=Unhealthy)", 
            new[] { "service", "category" });
    
    private static readonly Histogram HealthCheckDuration = Metrics
        .CreateHistogram("method_health_check_duration_seconds", "Health check duration in seconds",
            new[] { "service" });
    
    public PrometheusHealthMetrics(IHealthCheckService healthCheckService, ILogger<PrometheusHealthMetrics> logger)
    {
        _healthCheckService = healthCheckService;
        _logger = logger;
    }
    
    public Task StartAsync(CancellationToken cancellationToken)
    {
        _timer = new Timer(UpdateMetrics, null, TimeSpan.Zero, TimeSpan.FromMinutes(1));
        return Task.CompletedTask;
    }
    
    private async void UpdateMetrics(object state)
    {
        try
        {
            var healthReport = await _healthCheckService.CheckHealthAsync();
            
            foreach (var entry in healthReport.Entries)
            {
                var serviceName = entry.Key;
                var healthEntry = entry.Value;
                
                // Convert health status to numeric value
                var statusValue = healthEntry.Status switch
                {
                    HealthStatus.Healthy => 1.0,
                    HealthStatus.Degraded => 0.5,
                    HealthStatus.Unhealthy => 0.0,
                    _ => 0.0
                };
                
                var category = healthEntry.Tags?.FirstOrDefault() ?? "general";
                
                HealthStatus.WithLabels(serviceName, category).Set(statusValue);
                HealthCheckDuration.WithLabels(serviceName).Observe(healthEntry.Duration.TotalSeconds);
            }
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error updating Prometheus health metrics");
        }
    }
    
    public Task StopAsync(CancellationToken cancellationToken)
    {
        _timer?.Dispose();
        return Task.CompletedTask;
    }
}
```

### Health Check Alerts
```csharp
// HealthCheckAlertService.cs
public class HealthCheckAlertService : IHostedService
{
    private readonly IHealthCheckService _healthCheckService;
    private readonly INotificationService _notificationService;
    private readonly ILogger<HealthCheckAlertService> _logger;
    private Timer _timer;
    private HealthStatus _lastOverallStatus = HealthStatus.Healthy;
    private Dictionary<string, HealthStatus> _lastServiceStatuses = new();
    
    public HealthCheckAlertService(
        IHealthCheckService healthCheckService,
        INotificationService notificationService,
        ILogger<HealthCheckAlertService> logger)
    {
        _healthCheckService = healthCheckService;
        _notificationService = notificationService;
        _logger = logger;
    }
    
    public Task StartAsync(CancellationToken cancellationToken)
    {
        _timer = new Timer(CheckHealthAndAlert, null, TimeSpan.Zero, TimeSpan.FromMinutes(5));
        return Task.CompletedTask;
    }
    
    private async void CheckHealthAndAlert(object state)
    {
        try
        {
            var healthReport = await _healthCheckService.CheckHealthAsync();
            
            // Check for overall status changes
            if (healthReport.Status != _lastOverallStatus)
            {
                await SendOverallStatusAlert(healthReport.Status, _lastOverallStatus);
                _lastOverallStatus = healthReport.Status;
            }
            
            // Check for individual service status changes
            foreach (var entry in healthReport.Entries)
            {
                var serviceName = entry.Key;
                var currentStatus = entry.Value.Status;
                
                if (_lastServiceStatuses.TryGetValue(serviceName, out var lastStatus))
                {
                    if (currentStatus != lastStatus && currentStatus != HealthStatus.Healthy)
                    {
                        await SendServiceStatusAlert(serviceName, currentStatus, lastStatus, entry.Value);
                    }
                }
                
                _lastServiceStatuses[serviceName] = currentStatus;
            }
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error during health check alerting");
        }
    }
    
    private async Task SendOverallStatusAlert(HealthStatus currentStatus, HealthStatus previousStatus)
    {
        var severity = currentStatus switch
        {
            HealthStatus.Unhealthy => NotificationSeverity.Critical,
            HealthStatus.Degraded => NotificationSeverity.Warning,
            _ => NotificationSeverity.Info
        };
        
        var message = $"Method system health status changed from {previousStatus} to {currentStatus}";
        
        await _notificationService.SendNotificationAsync(new Notification
        {
            Title = "Method System Health Alert",
            Message = message,
            Severity = severity,
            Channel = NotificationChannel.Email | NotificationChannel.Slack,
            Tags = new[] { "health", "system", "alert" }
        });
    }
    
    private async Task SendServiceStatusAlert(string serviceName, HealthStatus currentStatus, HealthStatus previousStatus, HealthReportEntry entry)
    {
        var severity = currentStatus switch
        {
            HealthStatus.Unhealthy => NotificationSeverity.High,
            HealthStatus.Degraded => NotificationSeverity.Medium,
            _ => NotificationSeverity.Low
        };
        
        var message = $"Service '{serviceName}' status changed from {previousStatus} to {currentStatus}";
        if (!string.IsNullOrEmpty(entry.Description))
        {
            message += $"\nDescription: {entry.Description}";
        }
        
        await _notificationService.SendNotificationAsync(new Notification
        {
            Title = $"Method Service Alert - {serviceName}",
            Message = message,
            Severity = severity,
            Channel = NotificationChannel.Email,
            Tags = new[] { "health", "service", serviceName.ToLower() }
        });
    }
    
    public Task StopAsync(CancellationToken cancellationToken)
    {
        _timer?.Dispose();
        return Task.CompletedTask;
    }
}
```

## Best Practices

### Health Check Guidelines

1. **Keep Checks Fast** - Target <5 seconds for all checks combined
2. **Use Timeouts** - Prevent hanging checks from blocking the endpoint
3. **Include Dependencies** - Check critical dependencies like databases
4. **Provide Meaningful Data** - Include useful diagnostic information
5. **Use Appropriate Status Levels** - Healthy/Degraded/Unhealthy based on impact

### Monitoring Strategy

1. **Layer Monitoring** - Infrastructure, application, and business health
2. **Set Alert Thresholds** - Avoid alert fatigue with appropriate thresholds
3. **Include Recovery Actions** - Document remediation steps for common issues
4. **Test Regularly** - Verify health checks work as expected
5. **Monitor the Monitor** - Ensure health check system itself is healthy

### Security Considerations

1. **Limit Exposure** - Don't expose sensitive information in health data
2. **Control Access** - Restrict detailed health information to authorized users
3. **Secure Endpoints** - Use authentication for detailed health endpoints
4. **Audit Access** - Log who accesses health information
5. **Rate Limiting** - Prevent abuse of health check endpoints

**Next**: [Elasticsearch & Kibana Integration](./elasticsearch-kibana.md)
