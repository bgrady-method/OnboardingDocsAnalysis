# Elasticsearch & Kibana Integration

## Kibana Dashboard Setup for Method Development

This guide covers setting up Kibana to visualize Method's log data and create monitoring dashboards.

## Overview

Kibana provides powerful visualization capabilities for Method's Elasticsearch data:

- **Log Analysis** - Search and analyze application logs
- **Real-time Monitoring** - Live dashboards for system metrics
- **Alerting** - Proactive monitoring with Watcher
- **Data Discovery** - Explore patterns in Method's data
- **Custom Dashboards** - Tailored views for different teams

## Prerequisites

- Elasticsearch instance running (see [Elasticsearch Setup](./elasticsearch.md))
- Method logs being indexed to Elasticsearch
- Basic understanding of Elasticsearch queries
- Web browser with network access to Kibana

## Kibana Installation

### Docker Installation (Recommended)

#### Single Container Setup
```powershell
# Run Kibana container connected to Elasticsearch
docker run -d `
  --name kibana `
  --restart unless-stopped `
  -p 5601:5601 `
  -e "ELASTICSEARCH_HOSTS=http://host.docker.internal:9200" `
  -e "SERVER_NAME=kibana.method.local" `
  -e "SERVER_HOST=0.0.0.0" `
  docker.elastic.co/kibana/kibana:8.11.0

# Wait for startup
Start-Sleep -Seconds 30

# Verify Kibana is running
Invoke-WebRequest -Uri "http://localhost:5601" -UseBasicParsing
```

#### With Docker Compose
```yaml
# docker-compose.kibana.yml (extends elasticsearch compose)
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
    ports:
      - "9200:9200"
      - "9300:9300"
    volumes:
      - elasticsearch_data:/usr/share/elasticsearch/data
    networks:
      - elk-network

  kibana:
    image: docker.elastic.co/kibana/kibana:8.11.0
    container_name: kibana
    restart: unless-stopped
    ports:
      - "5601:5601"
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
      - SERVER_NAME=kibana.method.local
      - SERVER_HOST=0.0.0.0
      - LOGGING_LEVEL=warn
    depends_on:
      - elasticsearch
    networks:
      - elk-network

volumes:
  elasticsearch_data:

networks:
  elk-network:
    driver: bridge
```

### Manual Installation

#### Download and Setup
```powershell
# Download Kibana
$kibanaVersion = "8.11.0"
$downloadUrl = "https://artifacts.elastic.co/downloads/kibana/kibana-$kibanaVersion-windows-x86_64.zip"
$downloadPath = "C:\temp\kibana-$kibanaVersion.zip"
$extractPath = "C:\MethodDev\"

Invoke-WebRequest -Uri $downloadUrl -OutFile $downloadPath
Expand-Archive -Path $downloadPath -DestinationPath $extractPath -Force
Rename-Item -Path "$extractPath\kibana-$kibanaVersion-windows-x86_64" -NewName "Kibana"
```

#### Kibana Configuration
```yaml
# C:\MethodDev\Kibana\config\kibana.yml
server.port: 5601
server.host: "0.0.0.0"
server.name: "kibana.method.local"

# Elasticsearch configuration
elasticsearch.hosts: ["http://localhost:9200"]
elasticsearch.username: "kibana_system"
elasticsearch.password: "password"  # Only if security is enabled

# UI Configuration
server.publicBaseUrl: "http://localhost:5601"

# Logging
logging.appenders.default:
  type: file
  fileName: C:\MethodDev\Kibana\logs\kibana.log
  layout:
    type: json

logging.root.level: warn
logging.loggers:
  - name: plugins.security
    level: info

# Method-specific settings
newsfeed.enabled: false
telemetry.enabled: false
telemetry.optIn: false

