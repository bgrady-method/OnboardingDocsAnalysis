# Elasticsearch Setup

## Elasticsearch for Method Development

This guide covers setting up Elasticsearch for log aggregation, search, and analytics in Method's development environment.

## Overview

Elasticsearch provides:

- **Log Aggregation** - Centralized collection of application and system logs
- **Full-text Search** - Advanced search capabilities across log data
- **Real-time Analytics** - Performance metrics and trend analysis
- **Alerting** - Proactive monitoring and notification system
- **Visualization** - Data visualization through integration with Kibana

## Prerequisites

- Docker Desktop installed and running
- At least 4GB available RAM for Elasticsearch
- Java 11+ (if running standalone installation)
- Method development environment configured

## Installation Options

### Option 1: Docker Installation (Recommended)

#### Single Node Setup
```powershell
# Create elasticsearch data directory
New-Item -ItemType Directory -Path "C:\MethodDev\ElasticSearch\data" -Force
New-Item -ItemType Directory -Path "C:\MethodDev\ElasticSearch\logs" -Force

# Set permissions for elasticsearch user (UID 1000)
icacls "C:\MethodDev\ElasticSearch" /grant "Everyone:(OI)(CI)F"

# Run Elasticsearch container
docker run -d `
  --name elasticsearch `
  --restart unless-stopped `
  -p 9200:9200 `
  -p 9300:9300 `
  -e "discovery.type=single-node" `
  -e "ES_JAVA_OPTS=-Xms2g -Xmx2g" `
  -e "xpack.security.enabled=false" `
  -e "xpack.monitoring.collection.enabled=true" `
  -v "C:\MethodDev\ElasticSearch\data:/usr/share/elasticsearch/data" `
  -v "C:\MethodDev\ElasticSearch\logs:/usr/share/elasticsearch/logs" `
  docker.elastic.co/elasticsearch/elasticsearch:8.11.0

# Verify installation
Start-Sleep -Seconds 30
Invoke-RestMethod -Uri "http://localhost:9200" -Method Get
```

#### Docker Compose Setup
```yaml
# docker-compose.elasticsearch.yml
version: '3.8'
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.0
    container_name: elasticsearch
    restart: unless-stopped
    environment:
      - discovery.type=single-node
      - ES_JAVA_OPTS=-Xms2g -Xmx2g
      - xpack.security.enabled=false
      - xpack.monitoring.collection.enabled=true
      - cluster.name=method-cluster
      - node.name=method-node-1
    ports:
      - "9200:9200"
      - "9300:9300"
    volumes:
      - elasticsearch_data:/usr/share/elasticsearch/data
      - elasticsearch_logs:/usr/share/elasticsearch/logs
    networks:
      - method-network

  kibana:
    image: docker.elastic.co/kibana/kibana:8.11.0
    container_name: kibana
    restart: unless-stopped
    ports:
      - "5601:5601"
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
      - SERVER_NAME=kibana.method.local
    depends_on:
      - elasticsearch
    networks:
      - method-network

volumes:
  elasticsearch_data:
  elasticsearch_logs:

networks:
  method-network:
    driver: bridge
```

```powershell
# Start with Docker Compose
docker-compose -f docker-compose.elasticsearch.yml up -d

# Check status
docker-compose -f docker-compose.elasticsearch.yml ps
```

### Option 2: Manual Installation

#### Download and Install
```powershell
# Download Elasticsearch
$elasticVersion = "8.11.0"
$downloadUrl = "https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-$elasticVersion-windows-x86_64.zip"
$downloadPath = "C:\temp\elasticsearch-$elasticVersion.zip"
$extractPath = "C:\MethodDev\"

# Download
Invoke-WebRequest -Uri $downloadUrl -OutFile $downloadPath

# Extract
Expand-Archive -Path $downloadPath -DestinationPath $extractPath -Force

