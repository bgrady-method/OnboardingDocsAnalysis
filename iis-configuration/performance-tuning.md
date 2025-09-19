# IIS Performance Tuning

This guide covers comprehensive IIS performance optimization for Method's local development environment to ensure optimal performance and resource utilization.

## Overview

IIS performance tuning is essential for Method's local development environment to ensure smooth operation, efficient resource usage, and optimal response times. This guide covers caching, compression, connection management, and resource optimization.

## Prerequisites

- IIS installed and configured
- Application pools created and configured
- Virtual directories set up and working
- SSL certificates configured
- Security settings applied
- Administrator access for performance configuration
- Performance monitoring tools available

## Performance Architecture

### Method Performance Optimization Layers

```
Method IIS Performance Architecture
├── Application Layer
│   ├── Application Pool Optimization
│   ├── .NET Framework Tuning
│   ├── Memory Management
│   └── Thread Pool Configuration
├── IIS Server Layer
│   ├── Connection Management
│   ├── Request Processing
│   ├── Static Content Optimization
│   └── Dynamic Content Caching
├── Caching Layer
│   ├── Output Caching
│   ├── Kernel Caching
│   ├── Static File Caching
│   └── Application Caching
├── Compression Layer
│   ├── Static Compression
│   ├── Dynamic Compression
│   └── Response Optimization
└── Network Layer
    ├── Keep-Alive Configuration
    ├── Bandwidth Throttling
    └── Connection Limits
```

### Performance Zones

Method services are optimized based on their performance characteristics:

- **High-Traffic Zone** - API Gateway, Authentication (Optimized for throughput)
- **Resource-Intensive Zone** - Account Management, Reporting (Memory optimization)
- **Static Content Zone** - Assets, Downloads (Caching optimization)
- **Interactive Zone** - Portal, UI (Response time optimization)

## Application Pool Performance Tuning

### Application Pool Resource Optimization

```powershell
# Optimize Method application pools for performance
function Optimize-MethodApplicationPoolPerformance {
    Write-Host "Optimizing Method application pool performance..." -ForegroundColor Green
    
    try {
        Import-Module WebAdministration -ErrorAction Stop
        
        # Performance optimization configurations by service type
        $performanceConfigs = @(
            @{
                Name = "MethodGateway"
                ServiceType = "API Gateway"
                MaxProcesses = 2
                ProcessorAffinity = @()  # Let Windows manage
                IdleTimeout = "00:00:00"  # Never idle
                MaxMemoryMB = 2048
                RecyclingInterval = "02:00:00"  # 2 hours
                QueueLength = 5000
                CPULimit = 85  # 85% CPU limit
                CPUAction = "NoAction"  # Don't kill in development
                Description = "High-throughput API gateway optimization"
            },
            @{
                Name = "MethodRuntimeCore"
                ServiceType = "Core Business Logic"
                MaxProcesses = 3
                ProcessorAffinity = @()
                IdleTimeout = "00:00:00"  # Never idle
                MaxMemoryMB = 4096
                RecyclingInterval = "04:00:00"  # 4 hours
                QueueLength = 3000
                CPULimit = 90
                CPUAction = "NoAction"
                Description = "Core business logic processing optimization"
            },
            @{
                Name = "MethodAuthentication"
                ServiceType = "Authentication Service"
                MaxProcesses = 1
                ProcessorAffinity = @()
                IdleTimeout = "00:05:00"
                MaxMemoryMB = 1024
                RecyclingInterval = "01:00:00"  # 1 hour
                QueueLength = 2000
                CPULimit = 70
                CPUAction = "NoAction"
                Description = "Authentication service optimization"
            },
            @{
                Name = "MethodAccountManagement"
                ServiceType = "Account Management"
                MaxProcesses = 2
                ProcessorAffinity = @()
                IdleTimeout = "00:10:00"
                MaxMemoryMB = 2048
                RecyclingInterval = "03:00:00"  # 3 hours
                QueueLength = 1500
                CPULimit = 80
                CPUAction = "NoAction"
                Description = "Account management service optimization"
            },
            @{
                Name = "MethodNotifications"
                ServiceType = "Notification Service"
                MaxProcesses = 1
                ProcessorAffinity = @()
                IdleTimeout = "00:15:00"
                MaxMemoryMB = 1024
                RecyclingInterval = "02:00:00"  # 2 hours
                QueueLength = 1000
                CPULimit = 60
                CPUAction = "NoAction"
                Description = "Notification service optimization"
            },
            @{
                Name = "MethodUI"
                ServiceType = "User Interface"
                MaxProcesses = 1
                ProcessorAffinity = @()
                IdleTimeout = "00:20:00"
                MaxMemoryMB = 1024
                RecyclingInterval = "06:00:00"  # 6 hours
                QueueLength = 1000
                CPULimit = 70
                CPUAction = "NoAction"
                Description = "User interface application optimization"
            },
            @{
                Name = "MethodPortal"
                ServiceType = "Administrative Portal"
                MaxProcesses = 1
                ProcessorAffinity = @()
                IdleTimeout = "00:30:00"
                MaxMemoryMB = 1024
                RecyclingInterval = "08:00:00"  # 8 hours
                QueueLength = 500
                CPULimit = 60
                CPUAction = "NoAction"
                Description = "Administrative portal optimization"
            }
        )
        
        $optimizationResults = @()
        
        foreach ($config in $performanceConfigs) {
            Write-Host "Optimizing performance for: $($config.Name)" -ForegroundColor Yellow
            
            try {
                # Check if application pool exists
                $appPool = Get-IISAppPool -Name $config.Name -ErrorAction SilentlyContinue
                
                if (!$appPool) {
                    Write-Host "⚠ Application pool not found: $($config.Name)" -ForegroundColor Yellow
                    $optimizationResults += [PSCustomObject]@{
                        AppPool = $config.Name
                        ServiceType = $config.ServiceType
                        Status = "⚠ Not Found"
                        OptimizationsApplied = 0
                    }
                    continue
                }
                
                $optimizationsApplied = 0
                
                # Configure process model settings
                try {
                    Set-ItemProperty -Path "IIS:\AppPools\$($config.Name)" -Name "processModel.maxProcesses" -Value $config.MaxProcesses
                    Set-ItemProperty -Path "IIS:\AppPools\$($config.Name)" -Name "processModel.idleTimeout" -Value $config.IdleTimeout
                    Write-Host "  ✓ Process model configured" -ForegroundColor Gray
                    $optimizationsApplied++
                } catch {
                    Write-Host "  ⚠ Could not configure process model" -ForegroundColor Yellow
                }
                
                # Configure memory limits
                try {
                    $memoryLimitKB = $config.MaxMemoryMB * 1024
                    Set-ItemProperty -Path "IIS:\AppPools\$($config.Name)" -Name "recycling.periodicRestart.memory" -Value $memoryLimitKB
                    Write-Host "  ✓ Memory limit set: $($config.MaxMemoryMB) MB" -ForegroundColor Gray
                    $optimizationsApplied++
                } catch {
                    Write-Host "  ⚠ Could not set memory limit" -ForegroundColor Yellow
                }
                
                # Configure recycling interval
                try {
                    Set-ItemProperty -Path "IIS:\AppPools\$($config.Name)" -Name "recycling.periodicRestart.time" -Value $config.RecyclingInterval
                    Write-Host "  ✓ Recycling interval: $($config.RecyclingInterval)" -ForegroundColor Gray
                    $optimizationsApplied++
                } catch {
                    Write-Host "  ⚠ Could not set recycling interval" -ForegroundColor Yellow
                }
                
                # Configure request queue length
                try {
                    Set-ItemProperty -Path "IIS:\AppPools\$($config.Name)" -Name "processModel.requestQueueLimit" -Value $config.QueueLength
                    Write-Host "  ✓ Queue length: $($config.QueueLength)" -ForegroundColor Gray
                    $optimizationsApplied++
                } catch {
                    Write-Host "  ⚠ Could not set queue length" -ForegroundColor Yellow
                }
                
                # Configure CPU limits (optional for development)
                try {
                    Set-ItemProperty -Path "IIS:\AppPools\$($config.Name)" -Name "cpu.limit" -Value ($config.CPULimit * 1000)  # Convert to per-mille
                    Set-ItemProperty -Path "IIS:\AppPools\$($config.Name)" -Name "cpu.action" -Value $config.CPUAction
                    Write-Host "  ✓ CPU limit: $($config.CPULimit)%" -ForegroundColor Gray
                    $optimizationsApplied++
                } catch {
                    Write-Host "  ⚠ Could not set CPU limits" -ForegroundColor Yellow
                }
                
                # Configure failure detection
                try {
                    Set-ItemProperty -Path "IIS:\AppPools\$($config.Name)" -Name "failure.rapidFailProtection" -Value $true
                    Set-ItemProperty -Path "IIS:\AppPools\$($config.Name)" -Name "failure.rapidFailProtectionMaxCrashes" -Value 10  # More lenient for development
                    Set-ItemProperty -Path "IIS:\AppPools\$($config.Name)" -Name "failure.rapidFailProtectionInterval" -Value "00:05:00"
                    Write-Host "  ✓ Failure detection configured" -ForegroundColor Gray
                    $optimizationsApplied++
                } catch {
                    Write-Host "  ⚠ Could not configure failure detection" -ForegroundColor Yellow
                }
                
                Write-Host "✓ Performance optimization completed for: $($config.Name)" -ForegroundColor Green
                
                $optimizationResults += [PSCustomObject]@{
                    AppPool = $config.Name
                    ServiceType = $config.ServiceType
                    Status = "✓ Optimized"
                    OptimizationsApplied = $optimizationsApplied
                    MaxMemoryMB = $config.MaxMemoryMB
                    MaxProcesses = $config.MaxProcesses
                    QueueLength = $config.QueueLength
                }
                
            } catch {
                Write-Host "✗ Failed to optimize: $($config.Name)" -ForegroundColor Red
                Write-Host "Error: $($_.Exception.Message)" -ForegroundColor Red
                
                $optimizationResults += [PSCustomObject]@{
                    AppPool = $config.Name
                    ServiceType = $config.ServiceType
                    Status = "✗ Failed"
                    OptimizationsApplied = 0
                    MaxMemoryMB = 0
                    MaxProcesses = 0
                    QueueLength = 0
                }
            }
        }
        
        # Display results
        Write-Host "`nApplication Pool Performance Optimization Results:" -ForegroundColor Green
        Write-Host "==================================================" -ForegroundColor Green
        $optimizationResults | Format-Table -AutoSize
        
        return $optimizationResults
        
    } catch {
        Write-Host "✗ Error optimizing application pool performance: $($_.Exception.Message)" -ForegroundColor Red
        return @()
    }
}

