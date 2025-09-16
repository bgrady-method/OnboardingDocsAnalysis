# Debugging .NET Core Projects

## Visual Studio Debugging Setup

Debugging .NET Core projects in Method's ecosystem requires specific configuration due to the microservices architecture.

## Prerequisites

- Visual Studio 2019/2022 with .NET Core debugging tools
- Method development environment fully configured
- All required services running (SQL Server, Docker containers)

## Debugging Configuration

### Launch Settings (launchSettings.json)
```json
{
  "profiles": {
    "IIS Express": {
      "commandName": "IISExpress",
      "launchBrowser": true,
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development",
        "DOTNET_ENVIRONMENT": "Development"
      }
    },
    "SelfHosted": {
      "commandName": "Project",
      "launchBrowser": true,
      "applicationUrl": "https://localhost:5001;http://localhost:5000",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    }
  }
}
```

### Required Environment Variables
```bash
ASPNETCORE_ENVIRONMENT=Development
DOTNET_ENVIRONMENT=Development
DOTNET_HOST_PATH=C:\Program Files\dotnet\dotnet.exe
```

## Debugging Method Microservices

### RuntimeCore Engine
- **Project**: Method.RuntimeCore
- **Port**: Usually 5001 (HTTPS), 5000 (HTTP)
- **Dependencies**: SQL Server, MongoDB, Redis
- **Configuration**: appsettings.Development.json

### Gateway API
- **Project**: Method.GatewayAPI
- **Port**: Usually 5003 (HTTPS), 5002 (HTTP)
- **Dependencies**: All backend services
- **Configuration**: Routes configured in appsettings

### Authentication Service
- **Project**: Method.AuthenticationService
- **Port**: Usually 5005 (HTTPS), 5004 (HTTP)
- **Dependencies**: SQL Server, Redis
- **Configuration**: OAuth settings in appsettings

## Debugging Techniques

### Setting Breakpoints
1. **Conditional Breakpoints** - Right-click breakpoint → Conditions
2. **Logpoints** - Right-click breakpoint → Actions → Log message
3. **Exception Breakpoints** - Debug → Windows → Exception Settings

### Debugging Async Methods
```csharp
// Use ConfigureAwait(false) for library code
await SomeAsyncMethod().ConfigureAwait(false);

// Enable "Just My Code" debugging
// Tools → Options → Debugging → General → Enable Just My Code
```

### Debugging Dependency Injection
```csharp
// Add logging to constructor to verify DI
public class MyService
{
    private readonly ILogger<MyService> _logger;
    
    public MyService(ILogger<MyService> logger)
    {
        _logger = logger;
        _logger.LogInformation("MyService constructor called");
    }
}
```

## Multi-Service Debugging

### Debugging Multiple Projects Simultaneously
1. Right-click Solution → Properties
2. Select "Multiple startup projects"
3. Set Action to "Start" for required services
4. Configure start order (dependencies first)

### Service Dependencies Order
1. **SQL Server & Docker containers** (Infrastructure)
2. **AuthenticationService** (Core authentication)
3. **RuntimeCore** (Business logic)
4. **GatewayAPI** (API routing)
5. **Method-UI** (Frontend)

## Common Debugging Scenarios

### Database Connection Issues
```csharp
// Test connection in controller
[HttpGet("test-db")]
public async Task<IActionResult> TestDatabase()
{
    try
    {
        using var connection = new SqlConnection(connectionString);
        await connection.OpenAsync();
        return Ok("Database connection successful");
    }
    catch (Exception ex)
    {
        return BadRequest($"Database error: {ex.Message}");
    }
}
```

### Configuration Debugging
```csharp
// Inject IConfiguration to inspect settings
public class MyController : ControllerBase
{
    private readonly IConfiguration _config;
    
    public MyController(IConfiguration config)
    {
        _config = config;
    }
    
    [HttpGet("config")]
    public IActionResult GetConfig()
    {
        return Ok(new {
            Environment = _config["ASPNETCORE_ENVIRONMENT"],
            ConnectionString = _config.GetConnectionString("DefaultConnection"),
            CustomSetting = _config["MyCustomSetting"]
        });
    }
}
```

### Service-to-Service Communication
```csharp
// Debug HTTP client calls
public class MyService
{
    private readonly HttpClient _httpClient;
    private readonly ILogger<MyService> _logger;
    
    public async Task<T> CallOtherService<T>(string endpoint)
    {
        _logger.LogInformation($"Calling service: {endpoint}");
        
        var response = await _httpClient.GetAsync(endpoint);
        _logger.LogInformation($"Response status: {response.StatusCode}");
        
        var content = await response.Content.ReadAsStringAsync();
        _logger.LogInformation($"Response content: {content}");
        
        return JsonSerializer.Deserialize<T>(content);
    }
}
```

## Performance Debugging

### Using dotnet-trace
```bash
# Install global tool
dotnet tool install --global dotnet-trace

# Collect trace
dotnet-trace collect --process-id <PID> --providers Microsoft-Extensions-Logging

# Analyze in Visual Studio or PerfView
```

### Memory Debugging
```bash
# Install global tool
dotnet tool install --global dotnet-dump

# Create memory dump
dotnet-dump collect --process-id <PID>

# Analyze in Visual Studio
```

## Debugging Tools Integration

### Application Insights (if configured)
- Set up local Application Insights for telemetry
- Use Live Metrics Stream for real-time monitoring
- Query logs with KQL

### Health Checks Integration
```csharp
// Add health check endpoints for debugging
public void ConfigureServices(IServiceCollection services)
{
    services.AddHealthChecks()
        .AddSqlServer(connectionString)
        .AddRedis(redisConnectionString);
}

public void Configure(IApplicationBuilder app)
{
    app.UseHealthChecks("/health");
}
```

## Troubleshooting Common Issues

### Startup Failures
- Check launchSettings.json configuration
- Verify environment variables
- Confirm all dependencies are running
- Review startup logs in Output window

### Service Discovery Issues
- Verify service URLs in configuration
- Check network connectivity between services
- Confirm service registration in gateway

### Authentication Issues
- Verify JWT token configuration
- Check OAuth settings
- Confirm user permissions in database

**Next**: [Debugging .NET Framework Projects](./debug-netframework.md)