# Dashboard settings
dashboard.defaultDarkMode: false
```

## Initial Kibana Setup

### PowerShell Setup Script
```powershell
function Initialize-MethodKibana {
    $kibanaUrl = "http://localhost:5601"
    $elasticsearchUrl = "http://localhost:9200"
    
    Write-Host "Setting up Method Kibana configuration..." -ForegroundColor Green
    
    # Wait for Kibana to be ready
    do {
        try {
            $response = Invoke-WebRequest -Uri "$kibanaUrl/api/status" -UseBasicParsing -TimeoutSec 10
            $status = $response.Content | ConvertFrom-Json
            if ($status.status.overall.state -eq "green") {
                Write-Host "Kibana is ready!" -ForegroundColor Green
                break
            }
        }
        catch {
            Write-Host "Waiting for Kibana to start..." -ForegroundColor Yellow
            Start-Sleep -Seconds 10
        }
    } while ($true)
    
    # Create Method index patterns
    $indexPatterns = @(
        @{
            "id" = "method-logs-*"
            "attributes" = @{
                "title" = "method-logs-*"
                "timeFieldName" = "@timestamp"
            }
        },
        @{
            "id" = "method-metrics-*"
            "attributes" = @{
                "title" = "method-metrics-*"
                "timeFieldName" = "@timestamp"
            }
        }
    )
    
    foreach ($pattern in $indexPatterns) {
        try {
            $body = @{
                "attributes" = $pattern.attributes
            } | ConvertTo-Json -Depth 3
            
            $response = Invoke-RestMethod -Uri "$kibanaUrl/api/saved_objects/index-pattern/$($pattern.id)" -Method Post -Body $body -ContentType "application/json" -Headers @{"kbn-xsrf" = "true"}
            Write-Host "✓ Created index pattern: $($pattern.id)" -ForegroundColor Green
        }
        catch {
            if ($_.Exception.Response.StatusCode -eq 409) {
                Write-Host "⚠ Index pattern already exists: $($pattern.id)" -ForegroundColor Yellow
            } else {
                Write-Host "✗ Failed to create index pattern $($pattern.id): $($_.Exception.Message)" -ForegroundColor Red
            }
        }
    }
    
    Write-Host "Kibana setup completed!" -ForegroundColor Green
    Write-Host "Access Kibana at: $kibanaUrl" -ForegroundColor Cyan
}

