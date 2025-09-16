# Audit Trail Projects

## Method Audit Trail and Event Tracking

This guide covers setting up and using Method's audit trail system for tracking user actions, data changes, and system events.

## Overview

Method's audit trail system provides:

- **User Action Tracking** - Login attempts, data modifications, system access
- **Data Change Auditing** - Before/after values for critical data
- **System Event Logging** - API calls, service interactions, errors
- **Compliance Reporting** - Audit reports for regulatory requirements
- **Security Monitoring** - Suspicious activity detection and alerting

## Audit Trail Architecture

### Core Components

```
Method Audit System
├── Event Collectors      # Capture events from various sources
├── Event Processors      # Normalize and enrich event data
├── Storage Layer         # Event persistence (SQL + MongoDB)
├── Query Engine          # Search and retrieve audit data
└── Reporting Service     # Generate audit reports
```

### Event Types

**User Events:**
- Authentication (login, logout, password changes)
- Authorization (permission grants, role changes)
- Data access (view, export, print)
- Data modification (create, update, delete)

**System Events:**
- API requests and responses
- Service startup/shutdown
- Configuration changes
- Error conditions

**Business Events:**
- Workflow executions
- Integration events
- Notification deliveries
- Payment transactions

## Setup and Configuration

### 1. Audit Database Setup

#### SQL Server Audit Tables
```sql
-- Create audit database
CREATE DATABASE MethodAuditTrail;
GO

USE MethodAuditTrail;
GO

-- Core audit events table
CREATE TABLE AuditEvents (
    Id BIGINT IDENTITY(1,1) PRIMARY KEY,
    EventId UNIQUEIDENTIFIER NOT NULL DEFAULT NEWID(),
    Timestamp DATETIME2 NOT NULL DEFAULT GETUTCDATE(),
    EventType VARCHAR(100) NOT NULL,
    EventCategory VARCHAR(50) NOT NULL,
    UserId VARCHAR(100),
    UserName VARCHAR(255),
    SessionId VARCHAR(100),
    IpAddress VARCHAR(45),
    UserAgent VARCHAR(500),
    ResourceType VARCHAR(100),
    ResourceId VARCHAR(100),
    Action VARCHAR(100) NOT NULL,
    Result VARCHAR(20) NOT NULL, -- Success, Failure, Error
    Details NVARCHAR(MAX), -- JSON data
    BeforeData NVARCHAR(MAX), -- JSON data
    AfterData NVARCHAR(MAX), -- JSON data
    CorrelationId VARCHAR(100),
    ServiceName VARCHAR(100),
    INDEX IX_AuditEvents_Timestamp (Timestamp),
    INDEX IX_AuditEvents_UserId (UserId),
    INDEX IX_AuditEvents_EventType (EventType),
    INDEX IX_AuditEvents_ResourceType (ResourceType),
    INDEX IX_AuditEvents_CorrelationId (CorrelationId)
);

-- User session tracking
CREATE TABLE UserSessions (
    SessionId VARCHAR(100) PRIMARY KEY,
    UserId VARCHAR(100) NOT NULL,
    UserName VARCHAR(255),
    LoginTime DATETIME2 NOT NULL,
    LogoutTime DATETIME2,
    IpAddress VARCHAR(45),
    UserAgent VARCHAR(500),
    IsActive BIT NOT NULL DEFAULT 1,
    LastActivity DATETIME2 NOT NULL,
    INDEX IX_UserSessions_UserId (UserId),
    INDEX IX_UserSessions_LoginTime (LoginTime)
);

-- Security alerts
CREATE TABLE SecurityAlerts (
    Id BIGINT IDENTITY(1,1) PRIMARY KEY,
    AlertId UNIQUEIDENTIFIER NOT NULL DEFAULT NEWID(),
    Timestamp DATETIME2 NOT NULL DEFAULT GETUTCDATE(),
    AlertType VARCHAR(100) NOT NULL,
    Severity VARCHAR(20) NOT NULL, -- Low, Medium, High, Critical
    UserId VARCHAR(100),
    Description NVARCHAR(500),
    EventIds NVARCHAR(MAX), -- JSON array of related event IDs
    Status VARCHAR(20) NOT NULL DEFAULT 'Open', -- Open, Investigating, Resolved
    AssignedTo VARCHAR(100),
    ResolutionNotes NVARCHAR(MAX),
    INDEX IX_SecurityAlerts_Timestamp (Timestamp),
    INDEX IX_SecurityAlerts_AlertType (AlertType),
    INDEX IX_SecurityAlerts_Status (Status)
);
```