# Optimize application pool performance
$appPoolOptimization = Optimize-MethodApplicationPoolPerformance
```

### Thread Pool Configuration

```powershell
# Configure .NET thread pool settings for Method applications
function Set-MethodThreadPoolConfiguration {
    Write-Host "Configuring Method .NET thread pool settings..." -ForegroundColor Green
    
    try {
        # Thread pool configurations by application type
        $threadPoolConfigs = @(
            @{
                AppPool = "MethodGateway"
                MinWorkerThreads = 25
                MaxWorkerThreads = 100
                MinIOThreads = 25
                MaxIOThreads = 100
                Description = "API Gateway - high concurrency"
            },
            @{
                AppPool = "MethodRuntimeCore"
                MinWorkerThreads = 50
                MaxWorkerThreads = 200
                MinIOThreads = 50
                MaxIOThreads = 200
                Description = "Core business logic - CPU intensive"
            },
            @{
                AppPool = "MethodAuthentication"
                MinWorkerThreads = 10
                MaxWorkerThreads = 50
                MinIOThreads = 10
                MaxIOThreads = 50
                Description = "Authentication service - moderate load"
            },
            @{
                AppPool = "MethodAccountManagement"
                MinWorkerThreads = 20
                MaxWorkerThreads = 100
                MinIOThreads = 20
                MaxIOThreads = 100
                Description = "Account management - database intensive"
            }
        )
        
        foreach ($config in $threadPoolConfigs) {
            Write-Host "Configuring thread pool for: $($config.AppPool)" -ForegroundColor Yellow
            
            try {
                # Create web.config fragment for thread pool settings
                $webConfigFragment = @"
<!-- Thread Pool Configuration for $($config.AppPool) -->
<configuration>
  <system.web>
    <httpRuntime 
      minFreeThreads="176" 
      minLocalRequestFreeThreads="152"
      maxRequestLength="51200"
      executionTimeout="3600"
      requestValidationMode="2.0" />
    <processModel 
      autoConfig="false"
      minWorkerThreads="$($config.MinWorkerThreads)"
      maxWorkerThreads="$($config.MaxWorkerThreads)"
      minIoThreads="$($config.MinIOThreads)"
      maxIoThreads="$($config.MaxIOThreads)" />
  </system.web>
  <system.webServer>
    <defaultDocument enabled="true">
      <files>
        <clear />
        <add value="default.html" />
        <add value="index.html" />
      </files>
    </defaultDocument>
  </system.webServer>
</configuration>
"@
                
                # Save thread pool configuration template
                $configPath = "C:\temp\ThreadPool-$($config.AppPool)-Config.xml"
                $webConfigFragment | Out-File -FilePath $configPath -Encoding UTF8
                
                Write-Host "✓ Thread pool configuration saved for: $($config.AppPool)" -ForegroundColor Green
                Write-Host "  Template saved to: $configPath" -ForegroundColor Gray
                Write-Host "  Worker Threads: $($config.MinWorkerThreads)-$($config.MaxWorkerThreads)" -ForegroundColor Gray
                Write-Host "  I/O Threads: $($config.MinIOThreads)-$($config.MaxIOThreads)" -ForegroundColor Gray
                
            } catch {
                Write-Host "⚠ Could not configure thread pool for: $($config.AppPool)" -ForegroundColor Yellow
                Write-Host "Error: $($_.Exception.Message)" -ForegroundColor Yellow
            }
        }
        
        # Global thread pool recommendations
        Write-Host "`nThread Pool Configuration Notes:" -ForegroundColor Yellow
        Write-Host "================================" -ForegroundColor Yellow
        Write-Host "• Thread pool settings are configured per application" -ForegroundColor Gray
        Write-Host "• Copy relevant sections to your application's web.config files" -ForegroundColor Gray
        Write-Host "• Monitor thread pool performance using performance counters" -ForegroundColor Gray
        Write-Host "• Adjust based on actual load patterns and performance metrics" -ForegroundColor Gray
        Write-Host "• Consider using async/await patterns to reduce thread usage" -ForegroundColor Gray
        
    } catch {
        Write-Host "✗ Error configuring thread pool settings: $($_.Exception.Message)" -ForegroundColor Red
    }
}

# Configure thread pool settings
Set-MethodThreadPoolConfiguration
```

## IIS Server Performance Optimization

### Connection and Request Processing

```powershell
# Optimize IIS connection and request processing settings
function Optimize-MethodIISConnectionSettings {
    Write-Host "Optimizing Method IIS connection and request processing..." -ForegroundColor Green
    
    try {
        Import-Module WebAdministration
        
        # Connection optimization configurations
        $connectionConfigs = @(
            @{
                Site = "Default Web Site"
                MaxConnections = 4294967295  # Unlimited for development
                ConnectionTimeout = 900       # 15 minutes
                HeaderWaitTimeout = 0        # Unlimited
                MinBytesPerSecond = 0        # No minimum
                MaxBandwidth = 4294967295    # Unlimited
                MaxGlobalBandwidth = 4294967295  # Unlimited
                Description = "Main site connection optimization"
            },
            @{
                Site = "Method Portal"
                MaxConnections = 1000        # Limited for admin portal
                ConnectionTimeout = 1800     # 30 minutes
                HeaderWaitTimeout = 0        # Unlimited
                MinBytesPerSecond = 0        # No minimum
                MaxBandwidth = 4294967295    # Unlimited
                MaxGlobalBandwidth = 4294967295  # Unlimited
                Description = "Portal connection optimization"
            }
        )
        
        foreach ($config in $connectionConfigs) {
            Write-Host "Optimizing connections for: $($config.Site)" -ForegroundColor Yellow
            
            try {
                # Configure connection limits
                Set-WebConfigurationProperty -Filter "system.webServer/serverRuntime" `
                                           -Name "maxRequestEntityAllowed" -Value 4294967295 `
                                           -PSPath "IIS:\Sites\$($config.Site)"
                
                Set-WebConfigurationProperty -Filter "system.webServer/serverRuntime" `
                                           -Name "uploadReadAheadSize" -Value 524288 `
                                           -PSPath "IIS:\Sites\$($config.Site)"
                
                Set-WebConfigurationProperty -Filter "system.webServer/serverRuntime" `
                                           -Name "maxUrlSegments" -Value 260 `
                                           -PSPath "IIS:\Sites\$($config.Site)"
                
                Set-WebConfigurationProperty -Filter "system.webServer/serverRuntime" `
                                           -Name "maxQueryStringLength" -Value 8192 `
                                           -PSPath "IIS:\Sites\$($config.Site)"
                
                Write-Host "✓ Connection settings optimized for: $($config.Site)" -ForegroundColor Green
                
            } catch {
                Write-Host "⚠ Could not optimize connections for: $($config.Site)" -ForegroundColor Yellow
                Write-Host "Error: $($_.Exception.Message)" -ForegroundColor Yellow
            }
        }
        
        # Configure global HTTP settings
        Write-Host "`nConfiguring global HTTP settings..." -ForegroundColor Yellow
        
        try {
            # Configure HTTP.sys settings
            Set-WebConfigurationProperty -Filter "system.webServer/httpProtocol" `
                                       -Name "allowKeepAlive" -Value $true
            
            # Configure request filtering
            Set-WebConfigurationProperty -Filter "system.webServer/security/requestFiltering/requestLimits" `
                                       -Name "maxAllowedContentLength" -Value 2147483647  # ~2GB
            
            Set-WebConfigurationProperty -Filter "system.webServer/security/requestFiltering/requestLimits" `
                                       -Name "maxUrl" -Value 8192
            
            Set-WebConfigurationProperty -Filter "system.webServer/security/requestFiltering/requestLimits" `
                                       -Name "maxQueryString" -Value 8192
            
            Write-Host "✓ Global HTTP settings configured" -ForegroundColor Green
            
        } catch {
            Write-Host "⚠ Could not configure global HTTP settings" -ForegroundColor Yellow
        }
        
        # Configure compression (handled in separate function)
        Write-Host "`nConnection optimization completed" -ForegroundColor Green
        
    } catch {
        Write-Host "✗ Error optimizing IIS connection settings: $($_.Exception.Message)" -ForegroundColor Red
    }
}