# Run setup
Initialize-MethodKibana
```

## Method Dashboard Creation

### Application Logs Dashboard

#### Log Analysis Visualizations
```json
{
  "version": "8.11.0",
  "objects": [
    {
      "id": "method-logs-timeline",
      "type": "visualization",
      "attributes": {
        "title": "Method Logs Timeline",
        "visState": {
          "title": "Method Logs Timeline",
          "type": "histogram",
          "params": {
            "grid": {"categoryLines": false, "style": {"color": "#eee"}},
            "categoryAxes": [{"id": "CategoryAxis-1", "type": "category", "position": "bottom", "show": true, "style": {}, "scale": {"type": "linear"}, "labels": {"show": true, "truncate": 100}, "title": {}}],
            "valueAxes": [{"id": "ValueAxis-1", "name": "LeftAxis-1", "type": "value", "position": "left", "show": true, "style": {}, "scale": {"type": "linear", "mode": "normal"}, "labels": {"show": true, "rotate": 0, "filter": false, "truncate": 100}, "title": {"text": "Count"}}],
            "seriesParams": [{"show": "true", "type": "histogram", "mode": "stacked", "data": {"label": "Count", "id": "1"}, "valueAxis": "ValueAxis-1", "drawLinesBetweenPoints": true, "showCircles": true}],
            "addTooltip": true,
            "addLegend": true,
            "legendPosition": "right",
            "times": [],
            "addTimeMarker": false
          },
          "aggs": [
            {"id": "1", "enabled": true, "type": "count", "schema": "metric", "params": {}},
            {"id": "2", "enabled": true, "type": "date_histogram", "schema": "segment", "params": {"field": "@timestamp", "interval": "auto", "customInterval": "2h", "min_doc_count": 1, "extended_bounds": {}}}
          ]
        },
        "uiStateJSON": "{}",
        "description": "Timeline of Method application logs",
        "kibanaSavedObjectMeta": {
          "searchSourceJSON": {
            "index": "method-logs-*",
            "query": {"match_all": {}},
            "filter": []
          }
        }
      }
    },
    {
      "id": "method-log-levels",
      "type": "visualization",
      "attributes": {
        "title": "Method Log Levels",
        "visState": {
          "title": "Method Log Levels",
          "type": "pie",
          "params": {
            "addTooltip": true,
            "addLegend": true,
            "legendPosition": "right",
            "isDonut": true
          },
          "aggs": [
            {"id": "1", "enabled": true, "type": "count", "schema": "metric", "params": {}},
            {"id": "2", "enabled": true, "type": "terms", "schema": "segment", "params": {"field": "level", "size": 5, "order": "desc", "orderBy": "1"}}
          ]
        },
        "uiStateJSON": "{}",
        "description": "Distribution of log levels",
        "kibanaSavedObjectMeta": {
          "searchSourceJSON": {
            "index": "method-logs-*",
            "query": {"match_all": {}},
            "filter": []
          }
        }
      }
    },
    {
      "id": "method-applications",
      "type": "visualization",
      "attributes": {
        "title": "Method Applications",
        "visState": {
          "title": "Method Applications",
          "type": "histogram",
          "params": {
            "grid": {"categoryLines": false, "style": {"color": "#eee"}},
            "categoryAxes": [{"id": "CategoryAxis-1", "type": "category", "position": "bottom", "show": true, "style": {}, "scale": {"type": "linear"}, "labels": {"show": true, "truncate": 100}, "title": {}}],
            "valueAxes": [{"id": "ValueAxis-1", "name": "LeftAxis-1", "type": "value", "position": "left", "show": true, "style": {}, "scale": {"type": "linear", "mode": "normal"}, "labels": {"show": true, "rotate": 0, "filter": false, "truncate": 100}, "title": {"text": "Count"}}],
            "seriesParams": [{"show": "true", "type": "histogram", "mode": "stacked", "data": {"label": "Count", "id": "1"}, "valueAxis": "ValueAxis-1", "drawLinesBetweenPoints": true, "showCircles": true}],
            "addTooltip": true,
            "addLegend": true,
            "legendPosition": "right",
            "times": [],
            "addTimeMarker": false
          },
          "aggs": [
            {"id": "1", "enabled": true, "type": "count", "schema": "metric", "params": {}},
            {"id": "2", "enabled": true, "type": "terms", "schema": "segment", "params": {"field": "application", "size": 10, "order": "desc", "orderBy": "1"}}
          ]
        },
        "uiStateJSON": "{}",
        "description": "Log volume by Method application",
        "kibanaSavedObjectMeta": {
          "searchSourceJSON": {
            "index": "method-logs-*",
            "query": {"match_all": {}},
            "filter": []
          }
        }
      }
    }
  ]
}
```

### Performance Metrics Dashboard

#### System Metrics Visualizations
```json
{
  "version": "8.11.0",
  "objects": [
    {
      "id": "method-cpu-usage",
      "type": "visualization",
      "attributes": {
        "title": "Method CPU Usage",
        "visState": {
          "title": "Method CPU Usage",
          "type": "line",
          "params": {
            "grid": {"categoryLines": false, "style": {"color": "#eee"}},
            "categoryAxes": [{"id": "CategoryAxis-1", "type": "category", "position": "bottom", "show": true, "style": {}, "scale": {"type": "linear"}, "labels": {"show": true, "truncate": 100}, "title": {}}],
            "valueAxes": [{"id": "ValueAxis-1", "name": "LeftAxis-1", "type": "value", "position": "left", "show": true, "style": {}, "scale": {"type": "linear", "mode": "normal"}, "labels": {"show": true, "rotate": 0, "filter": false, "truncate": 100}, "title": {"text": "CPU %"}}],
            "seriesParams": [{"show": "true", "type": "line", "mode": "normal", "data": {"label": "Average metric_value", "id": "1"}, "valueAxis": "ValueAxis-1", "drawLinesBetweenPoints": true, "showCircles": true}],
            "addTooltip": true,
            "addLegend": true,
            "legendPosition": "right",
            "times": [],
            "addTimeMarker": false
          },
          "aggs": [
            {"id": "1", "enabled": true, "type": "avg", "schema": "metric", "params": {"field": "metric_value"}},
            {"id": "2", "enabled": true, "type": "date_histogram", "schema": "segment", "params": {"field": "@timestamp", "interval": "auto", "customInterval": "2h", "min_doc_count": 1, "extended_bounds": {}}}
          ]
        },
        "uiStateJSON": "{}",
        "description": "CPU usage over time",
        "kibanaSavedObjectMeta": {
          "searchSourceJSON": {
            "index": "method-metrics-*",
            "query": {"bool": {"must": [{"term": {"metric_name": "cpu_usage_percent"}}]}},
            "filter": []
          }
        }
      }
    },
    {
      "id": "method-memory-usage",
      "type": "visualization",
      "attributes": {
        "title": "Method Memory Usage",
        "visState": {
          "title": "Method Memory Usage",
          "type": "line",
          "params": {
            "grid": {"categoryLines": false, "style": {"color": "#eee"}},
            "categoryAxes": [{"id": "CategoryAxis-1", "type": "category", "position": "bottom", "show": true, "style": {}, "scale": {"type": "linear"}, "labels": {"show": true, "truncate": 100}, "title": {}}],
            "valueAxes": [{"id": "ValueAxis-1", "name": "LeftAxis-1", "type": "value", "position": "left", "show": true, "style": {}, "scale": {"type": "linear", "mode": "normal"}, "labels": {"show": true, "rotate": 0, "filter": false, "truncate": 100}, "title": {"text": "Memory MB"}}],
            "seriesParams": [{"show": "true", "type": "line", "mode": "normal", "data": {"label": "Average metric_value", "id": "1"}, "valueAxis": "ValueAxis-1", "drawLinesBetweenPoints": true, "showCircles": true}],
            "addTooltip": true,
            "addLegend": true,
            "legendPosition": "right",
            "times": [],
            "addTimeMarker": false
          },
          "aggs": [
            {"id": "1", "enabled": true, "type": "avg", "schema": "metric", "params": {"field": "metric_value"}},
            {"id": "2", "enabled": true, "type": "date_histogram", "schema": "segment", "params": {"field": "@timestamp", "interval": "auto", "customInterval": "2h", "min_doc_count": 1, "extended_bounds": {}}}
          ]
        },
        "uiStateJSON": "{}",
        "description": "Memory usage over time",
        "kibanaSavedObjectMeta": {
          "searchSourceJSON": {
            "index": "method-metrics-*",
            "query": {"bool": {"must": [{"term": {"metric_name": "memory_available_mb"}}]}},
            "filter": []
          }
        }
      }
    }
  ]
}
```

### PowerShell Dashboard Creation
```powershell
function New-MethodKibanaDashboards {
    param(
        [string]$KibanaUrl = "http://localhost:5601"
    )
    
    Write-Host "Creating Method Kibana dashboards..." -ForegroundColor Green
    
    # Dashboard definitions
    $dashboards = @{
        "method-logs-dashboard" = @{
            "title" = "Method Application Logs"
            "description" = "Overview of Method application log data"
            "panelsJSON" = @(
                @{
                    "id" = "method-logs-timeline"
                    "type" = "visualization"
                    "gridData" = @{
                        "x" = 0; "y" = 0; "w" = 48; "h" = 15
                    }
                },
                @{
                    "id" = "method-log-levels"
                    "type" = "visualization"
                    "gridData" = @{
                        "x" = 0; "y" = 15; "w" = 24; "h" = 15
                    }
                },
                @{
                    "id" = "method-applications"
                    "type" = "visualization"
                    "gridData" = @{
                        "x" = 24; "y" = 15; "w" = 24; "h" = 15
                    }
                }
            )
        }
        "method-performance-dashboard" = @{
            "title" = "Method System Performance"
            "description" = "System performance metrics for Method applications"
            "panelsJSON" = @(
                @{
                    "id" = "method-cpu-usage"
                    "type" = "visualization"
                    "gridData" = @{
                        "x" = 0; "y" = 0; "w" = 24; "h" = 15
                    }
                },
                @{
                    "id" = "method-memory-usage"
                    "type" = "visualization"
                    "gridData" = @{
                        "x" = 24; "y" = 0; "w" = 24; "h" = 15
                    }
                }
            )
        }
    }
    
    foreach ($dashboardId in $dashboards.Keys) {
        $dashboard = $dashboards[$dashboardId]
        
        try {
            $body = @{
                "attributes" = @{
                    "title" = $dashboard.title
                    "description" = $dashboard.description
                    "panelsJSON" = ($dashboard.panelsJSON | ConvertTo-Json -Depth 5)
                    "timeRestore" = $false
                    "version" = 1
                }
            } | ConvertTo-Json -Depth 10
            
            $response = Invoke-RestMethod -Uri "$KibanaUrl/api/saved_objects/dashboard/$dashboardId" -Method Post -Body $body -ContentType "application/json" -Headers @{"kbn-xsrf" = "true"}
            Write-Host "✓ Created dashboard: $($dashboard.title)" -ForegroundColor Green
        }
        catch {
            if ($_.Exception.Response.StatusCode -eq 409) {
                Write-Host "⚠ Dashboard already exists: $($dashboard.title)" -ForegroundColor Yellow
            } else {
                Write-Host "✗ Failed to create dashboard $($dashboard.title): $($_.Exception.Message)" -ForegroundColor Red
            }
        }
    }
    
    Write-Host "Dashboard creation completed!" -ForegroundColor Green
}