#### MongoDB Audit Collections
```javascript
// MongoDB collections for high-volume audit data
db.createCollection("auditEvents", {
    validator: {
        $jsonSchema: {
            bsonType: "object",
            required: ["eventId", "timestamp", "eventType", "action", "result"],
            properties: {
                eventId: { bsonType: "string" },
                timestamp: { bsonType: "date" },
                eventType: { bsonType: "string" },
                eventCategory: { bsonType: "string" },
                userId: { bsonType: "string" },
                action: { bsonType: "string" },
                result: { bsonType: "string" },
                details: { bsonType: "object" }
            }
        }
    }
});

// Create indexes for performance
db.auditEvents.createIndex({ "timestamp": -1 });
db.auditEvents.createIndex({ "userId": 1, "timestamp": -1 });
db.auditEvents.createIndex({ "eventType": 1, "timestamp": -1 });
db.auditEvents.createIndex({ "correlationId": 1 });

// High-volume API audit logs
db.createCollection("apiAuditLogs");
db.apiAuditLogs.createIndex({ "timestamp": -1 });
db.apiAuditLogs.createIndex({ "endpoint": 1, "timestamp": -1 });
db.apiAuditLogs.createIndex({ "userId": 1, "timestamp": -1 });
```

### 2. Audit Service Configuration

#### Audit Service Setup
```csharp
// AuditService.cs - Core audit logging service
public class AuditService : IAuditService
{
    private readonly ILogger<AuditService> _logger;
    private readonly IAuditRepository _auditRepository;
    private readonly IHttpContextAccessor _httpContextAccessor;
    
    public AuditService(
        ILogger<AuditService> logger,
        IAuditRepository auditRepository,
        IHttpContextAccessor httpContextAccessor)
    {
        _logger = logger;
        _auditRepository = auditRepository;
        _httpContextAccessor = httpContextAccessor;
    }
    
    public async Task LogEventAsync(AuditEvent auditEvent)
    {
        try
        {
            // Enrich event with context information
            EnrichEventContext(auditEvent);
            
            // Validate event data
            ValidateAuditEvent(auditEvent);
            
            // Store in appropriate storage
            await _auditRepository.SaveEventAsync(auditEvent);
            
            // Check for security alerts
            await CheckSecurityAlerts(auditEvent);
            
            _logger.LogDebug("Audit event logged: {EventType} for user {UserId}", 
                auditEvent.EventType, auditEvent.UserId);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Failed to log audit event: {EventType}", auditEvent.EventType);
            // Don't throw - audit failures shouldn't break business operations
        }
    }
    
    private void EnrichEventContext(AuditEvent auditEvent)
    {
        var httpContext = _httpContextAccessor.HttpContext;
        if (httpContext != null)
        {
            auditEvent.IpAddress ??= GetClientIpAddress(httpContext);
            auditEvent.UserAgent ??= httpContext.Request.Headers["User-Agent"];
            auditEvent.SessionId ??= httpContext.Session.Id;
            
            if (string.IsNullOrEmpty(auditEvent.UserId))
            {
                auditEvent.UserId = httpContext.User?.Identity?.Name;
            }
        }
        
        auditEvent.Timestamp = DateTime.UtcNow;
        auditEvent.CorrelationId ??= System.Diagnostics.Activity.Current?.Id ?? Guid.NewGuid().ToString();
    }
}
```