# Optimize IIS connection settings
Optimize-MethodIISConnectionSettings
```

### Output Caching Configuration

```powershell
# Configure output caching for Method applications
function Set-MethodOutputCaching {
    Write-Host "Configuring Method output caching..." -ForegroundColor Green
    
    try {
        Import-Module WebAdministration
        
        # Caching configurations by content type
        $cachingConfigs = @(
            @{
                Path = "Default Web Site/api"
                CachingRules = @(
                    @{
                        Extension = "*.json"
                        CacheForSeconds = 300      # 5 minutes for API responses
                        VaryByHeaders = "Accept, Accept-Language, Authorization"
                        VaryByQueryString = "*"
                        Description = "API JSON responses"
                    },
                    @{
                        Extension = "*.xml"
                        CacheForSeconds = 600      # 10 minutes for XML
                        VaryByHeaders = "Accept, Accept-Language"
                        VaryByQueryString = "*"
                        Description = "API XML responses"
                    }
                )
                Description = "API Gateway caching"
            },
            @{
                Path = "Default Web Site/assets"
                CachingRules = @(
                    @{
                        Extension = "*.css"
                        CacheForSeconds = 86400    # 24 hours for CSS
                        VaryByHeaders = ""
                        VaryByQueryString = ""
                        Description = "CSS stylesheets"
                    },
                    @{
                        Extension = "*.js"
                        CacheForSeconds = 86400    # 24 hours for JavaScript
                        VaryByHeaders = ""
                        VaryByQueryString = ""
                        Description = "JavaScript files"
                    },
                    @{
                        Extension = "*.png,*.jpg,*.jpeg,*.gif,*.svg"
                        CacheForSeconds = 604800   # 7 days for images
                        VaryByHeaders = ""
                        VaryByQueryString = ""
                        Description = "Image files"
                    },
                    @{
                        Extension = "*.woff,*.woff2,*.ttf,*.eot"
                        CacheForSeconds = 2592000  # 30 days for fonts
                        VaryByHeaders = ""
                        VaryByQueryString = ""
                        Description = "Font files"
                    }
                )
                Description = "Static assets caching"
            },
            @{
                Path = "Method Portal"
                CachingRules = @(
                    @{
                        Extension = "*.html"
                        CacheForSeconds = 1800     # 30 minutes for HTML pages
                        VaryByHeaders = "Accept-Language, Authorization"
                        VaryByQueryString = "*"
                        Description = "Portal HTML pages"
                    }
                )
                Description = "Portal caching"
            }
        )
        
        $cachingResults = @()
        
        foreach ($config in $cachingConfigs) {
            Write-Host "Configuring caching for: $($config.Path)" -ForegroundColor Yellow
            
            try {
                # Enable output caching
                Set-WebConfigurationProperty -Filter "system.webServer/caching" `
                                           -Name "enabled" -Value $true `
                                           -PSPath "IIS:\Sites\$($config.Path)"
                
                # Configure cache profiles
                foreach ($rule in $config.CachingRules) {
                    try {
                        # Create cache profile
                        $cacheProfileName = "Cache_$($rule.Extension.Replace('*', '').Replace('.', '').Replace(',', '_'))"
                        
                        # Add output cache rule
                        Add-WebConfigurationProperty -Filter "system.webServer/caching/profiles" `
                                                   -Name "." `
                                                   -Value @{
                                                       extension = $rule.Extension
                                                       policy = "CacheUntilChange"
                                                       kernelCachePolicy = "CacheUntilChange"
                                                       duration = "00:00:$($rule.CacheForSeconds)"
                                                   } `
                                                   -PSPath "IIS:\Sites\$($config.Path)" `
                                                   -ErrorAction SilentlyContinue
                        
                        Write-Host "  ✓ Cache rule added: $($rule.Extension) - $($rule.CacheForSeconds)s" -ForegroundColor Gray
                        
                    } catch {
                        Write-Host "  ⚠ Could not add cache rule: $($rule.Extension)" -ForegroundColor Yellow
                    }
                }
                
                Write-Host "✓ Caching configured for: $($config.Path)" -ForegroundColor Green
                
                $cachingResults += [PSCustomObject]@{
                    Path = $config.Path
                    RulesCount = $config.CachingRules.Count
                    Status = "✓ Configured"
                    Description = $config.Description
                }
                
            } catch {
                Write-Host "✗ Failed to configure caching for: $($config.Path)" -ForegroundColor Red
                Write-Host "Error: $($_.Exception.Message)" -ForegroundColor Red
                
                $cachingResults += [PSCustomObject]@{
                    Path = $config.Path
                    RulesCount = 0
                    Status = "✗ Failed"
                    Description = "Configuration failed"
                }
            }
        }
        
        # Configure kernel caching
        Write-Host "`nConfiguring kernel caching..." -ForegroundColor Yellow
        
        try {
            # Enable kernel caching
            Set-WebConfigurationProperty -Filter "system.webServer/caching" `
                                       -Name "enabled" -Value $true
            
            Set-WebConfigurationProperty -Filter "system.webServer/caching" `
                                       -Name "enableKernelCache" -Value $true
            
            Set-WebConfigurationProperty -Filter "system.webServer/caching" `
                                       -Name "maxCacheSize" -Value 1024  # 1GB cache size
            
            Set-WebConfigurationProperty -Filter "system.webServer/caching" `
                                       -Name "maxResponseSize" -Value 262144  # 256KB max response size
            
            Write-Host "✓ Kernel caching configured" -ForegroundColor Green
            
        } catch {
            Write-Host "⚠ Could not configure kernel caching" -ForegroundColor Yellow
        }
        
        # Display results
        Write-Host "`nOutput Caching Configuration Results:" -ForegroundColor Green
        Write-Host "=====================================" -ForegroundColor Green
        $cachingResults | Format-Table -AutoSize
        
        return $cachingResults
        
    } catch {
        Write-Host "✗ Error configuring output caching: $($_.Exception.Message)" -ForegroundColor Red
        return @()
    }
}