# Rename directory for easier access
Rename-Item -Path "$extractPath\elasticsearch-$elasticVersion" -NewName "ElasticSearch"
```

#### Configuration
```yaml
# C:\MethodDev\ElasticSearch\config\elasticsearch.yml
cluster.name: method-cluster
node.name: method-node-1
path.data: C:\MethodDev\ElasticSearch\data
path.logs: C:\MethodDev\ElasticSearch\logs
network.host: 0.0.0.0
http.port: 9200
discovery.type: single-node

# Security settings (for development)
xpack.security.enabled: false
xpack.monitoring.collection.enabled: true

# JVM heap size (adjust based on available RAM)
# -Xms2g
# -Xmx2g
```

#### JVM Configuration
```ini
# C:\MethodDev\ElasticSearch\config\jvm.options
# Heap size (customize based on system memory)
-Xms2g
-Xmx2g

# GC configuration
-XX:+UseG1GC
-XX:G1HeapRegionSize=4m
-XX:+UseLargePages
-XX:+UnlockExperimentalVMOptions
-XX:+UseTransparentHugePages

# Memory and performance
-Dfile.encoding=UTF-8
-Djava.awt.headless=true
-XX:+ExitOnOutOfMemoryError
```

## Index Configuration

### Index Templates for Method Logs

#### Application Logs Template
```json
{
  "index_patterns": ["method-logs-*"],
  "template": {
    "settings": {
      "number_of_shards": 1,
      "number_of_replicas": 0,
      "refresh_interval": "5s",
      "index.mapping.total_fields.limit": 2000
    },
    "mappings": {
      "properties": {
        "@timestamp": {
          "type": "date"
        },
        "level": {
          "type": "keyword"
        },
        "message": {
          "type": "text",
          "analyzer": "standard"
        },
        "logger": {
          "type": "keyword"
        },
        "thread": {
          "type": "keyword"
        },
        "exception": {
          "type": "text"
        },
        "application": {
          "type": "keyword"
        },
        "environment": {
          "type": "keyword"
        },
        "server": {
          "type": "keyword"
        },
        "user_id": {
          "type": "keyword"
        },
        "session_id": {
          "type": "keyword"
        },
        "correlation_id": {
          "type": "keyword"
        },
        "request_id": {
          "type": "keyword"
        },
        "method": {
          "type": "keyword"
        },
        "url": {
          "type": "keyword"
        },
        "status_code": {
          "type": "integer"
        },
        "duration": {
          "type": "integer"
        }
      }
    }
  }
}
```

#### Performance Metrics Template
```json
{
  "index_patterns": ["method-metrics-*"],
  "template": {
    "settings": {
      "number_of_shards": 1,
      "number_of_replicas": 0,
      "refresh_interval": "10s"
    },
    "mappings": {
      "properties": {
        "@timestamp": {
          "type": "date"
        },
        "metric_name": {
          "type": "keyword"
        },
        "metric_value": {
          "type": "double"
        },
        "metric_unit": {
          "type": "keyword"
        },
        "application": {
          "type": "keyword"
        },
        "server": {
          "type": "keyword"
        },
        "environment": {
          "type": "keyword"
        },
        "tags": {
          "type": "object"
        }
      }
    }
  }
}
```

### PowerShell Setup Script
```powershell
# Setup Elasticsearch indexes and templates
function Initialize-MethodElasticSearch {
    $elasticUrl = "http://localhost:9200"
    
    Write-Host "Setting up Method Elasticsearch configuration..." -ForegroundColor Green
    
    # Wait for Elasticsearch to be ready
    do {
        try {
            $response = Invoke-RestMethod -Uri $elasticUrl -Method Get -ErrorAction Stop
            Write-Host "Elasticsearch is ready: $($response.tagline)" -ForegroundColor Green
            break
        }
        catch {
            Write-Host "Waiting for Elasticsearch to start..." -ForegroundColor Yellow
            Start-Sleep -Seconds 5
        }
    } while ($true)
    
    # Create index templates
    $templates = @{
        "method-logs-template" = @{
            "index_patterns" = @("method-logs-*")
            "template" = @{
                "settings" = @{
                    "number_of_shards" = 1
                    "number_of_replicas" = 0
                    "refresh_interval" = "5s"
                }
                "mappings" = @{
                    "properties" = @{
                        "@timestamp" = @{ "type" = "date" }
                        "level" = @{ "type" = "keyword" }
                        "message" = @{ "type" = "text"; "analyzer" = "standard" }
                        "logger" = @{ "type" = "keyword" }
                        "application" = @{ "type" = "keyword" }
                        "environment" = @{ "type" = "keyword" }
                        "user_id" = @{ "type" = "keyword" }
                        "correlation_id" = @{ "type" = "keyword" }
                    }
                }
            }
        }
        "method-metrics-template" = @{
            "index_patterns" = @("method-metrics-*")
            "template" = @{
                "settings" = @{
                    "number_of_shards" = 1
                    "number_of_replicas" = 0
                    "refresh_interval" = "10s"
                }
                "mappings" = @{
                    "properties" = @{
                        "@timestamp" = @{ "type" = "date" }
                        "metric_name" = @{ "type" = "keyword" }
                        "metric_value" = @{ "type" = "double" }
                        "application" = @{ "type" = "keyword" }
                    }
                }
            }
        }
    }
    
    foreach ($templateName in $templates.Keys) {
        try {
            $templateJson = $templates[$templateName] | ConvertTo-Json -Depth 10
            $response = Invoke-RestMethod -Uri "$elasticUrl/_index_template/$templateName" -Method Put -Body $templateJson -ContentType "application/json"
            Write-Host "✓ Created index template: $templateName" -ForegroundColor Green
        }
        catch {
            Write-Host "✗ Failed to create template $templateName : $($_.Exception.Message)" -ForegroundColor Red
        }
    }
    
    # Create initial indexes
    $indexDate = Get-Date -Format "yyyy.MM.dd"
    $indexes = @(
        "method-logs-$indexDate",
        "method-metrics-$indexDate"
    )
    
    foreach ($index in $indexes) {
        try {
            $response = Invoke-RestMethod -Uri "$elasticUrl/$index" -Method Put
            Write-Host "✓ Created index: $index" -ForegroundColor Green
        }
        catch {
            Write-Host "✗ Failed to create index $index : $($_.Exception.Message)" -ForegroundColor Red
        }
    }
    
    Write-Host "Elasticsearch setup completed!" -ForegroundColor Green
}