# Create dashboards
New-MethodKibanaDashboards
```

## Search and Discovery

### Common Kibana Queries

#### Discover Queries for Method Logs
```lucene
# Search for errors in the last hour
level:ERROR AND @timestamp:[now-1h TO now]

# Find logs from specific application
application:"Method-RuntimeCore"

# Search for specific user activity
user_id:"john.doe@method.com"

# Find logs with exceptions
exception:*

# Search for specific correlation ID
correlation_id:"abc123-def456-ghi789"

# Performance issues (slow requests)
duration:>5000

# Authentication events
message:"login" OR message:"authentication"

# Database-related logs
message:"database" OR message:"sql" OR message:"query"

# API endpoint errors
status_code:>=400 AND url:"/api/*"

# Application startup/shutdown events
message:"starting" OR message:"stopped" OR message:"shutdown"
```

#### Advanced KQL Queries
```kql
// Method application errors in the last 24 hours
level : "ERROR" and @timestamp >= "now-24h"

// Specific application with high response times
application : "Method-Gateway" and duration > 1000

// Failed authentication attempts
event_type : "USER_AUTHENTICATION" and result : "Failure"

// Database connection issues
message : "database" and (message : "timeout" or message : "connection")

// High memory usage alerts
metric_name : "memory_usage_percent" and metric_value > 80

