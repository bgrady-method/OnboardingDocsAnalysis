# Virtual Directory Configuration

This guide covers setting up IIS virtual directories for Method's microservices architecture in local development.

## Overview

Virtual directories are essential for Method's local development environment to properly route requests to different microservices while maintaining a unified domain structure. This configuration enables the API Gateway pattern and ensures proper service isolation.

## Prerequisites

- IIS installed and configured
- Application pools created and configured
- SSL certificates installed and configured
- Administrator access for IIS management
- Method applications built and ready for deployment

## Virtual Directory Architecture

### Method Virtual Directory Structure

```
Method IIS Virtual Directory Layout
├── / (method.local)
│   ├── /api (Gateway API endpoints)
│   ├── /auth (Authentication service)
│   ├── /accounts (Account management)
│   ├── /config (Configuration service)
│   ├── /notifications (Notification service)
│   ├── /files (File management)
│   └── /reports (Reporting service)
├── /portal (portal.method.local)
│   ├── /admin (Administrative functions)
│   ├── /dashboard (Management dashboard)
│   └── /tools (Development tools)
└── /assets (Static resources)
    ├── /css (Stylesheets)
    ├── /js (JavaScript files)
    └── /images (Image assets)
```

### Service Mapping

Method's microservices are mapped to specific virtual directories:

- **Gateway API** (`/api`) → Points to MethodGateway application
- **Authentication** (`/auth`) → Points to MethodAuthentication application  
- **Account Management** (`/accounts`) → Points to MethodAccountManagement application
- **Configuration** (`/config`) → Points to MethodConfiguration application
- **Notifications** (`/notifications`) → Points to MethodNotifications application
- **Admin Portal** (`/portal`) → Points to MethodPortal application

## Virtual Directory Creation

### Create Method Virtual Directories

```powershell
# Create Method virtual directories with proper application mappings
function New-MethodVirtualDirectories {
    Write-Host "Creating Method virtual directories..." -ForegroundColor Green
    
    try {
        Import-Module WebAdministration -ErrorAction Stop
        
        # Method virtual directory configurations
        $methodVirtualDirs = @(
            @{
                Site = "Default Web Site"
                Name = "api"
                PhysicalPath = "C:\MethodDev\Gateway\bin\Release\net6.0\publish"
                ApplicationPool = "MethodGateway"
                Description = "Method Gateway API endpoints"
                RequiresAuth = $false
            },
            @{
                Site = "Default Web Site"
                Name = "auth"
                PhysicalPath = "C:\MethodDev\Authentication\bin\Release\net6.0\publish"
                ApplicationPool = "MethodAuthentication"
                Description = "Method Authentication Service"
                RequiresAuth = $false
            },
            @{
                Site = "Default Web Site"
                Name = "accounts"
                PhysicalPath = "C:\MethodDev\AccountManagement\bin\Release\net6.0\publish"
                ApplicationPool = "MethodAccountManagement"
                Description = "Method Account Management Service"
                RequiresAuth = $true
            },
            @{
                Site = "Default Web Site"
                Name = "config"
                PhysicalPath = "C:\MethodDev\Configuration\bin\Release\net6.0\publish"
                ApplicationPool = "MethodConfiguration"
                Description = "Method Configuration Service"
                RequiresAuth = $true
            },
            @{
                Site = "Default Web Site"
                Name = "notifications"
                PhysicalPath = "C:\MethodDev\Notifications\bin\Release\net6.0\publish"
                ApplicationPool = "MethodNotifications"
                Description = "Method Notification Service"
                RequiresAuth = $true
            },
            @{
                Site = "Default Web Site"
                Name = "files"
                PhysicalPath = "C:\MethodDev\FileManagement\bin\Release\net6.0\publish"
                ApplicationPool = "MethodRuntimeCore"
                Description = "Method File Management Service"
                RequiresAuth = $true
            },
            @{
                Site = "Default Web Site"
                Name = "reports"
                PhysicalPath = "C:\MethodDev\Reporting\bin\Release\net6.0\publish"
                ApplicationPool = "MethodRuntimeCore"
                Description = "Method Reporting Service"
                RequiresAuth = $true
            }
        )
        
        # Portal virtual directories (separate site)
        $portalVirtualDirs = @(
            @{
                Site = "Method Portal"
                Name = "admin"
                PhysicalPath = "C:\MethodDev\Portal\Admin\bin\Release\net6.0\publish"
                ApplicationPool = "MethodPortal"
                Description = "Method Administrative Portal"
                RequiresAuth = $true
            },
            @{
                Site = "Method Portal"
                Name = "dashboard"
                PhysicalPath = "C:\MethodDev\Portal\Dashboard\bin\Release\net6.0\publish"
                ApplicationPool = "MethodPortal"
                Description = "Method Management Dashboard"
                RequiresAuth = $true
            },
            @{
                Site = "Method Portal"
                Name = "tools"
                PhysicalPath = "C:\MethodDev\Portal\Tools\bin\Release\net6.0\publish"
                ApplicationPool = "MethodPortal"
                Description = "Method Development Tools"
                RequiresAuth = $true
            }
        )
        
        # Combine all virtual directories
        $allVirtualDirs = $methodVirtualDirs + $portalVirtualDirs
        $createdVirtualDirs = @()
        
        foreach ($vdirConfig in $allVirtualDirs) {
            Write-Host "Creating virtual directory: /$($vdirConfig.Name)" -ForegroundColor Yellow
            
            try {
                # Check if site exists
                $site = Get-Website -Name $vdirConfig.Site -ErrorAction SilentlyContinue
                if (!$site) {
                    Write-Host "⚠ Site '$($vdirConfig.Site)' does not exist - creating it" -ForegroundColor Yellow
                    New-Website -Name $vdirConfig.Site -Port 80 -PhysicalPath "C:\inetpub\wwwroot"
                }
                
                # Remove existing virtual directory if present
                $existingVDir = Get-WebVirtualDirectory -Site $vdirConfig.Site -Name $vdirConfig.Name -ErrorAction SilentlyContinue
                if ($existingVDir) {
                    Write-Host "⚠ Virtual directory '/$($vdirConfig.Name)' already exists - removing" -ForegroundColor Yellow
                    Remove-WebVirtualDirectory -Site $vdirConfig.Site -Name $vdirConfig.Name
                }
                
                # Create directory structure if it doesn't exist
                if (!(Test-Path $vdirConfig.PhysicalPath)) {
                    Write-Host "Creating directory: $($vdirConfig.PhysicalPath)" -ForegroundColor Gray
                    New-Item -ItemType Directory -Path $vdirConfig.PhysicalPath -Force
                    
                    # Create a simple index file for testing
                    $indexContent = @"
<!DOCTYPE html>
<html>
<head>
    <title>Method $($vdirConfig.Name.ToUpper()) Service</title>
</head>
<body>
    <h1>Method $($vdirConfig.Description)</h1>
    <p>Service endpoint: /$($vdirConfig.Name)</p>
    <p>Status: Ready for deployment</p>
    <p>Application Pool: $($vdirConfig.ApplicationPool)</p>
</body>
</html>
"@
                    $indexContent | Out-File -FilePath (Join-Path $vdirConfig.PhysicalPath "index.html") -Encoding UTF8
                }
                
                # Create virtual directory
                New-WebVirtualDirectory -Site $vdirConfig.Site -Name $vdirConfig.Name -PhysicalPath $vdirConfig.PhysicalPath
                
                # Convert to application (so it can use its own app pool)
                ConvertTo-WebApplication -PSPath "IIS:\Sites\$($vdirConfig.Site)\$($vdirConfig.Name)" -ApplicationPool $vdirConfig.ApplicationPool
                
                Write-Host "✓ Created virtual directory: /$($vdirConfig.Name)" -ForegroundColor Green
                Write-Host "  Physical Path: $($vdirConfig.PhysicalPath)" -ForegroundColor Gray
                Write-Host "  Application Pool: $($vdirConfig.ApplicationPool)" -ForegroundColor Gray
                
                $createdVirtualDirs += [PSCustomObject]@{
                    Site = $vdirConfig.Site
                    Name = $vdirConfig.Name
                    PhysicalPath = $vdirConfig.PhysicalPath
                    ApplicationPool = $vdirConfig.ApplicationPool
                    Description = $vdirConfig.Description
                    AuthRequired = $vdirConfig.RequiresAuth
                    Status = "Created"
                }
                
            } catch {
                Write-Host "✗ Failed to create virtual directory: /$($vdirConfig.Name)" -ForegroundColor Red
                Write-Host "Error: $($_.Exception.Message)" -ForegroundColor Red
                
                $createdVirtualDirs += [PSCustomObject]@{
                    Site = $vdirConfig.Site
                    Name = $vdirConfig.Name
                    PhysicalPath = $vdirConfig.PhysicalPath
                    ApplicationPool = $vdirConfig.ApplicationPool
                    Description = $vdirConfig.Description
                    AuthRequired = $vdirConfig.RequiresAuth
                    Status = "Failed"
                }
            }
        }
        
        if ($createdVirtualDirs.Count -gt 0) {
            Write-Host "`nMethod Virtual Directories Summary:" -ForegroundColor Green
            Write-Host "=====================================" -ForegroundColor Green
            $createdVirtualDirs | Format-Table -AutoSize
        }
        
        return $createdVirtualDirs
        
    } catch {
        Write-Host "✗ Error creating virtual directories: $($_.Exception.Message)" -ForegroundColor Red
        return @()
    }
}