# Run setup
Initialize-MethodElasticSearch
```

## Log Shipping Configuration

### Method Application Logging

#### Serilog Configuration
```csharp
// Program.cs or Startup.cs
using Serilog;
using Serilog.Sinks.Elasticsearch;

public static void Main(string[] args)
{
    Log.Logger = new LoggerConfiguration()
        .MinimumLevel.Information()
        .MinimumLevel.Override("Microsoft", LogEventLevel.Warning)
        .MinimumLevel.Override("System", LogEventLevel.Warning)
        .Enrich.FromLogContext()
        .Enrich.WithProperty("Application", "Method-RuntimeCore")
        .Enrich.WithProperty("Environment", Environment.GetEnvironmentVariable("ASPNETCORE_ENVIRONMENT") ?? "Development")
        .WriteTo.Console()
        .WriteTo.File("D:\\logs\\RuntimeCore\\app-.log", 
            rollingInterval: RollingInterval.Day,
            retainedFileCountLimit: 7)
        .WriteTo.Elasticsearch(new ElasticsearchSinkOptions(new Uri("http://localhost:9200"))
        {
            AutoRegisterTemplate = true,
            IndexFormat = "method-logs-{0:yyyy.MM.dd}",
            TypeName = "_doc",
            BatchAction = ElasticOpType.Index,
            Period = TimeSpan.FromSeconds(10),
            InlineFields = true,
            MinimumLogEventLevel = LogEventLevel.Information
        })
        .CreateLogger();

    try
    {
        Log.Information("Method application starting up");
        CreateHostBuilder(args).Build().Run();
    }
    catch (Exception ex)
    {
        Log.Fatal(ex, "Method application start-up failed");
    }
    finally
    {
        Log.CloseAndFlush();
    }
}
```

#### NLog Configuration
```xml
<!-- NLog.config -->
<?xml version="1.0" encoding="utf-8" ?>
<nlog xmlns="http://www.nlog-project.org/schemas/NLog.xsd"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  
  <extensions>
    <add assembly="NLog.Targets.ElasticSearch"/>
  </extensions>
  
  <targets>
    <target xsi:type="File" name="fileTarget"
            fileName="D:\logs\${processname}\app-${shortdate}.log"
            layout="${longdate} ${level:uppercase=true} ${logger} ${message} ${exception:format=tostring}" />
    
    <target xsi:type="ElasticSearch" name="elasticTarget"
            uri="http://localhost:9200"
            index="method-logs-${date:format=yyyy.MM.dd}"
            documentType="_doc"
            includeAllProperties="true">
      <field name="application" layout="Method-${processname}" />
      <field name="environment" layout="${environment:ASPNETCORE_ENVIRONMENT}" />
      <field name="server" layout="${machinename}" />
      <field name="level" layout="${level:uppercase=true}" />
      <field name="logger" layout="${logger}" />
      <field name="message" layout="${message}" />
      <field name="exception" layout="${exception:format=tostring}" />
    </target>
  </targets>
  
  <rules>
    <logger name="*" minlevel="Info" writeTo="fileTarget,elasticTarget" />
  </rules>