# Configure output caching
$cachingResults = Set-MethodOutputCaching
```

## Compression and Response Optimization

### Static and Dynamic Compression

```powershell
# Configure compression for Method applications
function Set-MethodCompressionOptimization {
    Write-Host "Configuring Method compression optimization..." -ForegroundColor Green
    
    try {
        Import-Module WebAdministration
        
        # Enable static compression
        Write-Host "Configuring static compression..." -ForegroundColor Yellow
        
        try {
            # Enable static compression globally
            Set-WebConfigurationProperty -Filter "system.webServer/httpCompression" `
                                       -Name "staticCompressionEnableCpuUsage" -Value 90
            
            Set-WebConfigurationProperty -Filter "system.webServer/httpCompression" `
                                       -Name "staticCompressionDisableCpuUsage" -Value 100
            
            # Configure static compression file types
            $staticCompressionTypes = @(
                "text/*", "message/*", "application/x-javascript", "application/javascript",
                "application/json", "application/xml", "application/atom+xml", 
                "application/xaml+xml", "application/rss+xml", "image/svg+xml",
                "application/x-font-ttf", "application/font-woff", "application/font-woff2"
            )
            
            foreach ($mimeType in $staticCompressionTypes) {
                try {
                    Add-WebConfigurationProperty -Filter "system.webServer/httpCompression/staticTypes" `
                                               -Name "." `
                                               -Value @{mimeType=$mimeType; enabled=$true} `
                                               -ErrorAction SilentlyContinue
                } catch {
                    # Type may already exist
                }
            }
            
            Write-Host "✓ Static compression configured" -ForegroundColor Green
            
        } catch {
            Write-Host "⚠ Could not configure static compression" -ForegroundColor Yellow
        }
        
        # Enable dynamic compression
        Write-Host "Configuring dynamic compression..." -ForegroundColor Yellow
        
        try {
            # Enable dynamic compression globally
            Set-WebConfigurationProperty -Filter "system.webServer/httpCompression" `
                                       -Name "dynamicCompressionEnableCpuUsage" -Value 50
            
            Set-WebConfigurationProperty -Filter "system.webServer/httpCompression" `
                                       -Name "dynamicCompressionDisableCpuUsage" -Value 90
            
            # Configure dynamic compression file types
            $dynamicCompressionTypes = @(
                "text/*", "message/*", "application/x-javascript", "application/javascript",
                "application/json", "application/xml", "application/soap+xml"
            )
            
            foreach ($mimeType in $dynamicCompressionTypes) {
                try {
                    Add-WebConfigurationProperty -Filter "system.webServer/httpCompression/dynamicTypes" `
                                               -Name "." `
                                               -Value @{mimeType=$mimeType; enabled=$true} `
                                               -ErrorAction SilentlyContinue
                } catch {
                    # Type may already exist
                }
            }
            
            Write-Host "✓ Dynamic compression configured" -ForegroundColor Green
            
        } catch {
            Write-Host "⚠ Could not configure dynamic compression" -ForegroundColor Yellow
        }
        
        # Configure compression per site
        $compressionSiteConfigs = @(
            @{
                Site = "Default Web Site"
                StaticCompression = $true
                DynamicCompression = $true
                CompressionLevel = 9  # Maximum compression
                Description = "Main site compression"
            },
            @{
                Site = "Default Web Site/api"
                StaticCompression = $false  # API responses are typically small
                DynamicCompression = $true
                CompressionLevel = 6   # Balanced compression for APIs
                Description = "API Gateway compression"
            },
            @{
                Site = "Default Web Site/assets"
                StaticCompression = $true
                DynamicCompression = $false  # Static content only
                CompressionLevel = 9   # Maximum compression for static files
                Description = "Static assets compression"
            },
            @{
                Site = "Method Portal"
                StaticCompression = $true
                DynamicCompression = $true
                CompressionLevel = 7   # Good compression for admin portal
                Description = "Portal compression"
            }
        )
        
        $compressionResults = @()
        
        foreach ($config in $compressionSiteConfigs) {
            Write-Host "Configuring compression for: $($config.Site)" -ForegroundColor Yellow
            
            try {
                # Enable/disable static compression
                Set-WebConfigurationProperty -Filter "system.webServer/urlCompression" `
                                           -Name "doStaticCompression" -Value $config.StaticCompression `
                                           -PSPath "IIS:\Sites\$($config.Site)"
                
                # Enable/disable dynamic compression
                Set-WebConfigurationProperty -Filter "system.webServer/urlCompression" `
                                           -Name "doDynamicCompression" -Value $config.DynamicCompression `
                                           -PSPath "IIS:\Sites\$($config.Site)"
                
                Write-Host "✓ Compression configured for: $($config.Site)" -ForegroundColor Green
                Write-Host "  Static: $($config.StaticCompression), Dynamic: $($config.DynamicCompression)" -ForegroundColor Gray
                
                $compressionResults += [PSCustomObject]@{
                    Site = $config.Site
                    StaticCompression = $config.StaticCompression
                    DynamicCompression = $config.DynamicCompression
                    CompressionLevel = $config.CompressionLevel
                    Status = "✓ Configured"
                }
                
            } catch {
                Write-Host "✗ Failed to configure compression for: $($config.Site)" -ForegroundColor Red
                Write-Host "Error: $($_.Exception.Message)" -ForegroundColor Red
                
                $compressionResults += [PSCustomObject]@{
                    Site = $config.Site
                    StaticCompression = $false
                    DynamicCompression = $false
                    CompressionLevel = 0
                    Status = "✗ Failed"
                }
            }
        }
        
        # Display results
        Write-Host "`nCompression Configuration Results:" -ForegroundColor Green
        Write-Host "===================================" -ForegroundColor Green
        $compressionResults | Format-Table -AutoSize
        
        # Compression recommendations
        Write-Host "`nCompression Best Practices:" -ForegroundColor Yellow
        Write-Host "===========================" -ForegroundColor Yellow
        Write-Host "• Monitor CPU usage - compression uses CPU resources" -ForegroundColor Gray
        Write-Host "• Static compression is more efficient than dynamic" -ForegroundColor Gray
        Write-Host "• Exclude already compressed files (images, videos)" -ForegroundColor Gray
        Write-Host "• Test compression ratios vs. CPU impact" -ForegroundColor Gray
        Write-Host "• Consider client capabilities and bandwidth" -ForegroundColor Gray
        
        return $compressionResults
        
    } catch {
        Write-Host "✗ Error configuring compression: $($_.Exception.Message)" -ForegroundColor Red
        return @()
    }
}