// Workflow execution failures
resource_type : "Workflow" and result : "Error"

// User session analysis
session_id : "session123" and @timestamp >= "now-1d"

// API rate limiting
status_code : 429 or message : "rate limit"

// Cross-application correlation tracking
correlation_id : "corr-abc123" and @timestamp >= "now-1h"
```

### Saved Searches

#### PowerShell Saved Search Creation
```powershell
function New-MethodKibanaSavedSearches {
    param(
        [string]$KibanaUrl = "http://localhost:5601"
    )
    
    $savedSearches = @{
        "method-error-logs" = @{
            "title" = "Method Error Logs"
            "description" = "All error and fatal level logs from Method applications"
            "kibanaSavedObjectMeta" = @{
                "searchSourceJSON" = @{
                    "index" = "method-logs-*"
                    "query" = @{
                        "bool" = @{
                            "should" = @(
                                @{ "term" = @{ "level" = "ERROR" } },
                                @{ "term" = @{ "level" = "FATAL" } }
                            )
                        }
                    }
                    "sort" = @(
                        @{ "@timestamp" = @{ "order" = "desc" } }
                    )
                    "fields" = @("@timestamp", "level", "application", "message", "exception")
                } | ConvertTo-Json -Depth 10
            }
        }
        "method-performance-slow" = @{
            "title" = "Method Slow Requests"
            "description" = "Requests with response time > 5 seconds"
            "kibanaSavedObjectMeta" = @{
                "searchSourceJSON" = @{
                    "index" = "method-logs-*"
                    "query" = @{
                        "range" = @{
                            "duration" = @{
                                "gte" = 5000
                            }
                        }
                    }
                    "sort" = @(
                        @{ "duration" = @{ "order" = "desc" } }
                    )
                    "fields" = @("@timestamp", "application", "url", "method", "duration", "status_code")
                } | ConvertTo-Json -Depth 10
            }
        }
        "method-user-activity" = @{
            "title" = "Method User Activity"
            "description" = "User authentication and action logs"
            "kibanaSavedObjectMeta" = @{
                "searchSourceJSON" = @{
                    "index" = "method-logs-*"
                    "query" = @{
                        "bool" = @{
                            "should" = @(
                                @{ "match" = @{ "event_type" = "USER_AUTHENTICATION" } },
                                @{ "match" = @{ "event_type" = "USER_AUTHORIZATION" } },
                                @{ "match" = @{ "event_type" = "DATA_ACCESS" } }
                            )
                        }
                    }
                    "sort" = @(
                        @{ "@timestamp" = @{ "order" = "desc" } }
                    )
                    "fields" = @("@timestamp", "user_id", "event_type", "action", "result", "ip_address")
                } | ConvertTo-Json -Depth 10
            }
        }
    }
    
    foreach ($searchId in $savedSearches.Keys) {
        $search = $savedSearches[$searchId]
        
        try {
            $body = @{
                "attributes" = $search
            } | ConvertTo-Json -Depth 15
            
            $response = Invoke-RestMethod -Uri "$KibanaUrl/api/saved_objects/search/$searchId" -Method Post -Body $body -ContentType "application/json" -Headers @{"kbn-xsrf" = "true"}
            Write-Host "✓ Created saved search: $($search.title)" -ForegroundColor Green
        }
        catch {
            Write-Host "✗ Failed to create saved search $($search.title): $($_.Exception.Message)" -ForegroundColor Red
        }
    }
}