</nlog>
```

### Filebeat Configuration

#### Install Filebeat
```powershell
# Download and install Filebeat
$filebeatVersion = "8.11.0"
$downloadUrl = "https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-$filebeatVersion-windows-x86_64.zip"
$downloadPath = "C:\temp\filebeat-$filebeatVersion.zip"
$extractPath = "C:\MethodDev\"

Invoke-WebRequest -Uri $downloadUrl -OutFile $downloadPath
Expand-Archive -Path $downloadPath -DestinationPath $extractPath -Force
Rename-Item -Path "$extractPath\filebeat-$filebeatVersion-windows-x86_64" -NewName "Filebeat"
```

#### Filebeat Configuration
```yaml
# C:\MethodDev\Filebeat\filebeat.yml
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - D:\logs\*\*.log
  fields:
    environment: development
    source: method-logs
  fields_under_root: true
  multiline.pattern: '^\d{4}-\d{2}-\d{2}'
  multiline.negate: true
  multiline.match: after

- type: log
  enabled: true
  paths:
    - C:\inetpub\logs\LogFiles\W3SVC*\*.log
  fields:
    log_type: iis
    environment: development
  fields_under_root: true

# IIS log processing
processors:
- if:
    equals:
      log_type: iis
  then:
  - dissect:
      tokenizer: "%{timestamp} %{server_ip} %{method} %{uri_stem} %{uri_query} %{server_port} %{username} %{client_ip} %{user_agent} %{referer} %{status} %{win32_status} %{time_taken}"
      field: "message"
      target_prefix: "iis"

output.elasticsearch:
  hosts: ["localhost:9200"]
  index: "method-logs-%{+yyyy.MM.dd}"
  template.enabled: true
  template.pattern: "method-logs-*"
  template.settings:
    index.number_of_shards: 1
    index.number_of_replicas: 0

logging.level: info
logging.to_files: true
logging.files:
  path: C:\MethodDev\Filebeat\logs
  name: filebeat
  keepfiles: 7
  permissions: 0644
```

#### Start Filebeat Service
```powershell
# Install as Windows service
cd "C:\MethodDev\Filebeat"
.\install-service-filebeat.ps1

# Start service
Start-Service filebeat

# Check status
Get-Service filebeat
```

## Performance Monitoring

### Custom Metrics Collection

#### Metrics Collection Service
```csharp
// MetricsCollectionService.cs
public class MetricsCollectionService : BackgroundService
{
    private readonly IElasticClient _elasticClient;
    private readonly ILogger<MetricsCollectionService> _logger;
    private readonly PerformanceCounter _cpuCounter;
    private readonly PerformanceCounter _memoryCounter;
    