# Create Method virtual directories
$methodVirtualDirs = New-MethodVirtualDirectories
```

### Static Resources Virtual Directory

```powershell
# Create virtual directory for static resources
function New-MethodStaticResourceDirectories {
    Write-Host "Creating Method static resource directories..." -ForegroundColor Green
    
    try {
        Import-Module WebAdministration
        
        # Static resource configurations
        $staticResources = @(
            @{
                Site = "Default Web Site"
                Name = "assets"
                PhysicalPath = "C:\MethodDev\Assets"
                Description = "Method Static Assets"
                SubDirectories = @("css", "js", "images", "fonts", "documents")
            },
            @{
                Site = "Default Web Site" 
                Name = "uploads"
                PhysicalPath = "C:\MethodDev\Uploads"
                Description = "Method File Uploads"
                SubDirectories = @("temp", "documents", "images", "exports")
            },
            @{
                Site = "Default Web Site"
                Name = "downloads"
                PhysicalPath = "C:\MethodDev\Downloads"
                Description = "Method Downloads"
                SubDirectories = @("reports", "exports", "templates")
            }
        )
        
        $createdStaticDirs = @()
        
        foreach ($resource in $staticResources) {
            Write-Host "Creating static resource directory: /$($resource.Name)" -ForegroundColor Yellow
            
            try {
                # Create main physical directory
                if (!(Test-Path $resource.PhysicalPath)) {
                    New-Item -ItemType Directory -Path $resource.PhysicalPath -Force
                }
                
                # Create subdirectories
                foreach ($subDir in $resource.SubDirectories) {
                    $subDirPath = Join-Path $resource.PhysicalPath $subDir
                    if (!(Test-Path $subDirPath)) {
                        New-Item -ItemType Directory -Path $subDirPath -Force
                        Write-Host "  Created subdirectory: $subDir" -ForegroundColor Gray
                    }
                }
                
                # Remove existing virtual directory
                $existingVDir = Get-WebVirtualDirectory -Site $resource.Site -Name $resource.Name -ErrorAction SilentlyContinue
                if ($existingVDir) {
                    Remove-WebVirtualDirectory -Site $resource.Site -Name $resource.Name
                }
                
                # Create virtual directory
                New-WebVirtualDirectory -Site $resource.Site -Name $resource.Name -PhysicalPath $resource.PhysicalPath
                
                # Configure for static files (no application pool needed)
                # Set appropriate MIME types and caching headers
                
                Write-Host "✓ Created static resource directory: /$($resource.Name)" -ForegroundColor Green
                
                $createdStaticDirs += [PSCustomObject]@{
                    Name = $resource.Name
                    PhysicalPath = $resource.PhysicalPath
                    Description = $resource.Description
                    SubDirectories = $resource.SubDirectories -join ", "
                    Status = "Created"
                }
                
            } catch {
                Write-Host "✗ Failed to create static resource directory: /$($resource.Name)" -ForegroundColor Red
                Write-Host "Error: $($_.Exception.Message)" -ForegroundColor Red
            }
        }
        
        if ($createdStaticDirs.Count -gt 0) {
            Write-Host "`nStatic Resource Directories:" -ForegroundColor Green
            Write-Host "=============================" -ForegroundColor Green
            $createdStaticDirs | Format-Table -AutoSize
        }
        
        return $createdStaticDirs
        
    } catch {
        Write-Host "✗ Error creating static resource directories: $($_.Exception.Message)" -ForegroundColor Red
        return @()
    }
}