#### Audit Event Model
```csharp
// AuditEvent.cs
public class AuditEvent
{
    public string EventId { get; set; } = Guid.NewGuid().ToString();
    public DateTime Timestamp { get; set; } = DateTime.UtcNow;
    public string EventType { get; set; }
    public string EventCategory { get; set; }
    public string UserId { get; set; }
    public string UserName { get; set; }
    public string SessionId { get; set; }
    public string IpAddress { get; set; }
    public string UserAgent { get; set; }
    public string ResourceType { get; set; }
    public string ResourceId { get; set; }
    public string Action { get; set; }
    public string Result { get; set; } // Success, Failure, Error
    public object Details { get; set; }
    public object BeforeData { get; set; }
    public object AfterData { get; set; }
    public string CorrelationId { get; set; }
    public string ServiceName { get; set; }
}

// Predefined event types
public static class AuditEventTypes
{
    public const string UserAuthentication = "USER_AUTHENTICATION";
    public const string UserAuthorization = "USER_AUTHORIZATION";
    public const string DataAccess = "DATA_ACCESS";
    public const string DataModification = "DATA_MODIFICATION";
    public const string SystemConfiguration = "SYSTEM_CONFIGURATION";
    public const string ApiRequest = "API_REQUEST";
    public const string WorkflowExecution = "WORKFLOW_EXECUTION";
    public const string SecurityAlert = "SECURITY_ALERT";
}

public static class AuditEventCategories
{
    public const string Security = "SECURITY";
    public const string Data = "DATA";
    public const string System = "SYSTEM";
    public const string Business = "BUSINESS";
    public const string Compliance = "COMPLIANCE";
}
```

### 3. Audit Middleware Implementation

#### ASP.NET Core Middleware
```csharp
// AuditMiddleware.cs
public class AuditMiddleware
{
    private readonly RequestDelegate _next;
    private readonly IAuditService _auditService;
    private readonly ILogger<AuditMiddleware> _logger;
    
    public AuditMiddleware(RequestDelegate next, IAuditService auditService, ILogger<AuditMiddleware> logger)
    {
        _next = next;
        _auditService = auditService;
        _logger = logger;
    }
    
    public async Task InvokeAsync(HttpContext context)
    {
        var stopwatch = Stopwatch.StartNew();
        var correlationId = Guid.NewGuid().ToString();
        
        // Store correlation ID for request tracking
        context.Items["CorrelationId"] = correlationId;
        
        // Capture request details
        var requestBody = await CaptureRequestBody(context.Request);
        
        try
        {
            await _next(context);
            
            // Log successful API request
            await LogApiRequest(context, requestBody, correlationId, stopwatch.ElapsedMilliseconds, "Success");
        }
        catch (Exception ex)
        {
            // Log failed API request
            await LogApiRequest(context, requestBody, correlationId, stopwatch.ElapsedMilliseconds, "Error");
            
            // Log security alert for certain types of errors
            if (IsSecurityRelevantError(ex))
            {
                await LogSecurityAlert(context, ex, correlationId);
            }
            
            throw;
        }
    }
    
    private async Task LogApiRequest(HttpContext context, string requestBody, string correlationId, long duration, string result)
    {
        var auditEvent = new AuditEvent
        {
            EventType = AuditEventTypes.ApiRequest,
            EventCategory = AuditEventCategories.System,
            Action = context.Request.Method,
            Result = result,
            CorrelationId = correlationId,
            Details = new
            {
                Path = context.Request.Path,
                QueryString = context.Request.QueryString.ToString(),
                RequestBody = requestBody,
                ResponseStatus = context.Response.StatusCode,
                Duration = duration,
                ContentType = context.Request.ContentType
            }
        };
        
        await _auditService.LogEventAsync(auditEvent);
    }
}
```

## Event Tracking Implementation

### 1. User Action Tracking