    public MetricsCollectionService(IElasticClient elasticClient, ILogger<MetricsCollectionService> logger)
    {
        _elasticClient = elasticClient;
        _logger = logger;
        _cpuCounter = new PerformanceCounter("Processor", "% Processor Time", "_Total");
        _memoryCounter = new PerformanceCounter("Memory", "Available MBytes");
    }
    
    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            try
            {
                await CollectAndSendMetrics();
                await Task.Delay(TimeSpan.FromMinutes(1), stoppingToken);
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Error collecting metrics");
                await Task.Delay(TimeSpan.FromSeconds(30), stoppingToken);
            }
        }
    }
    
    private async Task CollectAndSendMetrics()
    {
        var timestamp = DateTime.UtcNow;
        var indexName = $"method-metrics-{timestamp:yyyy.MM.dd}";
        
        var metrics = new List<object>
        {
            new
            {
                timestamp = timestamp,
                metric_name = "cpu_usage_percent",
                metric_value = _cpuCounter.NextValue(),
                metric_unit = "percent",
                application = "Method-System",
                server = Environment.MachineName,
                environment = Environment.GetEnvironmentVariable("ASPNETCORE_ENVIRONMENT") ?? "Development"
            },
            new
            {
                timestamp = timestamp,
                metric_name = "memory_available_mb",
                metric_value = _memoryCounter.NextValue(),
                metric_unit = "megabytes",
                application = "Method-System",
                server = Environment.MachineName,
                environment = Environment.GetEnvironmentVariable("ASPNETCORE_ENVIRONMENT") ?? "Development"
            }
        };
        
        // Add application-specific metrics
        metrics.AddRange(await CollectApplicationMetrics());
        
        // Bulk index to Elasticsearch
        var bulkRequest = new BulkRequest(indexName)
        {
            Operations = metrics.Select(m => new BulkIndexOperation<object>(m)).Cast<IBulkOperation>().ToList()
        };
        
        var response = await _elasticClient.BulkAsync(bulkRequest);
        
        if (!response.IsValid)
        {
            _logger.LogError("Failed to send metrics to Elasticsearch: {Error}", response.OriginalException?.Message);
        }
    }
    
    private async Task<List<object>> CollectApplicationMetrics()
    {
        var metrics = new List<object>();
        
        // Database connection pool metrics
        var dbMetrics = await GetDatabaseMetrics();
        metrics.AddRange(dbMetrics);
        
        // HTTP request metrics
        var httpMetrics = await GetHttpMetrics();
        metrics.AddRange(httpMetrics);
        
        // Custom business metrics
        var businessMetrics = await GetBusinessMetrics();
        metrics.AddRange(businessMetrics);
        
        return metrics;
    }
}
```

### Health Check Integration

#### Elasticsearch Health Check
```csharp
// ElasticsearchHealthCheck.cs
public class ElasticsearchHealthCheck : IHealthCheck
{
    private readonly IElasticClient _elasticClient;
    
    public ElasticsearchHealthCheck(IElasticClient elasticClient)
    {
        _elasticClient = elasticClient;
    }
    