# Create saved searches
New-MethodKibanaSavedSearches
```

## Alerting and Monitoring

### Watcher Alerts

#### High Error Rate Alert
```json
{
  "trigger": {
    "schedule": {
      "interval": "5m"
    }
  },
  "input": {
    "search": {
      "request": {
        "search_type": "query_then_fetch",
        "indices": ["method-logs-*"],
        "body": {
          "query": {
            "bool": {
              "must": [
                {
                  "range": {
                    "@timestamp": {
                      "gte": "now-5m"
                    }
                  }
                },
                {
                  "terms": {
                    "level": ["ERROR", "FATAL"]
                  }
                }
              ]
            }
          },
          "aggs": {
            "error_count": {
              "cardinality": {
                "field": "correlation_id"
              }
            }
          }
        }
      }
    }
  },
  "condition": {
    "compare": {
      "ctx.payload.aggregations.error_count.value": {
        "gt": 10
      }
    }
  },
  "actions": {
    "send_email": {
      "email": {
        "profile": "standard",
        "to": ["devops@method.com"],
        "subject": "Method High Error Rate Alert",
        "body": "High error rate detected in Method applications. {{ctx.payload.aggregations.error_count.value}} errors in the last 5 minutes."
      }
    },
    "log_alert": {
      "logging": {
        "level": "warn",
        "text": "High error rate alert triggered: {{ctx.payload.aggregations.error_count.value}} errors"
      }
    }
  }
}
```

#### Performance Degradation Alert
```json
{
  "trigger": {
    "schedule": {
      "interval": "2m"
    }
  },
  "input": {
    "search": {
      "request": {
        "search_type": "query_then_fetch",
        "indices": ["method-metrics-*"],
        "body": {
          "query": {
            "bool": {
              "must": [
                {
                  "range": {
                    "@timestamp": {
                      "gte": "now-10m"
                    }
                  }
                },
                {
                  "term": {
                    "metric_name": "cpu_usage_percent"
                  }
                }
              ]
            }
          },
          "aggs": {
            "avg_cpu": {
              "avg": {
                "field": "metric_value"
              }
            }
          }
        }
      }
    }
  },
  "condition": {
    "compare": {
      "ctx.payload.aggregations.avg_cpu.value": {
        "gt": 80
      }
    }
  },
  "actions": {
    "send_slack": {
      "slack": {
        "message": {
          "to": ["#method-alerts"],
          "text": "🚨 Method CPU usage is high: {{ctx.payload.aggregations.avg_cpu.value}}%"
        }
      }
    }
  }
}
```

### Custom Alerting with PowerShell

#### Kibana Alert Monitor
```powershell
function Start-MethodKibanaAlertMonitor {
    param(
        [string]$KibanaUrl = "http://localhost:5601",
        [string]$ElasticsearchUrl = "http://localhost:9200",
        [int]$IntervalSeconds = 300
    )
    
    Write-Host "Starting Method Kibana alert monitoring..." -ForegroundColor Green
    
    while ($true) {
        try {
            # Check for high error rates
            $errorQuery = @{
                "query" = @{
                    "bool" = @{
                        "must" = @(
                            @{
                                "range" = @{
                                    "@timestamp" = @{
                                        "gte" = "now-5m"
                                    }
                                }
                            },
                            @{
                                "terms" = @{
                                    "level" = @("ERROR", "FATAL")
                                }
                            }
                        )
                    }
                }
            }
            
            $errorResponse = Invoke-RestMethod -Uri "$ElasticsearchUrl/method-logs-*/_count" -Method Post -Body ($errorQuery | ConvertTo-Json -Depth 10) -ContentType "application/json"
            
            if ($errorResponse.count -gt 10) {
                Send-MethodAlert -Type "HighErrorRate" -Count $errorResponse.count
            }
            
            # Check for performance issues
            $perfQuery = @{
                "query" = @{
                    "bool" = @{
                        "must" = @(
                            @{
                                "range" = @{
                                    "@timestamp" = @{
                                        "gte" = "now-10m"
                                    }
                                }
                            },
                            @{
                                "term" = @{
                                    "metric_name" = "cpu_usage_percent"
                                }
                            }
                        )
                    }
                }
                "aggs" = @{
                    "avg_cpu" = @{
                        "avg" = @{
                            "field" = "metric_value"
                        }
                    }
                }
            }
            
            $perfResponse = Invoke-RestMethod -Uri "$ElasticsearchUrl/method-metrics-*/_search" -Method Post -Body ($perfQuery | ConvertTo-Json -Depth 10) -ContentType "application/json"
            
            $avgCpu = $perfResponse.aggregations.avg_cpu.value
            if ($avgCpu -gt 80) {
                Send-MethodAlert -Type "HighCpuUsage" -Value $avgCpu
            }
            
            Write-Host "$(Get-Date): Alert check completed - Errors: $($errorResponse.count), CPU: $([math]::Round($avgCpu, 2))%" -ForegroundColor Gray
            
        }
        catch {
            Write-Host "Alert monitoring error: $($_.Exception.Message)" -ForegroundColor Red
        }
        
        Start-Sleep -Seconds $IntervalSeconds
    }
}