#### Authentication Events
```csharp
// AuthenticationService.cs
public class AuthenticationService
{
    private readonly IAuditService _auditService;
    
    public async Task<AuthResult> LoginAsync(LoginRequest request)
    {
        var auditEvent = new AuditEvent
        {
            EventType = AuditEventTypes.UserAuthentication,
            EventCategory = AuditEventCategories.Security,
            Action = "LOGIN_ATTEMPT",
            UserId = request.Username,
            Details = new { Username = request.Username, RememberMe = request.RememberMe }
        };
        
        try
        {
            var result = await ValidateCredentialsAsync(request);
            
            if (result.Success)
            {
                auditEvent.Result = "Success";
                auditEvent.Details = new { 
                    Username = request.Username, 
                    LoginSuccess = true,
                    SessionDuration = result.SessionDuration
                };
            }
            else
            {
                auditEvent.Result = "Failure";
                auditEvent.Details = new { 
                    Username = request.Username, 
                    FailureReason = result.FailureReason,
                    AttemptsRemaining = result.AttemptsRemaining
                };
                
                // Check for brute force attempts
                if (result.AttemptsRemaining <= 0)
                {
                    await LogSecurityAlert("BRUTE_FORCE_ATTACK", request.Username, auditEvent.CorrelationId);
                }
            }
            
            await _auditService.LogEventAsync(auditEvent);
            return result;
        }
        catch (Exception ex)
        {
            auditEvent.Result = "Error";
            auditEvent.Details = new { Username = request.Username, Error = ex.Message };
            await _auditService.LogEventAsync(auditEvent);
            throw;
        }
    }
}
```

#### Data Modification Tracking
```csharp
// DataAuditAttribute.cs - Attribute for automatic data change tracking
[AttributeUsage(AttributeTargets.Method)]
public class DataAuditAttribute : Attribute
{
    public string ResourceType { get; set; }
    public string Action { get; set; }
    
    public DataAuditAttribute(string resourceType, string action)
    {
        ResourceType = resourceType;
        Action = action;
    }
}

// DataAuditInterceptor.cs - Intercepts data operations
public class DataAuditInterceptor : IInterceptor
{
    private readonly IAuditService _auditService;
    
    public void Intercept(IInvocation invocation)
    {
        var auditAttribute = invocation.Method.GetCustomAttribute<DataAuditAttribute>();
        if (auditAttribute != null)
        {
            var beforeData = CaptureBeforeData(invocation);
            
            try
            {
                invocation.Proceed();
                
                var afterData = CaptureAfterData(invocation);
                
                var auditEvent = new AuditEvent
                {
                    EventType = AuditEventTypes.DataModification,
                    EventCategory = AuditEventCategories.Data,
                    ResourceType = auditAttribute.ResourceType,
                    Action = auditAttribute.Action,
                    Result = "Success",
                    BeforeData = beforeData,
                    AfterData = afterData,
                    Details = new { Method = invocation.Method.Name }
                };
                
                Task.Run(() => _auditService.LogEventAsync(auditEvent));
            }
            catch (Exception ex)
            {
                var auditEvent = new AuditEvent
                {
                    EventType = AuditEventTypes.DataModification,
                    EventCategory = AuditEventCategories.Data,
                    ResourceType = auditAttribute.ResourceType,
                    Action = auditAttribute.Action,
                    Result = "Error",
                    BeforeData = beforeData,
                    Details = new { Method = invocation.Method.Name, Error = ex.Message }
                };
                
                Task.Run(() => _auditService.LogEventAsync(auditEvent));
                throw;
            }
        }
        else
        {
            invocation.Proceed();
        }
    }
}
```

### 2. Business Event Tracking

#### Workflow Execution Tracking
```csharp
// WorkflowAuditService.cs
public class WorkflowAuditService
{
    private readonly IAuditService _auditService;
    
    public async Task LogWorkflowStartAsync(string workflowId, string workflowType, object parameters)
    {
        var auditEvent = new AuditEvent
        {
            EventType = AuditEventTypes.WorkflowExecution,
            EventCategory = AuditEventCategories.Business,
            ResourceType = "Workflow",
            ResourceId = workflowId,
            Action = "START",
            Result = "Success",
            Details = new
            {
                WorkflowType = workflowType,
                Parameters = parameters,
                StartTime = DateTime.UtcNow
            }
        };
        
        await _auditService.LogEventAsync(auditEvent);
    }
    
    public async Task LogWorkflowStepAsync(string workflowId, string stepName, string stepResult, object stepData)
    {
        var auditEvent = new AuditEvent
        {
            EventType = AuditEventTypes.WorkflowExecution,
            EventCategory = AuditEventCategories.Business,
            ResourceType = "WorkflowStep",
            ResourceId = workflowId,
            Action = stepName,
            Result = stepResult,
            Details = new
            {
                StepName = stepName,
                StepData = stepData,
                ExecutionTime = DateTime.UtcNow
            }
        };
        
        await _auditService.LogEventAsync(auditEvent);
    }
}
```