    public async Task<HealthCheckResult> CheckHealthAsync(HealthCheckContext context, CancellationToken cancellationToken = default)
    {
        try
        {
            var response = await _elasticClient.Cluster.HealthAsync(ct: cancellationToken);
            
            if (!response.IsValid)
            {
                return HealthCheckResult.Unhealthy("Elasticsearch cluster is not responding", response.OriginalException);
            }
            
            var data = new Dictionary<string, object>
            {
                ["cluster_name"] = response.ClusterName,
                ["status"] = response.Status.ToString(),
                ["number_of_nodes"] = response.NumberOfNodes,
                ["active_primary_shards"] = response.ActivePrimaryShards,
                ["active_shards"] = response.ActiveShards
            };
            
            return response.Status switch
            {
                ElasticsearchHealthStatus.Green => HealthCheckResult.Healthy("Elasticsearch cluster is healthy", data),
                ElasticsearchHealthStatus.Yellow => HealthCheckResult.Degraded("Elasticsearch cluster is in yellow state", data),
                ElasticsearchHealthStatus.Red => HealthCheckResult.Unhealthy("Elasticsearch cluster is in red state", data),
                _ => HealthCheckResult.Unhealthy("Unknown Elasticsearch cluster state", data)
            };
        }
        catch (Exception ex)
        {
            return HealthCheckResult.Unhealthy("Failed to check Elasticsearch health", ex);
        }
    }
}
```

## Query and Analysis

### Common Elasticsearch Queries

#### PowerShell Query Functions
```powershell
# Elasticsearch query functions
function Search-MethodLogs {
    param(
        [string]$Query = "*",
        [string]$Level = "",
        [string]$Application = "",
        [datetime]$FromDate = (Get-Date).AddDays(-1),
        [datetime]$ToDate = (Get-Date),
        [int]$Size = 100
    )
    
    $elasticUrl = "http://localhost:9200"
    $indexPattern = "method-logs-*"
    
    $searchBody = @{
        size = $Size
        sort = @(
            @{ "@timestamp" = @{ order = "desc" } }
        )
        query = @{
            bool = @{
                must = @(
                    @{
                        range = @{
                            "@timestamp" = @{
                                gte = $FromDate.ToString("yyyy-MM-ddTHH:mm:ss.fffZ")
                                lte = $ToDate.ToString("yyyy-MM-ddTHH:mm:ss.fffZ")
                            }
                        }
                    }
                )
            }
        }
    }
    
    # Add query string if provided
    if ($Query -ne "*") {
        $searchBody.query.bool.must += @{
            query_string = @{
                query = $Query
                default_field = "message"
            }
        }
    }
    
    # Add level filter if provided
    if ($Level) {
        $searchBody.query.bool.must += @{
            term = @{ "level" = $Level.ToUpper() }
        }
    }
    
    # Add application filter if provided
    if ($Application) {
        $searchBody.query.bool.must += @{
            term = @{ "application" = $Application }
        }
    }
    
    $json = $searchBody | ConvertTo-Json -Depth 10
    
    try {
        $response = Invoke-RestMethod -Uri "$elasticUrl/$indexPattern/_search" -Method Post -Body $json -ContentType "application/json"
        
        Write-Host "Found $($response.hits.total.value) matching log entries" -ForegroundColor Green
        
        foreach ($hit in $response.hits.hits) {
            $source = $hit._source
            Write-Host "$($source.'@timestamp') [$($source.level)] $($source.application): $($source.message)" -ForegroundColor Cyan
        }
        
        return $response.hits.hits
    }
    catch {
        Write-Host "Error searching logs: $($_.Exception.Message)" -ForegroundColor Red
        return $null
    }
}

# Get error statistics
function Get-MethodErrorStats {
    param(
        [datetime]$FromDate = (Get-Date).AddDays(-1),
        [datetime]$ToDate = (Get-Date)
    )
    
    $elasticUrl = "http://localhost:9200"
    $indexPattern = "method-logs-*"
    
    $searchBody = @{
        size = 0
        query = @{
            bool = @{
                must = @(
                    @{
                        range = @{
                            "@timestamp" = @{
                                gte = $FromDate.ToString("yyyy-MM-ddTHH:mm:ss.fffZ")
                                lte = $ToDate.ToString("yyyy-MM-ddTHH:mm:ss.fffZ")
                            }
                        }
                    },
                    @{
                        terms = @{
                            level = @("ERROR", "FATAL")
                        }
                    }
                )
            }
        }
        aggs = @{
            applications = @{
                terms = @{
                    field = "application"
                    size = 20
                }
                aggs = @{
                    error_levels = @{
                        terms = @{
                            field = "level"
                        }
                    }
                }
            }
            hourly_distribution = @{
                date_histogram = @{
                    field = "@timestamp"
                    calendar_interval = "1h"
                }
            }
        }
    }
    
    $json = $searchBody | ConvertTo-Json -Depth 10
    
    try {
        $response = Invoke-RestMethod -Uri "$elasticUrl/$indexPattern/_search" -Method Post -Body $json -ContentType "application/json"
        
        Write-Host "Error Statistics for $($FromDate.ToString('yyyy-MM-dd')) to $($ToDate.ToString('yyyy-MM-dd'))" -ForegroundColor Yellow
        Write-Host "Total Errors: $($response.hits.total.value)" -ForegroundColor Red
        
        Write-Host "`nErrors by Application:" -ForegroundColor Yellow
        foreach ($bucket in $response.aggregations.applications.buckets) {
            Write-Host "  $($bucket.key): $($bucket.doc_count)" -ForegroundColor Cyan
            foreach ($errorLevel in $bucket.error_levels.buckets) {
                Write-Host "    $($errorLevel.key): $($errorLevel.doc_count)" -ForegroundColor Gray
            }
        }
        
        return $response
    }
    catch {
        Write-Host "Error getting statistics: $($_.Exception.Message)" -ForegroundColor Red
        return $null
    }
}