function Send-MethodAlert {
    param(
        [string]$Type,
        [int]$Count = 0,
        [double]$Value = 0
    )
    
    $message = switch ($Type) {
        "HighErrorRate" { "High error rate detected: $Count errors in the last 5 minutes" }
        "HighCpuUsage" { "High CPU usage detected: $([math]::Round($Value, 2))%" }
        default { "Method alert: $Type" }
    }
    
    Write-Host "🚨 ALERT: $message" -ForegroundColor Red
    
    # Send to external systems (Slack, email, etc.)
    # Implementation depends on your notification preferences
}

# Start monitoring
Start-MethodKibanaAlertMonitor
```

## Data Management

### Index Lifecycle Management

#### PowerShell ILM Setup
```powershell
function Set-MethodIndexLifecycle {
    param(
        [string]$ElasticsearchUrl = "http://localhost:9200"
    )
    
    # Define lifecycle policy for Method logs
    $logPolicy = @{
        "policy" = @{
            "phases" = @{
                "hot" = @{
                    "actions" = @{
                        "rollover" = @{
                            "max_size" = "1GB"
                            "max_age" = "1d"
                        }
                    }
                }
                "warm" = @{
                    "min_age" = "7d"
                    "actions" = @{
                        "allocate" = @{
                            "number_of_replicas" = 0
                        }
                    }
                }
                "delete" = @{
                    "min_age" = "30d"
                }
            }
        }
    }
    
    # Create lifecycle policy
    try {
        $response = Invoke-RestMethod -Uri "$ElasticsearchUrl/_ilm/policy/method-logs-policy" -Method Put -Body ($logPolicy | ConvertTo-Json -Depth 10) -ContentType "application/json"
        Write-Host "✓ Created ILM policy for Method logs" -ForegroundColor Green
    }
    catch {
        Write-Host "✗ Failed to create ILM policy: $($_.Exception.Message)" -ForegroundColor Red
    }
    
    # Apply policy to index template
    $indexTemplate = @{
        "index_patterns" = @("method-logs-*")
        "template" = @{
            "settings" = @{
                "index.lifecycle.name" = "method-logs-policy"
                "index.lifecycle.rollover_alias" = "method-logs"
            }
        }
    }
    
    try {
        $response = Invoke-RestMethod -Uri "$ElasticsearchUrl/_index_template/method-logs-template" -Method Put -Body ($indexTemplate | ConvertTo-Json -Depth 10) -ContentType "application/json"
        Write-Host "✓ Applied ILM policy to index template" -ForegroundColor Green
    }
    catch {
        Write-Host "✗ Failed to apply ILM policy: $($_.Exception.Message)" -ForegroundColor Red
    }
}