## Audit Query and Reporting

### 1. Audit Query Service

#### Query Implementation
```csharp
// AuditQueryService.cs
public class AuditQueryService : IAuditQueryService
{
    private readonly IAuditRepository _auditRepository;
    
    public async Task<PagedResult<AuditEvent>> SearchEventsAsync(AuditSearchCriteria criteria)
    {
        var query = BuildQuery(criteria);
        return await _auditRepository.SearchAsync(query);
    }
    
    public async Task<AuditTrail> GetUserAuditTrailAsync(string userId, DateTime fromDate, DateTime toDate)
    {
        var events = await _auditRepository.GetUserEventsAsync(userId, fromDate, toDate);
        
        return new AuditTrail
        {
            UserId = userId,
            FromDate = fromDate,
            ToDate = toDate,
            Events = events,
            Summary = new AuditSummary
            {
                TotalEvents = events.Count,
                LoginCount = events.Count(e => e.EventType == AuditEventTypes.UserAuthentication && e.Action == "LOGIN"),
                DataModifications = events.Count(e => e.EventType == AuditEventTypes.DataModification),
                SecurityAlerts = events.Count(e => e.EventCategory == AuditEventCategories.Security && e.Result != "Success")
            }
        };
    }
    
    public async Task<ComplianceReport> GenerateComplianceReportAsync(ComplianceReportRequest request)
    {
        var events = await _auditRepository.GetComplianceEventsAsync(request.FromDate, request.ToDate, request.ComplianceType);
        
        return new ComplianceReport
        {
            ReportType = request.ComplianceType,
            GeneratedDate = DateTime.UtcNow,
            PeriodStart = request.FromDate,
            PeriodEnd = request.ToDate,
            Events = events,
            Statistics = CalculateComplianceStatistics(events)
        };
    }
}
```

#### Search Criteria Model
```csharp
// AuditSearchCriteria.cs
public class AuditSearchCriteria
{
    public DateTime? FromDate { get; set; }
    public DateTime? ToDate { get; set; }
    public string UserId { get; set; }
    public string EventType { get; set; }
    public string EventCategory { get; set; }
    public string Action { get; set; }
    public string Result { get; set; }
    public string ResourceType { get; set; }
    public string ResourceId { get; set; }
    public string IpAddress { get; set; }
    public string SearchText { get; set; }
    public int PageNumber { get; set; } = 1;
    public int PageSize { get; set; } = 50;
    public string SortBy { get; set; } = "Timestamp";
    public string SortDirection { get; set; } = "DESC";
}
```

### 2. Audit Dashboard

#### Dashboard API Controller
```csharp
// AuditDashboardController.cs
[ApiController]
[Route("api/audit/dashboard")]
public class AuditDashboardController : ControllerBase
{
    private readonly IAuditQueryService _auditQueryService;
    
    [HttpGet("summary")]
    public async Task<ActionResult<AuditDashboardSummary>> GetSummary([FromQuery] int days = 30)
    {
        var fromDate = DateTime.UtcNow.AddDays(-days);
        var toDate = DateTime.UtcNow;
        
        var summary = await _auditQueryService.GetDashboardSummaryAsync(fromDate, toDate);
        return Ok(summary);
    }
    
    [HttpGet("events")]
    public async Task<ActionResult<PagedResult<AuditEvent>>> SearchEvents([FromQuery] AuditSearchCriteria criteria)
    {
        var events = await _auditQueryService.SearchEventsAsync(criteria);
        return Ok(events);
    }
    
    [HttpGet("user/{userId}/trail")]
    public async Task<ActionResult<AuditTrail>> GetUserTrail(string userId, [FromQuery] DateTime fromDate, [FromQuery] DateTime toDate)
    {
        var trail = await _auditQueryService.GetUserAuditTrailAsync(userId, fromDate, toDate);
        return Ok(trail);
    }
    
    [HttpGet("security-alerts")]
    public async Task<ActionResult<PagedResult<SecurityAlert>>> GetSecurityAlerts([FromQuery] SecurityAlertSearchCriteria criteria)
    {
        var alerts = await _auditQueryService.GetSecurityAlertsAsync(criteria);
        return Ok(alerts);
    }
}
```