# Performance metrics query
function Get-MethodPerformanceMetrics {
    param(
        [string]$MetricName = "",
        [datetime]$FromDate = (Get-Date).AddHours(-1),
        [datetime]$ToDate = (Get-Date)
    )
    
    $elasticUrl = "http://localhost:9200"
    $indexPattern = "method-metrics-*"
    
    $searchBody = @{
        size = 0
        query = @{
            bool = @{
                must = @(
                    @{
                        range = @{
                            "@timestamp" = @{
                                gte = $FromDate.ToString("yyyy-MM-ddTHH:mm:ss.fffZ")
                                lte = $ToDate.ToString("yyyy-MM-ddTHH:mm:ss.fffZ")
                            }
                        }
                    }
                )
            }
        }
        aggs = @{
            metrics = @{
                terms = @{
                    field = "metric_name"
                    size = 50
                }
                aggs = @{
                    avg_value = @{ avg = @{ field = "metric_value" } }
                    max_value = @{ max = @{ field = "metric_value" } }
                    min_value = @{ min = @{ field = "metric_value" } }
                    timeline = @{
                        date_histogram = @{
                            field = "@timestamp"
                            calendar_interval = "5m"
                        }
                        aggs = @{
                            avg_value = @{ avg = @{ field = "metric_value" } }
                        }
                    }
                }
            }
        }
    }
    
    # Add metric name filter if provided
    if ($MetricName) {
        $searchBody.query.bool.must += @{
            term = @{ "metric_name" = $MetricName }
        }
    }
    
    $json = $searchBody | ConvertTo-Json -Depth 10
    
    try {
        $response = Invoke-RestMethod -Uri "$elasticUrl/$indexPattern/_search" -Method Post -Body $json -ContentType "application/json"
        
        Write-Host "Performance Metrics Summary" -ForegroundColor Yellow
        
        foreach ($bucket in $response.aggregations.metrics.buckets) {
            Write-Host "`n$($bucket.key):" -ForegroundColor Cyan
            Write-Host "  Average: $([math]::Round($bucket.avg_value.value, 2))" -ForegroundColor Green
            Write-Host "  Maximum: $([math]::Round($bucket.max_value.value, 2))" -ForegroundColor Red
            Write-Host "  Minimum: $([math]::Round($bucket.min_value.value, 2))" -ForegroundColor Green
        }
        
        return $response
    }
    catch {
        Write-Host "Error getting metrics: $($_.Exception.Message)" -ForegroundColor Red
        return $null
    }
}
```

## Maintenance and Optimization

### Index Lifecycle Management

#### Automatic Index Cleanup
```powershell
# Cleanup old indexes
function Remove-OldElasticsearchIndexes {
    param(
        [int]$RetentionDays = 30,
        [string]$IndexPattern = "method-logs-*"
    )
    
    $elasticUrl = "http://localhost:9200"
    $cutoffDate = (Get-Date).AddDays(-$RetentionDays)
    
    try {
        # Get all indexes matching pattern
        $response = Invoke-RestMethod -Uri "$elasticUrl/_cat/indices/$IndexPattern?format=json" -Method Get
        
        foreach ($index in $response) {
            # Extract date from index name (assumes format like method-logs-2023.12.01)
            if ($index.index -match "(\d{4}\.\d{2}\.\d{2})") {
                $indexDateStr = $matches[1]
                $indexDate = [DateTime]::ParseExact($indexDateStr, "yyyy.MM.dd", $null)
                
                if ($indexDate -lt $cutoffDate) {
                    Write-Host "Deleting old index: $($index.index) (Date: $indexDateStr)" -ForegroundColor Yellow
                    
                    $deleteResponse = Invoke-RestMethod -Uri "$elasticUrl/$($index.index)" -Method Delete
                    
                    if ($deleteResponse.acknowledged) {
                        Write-Host "✓ Successfully deleted $($index.index)" -ForegroundColor Green
                    } else {
                        Write-Host "✗ Failed to delete $($index.index)" -ForegroundColor Red
                    }
                }
            }
        }
    }
    catch {
        Write-Host "Error cleaning up indexes: $($_.Exception.Message)" -ForegroundColor Red
    }
}