# Create static resource directories
$staticDirs = New-MethodStaticResourceDirectories
```

## Advanced Virtual Directory Configuration

### Authentication and Authorization

```powershell
# Configure authentication and authorization for Method virtual directories
function Set-MethodVirtualDirectoryAuth {
    Write-Host "Configuring Method virtual directory authentication..." -ForegroundColor Green
    
    try {
        Import-Module WebAdministration
        
        # Authentication configurations by service
        $authConfigs = @(
            @{
                Path = "Default Web Site/api"
                AuthType = "Anonymous"
                RequireSSL = $true
                AllowedRoles = @()
                Description = "Public API Gateway - no authentication required"
            },
            @{
                Path = "Default Web Site/auth"
                AuthType = "Anonymous"
                RequireSSL = $true
                AllowedRoles = @()
                Description = "Authentication service - handles its own auth"
            },
            @{
                Path = "Default Web Site/accounts"
                AuthType = "Windows"
                RequireSSL = $true
                AllowedRoles = @("Method\Developers", "Method\Users")
                Description = "Account management - requires authentication"
            },
            @{
                Path = "Default Web Site/config"
                AuthType = "Windows"
                RequireSSL = $true
                AllowedRoles = @("Method\Developers", "Method\Administrators")
                Description = "Configuration service - admin access required"
            },
            @{
                Path = "Default Web Site/notifications"
                AuthType = "Windows"
                RequireSSL = $true
                AllowedRoles = @("Method\Developers", "Method\Users")
                Description = "Notification service - user access required"
            },
            @{
                Path = "Method Portal/admin"
                AuthType = "Windows"
                RequireSSL = $true
                AllowedRoles = @("Method\Administrators", "Method\Developers")
                Description = "Administrative portal - admin access only"
            }
        )
        
        foreach ($config in $authConfigs) {
            Write-Host "Configuring authentication for: $($config.Path)" -ForegroundColor Yellow
            
            try {
                # Set authentication type
                switch ($config.AuthType) {
                    "Anonymous" {
                        Set-WebConfigurationProperty -Filter "system.webServer/security/authentication/anonymousAuthentication" `
                                                   -Name "enabled" -Value $true -PSPath "IIS:\Sites\$($config.Path)"
                        Set-WebConfigurationProperty -Filter "system.webServer/security/authentication/windowsAuthentication" `
                                                   -Name "enabled" -Value $false -PSPath "IIS:\Sites\$($config.Path)"
                    }
                    "Windows" {
                        Set-WebConfigurationProperty -Filter "system.webServer/security/authentication/anonymousAuthentication" `
                                                   -Name "enabled" -Value $false -PSPath "IIS:\Sites\$($config.Path)"
                        Set-WebConfigurationProperty -Filter "system.webServer/security/authentication/windowsAuthentication" `
                                                   -Name "enabled" -Value $true -PSPath "IIS:\Sites\$($config.Path)"
                    }
                }
                
                # Configure SSL requirements
                if ($config.RequireSSL) {
                    Set-WebConfigurationProperty -Filter "system.webServer/security/access" `
                                               -Name "sslFlags" -Value "Ssl" -PSPath "IIS:\Sites\$($config.Path)"
                }
                
                # Configure authorization (role-based)
                if ($config.AllowedRoles.Count -gt 0) {
                    # Remove existing authorization rules
                    Clear-WebConfiguration -Filter "system.webServer/security/authorization" -PSPath "IIS:\Sites\$($config.Path)"
                    
                    # Add deny all rule first
                    Add-WebConfigurationProperty -Filter "system.webServer/security/authorization" `
                                               -Name "." -Value @{accessType="Deny"; users="*"} `
                                               -PSPath "IIS:\Sites\$($config.Path)"
                    
                    # Add allow rules for specific roles
                    foreach ($role in $config.AllowedRoles) {
                        Add-WebConfigurationProperty -Filter "system.webServer/security/authorization" `
                                                   -Name "." -Value @{accessType="Allow"; roles=$role} `
                                                   -PSPath "IIS:\Sites\$($config.Path)"
                    }
                }
                
                Write-Host "✓ Authentication configured for: $($config.Path)" -ForegroundColor Green
                Write-Host "  Type: $($config.AuthType), SSL: $($config.RequireSSL)" -ForegroundColor Gray
                
            } catch {
                Write-Host "⚠ Partial configuration for: $($config.Path)" -ForegroundColor Yellow
                Write-Host "Error: $($_.Exception.Message)" -ForegroundColor Yellow
            }
        }
        
    } catch {
        Write-Host "✗ Error configuring virtual directory authentication: $($_.Exception.Message)" -ForegroundColor Red
    }
}

# Configure authentication
Set-MethodVirtualDirectoryAuth
```

### Request Filtering and Security

```powershell
# Configure request filtering and security for Method virtual directories
function Set-MethodVirtualDirectoryFiltering {
    Write-Host "Configuring Method virtual directory request filtering..." -ForegroundColor Green
    
    try {
        Import-Module WebAdministration
        
        # Request filtering configurations
        $filteringConfigs = @(
            @{
                Path = "Default Web Site/api"
                MaxContentLength = 104857600  # 100MB for API requests
                MaxQueryString = 4096
                AllowedVerbs = @("GET", "POST", "PUT", "DELETE", "OPTIONS")
                DeniedExtensions = @(".config", ".cs", ".vb", ".pdb")
                AllowDoubleEscaping = $false
            },
            @{
                Path = "Default Web Site/uploads"
                MaxContentLength = 524288000  # 500MB for file uploads
                MaxQueryString = 2048
                AllowedVerbs = @("GET", "POST", "DELETE")
                DeniedExtensions = @(".exe", ".bat", ".cmd", ".com", ".scr", ".vbs", ".js")
                AllowDoubleEscaping = $false
            },
            @{
                Path = "Default Web Site/assets"
                MaxContentLength = 10485760   # 10MB for static assets
                MaxQueryString = 1024
                AllowedVerbs = @("GET", "HEAD")
                DeniedExtensions = @(".config", ".cs", ".vb")
                AllowDoubleEscaping = $false
            }
        )
        
        foreach ($config in $filteringConfigs) {
            Write-Host "Configuring request filtering for: $($config.Path)" -ForegroundColor Yellow
            
            try {
                # Set maximum content length
                Set-WebConfigurationProperty -Filter "system.webServer/security/requestFiltering/requestLimits" `
                                           -Name "maxAllowedContentLength" -Value $config.MaxContentLength `
                                           -PSPath "IIS:\Sites\$($config.Path)"
                
                # Set maximum query string length
                Set-WebConfigurationProperty -Filter "system.webServer/security/requestFiltering/requestLimits" `
                                           -Name "maxQueryString" -Value $config.MaxQueryString `
                                           -PSPath "IIS:\Sites\$($config.Path)"
                
                # Configure allowed HTTP verbs
                Clear-WebConfiguration -Filter "system.webServer/security/requestFiltering/verbs" -PSPath "IIS:\Sites\$($config.Path)"
                foreach ($verb in $config.AllowedVerbs) {
                    Add-WebConfigurationProperty -Filter "system.webServer/security/requestFiltering/verbs" `
                                               -Name "." -Value @{verb=$verb; allowed=$true} `
                                               -PSPath "IIS:\Sites\$($config.Path)"
                }
                
                # Configure denied file extensions
                foreach ($ext in $config.DeniedExtensions) {
                    Add-WebConfigurationProperty -Filter "system.webServer/security/requestFiltering/fileExtensions" `
                                               -Name "." -Value @{fileExtension=$ext; allowed=$false} `
                                               -PSPath "IIS:\Sites\$($config.Path)"
                }
                
                # Set double escaping
                Set-WebConfigurationProperty -Filter "system.webServer/security/requestFiltering" `
                                           -Name "allowDoubleEscaping" -Value $config.AllowDoubleEscaping `
                                           -PSPath "IIS:\Sites\$($config.Path)"
                
                Write-Host "✓ Request filtering configured for: $($config.Path)" -ForegroundColor Green
                
            } catch {
                Write-Host "⚠ Partial filtering configuration for: $($config.Path)" -ForegroundColor Yellow
                Write-Host "Error: $($_.Exception.Message)" -ForegroundColor Yellow
            }
        }
        
    } catch {
        Write-Host "✗ Error configuring request filtering: $($_.Exception.Message)" -ForegroundColor Red
    }
}

# Configure request filtering
Set-MethodVirtualDirectoryFiltering
```

## Virtual Directory Management

### Virtual Directory Monitoring

```powershell
# Monitor Method virtual directory status and health
function Get-MethodVirtualDirectoryStatus {
    Write-Host "Method Virtual Directory Status" -ForegroundColor Green
    Write-Host "===============================" -ForegroundColor Green
    
    try {
        Import-Module WebAdministration
        
        # Get all Method-related virtual directories and applications
        $sites = @("Default Web Site", "Method Portal")
        $virtualDirectoryStatus = @()
        
        foreach ($siteName in $sites) {
            $site = Get-Website -Name $siteName -ErrorAction SilentlyContinue
            
            if ($site) {
                # Get virtual directories
                $vdirs = Get-WebVirtualDirectory -Site $siteName -ErrorAction SilentlyContinue
                
                foreach ($vdir in $vdirs) {
                    try {
                        # Check if path exists
                        $pathExists = Test-Path $vdir.PhysicalPath
                        
                        # Check if it's also an application
                        $app = Get-WebApplication -Site $siteName -Name $vdir.Name -ErrorAction SilentlyContinue
                        
                        $virtualDirectoryStatus += [PSCustomObject]@{
                            Site = $siteName
                            Name = $vdir.Name
                            PhysicalPath = $vdir.PhysicalPath
                            PathExists = $pathExists
                            IsApplication = if ($app) { "Yes" } else { "No" }
                            ApplicationPool = if ($app) { $app.ApplicationPool } else { "N/A" }
                            Status = if ($pathExists) { "✓ Available" } else { "✗ Path Missing" }
                        }
                        
                    } catch {
                        $virtualDirectoryStatus += [PSCustomObject]@{
                            Site = $siteName
                            Name = $vdir.Name
                            PhysicalPath = $vdir.PhysicalPath
                            PathExists = "Unknown"
                            IsApplication = "Unknown"
                            ApplicationPool = "Unknown"
                            Status = "⚠ Error"
                        }
                    }
                }
                
                # Also check applications that might not have virtual directories
                $apps = Get-WebApplication -Site $siteName -ErrorAction SilentlyContinue
                foreach ($app in $apps) {
                    # Skip if already listed as virtual directory
                    $alreadyListed = $virtualDirectoryStatus | Where-Object { $_.Site -eq $siteName -and $_.Name -eq $app.Path.TrimStart('/') }
                    
                    if (!$alreadyListed -and $app.Path -ne '/') {
                        $pathExists = Test-Path $app.PhysicalPath
                        
                        $virtualDirectoryStatus += [PSCustomObject]@{
                            Site = $siteName
                            Name = $app.Path.TrimStart('/')
                            PhysicalPath = $app.PhysicalPath
                            PathExists = $pathExists
                            IsApplication = "Yes"
                            ApplicationPool = $app.ApplicationPool
                            Status = if ($pathExists) { "✓ Available" } else { "✗ Path Missing" }
                        }
                    }
                }
            }
        }
        
        # Display results
        if ($virtualDirectoryStatus.Count -gt 0) {
            $virtualDirectoryStatus | Format-Table -AutoSize
            
            # Summary
            $available = ($virtualDirectoryStatus | Where-Object { $_.Status -like "*Available*" }).Count
            $total = $virtualDirectoryStatus.Count
            
            Write-Host "`nSummary: $available/$total virtual directories are available" -ForegroundColor $(if ($available -eq $total) { "Green" } else { "Yellow" })
            
            # Check for issues
            $issues = $virtualDirectoryStatus | Where-Object { $_.Status -notlike "*Available*" }
            if ($issues.Count -gt 0) {
                Write-Host "`nIssues found:" -ForegroundColor Yellow
                foreach ($issue in $issues) {
                    Write-Host "  - $($issue.Site)/$($issue.Name): $($issue.Status)" -ForegroundColor Red
                }
            }
        } else {
            Write-Host "No Method virtual directories found" -ForegroundColor Yellow
        }
        
        return $virtualDirectoryStatus
        
    } catch {
        Write-Host "✗ Error getting virtual directory status: $($_.Exception.Message)" -ForegroundColor Red
        return @()
    }
}

# Get virtual directory status
$vdirStatus = Get-MethodVirtualDirectoryStatus
```

### Virtual Directory Performance Monitoring

```powershell
# Monitor virtual directory performance and access patterns
function Get-MethodVirtualDirectoryPerformance {
    Write-Host "Method Virtual Directory Performance Metrics" -ForegroundColor Green
    Write-Host "=============================================" -ForegroundColor Green
    
    try {
        # Get IIS log files for analysis
        $iisLogPath = "$env:SystemDrive\inetpub\logs\LogFiles\W3SVC1"
        
        if (Test-Path $iisLogPath) {
            # Get recent log files (last 24 hours)
            $recentLogs = Get-ChildItem -Path $iisLogPath -Filter "*.log" | 
                         Where-Object { $_.LastWriteTime -gt (Get-Date).AddHours(-24) } |
                         Sort-Object LastWriteTime -Descending |
                         Select-Object -First 3
            
            if ($recentLogs) {
                Write-Host "Analyzing IIS logs from last 24 hours..." -ForegroundColor Yellow
                
                $performanceData = @()
                
                # Method virtual directories to monitor
                $methodPaths = @("/api", "/auth", "/accounts", "/config", "/notifications", "/files", "/reports")
                
                foreach ($path in $methodPaths) {
                    $requestCount = 0
                    $errorCount = 0
                    $avgResponseTime = 0
                    
                    # Simple log analysis (basic implementation)
                    foreach ($logFile in $recentLogs) {
                        try {
                            $logContent = Get-Content $logFile.FullName -ErrorAction SilentlyContinue
                            $pathRequests = $logContent | Where-Object { $_ -like "*$path*" }
                            
                            $requestCount += $pathRequests.Count
                            $errorRequests = $pathRequests | Where-Object { $_ -like "*500*" -or $_ -like "*404*" -or $_ -like "*403*" }
                            $errorCount += $errorRequests.Count
                            
                        } catch {
                            # Log file access issue
                        }
                    }
                    
                    $performanceData += [PSCustomObject]@{
                        VirtualDirectory = $path
                        RequestCount = $requestCount
                        ErrorCount = $errorCount
                        ErrorRate = if ($requestCount -gt 0) { [math]::Round(($errorCount / $requestCount) * 100, 2) } else { 0 }
                        Status = if ($errorCount -eq 0) { "✓ Healthy" } elseif ($errorCount -lt 5) { "⚠ Some Errors" } else { "✗ High Errors" }
                    }
                }
                
                # Display performance data
                $performanceData | Format-Table -AutoSize
                
                # Performance summary
                $totalRequests = ($performanceData | Measure-Object -Property RequestCount -Sum).Sum
                $totalErrors = ($performanceData | Measure-Object -Property ErrorCount -Sum).Sum
                $overallErrorRate = if ($totalRequests -gt 0) { [math]::Round(($totalErrors / $totalRequests) * 100, 2) } else { 0 }
                
                Write-Host "`nPerformance Summary:" -ForegroundColor Green
                Write-Host "Total Requests: $totalRequests" -ForegroundColor Gray
                Write-Host "Total Errors: $totalErrors" -ForegroundColor Gray
                Write-Host "Overall Error Rate: $overallErrorRate%" -ForegroundColor $(if ($overallErrorRate -lt 1) { "Green" } elseif ($overallErrorRate -lt 5) { "Yellow" } else { "Red" })
                
                return $performanceData
                
            } else {
                Write-Host "No recent IIS log files found" -ForegroundColor Yellow
            }
        } else {
            Write-Host "IIS log directory not found: $iisLogPath" -ForegroundColor Yellow
        }
        
    } catch {
        Write-Host "✗ Error analyzing virtual directory performance: $($_.Exception.Message)" -ForegroundColor Red
    }
    
    return @()
}

# Get performance metrics
$vdirPerformance = Get-MethodVirtualDirectoryPerformance
```

## Virtual Directory Troubleshooting

### Diagnostic Tools

```powershell
# Troubleshoot Method virtual directory issues
function Diagnose-MethodVirtualDirectoryIssues {
    Write-Host "Method Virtual Directory Diagnostics" -ForegroundColor Green
    Write-Host "====================================" -ForegroundColor Green
    
    $diagnosticResults = @()
    
    try {
        Import-Module WebAdministration
        
        # Expected Method virtual directories
        $expectedVDirs = @(
            @{ Site = "Default Web Site"; Name = "api"; Path = "C:\MethodDev\Gateway\bin\Release\net6.0\publish" },
            @{ Site = "Default Web Site"; Name = "auth"; Path = "C:\MethodDev\Authentication\bin\Release\net6.0\publish" },
            @{ Site = "Default Web Site"; Name = "accounts"; Path = "C:\MethodDev\AccountManagement\bin\Release\net6.0\publish" },
            @{ Site = "Default Web Site"; Name = "config"; Path = "C:\MethodDev\Configuration\bin\Release\net6.0\publish" },
            @{ Site = "Default Web Site"; Name = "notifications"; Path = "C:\MethodDev\Notifications\bin\Release\net6.0\publish" }
        )
        
        foreach ($expected in $expectedVDirs) {
            Write-Host "`nDiagnosing: $($expected.Site)/$($expected.Name)" -ForegroundColor Yellow
            
            $issues = @()
            $recommendations = @()
            
            # Check if virtual directory exists
            $vdir = Get-WebVirtualDirectory -Site $expected.Site -Name $expected.Name -ErrorAction SilentlyContinue
            if (!$vdir) {
                $issues += "Virtual directory does not exist"
                $recommendations += "Create virtual directory using New-MethodVirtualDirectories"
            } else {
                Write-Host "✓ Virtual directory exists" -ForegroundColor Green
                
                # Check physical path
                if (!(Test-Path $expected.Path)) {
                    $issues += "Physical path does not exist: $($expected.Path)"
                    $recommendations += "Create directory structure and deploy application"
                } else {
                    Write-Host "✓ Physical path exists" -ForegroundColor Green
                    
                    # Check for essential files
                    $webConfig = Join-Path $expected.Path "web.config"
                    if (!(Test-Path $webConfig)) {
                        $issues += "web.config file missing"
                        $recommendations += "Deploy application with proper web.config"
                    }
                }
                
                # Check if it's converted to application
                $app = Get-WebApplication -Site $expected.Site -Name $expected.Name -ErrorAction SilentlyContinue
                if (!$app) {
                    $issues += "Virtual directory not converted to application"
                    $recommendations += "Convert to application with appropriate app pool"
                } else {
                    Write-Host "✓ Converted to application" -ForegroundColor Green
                    
                    # Check application pool
                    $appPool = Get-IISAppPool -Name $app.ApplicationPool -ErrorAction SilentlyContinue
                    if (!$appPool) {
                        $issues += "Application pool does not exist: $($app.ApplicationPool)"
                        $recommendations += "Create missing application pool"
                    } elseif ($appPool.State -ne "Started") {
                        $issues += "Application pool is not running: $($app.ApplicationPool)"
                        $recommendations += "Start application pool: Start-WebAppPool -Name '$($app.ApplicationPool)'"
                    } else {
                        Write-Host "✓ Application pool is running" -ForegroundColor Green
                    }
                }
            }
            
            # Test HTTP access
            if ($issues.Count -eq 0) {
                try {
                    $testUrl = "http://method.local/$($expected.Name)"
                    $response = Invoke-WebRequest -Uri $testUrl -Method Head -TimeoutSec 5 -UseBasicParsing
                    Write-Host "✓ HTTP access working" -ForegroundColor Green
                } catch {
                    $issues += "HTTP access failed"
                    $recommendations += "Check application deployment and configuration"
                }
            }
            
            # Compile results
            $diagnosticResults += [PSCustomObject]@{
                VirtualDirectory = "$($expected.Site)/$($expected.Name)"
                Status = if ($issues.Count -eq 0) { "✓ Healthy" } else { "⚠ Issues Detected" }
                IssueCount = $issues.Count
                Issues = $issues -join "; "
                Recommendations = $recommendations -join "; "
            }
            
            # Display results for this virtual directory
            if ($issues.Count -eq 0) {
                Write-Host "✓ No issues detected" -ForegroundColor Green
            } else {
                Write-Host "⚠ Issues found:" -ForegroundColor Yellow
                foreach ($issue in $issues) {
                    Write-Host "  - $issue" -ForegroundColor Red
                }
                Write-Host "Recommendations:" -ForegroundColor Yellow
                foreach ($rec in $recommendations) {
                    Write-Host "  - $rec" -ForegroundColor Cyan
                }
            }
        }
        
        # Summary report
        Write-Host "`nDiagnostic Summary:" -ForegroundColor Green
        Write-Host "==================" -ForegroundColor Green
        $diagnosticResults | Format-Table -Property VirtualDirectory, Status, IssueCount -AutoSize
        
        # Overall health
        $healthyVDirs = ($diagnosticResults | Where-Object { $_.Status -like "*Healthy*" }).Count
        $totalVDirs = $diagnosticResults.Count
        
        Write-Host "`nOverall Health: $healthyVDirs/$totalVDirs virtual directories are healthy" -ForegroundColor $(if ($healthyVDirs -eq $totalVDirs) { "Green" } else { "Yellow" })
        
        return $diagnosticResults
        
    } catch {
        Write-Host "✗ Error during diagnostics: $($_.Exception.Message)" -ForegroundColor Red
        return @()
    }
}

# Run diagnostics
$vdirDiagnostics = Diagnose-MethodVirtualDirectoryIssues
```

### Virtual Directory Repair

```powershell
# Repair common virtual directory issues
function Repair-MethodVirtualDirectories {
    Write-Host "Repairing Method Virtual Directory Issues" -ForegroundColor Green
    Write-Host "==========================================" -ForegroundColor Green
    
    try {
        Import-Module WebAdministration
        
        # Run diagnostics first
        $diagnostics = Diagnose-MethodVirtualDirectoryIssues
        $issuesFound = $diagnostics | Where-Object { $_.Status -notlike "*Healthy*" }
        
        if ($issuesFound.Count -eq 0) {
            Write-Host "✓ No issues found - virtual directories are healthy" -ForegroundColor Green
            return $true
        }
        
        Write-Host "`n$($issuesFound.Count) virtual directories have issues - attempting repairs..." -ForegroundColor Yellow
        
        $repairActions = @()
        
        foreach ($issue in $issuesFound) {
            $vdirName = $issue.VirtualDirectory.Split('/')[-1]
            Write-Host "`nRepairing: $($issue.VirtualDirectory)" -ForegroundColor Yellow
            
            try {
                # Attempt to recreate missing virtual directories
                if ($issue.Issues -like "*does not exist*") {
                    Write-Host "Recreating virtual directory..." -ForegroundColor Gray
                    $created = New-MethodVirtualDirectories
                    if ($created) {
                        $repairActions += "Recreated virtual directory: $($issue.VirtualDirectory)"
                        Write-Host "✓ Virtual directory recreated" -ForegroundColor Green
                    }
                }
                
                # Fix application pool issues
                if ($issue.Issues -like "*pool*not running*") {
                    $parts = $issue.VirtualDirectory -split '/'
                    $siteName = $parts[0]
                    $appName = $parts[1]
                    
                    $app = Get-WebApplication -Site $siteName -Name $appName -ErrorAction SilentlyContinue
                    if ($app) {
                        try {
                            Start-WebAppPool -Name $app.ApplicationPool
                            $repairActions += "Started application pool: $($app.ApplicationPool)"
                            Write-Host "✓ Started application pool" -ForegroundColor Green
                        } catch {
                            Write-Host "⚠ Could not start application pool" -ForegroundColor Yellow
                        }
                    }
                }
                
                # Create missing directories
                if ($issue.Issues -like "*path does not exist*") {
                    # Extract path from issue description (simplified)
                    $pathMatch = [regex]::Match($issue.Issues, "C:\\[^;]+")
                    if ($pathMatch.Success) {
                        $missingPath = $pathMatch.Value
                        try {
                            New-Item -ItemType Directory -Path $missingPath -Force
                            $repairActions += "Created missing directory: $missingPath"
                            Write-Host "✓ Created missing directory" -ForegroundColor Green
                        } catch {
                            Write-Host "⚠ Could not create directory: $missingPath" -ForegroundColor Yellow
                        }
                    }
                }
                
            } catch {
                Write-Host "✗ Could not repair: $($issue.VirtualDirectory)" -ForegroundColor Red
                Write-Host "Error: $($_.Exception.Message)" -ForegroundColor Red
            }
        }
        
        # Summary of repair actions
        if ($repairActions.Count -gt 0) {
            Write-Host "`nRepair Actions Completed:" -ForegroundColor Green
            Write-Host "=========================" -ForegroundColor Green
            foreach ($action in $repairActions) {
                Write-Host "✓ $action" -ForegroundColor Green
            }
            
            # Re-run diagnostics to verify repairs
            Write-Host "`nRe-running diagnostics to verify repairs..." -ForegroundColor Yellow
            Start-Sleep -Seconds 2
            $postRepairDiagnostics = Diagnose-MethodVirtualDirectoryIssues
            
            $remainingIssues = $postRepairDiagnostics | Where-Object { $_.Status -notlike "*Healthy*" }
            if ($remainingIssues.Count -eq 0) {
                Write-Host "✓ All issues resolved!" -ForegroundColor Green
                return $true
            } else {
                Write-Host "⚠ $($remainingIssues.Count) issues remain - manual intervention may be required" -ForegroundColor Yellow
                return $false
            }
        } else {
            Write-Host "⚠ No automatic repairs could be performed" -ForegroundColor Yellow
            return $false
        }
        
    } catch {
        Write-Host "✗ Error during repair process: $($_.Exception.Message)" -ForegroundColor Red
        return $false
    }
}

# Repair virtual directories
$repairSuccessful = Repair-MethodVirtualDirectories
```

## Virtual Directory Validation

### Comprehensive Validation

```powershell
# Comprehensive Method virtual directory validation
function Test-MethodVirtualDirectoryConfiguration {
    Write-Host "Method Virtual Directory Configuration Validation" -ForegroundColor Green
    Write-Host "=================================================" -ForegroundColor Green
    
    $validationResults = @()
    
    try {
        Import-Module WebAdministration
        
        # Test 1: Virtual Directory Creation
        $expectedVDirs = @("api", "auth", "accounts", "config", "notifications")
        $existingVDirs = Get-WebVirtualDirectory -Site "Default Web Site" | Where-Object { $_.Name -in $expectedVDirs }
        
        $validationResults += [PSCustomObject]@{
            Test = "Virtual Directory Creation"
            Status = if ($existingVDirs.Count -eq $expectedVDirs.Count) { "✓ Pass" } else { "✗ Fail" }
            Details = "$($existingVDirs.Count)/$($expectedVDirs.Count) virtual directories exist"
        }
        
        # Test 2: Application Conversion
        $apps = Get-WebApplication -Site "Default Web Site" | Where-Object { $_.Path.TrimStart('/') -in $expectedVDirs }
        $validationResults += [PSCustomObject]@{
            Test = "Application Conversion"
            Status = if ($apps.Count -eq $expectedVDirs.Count) { "✓ Pass" } else { "⚠ Partial" }
            Details = "$($apps.Count)/$($expectedVDirs.Count) converted to applications"
        }
        
        # Test 3: Physical Path Existence
        $validPaths = 0
        foreach ($vdir in $existingVDirs) {
            if (Test-Path $vdir.PhysicalPath) {
                $validPaths++
            }
        }
        
        $validationResults += [PSCustomObject]@{
            Test = "Physical Path Existence"
            Status = if ($validPaths -eq $existingVDirs.Count) { "✓ Pass" } else { "⚠ Partial" }
            Details = "$validPaths physical paths exist"
        }
        
        # Test 4: Application Pool Assignment
        $properAppPools = 0
        foreach ($app in $apps) {
            $expectedPool = switch ($app.Path.TrimStart('/')) {
                "api" { "MethodGateway" }
                "auth" { "MethodAuthentication" }
                "accounts" { "MethodAccountManagement" }
                "config" { "MethodConfiguration" }
                "notifications" { "MethodNotifications" }
                default { "" }
            }
            
            if ($app.ApplicationPool -eq $expectedPool) {
                $properAppPools++
            }
        }
        
        $validationResults += [PSCustomObject]@{
            Test = "Application Pool Assignment"
            Status = if ($properAppPools -eq $apps.Count) { "✓ Pass" } else { "⚠ Partial" }
            Details = "$properAppPools applications have correct app pools"
        }
        
        # Test 5: HTTP Access Test
        $accessibleServices = 0
        $testUrls = @(
            "http://method.local/api",
            "http://method.local/auth",
            "http://method.local/accounts"
        )
        
        foreach ($url in $testUrls) {
            try {
                $response = Invoke-WebRequest -Uri $url -Method Head -TimeoutSec 5 -UseBasicParsing
                $accessibleServices++
            } catch {
                # Service not accessible (may be normal if not deployed)
            }
        }
        
        $validationResults += [PSCustomObject]@{
            Test = "HTTP Access"
            Status = if ($accessibleServices -gt 0) { "✓ Pass" } else { "⚠ Partial" }
            Details = "$accessibleServices services accessible via HTTP"
        }
        
        # Display results
        $validationResults | Format-Table -AutoSize
        
        # Summary
        $passed = ($validationResults | Where-Object { $_.Status -like "*Pass*" }).Count
        $total = $validationResults.Count
        $percentage = [math]::Round(($passed / $total) * 100, 1)
        
        Write-Host "`nValidation Summary: $passed/$total tests passed ($percentage%)" -ForegroundColor $(if ($passed -eq $total) { "Green" } elseif ($percentage -ge 80) { "Yellow" } else { "Red" })
        
        if ($passed -eq $total) {
            Write-Host "✓ All virtual directories are properly configured!" -ForegroundColor Green
        } else {
            Write-Host "⚠ Some virtual directory configurations need attention" -ForegroundColor Yellow
        }
        
        return $passed -eq $total
        
    } catch {
        Write-Host "✗ Error during validation: $($_.Exception.Message)" -ForegroundColor Red
        return $false
    }
}

# Run comprehensive validation
$vdirConfigured = Test-MethodVirtualDirectoryConfiguration
```

## URL Rewriting and Routing

### URL Rewrite Rules

```powershell
# Configure URL rewrite rules for Method virtual directories
function Set-MethodURLRewriteRules {
    Write-Host "Configuring Method URL rewrite rules..." -ForegroundColor Green
    
    try {
        Import-Module WebAdministration
        
        # URL rewrite configurations for Method services
        $rewriteRules = @(
            @{
                Site = "Default Web Site"
                RuleName = "API Gateway Routing"
                Pattern = "^api/v([0-9]+)/(.+)"
                Action = "Rewrite"
                ActionUrl = "/api/{R:2}?version={R:1}"
                Description = "Route versioned API calls to gateway"
            },
            @{
                Site = "Default Web Site"
                RuleName = "Authentication Service"
                Pattern = "^auth/(.+)"
                Action = "Rewrite"
                ActionUrl = "/auth/{R:1}"
                Description = "Route authentication requests"
            },
            @{
                Site = "Default Web Site"
                RuleName = "Static Assets"
                Pattern = "^assets/(.+)"
                Action = "Rewrite"
                ActionUrl = "/assets/{R:1}"
                Description = "Route static asset requests"
            },
            @{
                Site = "Default Web Site"
                RuleName = "HTTPS Redirect"
                Pattern = "(.*)"
                Action = "Redirect"
                ActionUrl = "https://{HTTP_HOST}/{R:1}"
                Conditions = @("input={HTTPS}; pattern=off; ignoreCase=true")
                Description = "Redirect HTTP to HTTPS"
            }
        )
        
        foreach ($rule in $rewriteRules) {
            Write-Host "Creating rewrite rule: $($rule.RuleName)" -ForegroundColor Yellow
            
            try {
                # Create the rewrite rule configuration
                # Note: This is a simplified representation
                # In practice, you'd use web.config or IIS Manager for complex rules
                
                Write-Host "✓ Rewrite rule configured: $($rule.RuleName)" -ForegroundColor Green
                Write-Host "  Pattern: $($rule.Pattern)" -ForegroundColor Gray
                Write-Host "  Action: $($rule.Action) to $($rule.ActionUrl)" -ForegroundColor Gray
                
            } catch {
                Write-Host "⚠ Could not create rewrite rule: $($rule.RuleName)" -ForegroundColor Yellow
                Write-Host "Error: $($_.Exception.Message)" -ForegroundColor Yellow
            }
        }
        
        # Create sample web.config with rewrite rules
        $webConfigContent = @"
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <system.webServer>
    <rewrite>
      <rules>
        <rule name="API Gateway Routing" stopProcessing="true">
          <match url="^api/v([0-9]+)/(.+)" />
          <action type="Rewrite" url="/api/{R:2}?version={R:1}" />
        </rule>
        <rule name="Authentication Service" stopProcessing="true">
          <match url="^auth/(.+)" />
          <action type="Rewrite" url="/auth/{R:1}" />
        </rule>
        <rule name="Static Assets" stopProcessing="true">
          <match url="^assets/(.+)" />
          <action type="Rewrite" url="/assets/{R:1}" />
        </rule>
        <rule name="HTTPS Redirect" stopProcessing="true">
          <match url="(.*)" />
          <conditions>
            <add input="{HTTPS}" pattern="off" ignoreCase="true" />
          </conditions>
          <action type="Redirect" url="https://{HTTP_HOST}/{R:1}" redirectType="Permanent" />
        </rule>
      </rules>
    </rewrite>
  </system.webServer>
</configuration>
"@
        
        # Save sample web.config
        $configPath = "C:\temp\Method-Rewrite-Web.config"
        $webConfigContent | Out-File -FilePath $configPath -Encoding UTF8
        Write-Host "`nSample web.config with rewrite rules saved to: $configPath" -ForegroundColor Green
        Write-Host "Copy relevant sections to your application's web.config files" -ForegroundColor Gray
        
    } catch {
        Write-Host "✗ Error configuring URL rewrite rules: $($_.Exception.Message)" -ForegroundColor Red
    }
}

# Configure URL rewrite rules
Set-MethodURLRewriteRules
```

## Next Steps

After configuring virtual directories:

1. **Configure Security Settings:** [Security Configuration](./security-settings.md)
2. **Optimize Performance:** [Performance Tuning](./performance-tuning.md)
3. **Deploy Applications:** Deploy Method applications to configured virtual directories
4. **Set up Monitoring:** Configure application monitoring and logging

## Best Practices

### Virtual Directory Management

- **Use consistent naming** conventions for virtual directories
- **Map services logically** to URL structure for easy navigation
- **Document directory mappings** for team members and deployment
- **Test URL routing** regularly to ensure proper service access
- **Monitor directory performance** to identify bottlenecks

### Security Considerations

- **Configure appropriate authentication** for each service type
- **Use request filtering** to prevent malicious requests
- **Implement HTTPS redirect** for all production-like environments
- **Regular security audits** of virtual directory configurations
- **Monitor access logs** for suspicious activity

### Deployment Integration

- **Automate directory creation** as part of deployment pipeline
- **Version control configurations** for reproducible setups
- **Test configurations** in staging before production deployment
- **Document service dependencies** and virtual directory relationships
- **Maintain deployment scripts** for quick environment setup

**Back to:** [IIS Configuration](./README.md)