### 3. Frontend Dashboard Component

#### React Audit Dashboard
```javascript
// AuditDashboard.js
import React, { useState, useEffect } from 'react';
import { Card, Table, Select, DatePicker, Input, Button, Alert } from 'antd';

const AuditDashboard = () => {
    const [summary, setSummary] = useState(null);
    const [events, setEvents] = useState([]);
    const [loading, setLoading] = useState(false);
    const [searchCriteria, setSearchCriteria] = useState({
        fromDate: new Date(Date.now() - 30 * 24 * 60 * 60 * 1000),
        toDate: new Date(),
        eventType: '',
        userId: '',
        pageNumber: 1,
        pageSize: 50
    });
    
    useEffect(() => {
        loadDashboardData();
    }, []);
    
    const loadDashboardData = async () => {
        setLoading(true);
        try {
            const [summaryResponse, eventsResponse] = await Promise.all([
                fetch('/api/audit/dashboard/summary'),
                fetch('/api/audit/dashboard/events?' + new URLSearchParams(searchCriteria))
            ]);
            
            const summaryData = await summaryResponse.json();
            const eventsData = await eventsResponse.json();
            
            setSummary(summaryData);
            setEvents(eventsData.items);
        } catch (error) {
            console.error('Failed to load audit data:', error);
        } finally {
            setLoading(false);
        }
    };
    
    const columns = [
        {
            title: 'Timestamp',
            dataIndex: 'timestamp',
            key: 'timestamp',
            render: (timestamp) => new Date(timestamp).toLocaleString()
        },
        {
            title: 'User',
            dataIndex: 'userName',
            key: 'userName'
        },
        {
            title: 'Event Type',
            dataIndex: 'eventType',
            key: 'eventType'
        },
        {
            title: 'Action',
            dataIndex: 'action',
            key: 'action'
        },
        {
            title: 'Resource',
            dataIndex: 'resourceType',
            key: 'resourceType'
        },
        {
            title: 'Result',
            dataIndex: 'result',
            key: 'result',
            render: (result) => (
                <span className={`status ${result.toLowerCase()}`}>
                    {result}
                </span>
            )
        },
        {
            title: 'IP Address',
            dataIndex: 'ipAddress',
            key: 'ipAddress'
        }
    ];
    
    return (
        <div className="audit-dashboard">
            <h1>Audit Trail Dashboard</h1>
            
            {summary && (
                <div className="summary-cards">
                    <Card title="Total Events" className="summary-card">
                        <div className="metric">{summary.totalEvents}</div>
                    </Card>
                    <Card title="Unique Users" className="summary-card">
                        <div className="metric">{summary.uniqueUsers}</div>
                    </Card>
                    <Card title="Security Alerts" className="summary-card">
                        <div className="metric">{summary.securityAlerts}</div>
                    </Card>
                    <Card title="Failed Logins" className="summary-card">
                        <div className="metric">{summary.failedLogins}</div>
                    </Card>
                </div>
            )}
            
            <Card title="Search Audit Events" className="search-card">
                <div className="search-form">
                    <DatePicker.RangePicker
                        value={[searchCriteria.fromDate, searchCriteria.toDate]}
                        onChange={(dates) => setSearchCriteria({
                            ...searchCriteria,
                            fromDate: dates[0],
                            toDate: dates[1]
                        })}
                    />
                    
                    <Select
                        placeholder="Event Type"
                        value={searchCriteria.eventType}
                        onChange={(value) => setSearchCriteria({...searchCriteria, eventType: value})}
                        style={{ width: 200 }}
                    >
                        <Select.Option value="">All Events</Select.Option>
                        <Select.Option value="USER_AUTHENTICATION">Authentication</Select.Option>
                        <Select.Option value="DATA_MODIFICATION">Data Changes</Select.Option>
                        <Select.Option value="API_REQUEST">API Requests</Select.Option>
                    </Select>
                    
                    <Input
                        placeholder="User ID"
                        value={searchCriteria.userId}
                        onChange={(e) => setSearchCriteria({...searchCriteria, userId: e.target.value})}
                        style={{ width: 200 }}
                    />
                    
                    <Button type="primary" onClick={loadDashboardData} loading={loading}>
                        Search
                    </Button>
                </div>
            </Card>
            
            <Card title="Audit Events" className="events-table">
                <Table
                    columns={columns}
                    dataSource={events}
                    loading={loading}
                    rowKey="eventId"
                    pagination={{
                        current: searchCriteria.pageNumber,
                        pageSize: searchCriteria.pageSize,
                        onChange: (page, size) => setSearchCriteria({
                            ...searchCriteria,
                            pageNumber: page,
                            pageSize: size
                        })
                    }}
                />
            </Card>
        </div>
    );
};

export default AuditDashboard;
```