# Schedule cleanup task
Register-ScheduledTask -TaskName "ElasticsearchCleanup" -Trigger (New-ScheduledTaskTrigger -Daily -At "2:00AM") -Action (New-ScheduledTaskAction -Execute "PowerShell.exe" -Argument "-File C:\Scripts\ElasticsearchCleanup.ps1")
```

### Performance Optimization

#### Elasticsearch Tuning
```yaml
# elasticsearch.yml optimizations
# Increase bulk queue size
thread_pool.bulk.queue_size: 1000

# Optimize for logging workload
index.refresh_interval: 30s
index.number_of_replicas: 0
index.merge.scheduler.max_thread_count: 1

# Memory optimization
indices.memory.index_buffer_size: 20%
indices.memory.min_index_buffer_size: 96mb

# Fielddata cache optimization
indices.fielddata.cache.size: 20%
indices.queries.cache.size: 10%
```

#### Monitoring Script
```powershell
function Monitor-ElasticsearchHealth {
    $elasticUrl = "http://localhost:9200"
    
    try {
        # Cluster health
        $health = Invoke-RestMethod -Uri "$elasticUrl/_cluster/health" -Method Get
        Write-Host "Cluster Health: $($health.status)" -ForegroundColor $(if($health.status -eq "green") {"Green"} elseif($health.status -eq "yellow") {"Yellow"} else {"Red"})
        
        # Node stats
        $stats = Invoke-RestMethod -Uri "$elasticUrl/_nodes/stats" -Method Get
        $node = $stats.nodes.PSObject.Properties.Value | Select-Object -First 1
        
        $jvmHeapUsed = [math]::Round($node.jvm.mem.heap_used_percent, 2)
        $diskUsed = [math]::Round(($node.fs.total.total_in_bytes - $node.fs.total.available_in_bytes) / $node.fs.total.total_in_bytes * 100, 2)
        
        Write-Host "JVM Heap Used: $jvmHeapUsed%" -ForegroundColor $(if($jvmHeapUsed -lt 75) {"Green"} elseif($jvmHeapUsed -lt 90) {"Yellow"} else {"Red"})
        Write-Host "Disk Used: $diskUsed%" -ForegroundColor $(if($diskUsed -lt 80) {"Green"} elseif($diskUsed -lt 90) {"Yellow"} else {"Red"})
        
        # Index information
        $indices = Invoke-RestMethod -Uri "$elasticUrl/_cat/indices?format=json" -Method Get
        $methodIndices = $indices | Where-Object { $_.index -like "method-*" }
        
        Write-Host "`nMethod Indexes:" -ForegroundColor Yellow
        foreach ($index in $methodIndices) {
            Write-Host "  $($index.index): $($index.'docs.count') docs, $($index.'store.size')" -ForegroundColor Cyan
        }
        
    }
    catch {
        Write-Host "Error monitoring Elasticsearch: $($_.Exception.Message)" -ForegroundColor Red
    }
}

# Run monitoring
Monitor-ElasticsearchHealth
```

**Next**: [Health Check Application](./health-check.md)