# Set up index lifecycle management
Set-MethodIndexLifecycle
```

### Kibana Security

#### User and Role Management
```powershell
function Set-MethodKibanaSecurity {
    param(
        [string]$ElasticsearchUrl = "http://localhost:9200"
    )
    
    # Create Method developer role
    $developerRole = @{
        "cluster" = @("monitor")
        "indices" = @(
            @{
                "names" = @("method-logs-*", "method-metrics-*")
                "privileges" = @("read", "view_index_metadata")
            }
        )
        "applications" = @(
            @{
                "application" = "kibana-.kibana"
                "privileges" = @("feature_discover.read", "feature_visualize.read", "feature_dashboard.read")
                "resources" = @("*")
            }
        )
    }
    
    try {
        $response = Invoke-RestMethod -Uri "$ElasticsearchUrl/_security/role/method-developer" -Method Put -Body ($developerRole | ConvertTo-Json -Depth 10) -ContentType "application/json"
        Write-Host "✓ Created Method developer role" -ForegroundColor Green
    }
    catch {
        Write-Host "✗ Failed to create role: $($_.Exception.Message)" -ForegroundColor Red
    }
    
    # Create Method admin role
    $adminRole = @{
        "cluster" = @("all")
        "indices" = @(
            @{
                "names" = @("method-*")
                "privileges" = @("all")
            }
        )
        "applications" = @(
            @{
                "application" = "kibana-.kibana"
                "privileges" = @("all")
                "resources" = @("*")
            }
        )
    }
    
    try {
        $response = Invoke-RestMethod -Uri "$ElasticsearchUrl/_security/role/method-admin" -Method Put -Body ($adminRole | ConvertTo-Json -Depth 10) -ContentType "application/json"
        Write-Host "✓ Created Method admin role" -ForegroundColor Green
    }
    catch {
        Write-Host "✗ Failed to create admin role: $($_.Exception.Message)" -ForegroundColor Red
    }
}

# Set up Kibana security
Set-MethodKibanaSecurity
```

## Best Practices

### Dashboard Design
1. **Keep It Simple** - Focus on key metrics and avoid clutter
2. **Use Consistent Time Ranges** - Ensure all visualizations use the same time frame
3. **Color Coding** - Use consistent colors for status indicators
4. **Performance** - Limit the number of visualizations per dashboard
5. **User-Focused** - Design dashboards for specific user roles and needs

### Query Optimization
1. **Use Filters** - Apply filters to reduce data processing
2. **Time Bounds** - Always include time range filters
3. **Field Limiting** - Only include necessary fields in results
4. **Aggregation Efficiency** - Choose appropriate aggregation intervals
5. **Index Patterns** - Use specific index patterns when possible

### Maintenance
1. **Regular Cleanup** - Remove unused dashboards and visualizations
2. **Index Management** - Monitor index sizes and performance
3. **User Training** - Ensure team members know how to use Kibana effectively
4. **Security Reviews** - Regularly review user permissions and access
5. **Performance Monitoring** - Monitor Kibana and Elasticsearch performance

### Integration Tips
1. **Standardize Logging** - Ensure consistent log formats across applications
2. **Correlation IDs** - Use correlation IDs for tracing requests across services
3. **Structured Logging** - Use structured logging formats (JSON) for better searchability
4. **Metric Naming** - Use consistent metric naming conventions
5. **Alert Thresholds** - Set appropriate alert thresholds to avoid noise

This completes the comprehensive Method development environment documentation. The Elasticsearch and Kibana integration provides powerful log analysis and monitoring capabilities for the Method development team.

**Back to:** [Additional Tools Home](./README.md)