## Security Monitoring and Alerts

### Security Alert Rules Engine
```csharp
// SecurityAlertEngine.cs
public class SecurityAlertEngine
{
    private readonly IAuditService _auditService;
    private readonly INotificationService _notificationService;
    
    public async Task ProcessEventForSecurityAlerts(AuditEvent auditEvent)
    {
        var alerts = new List<SecurityAlert>();
        
        // Check for suspicious login patterns
        if (auditEvent.EventType == AuditEventTypes.UserAuthentication)
        {
            alerts.AddRange(await CheckSuspiciousLoginPatterns(auditEvent));
        }
        
        // Check for data access violations
        if (auditEvent.EventType == AuditEventTypes.DataAccess)
        {
            alerts.AddRange(await CheckDataAccessViolations(auditEvent));
        }
        
        // Check for privilege escalation
        if (auditEvent.EventType == AuditEventTypes.UserAuthorization)
        {
            alerts.AddRange(await CheckPrivilegeEscalation(auditEvent));
        }
        
        // Process any generated alerts
        foreach (var alert in alerts)
        {
            await ProcessSecurityAlert(alert);
        }
    }
    
    private async Task<List<SecurityAlert>> CheckSuspiciousLoginPatterns(AuditEvent auditEvent)
    {
        var alerts = new List<SecurityAlert>();
        
        // Check for multiple failed logins
        var recentFailures = await GetRecentFailedLogins(auditEvent.UserId, TimeSpan.FromMinutes(15));
        if (recentFailures.Count >= 5)
        {
            alerts.Add(new SecurityAlert
            {
                AlertType = "BRUTE_FORCE_ATTACK",
                Severity = "High",
                UserId = auditEvent.UserId,
                Description = $"Multiple failed login attempts detected for user {auditEvent.UserId}",
                RelatedEventIds = recentFailures.Select(e => e.EventId).ToList()
            });
        }
        
        // Check for logins from unusual locations
        var unusualLocation = await CheckUnusualLoginLocation(auditEvent.UserId, auditEvent.IpAddress);
        if (unusualLocation)
        {
            alerts.Add(new SecurityAlert
            {
                AlertType = "UNUSUAL_LOGIN_LOCATION",
                Severity = "Medium",
                UserId = auditEvent.UserId,
                Description = $"Login from unusual location detected: {auditEvent.IpAddress}",
                RelatedEventIds = new List<string> { auditEvent.EventId }
            });
        }
        
        return alerts;
    }
}
```

## Compliance Reporting