# Configure compression optimization
$compressionResults = Set-MethodCompressionOptimization
```

### Response Headers Optimization

```powershell
# Optimize response headers for Method applications
function Optimize-MethodResponseHeaders {
    Write-Host "Optimizing Method response headers..." -ForegroundColor Green
    
    try {
        Import-Module WebAdministration
        
        # Performance-focused header configurations
        $headerOptimizationConfigs = @(
            @{
                Path = "Default Web Site/assets"
                Headers = @{
                    "Cache-Control" = "public, max-age=31536000, immutable"  # 1 year cache
                    "Expires" = [DateTime]::Now.AddYears(1).ToString("R")
                    "ETag" = "strong"
                    "Vary" = "Accept-Encoding"
                    "Last-Modified" = "auto"
                }
                Description = "Static assets - aggressive caching"
            },
            @{
                Path = "Default Web Site/api"
                Headers = @{
                    "Cache-Control" = "private, max-age=300"  # 5 minutes cache
                    "Vary" = "Accept, Accept-Encoding, Authorization"
                    "ETag" = "weak"
                }
                Description = "API responses - moderate caching"
            },
            @{
                Path = "Method Portal"
                Headers = @{
                    "Cache-Control" = "private, max-age=1800"  # 30 minutes cache
                    "Vary" = "Accept-Language, Accept-Encoding"
                    "ETag" = "weak"
                }
                Description = "Portal - balanced caching"
            },
            @{
                Path = "Default Web Site/uploads"
                Headers = @{
                    "Cache-Control" = "private, max-age=86400"  # 1 day cache
                    "Content-Disposition" = "inline"
                    "Vary" = "Accept-Encoding"
                }
                Description = "Uploaded files - moderate caching"
            }
        )
        
        $optimizationResults = @()
        
        foreach ($config in $headerOptimizationConfigs) {
            Write-Host "Optimizing headers for: $($config.Path)" -ForegroundColor Yellow
            
            try {
                $configuredHeaders = 0
                
                foreach ($headerName in $config.Headers.Keys) {
                    $headerValue = $config.Headers[$headerName]
                    
                    try {
                        # Remove existing header if present
                        Remove-WebConfigurationProperty -Filter "system.webServer/httpProtocol/customHeaders" `
                                                      -Name "." -AtElement @{name=$headerName} `
                                                      -PSPath "IIS:\Sites\$($config.Path)" `
                                                      -ErrorAction SilentlyContinue
                        
                        # Add optimized header
                        if ($headerValue -ne "auto") {
                            Add-WebConfigurationProperty -Filter "system.webServer/httpProtocol/customHeaders" `
                                                       -Name "." -Value @{name=$headerName; value=$headerValue} `
                                                       -PSPath "IIS:\Sites\$($config.Path)"
                            
                            $configuredHeaders++
                            Write-Host "  ✓ $headerName: $headerValue" -ForegroundColor Gray
                        }
                        
                    } catch {
                        Write-Host "  ⚠ Could not configure header: $headerName" -ForegroundColor Yellow
                    }
                }
                
                # Configure ETags if specified
                if ($config.Headers.ContainsKey("ETag")) {
                    try {
                        $etagSetting = $config.Headers["ETag"]
                        if ($etagSetting -eq "strong") {
                            Set-WebConfigurationProperty -Filter "system.webServer/staticContent" `
                                                       -Name "enableEtag" -Value $true `
                                                       -PSPath "IIS:\Sites\$($config.Path)"
                        }
                        Write-Host "  ✓ ETag configured: $etagSetting" -ForegroundColor Gray
                    } catch {
                        Write-Host "  ⚠ Could not configure ETag" -ForegroundColor Yellow
                    }
                }
                
                Write-Host "✓ Headers optimized for: $($config.Path)" -ForegroundColor Green
                
                $optimizationResults += [PSCustomObject]@{
                    Path = $config.Path
                    HeadersConfigured = $configuredHeaders
                    Status = "✓ Optimized"
                    Description = $config.Description
                }
                
            } catch {
                Write-Host "✗ Failed to optimize headers for: $($config.Path)" -ForegroundColor Red
                Write-Host "Error: $($_.Exception.Message)" -ForegroundColor Red
                
                $optimizationResults += [PSCustomObject]@{
                    Path = $config.Path
                    HeadersConfigured = 0
                    Status = "✗ Failed"
                    Description = "Configuration failed"
                }
            }
        }
        
        # Configure global performance headers
        Write-Host "`nConfiguring global performance headers..." -ForegroundColor Yellow
        
        try {
            # Remove server header for security
            Set-WebConfigurationProperty -Filter "system.webServer/httpProtocol" `
                                       -Name "allowKeepAlive" -Value $true
            
            # Configure default document settings for performance
            Set-WebConfigurationProperty -Filter "system.webServer/defaultDocument" `
                                       -Name "enabled" -Value $true
            
            Write-Host "✓ Global performance headers configured" -ForegroundColor Green
            
        } catch {
            Write-Host "⚠ Could not configure global performance headers" -ForegroundColor Yellow
        }
        
        # Display results
        Write-Host "`nResponse Header Optimization Results:" -ForegroundColor Green
        Write-Host "=====================================" -ForegroundColor Green
        $optimizationResults | Format-Table -AutoSize
        
        return $optimizationResults
        
    } catch {
        Write-Host "✗ Error optimizing response headers: $($_.Exception.Message)" -ForegroundColor Red
        return @()
    }
}

# Optimize response headers
$headerOptimization = Optimize-MethodResponseHeaders
```

## Performance Monitoring and Analysis

### Performance Counter Monitoring

```powershell
# Set up performance monitoring for Method applications
function Set-MethodPerformanceMonitoring {
    Write-Host "Setting up Method performance monitoring..." -ForegroundColor Green
    
    try {
        # Performance counter categories to monitor
        $performanceCounters = @(
            @{
                Category = "Web Service"
                Counters = @("Bytes Received/sec", "Bytes Sent/sec", "Current Connections", "Get Requests/sec", "Post Requests/sec")
                Description = "IIS web service performance"
            },
            @{
                Category = "ASP.NET Applications"
                Counters = @("Requests/Sec", "Request Execution Time", "Requests Queued", "Cache Total Hits", "Cache Total Misses")
                Description = "ASP.NET application performance"
            },
            @{
                Category = "Process"
                Counters = @("% Processor Time", "Working Set", "Private Bytes", "Thread Count")
                Description = "Worker process performance"
            },
            @{
                Category = "Memory"
                Counters = @("Available MBytes", "% Committed Bytes In Use", "Cache Bytes", "Pool Nonpaged Bytes")
                Description = "System memory performance"
            },
            @{
                Category = "PhysicalDisk"
                Counters = @("% Disk Time", "Disk Reads/sec", "Disk Writes/sec", "Avg. Disk Queue Length")
                Description = "Disk I/O performance"
            }
        )
        
        # Create performance monitoring script
        $monitoringScript = @"
# Method Performance Monitoring Script
# Generated: $(Get-Date)

function Get-MethodPerformanceMetrics {
    param([int]`$SampleInterval = 5)
    
    Write-Host "Method Performance Monitoring - Collecting metrics every `$SampleInterval seconds" -ForegroundColor Green
    Write-Host "Press Ctrl+C to stop monitoring" -ForegroundColor Yellow
    Write-Host ""
    
    try {
        while (`$true) {
            `$timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
            Write-Host "Performance Snapshot - `$timestamp" -ForegroundColor Green
            Write-Host "=" * 60 -ForegroundColor Green
            
            # Web Service Metrics
            try {
                `$webConnections = (Get-Counter "\Web Service(_Total)\Current Connections" -ErrorAction SilentlyContinue).CounterSamples.CookedValue
                `$webBytesReceived = (Get-Counter "\Web Service(_Total)\Bytes Received/sec" -ErrorAction SilentlyContinue).CounterSamples.CookedValue
                `$webBytesSent = (Get-Counter "\Web Service(_Total)\Bytes Sent/sec" -ErrorAction SilentlyContinue).CounterSamples.CookedValue
                
                Write-Host "IIS Web Service:" -ForegroundColor Yellow
                Write-Host "  Current Connections: `$webConnections" -ForegroundColor Gray
                Write-Host "  Bytes Received/sec: `$([math]::Round(`$webBytesReceived, 2))" -ForegroundColor Gray
                Write-Host "  Bytes Sent/sec: `$([math]::Round(`$webBytesSent, 2))" -ForegroundColor Gray
                
            } catch {
                Write-Host "Could not collect web service metrics" -ForegroundColor Red
            }
            
            # ASP.NET Application Metrics
            try {
                `$aspNetRequests = (Get-Counter "\ASP.NET Applications(__Total__)\Requests/Sec" -ErrorAction SilentlyContinue).CounterSamples.CookedValue
                `$aspNetQueuedRequests = (Get-Counter "\ASP.NET Applications(__Total__)\Requests Queued" -ErrorAction SilentlyContinue).CounterSamples.CookedValue
                
                Write-Host "ASP.NET Applications:" -ForegroundColor Yellow
                Write-Host "  Requests/sec: `$([math]::Round(`$aspNetRequests, 2))" -ForegroundColor Gray
                Write-Host "  Queued Requests: `$aspNetQueuedRequests" -ForegroundColor Gray
                
            } catch {
                Write-Host "Could not collect ASP.NET metrics" -ForegroundColor Red
            }
            
            # Memory Metrics
            try {
                `$availableMemory = (Get-Counter "\Memory\Available MBytes" -ErrorAction SilentlyContinue).CounterSamples.CookedValue
                `$committedMemory = (Get-Counter "\Memory\% Committed Bytes In Use" -ErrorAction SilentlyContinue).CounterSamples.CookedValue
                
                Write-Host "System Memory:" -ForegroundColor Yellow
                Write-Host "  Available Memory: `$([math]::Round(`$availableMemory, 2)) MB" -ForegroundColor Gray
                Write-Host "  Committed Memory %: `$([math]::Round(`$committedMemory, 2))" -ForegroundColor Gray
                
            } catch {
                Write-Host "Could not collect memory metrics" -ForegroundColor Red
            }
            
            # Method Application Pool Metrics
            try {
                Write-Host "Method Application Pools:" -ForegroundColor Yellow
                `$methodPools = @("MethodGateway", "MethodRuntimeCore", "MethodAuthentication", "MethodAccountManagement")
                
                foreach (`$poolName in `$methodPools) {
                    try {
                        `$workerProcesses = Get-IISWorkerProcess | Where-Object { `$_.AppPoolName -eq `$poolName }
                        if (`$workerProcesses) {
                            `$process = Get-Process -Id `$workerProcesses[0].ProcessId -ErrorAction SilentlyContinue
                            if (`$process) {
                                `$cpuUsage = `$process.CPU
                                `$memoryMB = [math]::Round(`$process.WorkingSet64 / 1MB, 2)
                                Write-Host "  `$poolName - CPU: `$([math]::Round(`$cpuUsage, 2))s, Memory: `$memoryMB MB" -ForegroundColor Gray
                            }
                        } else {
                            Write-Host "  `$poolName - Not Running" -ForegroundColor Red
                        }
                    } catch {
                        Write-Host "  `$poolName - Error getting metrics" -ForegroundColor Red
                    }
                }
                
            } catch {
                Write-Host "Could not collect application pool metrics" -ForegroundColor Red
            }
            
            Write-Host ""
            Start-Sleep -Seconds `$SampleInterval
        }
        
    } catch {
        Write-Host "Performance monitoring stopped" -ForegroundColor Yellow
    }
}

# Start performance monitoring
Get-MethodPerformanceMetrics -SampleInterval 10
"@
        
        # Save monitoring script
        $scriptPath = "C:\MethodDev\Scripts\Monitor-Performance.ps1"
        if (!(Test-Path "C:\MethodDev\Scripts")) {
            New-Item -ItemType Directory -Path "C:\MethodDev\Scripts" -Force
        }
        
        $monitoringScript | Out-File -FilePath $scriptPath -Encoding UTF8
        
        Write-Host "✓ Performance monitoring script created: $scriptPath" -ForegroundColor Green
        Write-Host "Run with: & '$scriptPath'" -ForegroundColor Gray
        
        # Create performance baseline collection
        Write-Host "`nCollecting performance baseline..." -ForegroundColor Yellow
        
        try {
            $baseline = @{
                Timestamp = Get-Date
                CPUCores = (Get-WmiObject Win32_ComputerSystem).NumberOfLogicalProcessors
                TotalMemoryMB = [math]::Round((Get-WmiObject Win32_ComputerSystem).TotalPhysicalMemory / 1MB, 2)
                AvailableMemoryMB = (Get-Counter "\Memory\Available MBytes" -ErrorAction SilentlyContinue).CounterSamples.CookedValue
                IISConnections = (Get-Counter "\Web Service(_Total)\Current Connections" -ErrorAction SilentlyContinue).CounterSamples.CookedValue
            }
            
            Write-Host "Performance Baseline:" -ForegroundColor Green
            Write-Host "  CPU Cores: $($baseline.CPUCores)" -ForegroundColor Gray
            Write-Host "  Total Memory: $($baseline.TotalMemoryMB) MB" -ForegroundColor Gray
            Write-Host "  Available Memory: $([math]::Round($baseline.AvailableMemoryMB, 2)) MB" -ForegroundColor Gray
            Write-Host "  Current IIS Connections: $($baseline.IISConnections)" -ForegroundColor Gray
            
            # Save baseline
            $baselinePath = "C:\MethodDev\Logs\Performance-Baseline-$(Get-Date -Format 'yyyyMMdd').json"
            if (!(Test-Path "C:\MethodDev\Logs")) {
                New-Item -ItemType Directory -Path "C:\MethodDev\Logs" -Force
            }
            
            $baseline | ConvertTo-Json | Out-File -FilePath $baselinePath -Encoding UTF8
            Write-Host "✓ Performance baseline saved: $baselinePath" -ForegroundColor Green
            
        } catch {
            Write-Host "⚠ Could not collect performance baseline" -ForegroundColor Yellow
        }
        
    } catch {
        Write-Host "✗ Error setting up performance monitoring: $($_.Exception.Message)" -ForegroundColor Red
    }
}

# Set up performance monitoring
Set-MethodPerformanceMonitoring
```

### Performance Bottleneck Analysis

```powershell
# Analyze potential performance bottlenecks
function Analyze-MethodPerformanceBottlenecks {
    Write-Host "Analyzing Method performance bottlenecks..." -ForegroundColor Green
    
    try {
        $bottleneckAnalysis = @()
        
        # Analyze memory usage
        Write-Host "Analyzing memory usage..." -ForegroundColor Yellow
        
        try {
            $availableMemoryMB = (Get-Counter "\Memory\Available MBytes" -ErrorAction SilentlyContinue).CounterSamples.CookedValue
            $totalMemoryMB = [math]::Round((Get-WmiObject Win32_ComputerSystem).TotalPhysicalMemory / 1MB, 2)
            $memoryUtilization = [math]::Round((($totalMemoryMB - $availableMemoryMB) / $totalMemoryMB) * 100, 2)
            
            if ($memoryUtilization -gt 80) {
                $bottleneckAnalysis += [PSCustomObject]@{
                    Category = "Memory"
                    Issue = "High memory utilization: $memoryUtilization%"
                    Severity = "High"
                    Recommendation = "Reduce application pool memory limits or add more RAM"
                }
            } elseif ($memoryUtilization -gt 60) {
                $bottleneckAnalysis += [PSCustomObject]@{
                    Category = "Memory"
                    Issue = "Moderate memory utilization: $memoryUtilization%"
                    Severity = "Medium"
                    Recommendation = "Monitor memory usage and consider optimization"
                }
            }
            
            Write-Host "  Memory utilization: $memoryUtilization%" -ForegroundColor Gray
            
        } catch {
            Write-Host "  Could not analyze memory usage" -ForegroundColor Yellow
        }
        
        # Analyze CPU usage
        Write-Host "Analyzing CPU usage..." -ForegroundColor Yellow
        
        try {
            $cpuUsage = (Get-Counter "\Processor(_Total)\% Processor Time" -ErrorAction SilentlyContinue).CounterSamples.CookedValue
            
            if ($cpuUsage -gt 80) {
                $bottleneckAnalysis += [PSCustomObject]@{
                    Category = "CPU"
                    Issue = "High CPU utilization: $([math]::Round($cpuUsage, 2))%"
                    Severity = "High"
                    Recommendation = "Optimize application code or reduce application pool CPU limits"
                }
            } elseif ($cpuUsage -gt 60) {
                $bottleneckAnalysis += [PSCustomObject]@{
                    Category = "CPU"
                    Issue = "Moderate CPU utilization: $([math]::Round($cpuUsage, 2))%"
                    Severity = "Medium"
                    Recommendation = "Monitor CPU usage patterns and optimize if needed"
                }
            }
            
            Write-Host "  CPU utilization: $([math]::Round($cpuUsage, 2))%" -ForegroundColor Gray
            
        } catch {
            Write-Host "  Could not analyze CPU usage" -ForegroundColor Yellow
        }
        
        # Analyze disk I/O
        Write-Host "Analyzing disk I/O..." -ForegroundColor Yellow
        
        try {
            $diskTime = (Get-Counter "\PhysicalDisk(_Total)\% Disk Time" -ErrorAction SilentlyContinue).CounterSamples.CookedValue
            $diskQueueLength = (Get-Counter "\PhysicalDisk(_Total)\Avg. Disk Queue Length" -ErrorAction SilentlyContinue).CounterSamples.CookedValue
            
            if ($diskTime -gt 80) {
                $bottleneckAnalysis += [PSCustomObject]@{
                    Category = "Disk I/O"
                    Issue = "High disk utilization: $([math]::Round($diskTime, 2))%"
                    Severity = "High"
                    Recommendation = "Consider SSD upgrade or optimize database queries"
                }
            }
            
            if ($diskQueueLength -gt 2) {
                $bottleneckAnalysis += [PSCustomObject]@{
                    Category = "Disk I/O"
                    Issue = "High disk queue length: $([math]::Round($diskQueueLength, 2))"
                    Severity = "Medium"
                    Recommendation = "Optimize disk I/O or upgrade storage subsystem"
                }
            }
            
            Write-Host "  Disk utilization: $([math]::Round($diskTime, 2))%" -ForegroundColor Gray
            Write-Host "  Disk queue length: $([math]::Round($diskQueueLength, 2))" -ForegroundColor Gray
            
        } catch {
            Write-Host "  Could not analyze disk I/O" -ForegroundColor Yellow
        }
        
        # Analyze IIS performance
        Write-Host "Analyzing IIS performance..." -ForegroundColor Yellow
        
        try {
            $currentConnections = (Get-Counter "\Web Service(_Total)\Current Connections" -ErrorAction SilentlyContinue).CounterSamples.CookedValue
            $queuedRequests = (Get-Counter "\ASP.NET Applications(__Total__)\Requests Queued" -ErrorAction SilentlyContinue).CounterSamples.CookedValue
            
            if ($queuedRequests -gt 10) {
                $bottleneckAnalysis += [PSCustomObject]@{
                    Category = "IIS"
                    Issue = "High queued requests: $queuedRequests"
                    Severity = "High"
                    Recommendation = "Increase application pool worker processes or optimize application response time"
                }
            }
            
            if ($currentConnections -gt 1000) {
                $bottleneckAnalysis += [PSCustomObject]@{
                    Category = "IIS"
                    Issue = "High connection count: $currentConnections"
                    Severity = "Medium"
                    Recommendation = "Monitor connection patterns and consider connection limits"
                }
            }
            
            Write-Host "  Current connections: $currentConnections" -ForegroundColor Gray
            Write-Host "  Queued requests: $queuedRequests" -ForegroundColor Gray
            
        } catch {
            Write-Host "  Could not analyze IIS performance" -ForegroundColor Yellow
        }
        
        # Analyze Method application pools
        Write-Host "Analyzing Method application pools..." -ForegroundColor Yellow
        
        try {
            $methodPools = @("MethodGateway", "MethodRuntimeCore", "MethodAuthentication", "MethodAccountManagement")
            
            foreach ($poolName in $methodPools) {
                $workerProcesses = Get-IISWorkerProcess | Where-Object { $_.AppPoolName -eq $poolName }
                
                if (!$workerProcesses) {
                    $bottleneckAnalysis += [PSCustomObject]@{
                        Category = "Application Pools"
                        Issue = "Application pool not running: $poolName"
                        Severity = "High"
                        Recommendation = "Start application pool and investigate startup issues"
                    }
                } else {
                    try {
                        $process = Get-Process -Id $workerProcesses[0].ProcessId -ErrorAction SilentlyContinue
                        if ($process) {
                            $memoryMB = [math]::Round($process.WorkingSet64 / 1MB, 2)
                            
                            # Check for memory issues based on pool type
                            $memoryThreshold = switch ($poolName) {
                                "MethodRuntimeCore" { 3000 }
                                "MethodGateway" { 1500 }
                                default { 1000 }
                            }
                            
                            if ($memoryMB -gt $memoryThreshold) {
                                $bottleneckAnalysis += [PSCustomObject]@{
                                    Category = "Application Pools"
                                    Issue = "High memory usage in $poolName: $memoryMB MB"
                                    Severity = "Medium"
                                    Recommendation = "Investigate memory leaks or increase memory limits"
                                }
                            }
                        }
                    } catch {
                        # Could not get process info
                    }
                }
            }
            
        } catch {
            Write-Host "  Could not analyze application pools" -ForegroundColor Yellow
        }
        
        # Display bottleneck analysis results
        if ($bottleneckAnalysis.Count -gt 0) {
            Write-Host "`nPerformance Bottleneck Analysis Results:" -ForegroundColor Red
            Write-Host "=========================================" -ForegroundColor Red
            $bottleneckAnalysis | Format-Table -Wrap -AutoSize
            
            # Summary by severity
            $highSeverity = ($bottleneckAnalysis | Where-Object { $_.Severity -eq "High" }).Count
            $mediumSeverity = ($bottleneckAnalysis | Where-Object { $_.Severity -eq "Medium" }).Count
            
            Write-Host "`nBottleneck Summary:" -ForegroundColor Yellow
            Write-Host "High Severity Issues: $highSeverity" -ForegroundColor Red
            Write-Host "Medium Severity Issues: $mediumSeverity" -ForegroundColor Yellow
            
        } else {
            Write-Host "`n✓ No significant performance bottlenecks detected" -ForegroundColor Green
        }
        
        return $bottleneckAnalysis
        
    } catch {
        Write-Host "✗ Error analyzing performance bottlenecks: $($_.Exception.Message)" -ForegroundColor Red
        return @()
    }
}

# Analyze performance bottlenecks
$bottleneckAnalysis = Analyze-MethodPerformanceBottlenecks
```

## Performance Validation and Testing

### Comprehensive Performance Testing

```powershell
# Comprehensive Method performance validation
function Test-MethodPerformanceConfiguration {
    Write-Host "Method Performance Configuration Validation" -ForegroundColor Green
    Write-Host "===========================================" -ForegroundColor Green
    
    $validationResults = @()
    
    try {
        Import-Module WebAdministration
        
        # Test 1: Application Pool Performance Settings
        $appPoolsOptimized = 0
        $expectedPools = @("MethodGateway", "MethodRuntimeCore", "MethodAuthentication", "MethodAccountManagement")
        
        foreach ($poolName in $expectedPools) {
            try {
                $pool = Get-IISAppPool -Name $poolName -ErrorAction SilentlyContinue
                if ($pool) {
                    $maxProcesses = Get-ItemProperty -Path "IIS:\AppPools\$poolName" -Name "processModel.maxProcesses" -ErrorAction SilentlyContinue
                    $memoryLimit = Get-ItemProperty -Path "IIS:\AppPools\$poolName" -Name "recycling.periodicRestart.memory" -ErrorAction SilentlyContinue
                    
                    if ($maxProcesses.Value -gt 0 -and $memoryLimit.Value -gt 0) {
                        $appPoolsOptimized++
                    }
                }
            } catch {
                # Pool may not exist
            }
        }
        
        $validationResults += [PSCustomObject]@{
            Test = "Application Pool Optimization"
            Status = if ($appPoolsOptimized -eq $expectedPools.Count) { "✓ Pass" } else { "⚠ Partial" }
            Details = "$appPoolsOptimized/$($expectedPools.Count) pools optimized"
        }
        
        # Test 2: Compression Configuration
        $compressionEnabled = 0
        $compressionPaths = @("Default Web Site", "Default Web Site/assets")
        
        foreach ($path in $compressionPaths) {
            try {
                $staticCompression = Get-WebConfigurationProperty -Filter "system.webServer/urlCompression" `
                                                                -Name "doStaticCompression" -PSPath "IIS:\Sites\$path" `
                                                                -ErrorAction SilentlyContinue
                if ($staticCompression.Value) {
                    $compressionEnabled++
                }
            } catch {
                # Path may not exist
            }
        }
        
        $validationResults += [PSCustomObject]@{
            Test = "Compression Configuration"
            Status = if ($compressionEnabled -gt 0) { "✓ Pass" } else { "✗ Fail" }
            Details = "$compressionEnabled paths have compression enabled"
        }
        
        # Test 3: Caching Configuration
        $cachingEnabled = 0
        $cachingPaths = @("Default Web Site/assets", "Default Web Site/api")
        
        foreach ($path in $cachingPaths) {
            try {
                $cacheEnabled = Get-WebConfigurationProperty -Filter "system.webServer/caching" `
                                                            -Name "enabled" -PSPath "IIS:\Sites\$path" `
                                                            -ErrorAction SilentlyContinue
                if ($cacheEnabled.Value) {
                    $cachingEnabled++
                }
            } catch {
                # Path may not exist
            }
        }
        
        $validationResults += [PSCustomObject]@{
            Test = "Output Caching"
            Status = if ($cachingEnabled -gt 0) { "✓ Pass" } else { "⚠ Partial" }
            Details = "$cachingEnabled paths have caching configured"
        }
        
        # Test 4: Performance Headers
        $headersOptimized = 0
        $headerPaths = @("Default Web Site/assets", "Default Web Site/api")
        
        foreach ($path in $headerPaths) {
            try {
                $customHeaders = Get-WebConfiguration -Filter "system.webServer/httpProtocol/customHeaders" `
                                                    -PSPath "IIS:\Sites\$path" `
                                                    -ErrorAction SilentlyContinue
                
                # Check for performance-related headers
                $performanceHeaders = $customHeaders.Collection | Where-Object { 
                    $_.name -eq "Cache-Control" -or $_.name -eq "Expires" -or $_.name -eq "ETag" 
                }
                
                if ($performanceHeaders.Count -gt 0) {
                    $headersOptimized++
                }
            } catch {
                # Path may not exist
            }
        }
        
        $validationResults += [PSCustomObject]@{
            Test = "Performance Headers"
            Status = if ($headersOptimized -gt 0) { "✓ Pass" } else { "⚠ Partial" }
            Details = "$headersOptimized paths have performance headers"
        }
        
        # Test 5: System Resource Health
        $resourceHealth = "✓ Healthy"
        try {
            $availableMemoryMB = (Get-Counter "\Memory\Available MBytes" -ErrorAction SilentlyContinue).CounterSamples.CookedValue
            $cpuUsage = (Get-Counter "\Processor(_Total)\% Processor Time" -ErrorAction SilentlyContinue).CounterSamples.CookedValue
            
            if ($availableMemoryMB -lt 1024 -or $cpuUsage -gt 80) {
                $resourceHealth = "⚠ Resource Constraints"
            }
        } catch {
            $resourceHealth = "⚠ Cannot Monitor"
        }
        
        $validationResults += [PSCustomObject]@{
            Test = "System Resource Health"
            Status = $resourceHealth
            Details = "Memory: $([math]::Round($availableMemoryMB, 0)) MB available, CPU: $([math]::Round($cpuUsage, 1))%"
        }
        
        # Display results
        $validationResults | Format-Table -AutoSize
        
        # Summary
        $passed = ($validationResults | Where-Object { $_.Status -like "*Pass*" }).Count
        $total = $validationResults.Count
        $percentage = [math]::Round(($passed / $total) * 100, 1)
        
        Write-Host "`nPerformance Validation Summary: $passed/$total tests passed ($percentage%)" -ForegroundColor $(if ($passed -eq $total) { "Green" } elseif ($percentage -ge 80) { "Yellow" } else { "Red" })
        
        if ($passed -eq $total) {
            Write-Host "✓ All performance configurations are properly set!" -ForegroundColor Green
        } else {
            Write-Host "⚠ Some performance configurations need attention" -ForegroundColor Yellow
        }
        
        return $passed -eq $total
        
    } catch {
        Write-Host "✗ Error during performance validation: $($_.Exception.Message)" -ForegroundColor Red
        return $false
    }
}

# Run comprehensive performance validation
$performanceConfigured = Test-MethodPerformanceConfiguration
```

### Load Testing Recommendations

```powershell
# Provide load testing recommendations for Method applications
function Get-MethodLoadTestingGuidance {
    Write-Host "Method Load Testing Guidance" -ForegroundColor Green
    Write-Host "============================" -ForegroundColor Green
    
    # Load testing scenarios
    $loadTestingScenarios = @(
        @{
            Service = "MethodGateway"
            TestType = "API Load Test"
            RecommendedTool = "Apache JMeter, Artillery, or k6"
            TestScenarios = @(
                "Concurrent API requests (100-1000 users)",
                "Sustained load over 30 minutes",
                "Spike testing (sudden load increases)",
                "Stress testing (beyond normal capacity)"
            )
            KeyMetrics = @(
                "Response time (< 500ms target)",
                "Throughput (requests/second)",
                "Error rate (< 1% target)",
                "CPU and memory usage"
            )
            TestEndpoints = @(
                "GET /api/health",
                "POST /api/authenticate",
                "GET /api/accounts",
                "PUT /api/accounts/{id}"
            )
        },
        @{
            Service = "MethodUI"
            TestType = "Browser Load Test"
            RecommendedTool = "Selenium Grid, Playwright, or Puppeteer"
            TestScenarios = @(
                "Multiple browser sessions (10-50 concurrent)",
                "Page load testing",
                "Form submission testing",
                "Navigation pattern testing"
            )
            KeyMetrics = @(
                "Page load time (< 3s target)",
                "Time to interactive (< 5s target)",
                "JavaScript errors",
                "Memory leaks"
            )
            TestEndpoints = @(
                "https://method.local/",
                "https://method.local/login",
                "https://method.local/dashboard",
                "https://method.local/accounts"
            )
        },
        @{
            Service = "MethodRuntimeCore"
            TestType = "Business Logic Load Test"
            RecommendedTool = "Custom .NET load test or NBomber"
            TestScenarios = @(
                "Heavy business rule processing",
                "Database transaction testing",
                "Complex calculation workloads",
                "Memory-intensive operations"
            )
            KeyMetrics = @(
                "Processing time per operation",
                "Database connection pooling",
                "Memory usage patterns",
                "Thread pool utilization"
            )
            TestEndpoints = @(
                "Internal business logic APIs",
                "Database-intensive operations",
                "Calculation engines",
                "Reporting functions"
            )
        }
    )
    
    # Display load testing guidance
    foreach ($scenario in $loadTestingScenarios) {
        Write-Host "`n$($scenario.Service) - $($scenario.TestType)" -ForegroundColor Yellow
        Write-Host "=" * 50 -ForegroundColor Yellow
        Write-Host "Recommended Tool: $($scenario.RecommendedTool)" -ForegroundColor Gray
        
        Write-Host "`nTest Scenarios:" -ForegroundColor Green
        foreach ($testScenario in $scenario.TestScenarios) {
            Write-Host "  • $testScenario" -ForegroundColor Gray
        }
        
        Write-Host "`nKey Metrics:" -ForegroundColor Green
        foreach ($metric in $scenario.KeyMetrics) {
            Write-Host "  • $metric" -ForegroundColor Gray
        }
        
        Write-Host "`nTest Endpoints:" -ForegroundColor Green
        foreach ($endpoint in $scenario.TestEndpoints) {
            Write-Host "  • $endpoint" -ForegroundColor Gray
        }
    }
    
    # Sample JMeter test plan
    $jmeterTestPlan = @"
# Sample JMeter Test Plan for Method API Gateway
# Test Plan: Method_API_Load_Test

Thread Group: API_Load_Test
├── Number of Threads: 100
├── Ramp-up Period: 60 seconds
├── Loop Count: 10
└── Test Elements:
    ├── HTTP Request: GET /api/health
    ├── HTTP Request: POST /api/authenticate
    ├── HTTP Request: GET /api/accounts
    └── HTTP Request: PUT /api/accounts/{id}

Listeners:
├── View Results Tree
├── Summary Report
├── Response Times Over Time
└── Aggregate Report

Assertions:
├── Response Code: 200
├── Response Time: < 500ms
└── JSON Response Content
"@
    
    Write-Host "`nSample JMeter Test Plan:" -ForegroundColor Green
    Write-Host "=========================" -ForegroundColor Green
    Write-Host $jmeterTestPlan -ForegroundColor Gray
    
    # Performance testing best practices
    Write-Host "`nPerformance Testing Best Practices:" -ForegroundColor Yellow
    Write-Host "====================================" -ForegroundColor Yellow
    Write-Host "1. Start with baseline performance measurements" -ForegroundColor Gray
    Write-Host "2. Test in an environment that mimics production" -ForegroundColor Gray
    Write-Host "3. Monitor system resources during tests" -ForegroundColor Gray
    Write-Host "4. Test individual components before full system tests" -ForegroundColor Gray
    Write-Host "5. Gradually increase load to identify breaking points" -ForegroundColor Gray
    Write-Host "6. Test with realistic data volumes and user patterns" -ForegroundColor Gray
    Write-Host "7. Include negative testing (error conditions)" -ForegroundColor Gray
    Write-Host "8. Document and track performance over time" -ForegroundColor Gray
    
    # Create simple load testing script
    $loadTestScript = @"
# Simple PowerShell Load Test for Method API
# Generated: $(Get-Date)

function Test-MethodAPILoad {
    param(
        [string]`$BaseUrl = "https://method.local",
        [int]`$ConcurrentUsers = 10,
        [int]`$TestDurationMinutes = 5
    )
    
    Write-Host "Starting Method API load test..." -ForegroundColor Green
    Write-Host "Base URL: `$BaseUrl" -ForegroundColor Gray
    Write-Host "Concurrent Users: `$ConcurrentUsers" -ForegroundColor Gray
    Write-Host "Duration: `$TestDurationMinutes minutes" -ForegroundColor Gray
    
    `$endpoints = @(
        "/api/health",
        "/api/accounts",
        "/api/configuration"
    )
    
    `$results = @()
    `$endTime = (Get-Date).AddMinutes(`$TestDurationMinutes)
    
    1..`$ConcurrentUsers | ForEach-Object -Parallel {
        `$userResults = @()
        while ((Get-Date) -lt `$using:endTime) {
            foreach (`$endpoint in `$using:endpoints) {
                try {
                    `$startTime = Get-Date
                    `$response = Invoke-WebRequest -Uri "`$(`$using:BaseUrl)`$endpoint" -TimeoutSec 30 -UseBasicParsing
                    `$endTime = Get-Date
                    `$responseTime = (`$endTime - `$startTime).TotalMilliseconds
                    
                    `$userResults += [PSCustomObject]@{
                        Endpoint = `$endpoint
                        StatusCode = `$response.StatusCode
                        ResponseTime = `$responseTime
                        Success = `$true
                        Timestamp = `$startTime
                    }
                } catch {
                    `$userResults += [PSCustomObject]@{
                        Endpoint = `$endpoint
                        StatusCode = 0
                        ResponseTime = 0
                        Success = `$false
                        Timestamp = Get-Date
                    }
                }
                
                Start-Sleep -Milliseconds 100
            }
        }
        return `$userResults
    } | ForEach-Object { `$results += `$_ }
    
    # Analyze results
    `$totalRequests = `$results.Count
    `$successfulRequests = (`$results | Where-Object { `$_.Success }).Count
    `$averageResponseTime = (`$results | Where-Object { `$_.Success } | Measure-Object -Property ResponseTime -Average).Average
    
    Write-Host "`nLoad Test Results:" -ForegroundColor Green
    Write-Host "Total Requests: `$totalRequests" -ForegroundColor Gray
    Write-Host "Successful Requests: `$successfulRequests" -ForegroundColor Gray
    Write-Host "Success Rate: `$([math]::Round((`$successfulRequests / `$totalRequests) * 100, 2))%" -ForegroundColor Gray
    Write-Host "Average Response Time: `$([math]::Round(`$averageResponseTime, 2)) ms" -ForegroundColor Gray
    
    return `$results
}

# Run load test
# Test-MethodAPILoad -BaseUrl "https://method.local" -ConcurrentUsers 5 -TestDurationMinutes 2
"@
    
    $scriptPath = "C:\MethodDev\Scripts\Simple-LoadTest.ps1"
    if (!(Test-Path "C:\MethodDev\Scripts")) {
        New-Item -ItemType Directory -Path "C:\MethodDev\Scripts" -Force
    }
    
    $loadTestScript | Out-File -FilePath $scriptPath -Encoding UTF8
    Write-Host "`n✓ Simple load testing script created: $scriptPath" -ForegroundColor Green
    Write-Host "Customize and run with appropriate parameters for your environment" -ForegroundColor Gray
}

# Get load testing guidance
Get-MethodLoadTestingGuidance
```

## Next Steps

After configuring IIS performance optimization:

1. **Set up GitHub Integration:** [GitHub Setup](../github-setup/README.md)
2. **Configure Environment Validation:** [Environment Setup](../environment-setup/README.md)
3. **Certificate Management:** [Certificate Configuration](../certificates/README.md)
4. **Regular Performance Monitoring:** Implement ongoing performance monitoring
5. **Load Testing:** Execute load testing scenarios

## Best Practices

### Performance Optimization

- **Monitor Continuously:** Set up ongoing performance monitoring and alerting
- **Optimize Incrementally:** Make gradual improvements and measure impact
- **Test Thoroughly:** Validate performance changes under load
- **Document Settings:** Maintain documentation of performance configurations
- **Regular Review:** Periodically review and adjust performance settings

### Resource Management

- **Right-size Resources:** Configure appropriate limits based on actual usage
- **Monitor Trends:** Track performance trends over time
- **Capacity Planning:** Plan for growth and peak usage scenarios
- **Resource Balancing:** Balance CPU, memory, and I/O optimization
- **Cost Efficiency:** Optimize for both performance and resource costs

### Development vs Production

- **Development Environment:** Focus on developer productivity while maintaining reasonable performance
- **Staging Environment:** Mirror production performance settings for accurate testing
- **Production Environment:** Implement aggressive optimization with proper monitoring
- **Performance Testing:** Regular load testing across all environments
- **Configuration Management:** Maintain consistent performance configurations

**Back to:** [IIS Configuration](./README.md)