### Compliance Report Generator
```csharp
// ComplianceReportService.cs
public class ComplianceReportService
{
    public async Task<ComplianceReport> GenerateSOXReportAsync(DateTime fromDate, DateTime toDate)
    {
        var criteria = new AuditSearchCriteria
        {
            FromDate = fromDate,
            ToDate = toDate,
            EventCategories = new[] { "DATA", "SECURITY" }
        };
        
        var events = await _auditQueryService.GetComplianceEventsAsync(criteria);
        
        return new ComplianceReport
        {
            ReportType = "SOX",
            Title = "Sarbanes-Oxley Compliance Report",
            Events = events.Where(e => IsSOXRelevant(e)).ToList(),
            Sections = new List<ComplianceSection>
            {
                new ComplianceSection
                {
                    Title = "Financial Data Access",
                    Events = events.Where(e => e.ResourceType == "FinancialData").ToList()
                },
                new ComplianceSection
                {
                    Title = "System Administration",
                    Events = events.Where(e => e.EventType == "SYSTEM_CONFIGURATION").ToList()
                }
            }
        };
    }
    
    public async Task<ComplianceReport> GenerateGDPRReportAsync(string userId, DateTime fromDate, DateTime toDate)
    {
        var userEvents = await _auditQueryService.GetUserEventsAsync(userId, fromDate, toDate);
        
        return new ComplianceReport
        {
            ReportType = "GDPR",
            Title = $"GDPR Data Processing Report for User {userId}",
            Events = userEvents,
            Sections = new List<ComplianceSection>
            {
                new ComplianceSection
                {
                    Title = "Personal Data Access",
                    Events = userEvents.Where(e => e.EventType == "DATA_ACCESS").ToList()
                },
                new ComplianceSection
                {
                    Title = "Personal Data Modifications",
                    Events = userEvents.Where(e => e.EventType == "DATA_MODIFICATION").ToList()
                },
                new ComplianceSection
                {
                    Title = "Data Exports",
                    Events = userEvents.Where(e => e.Action == "EXPORT").ToList()
                }
            }
        };
    }
}
```

## Performance Considerations

### Audit Performance Optimization
```csharp
// High-performance audit implementation
public class HighPerformanceAuditService : IAuditService
{
    private readonly IMemoryCache _cache;
    private readonly Channel<AuditEvent> _auditChannel;
    private readonly IServiceProvider _serviceProvider;
    
    public HighPerformanceAuditService(IMemoryCache cache, IServiceProvider serviceProvider)
    {
        _cache = cache;
        _serviceProvider = serviceProvider;
        
        // Create high-throughput channel for audit events
        var options = new BoundedChannelOptions(10000)
        {
            FullMode = BoundedChannelFullMode.Wait,
            SingleReader = false,
            SingleWriter = false
        };
        
        var channel = Channel.CreateBounded<AuditEvent>(options);
        _auditChannel = channel;
        
        // Start background processor
        _ = Task.Run(ProcessAuditEventsAsync);
    }
    
    public async Task LogEventAsync(AuditEvent auditEvent)
    {
        // Non-blocking write to channel
        if (!_auditChannel.Writer.TryWrite(auditEvent))
        {
            // Channel is full - use fallback strategy
            await _auditChannel.Writer.WriteAsync(auditEvent);
        }
    }
    
    private async Task ProcessAuditEventsAsync()
    {
        var events = new List<AuditEvent>();
        
        await foreach (var auditEvent in _auditChannel.Reader.ReadAllAsync())
        {
            events.Add(auditEvent);
            
            // Batch process events for better performance
            if (events.Count >= 100 || events.Any(e => (DateTime.UtcNow - e.Timestamp).TotalSeconds > 5))
            {
                await ProcessEventBatch(events);
                events.Clear();
            }
        }
    }
    
    private async Task ProcessEventBatch(List<AuditEvent> events)
    {
        using var scope = _serviceProvider.CreateScope();
        var repository = scope.ServiceProvider.GetRequiredService<IAuditRepository>();
        
        try
        {
            await repository.SaveEventsBatchAsync(events);
        }
        catch (Exception ex)
        {
            // Log error but don't stop processing
            var logger = scope.ServiceProvider.GetRequiredService<ILogger<HighPerformanceAuditService>>();
            logger.LogError(ex, "Failed to save audit event batch of {Count} events", events.Count);
        }
    }
}
```

**Next**: [Elasticsearch Setup](./elasticsearch.md)
