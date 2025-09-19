# IIS Security Configuration

This guide covers comprehensive IIS security configuration for Method's local development environment to ensure secure operation while maintaining development productivity.

## Overview

IIS security configuration is crucial for Method's local development environment to simulate production security controls while maintaining accessibility for development work. This guide covers authentication, authorization, request filtering, and security hardening.

## Prerequisites

- IIS installed and configured
- Application pools created and configured
- Virtual directories set up and working
- SSL certificates installed and configured
- Administrator access for security configuration
- Understanding of Method's security requirements

## Security Architecture

### Method Security Layers

```
Method IIS Security Architecture
├── Network Security
│   ├── SSL/TLS Encryption (HTTPS)
│   ├── IP Restrictions (if needed)
│   └── Port Configuration
├── Authentication Layer
│   ├── Anonymous (Public APIs)
│   ├── Windows Authentication (Internal services)
│   ├── Forms Authentication (Web applications)
│   └── Bearer Token Authentication (APIs)
├── Authorization Layer
│   ├── Role-based Access Control
│   ├── User-based Permissions
│   └── Application-specific Rules
├── Request Filtering
│   ├── File Extension Filtering
│   ├── HTTP Verb Restrictions
│   ├── Content Length Limits
│   └── Query String Limits
└── Application Security
    ├── Web.config Security Settings
    ├── Custom Headers
    └── Error Page Configuration
```

### Security Zones

Method services are categorized into security zones:

- **Public Zone** - API Gateway, Authentication (Anonymous access)
- **Internal Zone** - Account Management, Configuration (Authenticated access)
- **Admin Zone** - Portal, Administrative functions (Elevated access)
- **Static Zone** - Assets, Downloads (Minimal security)

## Authentication Configuration

### Configure Authentication Methods

```powershell
# Configure IIS authentication methods for Method services
function Set-MethodIISAuthentication {
    Write-Host "Configuring Method IIS authentication methods..." -ForegroundColor Green
    
    try {
        Import-Module WebAdministration -ErrorAction Stop
        
        # Authentication configurations by service zone
        $authenticationConfigs = @(
            @{
                Path = "Default Web Site"
                ServiceType = "Main Site"
                AnonymousAuth = $true
                WindowsAuth = $false
                FormsAuth = $false
                BasicAuth = $false
                Description = "Default site with anonymous access"
            },
            @{
                Path = "Default Web Site/api"
                ServiceType = "API Gateway"
                AnonymousAuth = $true
                WindowsAuth = $false
                FormsAuth = $false
                BasicAuth = $false
                Description = "Public API gateway - anonymous access"
            },
            @{
                Path = "Default Web Site/auth"
                ServiceType = "Authentication Service"
                AnonymousAuth = $true
                WindowsAuth = $false
                FormsAuth = $false
                BasicAuth = $false
                Description = "Authentication service - handles its own auth"
            },
            @{
                Path = "Default Web Site/accounts"
                ServiceType = "Account Management"
                AnonymousAuth = $false
                WindowsAuth = $true
                FormsAuth = $false
                BasicAuth = $false
                Description = "Account management - requires Windows authentication"
            },
            @{
                Path = "Default Web Site/config"
                ServiceType = "Configuration Service"
                AnonymousAuth = $false
                WindowsAuth = $true
                FormsAuth = $false
                BasicAuth = $false
                Description = "Configuration service - Windows authentication required"
            },
            @{
                Path = "Default Web Site/notifications"
                ServiceType = "Notifications Service"
                AnonymousAuth = $false
                WindowsAuth = $true
                FormsAuth = $false
                BasicAuth = $false
                Description = "Notifications - Windows authentication required"
            },
            @{
                Path = "Method Portal"
                ServiceType = "Administrative Portal"
                AnonymousAuth = $false
                WindowsAuth = $true
                FormsAuth = $false
                BasicAuth = $false
                Description = "Administrative portal - Windows authentication required"
            },
            @{
                Path = "Default Web Site/assets"
                ServiceType = "Static Assets"
                AnonymousAuth = $true
                WindowsAuth = $false
                FormsAuth = $false
                BasicAuth = $false
                Description = "Static assets - anonymous access"
            }
        )
        
        $configurationResults = @()
        
        foreach ($config in $authenticationConfigs) {
            Write-Host "Configuring authentication for: $($config.Path)" -ForegroundColor Yellow
            
            try {
                # Check if path exists
                $pathExists = $false
                if ($config.Path.Contains("/")) {
                    # This is a virtual directory or application
                    $pathExists = Get-WebVirtualDirectory -Site ($config.Path.Split("/")[0]) -Name ($config.Path.Split("/")[1]) -ErrorAction SilentlyContinue
                    if (!$pathExists) {
                        $pathExists = Get-WebApplication -Site ($config.Path.Split("/")[0]) -Name ($config.Path.Split("/")[1]) -ErrorAction SilentlyContinue
                    }
                } else {
                    # This is a site
                    $pathExists = Get-Website -Name $config.Path -ErrorAction SilentlyContinue
                }
                
                if (!$pathExists) {
                    Write-Host "⚠ Path does not exist: $($config.Path)" -ForegroundColor Yellow
                    $configurationResults += [PSCustomObject]@{
                        Path = $config.Path
                        ServiceType = $config.ServiceType
                        Status = "⚠ Path Missing"
                        Configuration = "Skipped"
                    }
                    continue
                }
                
                # Configure Anonymous Authentication
                try {
                    Set-WebConfigurationProperty -Filter "system.webServer/security/authentication/anonymousAuthentication" `
                                               -Name "enabled" -Value $config.AnonymousAuth `
                                               -PSPath "IIS:\Sites\$($config.Path)"
                    Write-Host "  Anonymous Auth: $($config.AnonymousAuth)" -ForegroundColor Gray
                } catch {
                    Write-Host "  ⚠ Could not configure Anonymous Auth" -ForegroundColor Yellow
                }
                
                # Configure Windows Authentication
                try {
                    Set-WebConfigurationProperty -Filter "system.webServer/security/authentication/windowsAuthentication" `
                                               -Name "enabled" -Value $config.WindowsAuth `
                                               -PSPath "IIS:\Sites\$($config.Path)"
                    Write-Host "  Windows Auth: $($config.WindowsAuth)" -ForegroundColor Gray
                } catch {
                    Write-Host "  ⚠ Could not configure Windows Auth" -ForegroundColor Yellow
                }
                
                # Configure Forms Authentication (if enabled)
                if ($config.FormsAuth) {
                    try {
                        Set-WebConfigurationProperty -Filter "system.web/authentication" `
                                                   -Name "mode" -Value "Forms" `
                                                   -PSPath "IIS:\Sites\$($config.Path)"
                        Write-Host "  Forms Auth: $($config.FormsAuth)" -ForegroundColor Gray
                    } catch {
                        Write-Host "  ⚠ Could not configure Forms Auth" -ForegroundColor Yellow
                    }
                }
                
                # Configure Basic Authentication
                try {
                    Set-WebConfigurationProperty -Filter "system.webServer/security/authentication/basicAuthentication" `
                                               -Name "enabled" -Value $config.BasicAuth `
                                               -PSPath "IIS:\Sites\$($config.Path)"
                    Write-Host "  Basic Auth: $($config.BasicAuth)" -ForegroundColor Gray
                } catch {
                    Write-Host "  ⚠ Could not configure Basic Auth" -ForegroundColor Yellow
                }
                
                Write-Host "✓ Authentication configured for: $($config.Path)" -ForegroundColor Green
                
                $configurationResults += [PSCustomObject]@{
                    Path = $config.Path
                    ServiceType = $config.ServiceType
                    Status = "✓ Configured"
                    Configuration = "Anonymous:$($config.AnonymousAuth), Windows:$($config.WindowsAuth)"
                }
                
            } catch {
                Write-Host "✗ Failed to configure authentication for: $($config.Path)" -ForegroundColor Red
                Write-Host "Error: $($_.Exception.Message)" -ForegroundColor Red
                
                $configurationResults += [PSCustomObject]@{
                    Path = $config.Path
                    ServiceType = $config.ServiceType
                    Status = "✗ Failed"
                    Configuration = "Error"
                }
            }
        }
        
        # Display results
        Write-Host "`nAuthentication Configuration Results:" -ForegroundColor Green
        Write-Host "=====================================" -ForegroundColor Green
        $configurationResults | Format-Table -AutoSize
        
        return $configurationResults
        
    } catch {
        Write-Host "✗ Error configuring IIS authentication: $($_.Exception.Message)" -ForegroundColor Red
        return @()
    }
}

# Configure Method IIS authentication
$authResults = Set-MethodIISAuthentication
```

### Advanced Authentication Settings

```powershell
# Configure advanced authentication settings
function Set-MethodAdvancedAuthentication {
    Write-Host "Configuring advanced authentication settings..." -ForegroundColor Green
    
    try {
        Import-Module WebAdministration
        
        # Advanced authentication configurations
        $advancedConfigs = @(
            @{
                Path = "Default Web Site/accounts"
                WindowsAuthSettings = @{
                    UseAppPoolCredentials = $false
                    UseKernelMode = $true
                    Providers = @("Negotiate", "NTLM")
                }
                Description = "Account management advanced auth settings"
            },
            @{
                Path = "Default Web Site/config"
                WindowsAuthSettings = @{
                    UseAppPoolCredentials = $false
                    UseKernelMode = $true
                    Providers = @("Negotiate", "NTLM")
                }
                Description = "Configuration service advanced auth settings"
            },
            @{
                Path = "Method Portal"
                WindowsAuthSettings = @{
                    UseAppPoolCredentials = $false
                    UseKernelMode = $true
                    Providers = @("Negotiate", "NTLM")
                }
                Description = "Administrative portal advanced auth settings"
            }
        )
        
        foreach ($config in $advancedConfigs) {
            Write-Host "Configuring advanced auth for: $($config.Path)" -ForegroundColor Yellow
            
            try {
                # Configure Windows Authentication providers
                if ($config.WindowsAuthSettings.Providers) {
                    # Clear existing providers
                    Clear-WebConfiguration -Filter "system.webServer/security/authentication/windowsAuthentication/providers" `
                                         -PSPath "IIS:\Sites\$($config.Path)"
                    
                    # Add configured providers
                    foreach ($provider in $config.WindowsAuthSettings.Providers) {
                        Add-WebConfigurationProperty -Filter "system.webServer/security/authentication/windowsAuthentication/providers" `
                                                   -Name "." -Value @{value=$provider} `
                                                   -PSPath "IIS:\Sites\$($config.Path)"
                        Write-Host "  Added provider: $provider" -ForegroundColor Gray
                    }
                }
                
                # Configure kernel mode authentication
                Set-WebConfigurationProperty -Filter "system.webServer/security/authentication/windowsAuthentication" `
                                           -Name "useKernelMode" -Value $config.WindowsAuthSettings.UseKernelMode `
                                           -PSPath "IIS:\Sites\$($config.Path)"
                
                # Configure app pool credentials usage
                Set-WebConfigurationProperty -Filter "system.webServer/security/authentication/windowsAuthentication" `
                                           -Name "useAppPoolCredentials" -Value $config.WindowsAuthSettings.UseAppPoolCredentials `
                                           -PSPath "IIS:\Sites\$($config.Path)"
                
                Write-Host "✓ Advanced authentication configured for: $($config.Path)" -ForegroundColor Green
                
            } catch {
                Write-Host "⚠ Partial advanced authentication configuration for: $($config.Path)" -ForegroundColor Yellow
                Write-Host "Error: $($_.Exception.Message)" -ForegroundColor Yellow
            }
        }
        
    } catch {
        Write-Host "✗ Error configuring advanced authentication: $($_.Exception.Message)" -ForegroundColor Red
    }
}

# Configure advanced authentication
Set-MethodAdvancedAuthentication
```

## Authorization Configuration

### Role-Based Authorization

```powershell
# Configure role-based authorization for Method services
function Set-MethodAuthorizationRules {
    Write-Host "Configuring Method authorization rules..." -ForegroundColor Green
    
    try {
        Import-Module WebAdministration
        
        # Authorization rule configurations
        $authorizationConfigs = @(
            @{
                Path = "Default Web Site/accounts"
                Rules = @(
                    @{ Type = "Deny"; Users = "*"; Description = "Deny all by default" },
                    @{ Type = "Allow"; Roles = "Method\Developers"; Description = "Allow Method developers" },
                    @{ Type = "Allow"; Roles = "Method\Users"; Description = "Allow Method users" },
                    @{ Type = "Allow"; Roles = "BUILTIN\Administrators"; Description = "Allow local administrators" }
                )
                Description = "Account management authorization"
            },
            @{
                Path = "Default Web Site/config"
                Rules = @(
                    @{ Type = "Deny"; Users = "*"; Description = "Deny all by default" },
                    @{ Type = "Allow"; Roles = "Method\Developers"; Description = "Allow Method developers" },
                    @{ Type = "Allow"; Roles = "Method\Administrators"; Description = "Allow Method administrators" },
                    @{ Type = "Allow"; Roles = "BUILTIN\Administrators"; Description = "Allow local administrators" }
                )
                Description = "Configuration service authorization - admin access required"
            },
            @{
                Path = "Default Web Site/notifications"
                Rules = @(
                    @{ Type = "Deny"; Users = "*"; Description = "Deny all by default" },
                    @{ Type = "Allow"; Roles = "Method\Developers"; Description = "Allow Method developers" },
                    @{ Type = "Allow"; Roles = "Method\Users"; Description = "Allow Method users" },
                    @{ Type = "Allow"; Roles = "BUILTIN\Administrators"; Description = "Allow local administrators" }
                )
                Description = "Notification service authorization"
            },
            @{
                Path = "Method Portal"
                Rules = @(
                    @{ Type = "Deny"; Users = "*"; Description = "Deny all by default" },
                    @{ Type = "Allow"; Roles = "Method\Administrators"; Description = "Allow Method administrators only" },
                    @{ Type = "Allow"; Roles = "Method\Developers"; Description = "Allow Method developers" },
                    @{ Type = "Allow"; Roles = "BUILTIN\Administrators"; Description = "Allow local administrators" }
                )
                Description = "Administrative portal - admin access only"
            },
            @{
                Path = "Method Portal/admin"
                Rules = @(
                    @{ Type = "Deny"; Users = "*"; Description = "Deny all by default" },
                    @{ Type = "Allow"; Roles = "Method\Administrators"; Description = "Allow Method administrators only" },
                    @{ Type = "Allow"; Roles = "BUILTIN\Administrators"; Description = "Allow local administrators" }
                )
                Description = "Administrative functions - restricted access"
            }
        )
        
        $authorizationResults = @()
        
        foreach ($config in $authorizationConfigs) {
            Write-Host "Configuring authorization for: $($config.Path)" -ForegroundColor Yellow
            
            try {
                # Clear existing authorization rules
                Clear-WebConfiguration -Filter "system.webServer/security/authorization" `
                                     -PSPath "IIS:\Sites\$($config.Path)"
                
                # Add configured authorization rules
                foreach ($rule in $config.Rules) {
                    if ($rule.Type -eq "Allow") {
                        if ($rule.Users) {
                            Add-WebConfigurationProperty -Filter "system.webServer/security/authorization" `
                                                       -Name "." -Value @{accessType="Allow"; users=$rule.Users} `
                                                       -PSPath "IIS:\Sites\$($config.Path)"
                            Write-Host "  Added Allow rule for users: $($rule.Users)" -ForegroundColor Gray
                        }
                        if ($rule.Roles) {
                            Add-WebConfigurationProperty -Filter "system.webServer/security/authorization" `
                                                       -Name "." -Value @{accessType="Allow"; roles=$rule.Roles} `
                                                       -PSPath "IIS:\Sites\$($config.Path)"
                            Write-Host "  Added Allow rule for roles: $($rule.Roles)" -ForegroundColor Gray
                        }
                    } elseif ($rule.Type -eq "Deny") {
                        if ($rule.Users) {
                            Add-WebConfigurationProperty -Filter "system.webServer/security/authorization" `
                                                       -Name "." -Value @{accessType="Deny"; users=$rule.Users} `
                                                       -PSPath "IIS:\Sites\$($config.Path)"
                            Write-Host "  Added Deny rule for users: $($rule.Users)" -ForegroundColor Gray
                        }
                        if ($rule.Roles) {
                            Add-WebConfigurationProperty -Filter "system.webServer/security/authorization" `
                                                       -Name "." -Value @{accessType="Deny"; roles=$rule.Roles} `
                                                       -PSPath "IIS:\Sites\$($config.Path)"
                            Write-Host "  Added Deny rule for roles: $($rule.Roles)" -ForegroundColor Gray
                        }
                    }
                }
                
                Write-Host "✓ Authorization rules configured for: $($config.Path)" -ForegroundColor Green
                
                $authorizationResults += [PSCustomObject]@{
                    Path = $config.Path
                    RulesCount = $config.Rules.Count
                    Status = "✓ Configured"
                    Description = $config.Description
                }
                
            } catch {
                Write-Host "✗ Failed to configure authorization for: $($config.Path)" -ForegroundColor Red
                Write-Host "Error: $($_.Exception.Message)" -ForegroundColor Red
                
                $authorizationResults += [PSCustomObject]@{
                    Path = $config.Path
                    RulesCount = 0
                    Status = "✗ Failed"
                    Description = "Configuration failed"
                }
            }
        }
        
        # Display results
        Write-Host "`nAuthorization Configuration Results:" -ForegroundColor Green
        Write-Host "====================================" -ForegroundColor Green
        $authorizationResults | Format-Table -AutoSize
        
        return $authorizationResults
        
    } catch {
        Write-Host "✗ Error configuring authorization rules: $($_.Exception.Message)" -ForegroundColor Red
        return @()
    }
}

# Configure authorization rules
$authzResults = Set-MethodAuthorizationRules
```

### IP and Domain Restrictions

```powershell
# Configure IP and domain restrictions for Method services
function Set-MethodIPRestrictions {
    Write-Host "Configuring Method IP and domain restrictions..." -ForegroundColor Green
    
    try {
        Import-Module WebAdministration
        
        # IP restriction configurations (for development, typically more permissive)
        $ipRestrictionConfigs = @(
            @{
                Path = "Method Portal/admin"
                DefaultAction = "Deny"
                AllowedIPs = @("127.0.0.1", "::1", "192.168.1.0/24", "10.0.0.0/8")
                DeniedIPs = @()
                Description = "Administrative portal - restrict to local and internal networks"
            },
            @{
                Path = "Default Web Site/config"
                DefaultAction = "Allow"
                AllowedIPs = @()
                DeniedIPs = @("192.168.100.0/24")  # Example blocked subnet
                Description = "Configuration service - block specific subnets if needed"
            }
        )
        
        foreach ($config in $ipRestrictionConfigs) {
            Write-Host "Configuring IP restrictions for: $($config.Path)" -ForegroundColor Yellow
            
            try {
                # Set default action
                Set-WebConfigurationProperty -Filter "system.webServer/security/ipSecurity" `
                                           -Name "allowUnlisted" -Value ($config.DefaultAction -eq "Allow") `
                                           -PSPath "IIS:\Sites\$($config.Path)"
                
                # Clear existing IP restrictions
                Clear-WebConfiguration -Filter "system.webServer/security/ipSecurity" `
                                     -PSPath "IIS:\Sites\$($config.Path)"
                
                # Add allowed IPs
                foreach ($allowedIP in $config.AllowedIPs) {
                    Add-WebConfigurationProperty -Filter "system.webServer/security/ipSecurity" `
                                               -Name "." -Value @{ipAddress=$allowedIP; allowed=$true} `
                                               -PSPath "IIS:\Sites\$($config.Path)"
                    Write-Host "  Added allowed IP: $allowedIP" -ForegroundColor Gray
                }
                
                # Add denied IPs
                foreach ($deniedIP in $config.DeniedIPs) {
                    Add-WebConfigurationProperty -Filter "system.webServer/security/ipSecurity" `
                                               -Name "." -Value @{ipAddress=$deniedIP; allowed=$false} `
                                               -PSPath "IIS:\Sites\$($config.Path)"
                    Write-Host "  Added denied IP: $deniedIP" -ForegroundColor Gray
                }
                
                Write-Host "✓ IP restrictions configured for: $($config.Path)" -ForegroundColor Green
                
            } catch {
                Write-Host "⚠ Could not configure IP restrictions for: $($config.Path)" -ForegroundColor Yellow
                Write-Host "Error: $($_.Exception.Message)" -ForegroundColor Yellow
            }
        }
        
        # Note about development IP restrictions
        Write-Host "`nDevelopment IP Restrictions Notes:" -ForegroundColor Yellow
        Write-Host "==================================" -ForegroundColor Yellow
        Write-Host "• IP restrictions are typically minimal in development" -ForegroundColor Gray
        Write-Host "• Consider more restrictive settings for production-like environments" -ForegroundColor Gray
        Write-Host "• Always allow localhost (127.0.0.1) and local IPv6 (::1)" -ForegroundColor Gray
        Write-Host "• Update restrictions based on your network topology" -ForegroundColor Gray
        
    } catch {
        Write-Host "✗ Error configuring IP restrictions: $($_.Exception.Message)" -ForegroundColor Red
    }
}

# Configure IP restrictions (optional for development)
# Set-MethodIPRestrictions
```

## Request Filtering Security

### Advanced Request Filtering

```powershell
# Configure advanced request filtering for Method services
function Set-MethodRequestFiltering {
    Write-Host "Configuring Method advanced request filtering..." -ForegroundColor Green
    
    try {
        Import-Module WebAdministration
        
        # Request filtering configurations by service type
        $filteringConfigs = @(
            @{
                Path = "Default Web Site/api"
                MaxContentLength = 104857600  # 100MB
                MaxQueryString = 4096
                MaxUrl = 4096
                AllowedVerbs = @("GET", "POST", "PUT", "DELETE", "OPTIONS", "HEAD")
                DeniedExtensions = @(".config", ".cs", ".vb", ".pdb", ".dll")
                AllowedExtensions = @()  # Empty means allow all not explicitly denied
                DenyUrlSequences = @("..", "~", "%", "&lt;", "&gt;")
                AllowDoubleEscaping = $false
                AllowHighBitCharacters = $true
                Description = "API Gateway - secure but flexible filtering"
            },
            @{
                Path = "Default Web Site/auth"
                MaxContentLength = 52428800   # 50MB
                MaxQueryString = 2048
                MaxUrl = 2048
                AllowedVerbs = @("GET", "POST", "OPTIONS", "HEAD")
                DeniedExtensions = @(".config", ".cs", ".vb", ".pdb", ".dll", ".exe", ".bat")
                AllowedExtensions = @()
                DenyUrlSequences = @("..", "~", "%", "&lt;", "&gt;", "javascript:")
                AllowDoubleEscaping = $false
                AllowHighBitCharacters = $true
                Description = "Authentication service - moderate security"
            },
            @{
                Path = "Default Web Site/uploads"
                MaxContentLength = 524288000  # 500MB for file uploads
                MaxQueryString = 2048
                MaxUrl = 2048
                AllowedVerbs = @("GET", "POST", "DELETE")
                DeniedExtensions = @(".exe", ".bat", ".cmd", ".com", ".scr", ".vbs", ".js", ".jar", ".app", ".deb", ".pkg", ".dmg")
                AllowedExtensions = @(".pdf", ".doc", ".docx", ".xls", ".xlsx", ".ppt", ".pptx", ".txt", ".jpg", ".jpeg", ".png", ".gif", ".zip")
                DenyUrlSequences = @("..", "~", "%", "&lt;", "&gt;", "javascript:")
                AllowDoubleEscaping = $false
                AllowHighBitCharacters = $true
                Description = "File uploads - strict executable filtering"
            },
            @{
                Path = "Default Web Site/assets"
                MaxContentLength = 10485760   # 10MB
                MaxQueryString = 1024
                MaxUrl = 2048
                AllowedVerbs = @("GET", "HEAD")
                DeniedExtensions = @(".config", ".cs", ".vb", ".pdb", ".dll", ".exe", ".bat")
                AllowedExtensions = @(".css", ".js", ".png", ".jpg", ".jpeg", ".gif", ".svg", ".woff", ".woff2", ".ttf", ".eot")
                DenyUrlSequences = @("..", "~", "%", "&lt;", "&gt;")
                AllowDoubleEscaping = $false
                AllowHighBitCharacters = $false
                Description = "Static assets - restrictive filtering"
            },
            @{
                Path = "Method Portal"
                MaxContentLength = 52428800   # 50MB
                MaxQueryString = 2048
                MaxUrl = 2048
                AllowedVerbs = @("GET", "POST", "PUT", "DELETE", "OPTIONS", "HEAD")
                DeniedExtensions = @(".config", ".cs", ".vb", ".pdb", ".dll", ".exe", ".bat", ".cmd")
                AllowedExtensions = @()
                DenyUrlSequences = @("..", "~", "%", "&lt;", "&gt;", "javascript:", "vbscript:")
                AllowDoubleEscaping = $false
                AllowHighBitCharacters = $true
                Description = "Administrative portal - enhanced security"
            }
        )
        
        $filteringResults = @()
        
        foreach ($config in $filteringConfigs) {
            Write-Host "Configuring request filtering for: $($config.Path)" -ForegroundColor Yellow
            
            try {
                # Set request limits
                Set-WebConfigurationProperty -Filter "system.webServer/security/requestFiltering/requestLimits" `
                                           -Name "maxAllowedContentLength" -Value $config.MaxContentLength `
                                           -PSPath "IIS:\Sites\$($config.Path)"
                
                Set-WebConfigurationProperty -Filter "system.webServer/security/requestFiltering/requestLimits" `
                                           -Name "maxQueryString" -Value $config.MaxQueryString `
                                           -PSPath "IIS:\Sites\$($config.Path)"
                
                Set-WebConfigurationProperty -Filter "system.webServer/security/requestFiltering/requestLimits" `
                                           -Name "maxUrl" -Value $config.MaxUrl `
                                           -PSPath "IIS:\Sites\$($config.Path)"
                
                # Configure allowed HTTP verbs
                Clear-WebConfiguration -Filter "system.webServer/security/requestFiltering/verbs" `
                                     -PSPath "IIS:\Sites\$($config.Path)"
                
                foreach ($verb in $config.AllowedVerbs) {
                    Add-WebConfigurationProperty -Filter "system.webServer/security/requestFiltering/verbs" `
                                               -Name "." -Value @{verb=$verb; allowed=$true} `
                                               -PSPath "IIS:\Sites\$($config.Path)"
                }
                
                Write-Host "  Configured HTTP verbs: $($config.AllowedVerbs -join ', ')" -ForegroundColor Gray
                
                # Configure file extensions
                foreach ($ext in $config.DeniedExtensions) {
                    Add-WebConfigurationProperty -Filter "system.webServer/security/requestFiltering/fileExtensions" `
                                               -Name "." -Value @{fileExtension=$ext; allowed=$false} `
                                               -PSPath "IIS:\Sites\$($config.Path)"
                }
                
                if ($config.AllowedExtensions.Count -gt 0) {
                    # If specific extensions are allowed, deny all others first
                    Set-WebConfigurationProperty -Filter "system.webServer/security/requestFiltering/fileExtensions" `
                                               -Name "allowUnlisted" -Value $false `
                                               -PSPath "IIS:\Sites\$($config.Path)"
                    
                    foreach ($ext in $config.AllowedExtensions) {
                        Add-WebConfigurationProperty -Filter "system.webServer/security/requestFiltering/fileExtensions" `
                                                   -Name "." -Value @{fileExtension=$ext; allowed=$true} `
                                                   -PSPath "IIS:\Sites\$($config.Path)"
                    }
                    
                    Write-Host "  Allowed extensions: $($config.AllowedExtensions -join ', ')" -ForegroundColor Gray
                } else {
                    Write-Host "  Denied extensions: $($config.DeniedExtensions -join ', ')" -ForegroundColor Gray
                }
                
                # Configure denied URL sequences
                foreach ($sequence in $config.DenyUrlSequences) {
                    Add-WebConfigurationProperty -Filter "system.webServer/security/requestFiltering/denyUrlSequences" `
                                               -Name "." -Value @{sequence=$sequence} `
                                               -PSPath "IIS:\Sites\$($config.Path)"
                }
                
                # Set double escaping and high bit characters
                Set-WebConfigurationProperty -Filter "system.webServer/security/requestFiltering" `
                                           -Name "allowDoubleEscaping" -Value $config.AllowDoubleEscaping `
                                           -PSPath "IIS:\Sites\$($config.Path)"
                
                Set-WebConfigurationProperty -Filter "system.webServer/security/requestFiltering" `
                                           -Name "allowHighBitCharacters" -Value $config.AllowHighBitCharacters `
                                           -PSPath "IIS:\Sites\$($config.Path)"
                
                Write-Host "✓ Request filtering configured for: $($config.Path)" -ForegroundColor Green
                
                $filteringResults += [PSCustomObject]@{
                    Path = $config.Path
                    MaxContentMB = [math]::Round($config.MaxContentLength / 1MB, 2)
                    AllowedVerbs = $config.AllowedVerbs.Count
                    DeniedExtensions = $config.DeniedExtensions.Count
                    Status = "✓ Configured"
                    Description = $config.Description
                }
                
            } catch {
                Write-Host "✗ Failed to configure request filtering for: $($config.Path)" -ForegroundColor Red
                Write-Host "Error: $($_.Exception.Message)" -ForegroundColor Red
                
                $filteringResults += [PSCustomObject]@{
                    Path = $config.Path
                    MaxContentMB = 0
                    AllowedVerbs = 0
                    DeniedExtensions = 0
                    Status = "✗ Failed"
                    Description = "Configuration failed"
                }
            }
        }
        
        # Display results
        Write-Host "`nRequest Filtering Configuration Results:" -ForegroundColor Green
        Write-Host "========================================" -ForegroundColor Green
        $filteringResults | Format-Table -AutoSize
        
        return $filteringResults
        
    } catch {
        Write-Host "✗ Error configuring request filtering: $($_.Exception.Message)" -ForegroundColor Red
        return @()
    }
}

# Configure request filtering
$filteringResults = Set-MethodRequestFiltering
```

### Security Headers Configuration

```powershell
# Configure security headers for Method applications
function Set-MethodSecurityHeaders {
    Write-Host "Configuring Method security headers..." -ForegroundColor Green
    
    try {
        Import-Module WebAdministration
        
        # Security header configurations
        $securityHeaderConfigs = @(
            @{
                Path = "Default Web Site"
                Headers = @{
                    "X-Frame-Options" = "SAMEORIGIN"
                    "X-Content-Type-Options" = "nosniff"
                    "X-XSS-Protection" = "1; mode=block"
                    "Referrer-Policy" = "strict-origin-when-cross-origin"
                    "Content-Security-Policy" = "default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'; img-src 'self' data:; connect-src 'self'"
                }
                Description = "Main site security headers"
            },
            @{
                Path = "Default Web Site/api"
                Headers = @{
                    "X-Frame-Options" = "DENY"
                    "X-Content-Type-Options" = "nosniff"
                    "X-XSS-Protection" = "1; mode=block"
                    "Access-Control-Allow-Origin" = "*"  # Development setting - restrict in production
                    "Access-Control-Allow-Methods" = "GET, POST, PUT, DELETE, OPTIONS"
                    "Access-Control-Allow-Headers" = "Content-Type, Authorization, X-Requested-With"
                }
                Description = "API Gateway CORS and security headers"
            },
            @{
                Path = "Method Portal"
                Headers = @{
                    "X-Frame-Options" = "DENY"
                    "X-Content-Type-Options" = "nosniff"
                    "X-XSS-Protection" = "1; mode=block"
                    "Strict-Transport-Security" = "max-age=31536000; includeSubDomains"
                    "Referrer-Policy" = "no-referrer"
                    "Content-Security-Policy" = "default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'; img-src 'self'"
                }
                Description = "Administrative portal - strict security headers"
            },
            @{
                Path = "Default Web Site/assets"
                Headers = @{
                    "X-Frame-Options" = "DENY"
                    "X-Content-Type-Options" = "nosniff"
                    "Cache-Control" = "public, max-age=31536000"
                    "Access-Control-Allow-Origin" = "*"
                }
                Description = "Static assets - caching and basic security"
            }
        )
        
        $headerResults = @()
        
        foreach ($config in $securityHeaderConfigs) {
            Write-Host "Configuring security headers for: $($config.Path)" -ForegroundColor Yellow
            
            try {
                foreach ($headerName in $config.Headers.Keys) {
                    $headerValue = $config.Headers[$headerName]
                    
                    try {
                        # Remove existing header if present
                        Remove-WebConfigurationProperty -Filter "system.webServer/httpProtocol/customHeaders" `
                                                      -Name "." -AtElement @{name=$headerName} `
                                                      -PSPath "IIS:\Sites\$($config.Path)" `
                                                      -ErrorAction SilentlyContinue
                        
                        # Add new header
                        Add-WebConfigurationProperty -Filter "system.webServer/httpProtocol/customHeaders" `
                                                   -Name "." -Value @{name=$headerName; value=$headerValue} `
                                                   -PSPath "IIS:\Sites\$($config.Path)"
                        
                        Write-Host "  Added header: $headerName = $headerValue" -ForegroundColor Gray
                        
                    } catch {
                        Write-Host "  ⚠ Could not add header: $headerName" -ForegroundColor Yellow
                    }
                }
                
                Write-Host "✓ Security headers configured for: $($config.Path)" -ForegroundColor Green
                
                $headerResults += [PSCustomObject]@{
                    Path = $config.Path
                    HeadersCount = $config.Headers.Count
                    Status = "✓ Configured"
                    Description = $config.Description
                }
                
            } catch {
                Write-Host "✗ Failed to configure security headers for: $($config.Path)" -ForegroundColor Red
                Write-Host "Error: $($_.Exception.Message)" -ForegroundColor Red
                
                $headerResults += [PSCustomObject]@{
                    Path = $config.Path
                    HeadersCount = 0
                    Status = "✗ Failed"
                    Description = "Configuration failed"
                }
            }
        }
        
        # Display results
        Write-Host "`nSecurity Headers Configuration Results:" -ForegroundColor Green
        Write-Host "=======================================" -ForegroundColor Green
        $headerResults | Format-Table -AutoSize
        
        return $headerResults
        
    } catch {
        Write-Host "✗ Error configuring security headers: $($_.Exception.Message)" -ForegroundColor Red
        return @()
    }
}

# Configure security headers
$headerResults = Set-MethodSecurityHeaders
```

## SSL/TLS Security Configuration

### SSL Security Settings

```powershell
# Configure SSL security settings for Method applications
function Set-MethodSSLSecurity {
    Write-Host "Configuring Method SSL security settings..." -ForegroundColor Green
    
    try {
        Import-Module WebAdministration
        
        # SSL configurations by security zone
        $sslConfigs = @(
            @{
                Path = "Default Web Site"
                RequireSSL = $true
                SslFlags = "Ssl"
                ClientCertificate = "Ignore"
                Description = "Main site SSL settings"
            },
            @{
                Path = "Default Web Site/api"
                RequireSSL = $true
                SslFlags = "Ssl, SslNegotiateCert"
                ClientCertificate = "Accept"
                Description = "API Gateway SSL settings"
            },
            @{
                Path = "Default Web Site/accounts"
                RequireSSL = $true
                SslFlags = "Ssl"
                ClientCertificate = "Ignore"
                Description = "Account management SSL settings"
            },
            @{
                Path = "Method Portal"
                RequireSSL = $true
                SslFlags = "Ssl, SslRequireCert"
                ClientCertificate = "Require"
                Description = "Administrative portal - require client certificates"
            },
            @{
                Path = "Method Portal/admin"
                RequireSSL = $true
                SslFlags = "Ssl, SslRequireCert, Ssl128"
                ClientCertificate = "Require"
                Description = "Administrative functions - strict SSL requirements"
            }
        )
        
        $sslResults = @()
        
        foreach ($config in $sslConfigs) {
            Write-Host "Configuring SSL for: $($config.Path)" -ForegroundColor Yellow
            
            try {
                # Set SSL requirement
                Set-WebConfigurationProperty -Filter "system.webServer/security/access" `
                                           -Name "sslFlags" -Value $config.SslFlags `
                                           -PSPath "IIS:\Sites\$($config.Path)"
                
                Write-Host "  SSL Flags: $($config.SslFlags)" -ForegroundColor Gray
                Write-Host "  Client Certificate: $($config.ClientCertificate)" -ForegroundColor Gray
                
                Write-Host "✓ SSL configured for: $($config.Path)" -ForegroundColor Green
                
                $sslResults += [PSCustomObject]@{
                    Path = $config.Path
                    RequireSSL = $config.RequireSSL
                    SslFlags = $config.SslFlags
                    ClientCert = $config.ClientCertificate
                    Status = "✓ Configured"
                }
                
            } catch {
                Write-Host "⚠ Could not configure SSL for: $($config.Path)" -ForegroundColor Yellow
                Write-Host "Error: $($_.Exception.Message)" -ForegroundColor Yellow
                
                $sslResults += [PSCustomObject]@{
                    Path = $config.Path
                    RequireSSL = $false
                    SslFlags = "None"
                    ClientCert = "None"
                    Status = "✗ Failed"
                }
            }
        }
        
        # Display results
        Write-Host "`nSSL Security Configuration Results:" -ForegroundColor Green
        Write-Host "===================================" -ForegroundColor Green
        $sslResults | Format-Table -AutoSize
        
        # SSL best practices notes
        Write-Host "`nSSL Security Notes:" -ForegroundColor Yellow
        Write-Host "==================" -ForegroundColor Yellow
        Write-Host "• Client certificate requirements are strict for development" -ForegroundColor Gray
        Write-Host "• Consider relaxing client cert requirements for local development" -ForegroundColor Gray
        Write-Host "• Ensure all production-bound services require SSL" -ForegroundColor Gray
        Write-Host "• Test SSL configurations with browsers and API clients" -ForegroundColor Gray
        
        return $sslResults
        
    } catch {
        Write-Host "✗ Error configuring SSL security: $($_.Exception.Message)" -ForegroundColor Red
        return @()
    }
}

# Configure SSL security (adjust based on development needs)
$sslResults = Set-MethodSSLSecurity
```

### TLS Protocol Configuration

```powershell
# Configure TLS protocols and cipher suites
function Set-MethodTLSConfiguration {
    Write-Host "Configuring Method TLS protocols and cipher suites..." -ForegroundColor Green
    
    try {
        # TLS configuration is typically done at the Windows system level
        # This function provides guidance and some basic registry settings
        
        Write-Host "Checking current TLS configuration..." -ForegroundColor Yellow
        
        # Check enabled protocols
        $tlsVersions = @("SSL 2.0", "SSL 3.0", "TLS 1.0", "TLS 1.1", "TLS 1.2", "TLS 1.3")
        $tlsStatus = @()
        
        foreach ($version in $tlsVersions) {
            $serverKey = "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\$version\Server"
            $clientKey = "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\$version\Client"
            
            $serverEnabled = $false
            $clientEnabled = $false
            
            try {
                if (Test-Path $serverKey) {
                    $serverValue = Get-ItemProperty -Path $serverKey -Name "Enabled" -ErrorAction SilentlyContinue
                    $serverEnabled = $serverValue.Enabled -eq 1
                }
                
                if (Test-Path $clientKey) {
                    $clientValue = Get-ItemProperty -Path $clientKey -Name "Enabled" -ErrorAction SilentlyContinue
                    $clientEnabled = $clientValue.Enabled -eq 1
                }
            } catch {
                # Registry key may not exist
            }
            
            $tlsStatus += [PSCustomObject]@{
                Protocol = $version
                ServerEnabled = $serverEnabled
                ClientEnabled = $clientEnabled
                Recommended = ($version -eq "TLS 1.2" -or $version -eq "TLS 1.3")
                SecurityLevel = switch ($version) {
                    "SSL 2.0" { "Insecure" }
                    "SSL 3.0" { "Insecure" }
                    "TLS 1.0" { "Weak" }
                    "TLS 1.1" { "Weak" }
                    "TLS 1.2" { "Secure" }
                    "TLS 1.3" { "Most Secure" }
                    default { "Unknown" }
                }
            }\n        }\n        \n        # Display TLS status\n        Write-Host \"`nTLS Protocol Status:\" -ForegroundColor Green\n        Write-Host \"====================\" -ForegroundColor Green\n        $tlsStatus | Format-Table -AutoSize\n        \n        # TLS recommendations\n        Write-Host \"`nTLS Configuration Recommendations:\" -ForegroundColor Yellow\n        Write-Host \"==================================\" -ForegroundColor Yellow\n        Write-Host \"• Enable TLS 1.2 and TLS 1.3 for maximum security\" -ForegroundColor Gray\n        Write-Host \"• Disable SSL 2.0, SSL 3.0, TLS 1.0, and TLS 1.1\" -ForegroundColor Gray\n        Write-Host \"• Use strong cipher suites (AES-256, ECDHE)\" -ForegroundColor Gray\n        Write-Host \"• Test with SSL Labs or similar tools\" -ForegroundColor Gray\n        Write-Host \"• Consider Perfect Forward Secrecy (PFS)\" -ForegroundColor Gray\n        \n        # Optional: Configure recommended TLS settings (requires admin rights and reboot)\n        $configureTLS = Read-Host \"`nConfigure recommended TLS settings? (This requires administrative rights and system reboot) [y/n]\"\n        \n        if ($configureTLS -eq 'y' -or $configureTLS -eq 'Y') {\n            Write-Host \"Configuring TLS settings...\" -ForegroundColor Yellow\n            Write-Host \"⚠ This will modify system registry and require a reboot\" -ForegroundColor Yellow\n            \n            # This is a simplified example - full TLS hardening requires more comprehensive changes\n            try {\n                # Enable TLS 1.2\n                $tls12ServerPath = \"HKLM:\\SYSTEM\\CurrentControlSet\\Control\\SecurityProviders\\SCHANNEL\\Protocols\\TLS 1.2\\Server\"\n                if (!(Test-Path $tls12ServerPath)) {\n                    New-Item -Path $tls12ServerPath -Force\n                }\n                Set-ItemProperty -Path $tls12ServerPath -Name \"Enabled\" -Value 1 -Type DWord\n                Set-ItemProperty -Path $tls12ServerPath -Name \"DisabledByDefault\" -Value 0 -Type DWord\n                \n                Write-Host \"✓ TLS 1.2 enabled\" -ForegroundColor Green\n                Write-Host \"⚠ System reboot required for changes to take effect\" -ForegroundColor Yellow\n                \n            } catch {\n                Write-Host \"✗ Could not configure TLS settings: $($_.Exception.Message)\" -ForegroundColor Red\n                Write-Host \"Run PowerShell as Administrator to modify TLS settings\" -ForegroundColor Gray\n            }\n        }\n        \n        return $tlsStatus\n        \n    } catch {\n        Write-Host \"✗ Error checking TLS configuration: $($_.Exception.Message)\" -ForegroundColor Red\n        return @()\n    }\n}\n\n# Check TLS configuration\n$tlsStatus = Set-MethodTLSConfiguration\n```\n\n## Security Monitoring and Auditing\n\n### Security Event Logging\n\n```powershell\n# Configure security event logging for Method applications\nfunction Set-MethodSecurityLogging {\n    Write-Host \"Configuring Method security event logging...\" -ForegroundColor Green\n    \n    try {\n        Import-Module WebAdministration\n        \n        # Logging configurations\n        $loggingConfigs = @(\n            @{\n                Path = \"Default Web Site\"\n                LogFormat = \"W3C\"\n                LogFields = @(\n                    \"date\", \"time\", \"s-sitename\", \"s-computername\", \"s-ip\", \"cs-method\", \n                    \"cs-uri-stem\", \"cs-uri-query\", \"s-port\", \"cs-username\", \"c-ip\", \n                    \"cs(User-Agent)\", \"cs(Cookie)\", \"cs(Referer)\", \"sc-status\", \"sc-substatus\", \n                    \"sc-win32-status\", \"sc-bytes\", \"cs-bytes\", \"time-taken\"\n                )\n                LogDirectory = \"C:\\MethodDev\\Logs\\IIS\"\n                Description = \"Main site comprehensive logging\"\n            },\n            @{\n                Path = \"Method Portal\"\n                LogFormat = \"W3C\"\n                LogFields = @(\n                    \"date\", \"time\", \"s-sitename\", \"s-ip\", \"cs-method\", \"cs-uri-stem\", \n                    \"cs-username\", \"c-ip\", \"cs(User-Agent)\", \"sc-status\", \"sc-substatus\", \n                    \"sc-win32-status\", \"time-taken\"\n                )\n                LogDirectory = \"C:\\MethodDev\\Logs\\Portal\"\n                Description = \"Administrative portal security logging\"\n            }\n        )\n        \n        foreach ($config in $loggingConfigs) {\n            Write-Host \"Configuring logging for: $($config.Path)\" -ForegroundColor Yellow\n            \n            try {\n                # Create log directory if it doesn't exist\n                if (!(Test-Path $config.LogDirectory)) {\n                    New-Item -ItemType Directory -Path $config.LogDirectory -Force\n                    Write-Host \"  Created log directory: $($config.LogDirectory)\" -ForegroundColor Gray\n                }\n                \n                # Configure logging format\n                Set-WebConfigurationProperty -Filter \"system.applicationHost/sites/site[@name='$($config.Path)']/logFile\" `\n                                           -Name \"logFormat\" -Value $config.LogFormat\n                \n                # Set log directory\n                Set-WebConfigurationProperty -Filter \"system.applicationHost/sites/site[@name='$($config.Path)']/logFile\" `\n                                           -Name \"directory\" -Value $config.LogDirectory\n                \n                # Configure log fields\n                Set-WebConfigurationProperty -Filter \"system.applicationHost/sites/site[@name='$($config.Path)']/logFile\" `\n                                           -Name \"logExtFileFlags\" -Value ($config.LogFields -join \",\")\n                \n                # Enable logging\n                Set-WebConfigurationProperty -Filter \"system.applicationHost/sites/site[@name='$($config.Path)']/logFile\" `\n                                           -Name \"enabled\" -Value $true\n                \n                Write-Host \"✓ Logging configured for: $($config.Path)\" -ForegroundColor Green\n                Write-Host \"  Log Directory: $($config.LogDirectory)\" -ForegroundColor Gray\n                \n            } catch {\n                Write-Host \"⚠ Could not configure logging for: $($config.Path)\" -ForegroundColor Yellow\n                Write-Host \"Error: $($_.Exception.Message)\" -ForegroundColor Yellow\n            }\n        }\n        \n        # Configure failed request tracing for security events\n        Write-Host \"`nConfiguring Failed Request Tracing...\" -ForegroundColor Yellow\n        \n        try {\n            # Enable failed request tracing at site level\n            Set-WebConfigurationProperty -Filter \"system.webServer/tracing/traceFailedRequests\" `\n                                       -Name \"enabled\" -Value $true `\n                                       -PSPath \"IIS:\\Sites\\Default Web Site\"\n            \n            # Configure trace rules for security-related failures\n            Add-WebConfigurationProperty -Filter \"system.webServer/tracing/traceFailedRequests\" `\n                                       -Name \".\" `\n                                       -Value @{\n                                           path=\"*\"\n                                           statusCodes=\"401-403,500-599\"\n                                           timeTaken=\"00:00:30\"\n                                       } `\n                                       -PSPath \"IIS:\\Sites\\Default Web Site\"\n            \n            Write-Host \"✓ Failed Request Tracing configured\" -ForegroundColor Green\n            \n        } catch {\n            Write-Host \"⚠ Could not configure Failed Request Tracing\" -ForegroundColor Yellow\n        }\n        \n        # Create log analysis script\n        $logAnalysisScript = @\"\n# Method Security Log Analysis Script\n# Generated: $(Get-Date)\n\n# Function to analyze IIS logs for security events\nfunction Analyze-MethodSecurityLogs {\n    param([string]`$LogPath = \"C:\\MethodDev\\Logs\\IIS\")\n    \n    Write-Host \"Analyzing Method security logs...\" -ForegroundColor Green\n    \n    # Get recent log files\n    `$logFiles = Get-ChildItem -Path `$LogPath -Filter \"*.log\" | \n                 Where-Object { `$_.LastWriteTime -gt (Get-Date).AddDays(-7) }\n    \n    if (`$logFiles) {\n        # Analyze for security events\n        `$securityEvents = @()\n        \n        foreach (`$logFile in `$logFiles) {\n            `$content = Get-Content `$logFile.FullName\n            \n            # Look for authentication failures (401)\n            `$authFailures = `$content | Where-Object { `$_ -like \"*401*\" }\n            `$securityEvents += `$authFailures.Count\n            \n            # Look for forbidden access (403)\n            `$forbiddenAccess = `$content | Where-Object { `$_ -like \"*403*\" }\n            `$securityEvents += `$forbiddenAccess.Count\n        }\n        \n        Write-Host \"Security events found: `$(`$securityEvents.Count)\" -ForegroundColor Yellow\n    } else {\n        Write-Host \"No recent log files found\" -ForegroundColor Yellow\n    }\n}\n\n# Run security log analysis\nAnalyze-MethodSecurityLogs\n\"@\n        \n        $scriptPath = \"C:\\MethodDev\\Scripts\\Analyze-SecurityLogs.ps1\"\n        if (!(Test-Path \"C:\\MethodDev\\Scripts\")) {\n            New-Item -ItemType Directory -Path \"C:\\MethodDev\\Scripts\" -Force\n        }\n        \n        $logAnalysisScript | Out-File -FilePath $scriptPath -Encoding UTF8\n        Write-Host \"\\n✓ Security log analysis script created: $scriptPath\" -ForegroundColor Green\n        \n    } catch {\n        Write-Host \"✗ Error configuring security logging: $($_.Exception.Message)\" -ForegroundColor Red\n    }\n}\n\n# Configure security logging\nSet-MethodSecurityLogging\n```\n\n## Security Validation and Testing\n\n### Comprehensive Security Validation\n\n```powershell\n# Comprehensive Method security configuration validation\nfunction Test-MethodSecurityConfiguration {\n    Write-Host \"Method Security Configuration Validation\" -ForegroundColor Green\n    Write-Host \"=========================================\" -ForegroundColor Green\n    \n    $validationResults = @()\n    \n    try {\n        Import-Module WebAdministration\n        \n        # Test 1: Authentication Configuration\n        $authConfigured = 0\n        $authPaths = @(\"Default Web Site/accounts\", \"Default Web Site/config\", \"Method Portal\")\n        \n        foreach ($path in $authPaths) {\n            try {\n                $windowsAuth = Get-WebConfigurationProperty -Filter \"system.webServer/security/authentication/windowsAuthentication\" `\n                                                          -Name \"enabled\" -PSPath \"IIS:\\Sites\\$path\" `\n                                                          -ErrorAction SilentlyContinue\n                if ($windowsAuth.Value) {\n                    $authConfigured++\n                }\n            } catch {\n                # Path may not exist or be configured\n            }\n        }\n        \n        $validationResults += [PSCustomObject]@{\n            Test = \"Authentication Configuration\"\n            Status = if ($authConfigured -gt 0) { \"✓ Pass\" } else { \"✗ Fail\" }\n            Details = \"$authConfigured/$($authPaths.Count) paths have authentication configured\"\n        }\n        \n        # Test 2: SSL Requirements\n        $sslConfigured = 0\n        $sslPaths = @(\"Default Web Site\", \"Method Portal\")\n        \n        foreach ($path in $sslPaths) {\n            try {\n                $sslFlags = Get-WebConfigurationProperty -Filter \"system.webServer/security/access\" `\n                                                       -Name \"sslFlags\" -PSPath \"IIS:\\Sites\\$path\" `\n                                                       -ErrorAction SilentlyContinue\n                if ($sslFlags.Value -like \"*Ssl*\") {\n                    $sslConfigured++\n                }\n            } catch {\n                # Site may not exist\n            }\n        }\n        \n        $validationResults += [PSCustomObject]@{\n            Test = \"SSL Configuration\"\n            Status = if ($sslConfigured -eq $sslPaths.Count) { \"✓ Pass\" } else { \"⚠ Partial\" }\n            Details = \"$sslConfigured/$($sslPaths.Count) sites require SSL\"\n        }\n        \n        # Test 3: Request Filtering\n        $filteringConfigured = 0\n        $filteringPaths = @(\"Default Web Site/api\", \"Default Web Site/uploads\", \"Default Web Site/assets\")\n        \n        foreach ($path in $filteringPaths) {\n            try {\n                $maxContent = Get-WebConfigurationProperty -Filter \"system.webServer/security/requestFiltering/requestLimits\" `\n                                                         -Name \"maxAllowedContentLength\" -PSPath \"IIS:\\Sites\\$path\" `\n                                                         -ErrorAction SilentlyContinue\n                if ($maxContent.Value -gt 0) {\n                    $filteringConfigured++\n                }\n            } catch {\n                # Path may not exist\n            }\n        }\n        \n        $validationResults += [PSCustomObject]@{\n            Test = \"Request Filtering\"\n            Status = if ($filteringConfigured -gt 0) { \"✓ Pass\" } else { \"⚠ Partial\" }\n            Details = \"$filteringConfigured paths have request filtering configured\"\n        }\n        \n        # Test 4: Security Headers\n        $headersConfigured = 0\n        $headerPaths = @(\"Default Web Site\", \"Method Portal\")\n        \n        foreach ($path in $headerPaths) {\n            try {\n                $customHeaders = Get-WebConfiguration -Filter \"system.webServer/httpProtocol/customHeaders\" `\n                                                    -PSPath \"IIS:\\Sites\\$path\" `\n                                                    -ErrorAction SilentlyContinue\n                if ($customHeaders.Collection.Count -gt 0) {\n                    $headersConfigured++\n                }\n            } catch {\n                # Site may not exist\n            }\n        }\n        \n        $validationResults += [PSCustomObject]@{\n            Test = \"Security Headers\"\n            Status = if ($headersConfigured -gt 0) { \"✓ Pass\" } else { \"⚠ Partial\" }\n            Details = \"$headersConfigured sites have security headers configured\"\n        }\n        \n        # Test 5: Logging Configuration\n        $loggingEnabled = 0\n        $logPaths = @(\"Default Web Site\", \"Method Portal\")\n        \n        foreach ($path in $logPaths) {\n            try {\n                $logEnabled = Get-WebConfigurationProperty -Filter \"system.applicationHost/sites/site[@name='$path']/logFile\" `\n                                                         -Name \"enabled\" -ErrorAction SilentlyContinue\n                if ($logEnabled.Value) {\n                    $loggingEnabled++\n                }\n            } catch {\n                # Site may not exist\n            }\n        }\n        \n        $validationResults += [PSCustomObject]@{\n            Test = \"Security Logging\"\n            Status = if ($loggingEnabled -eq $logPaths.Count) { \"✓ Pass\" } else { \"⚠ Partial\" }\n            Details = \"$loggingEnabled/$($logPaths.Count) sites have logging enabled\"\n        }\n        \n        # Display results\n        $validationResults | Format-Table -AutoSize\n        \n        # Summary\n        $passed = ($validationResults | Where-Object { $_.Status -like \"*Pass*\" }).Count\n        $total = $validationResults.Count\n        $percentage = [math]::Round(($passed / $total) * 100, 1)\n        \n        Write-Host \"`nSecurity Validation Summary: $passed/$total tests passed ($percentage%)\" -ForegroundColor $(if ($passed -eq $total) { \"Green\" } elseif ($percentage -ge 80) { \"Yellow\" } else { \"Red\" })\n        \n        if ($passed -eq $total) {\n            Write-Host \"✓ All security configurations are properly set!\" -ForegroundColor Green\n        } else {\n            Write-Host \"⚠ Some security configurations need attention\" -ForegroundColor Yellow\n        }\n        \n        return $passed -eq $total\n        \n    } catch {\n        Write-Host \"✗ Error during security validation: $($_.Exception.Message)\" -ForegroundColor Red\n        return $false\n    }\n}\n\n# Run comprehensive security validation\n$securityConfigured = Test-MethodSecurityConfiguration\n```\n\n### Security Penetration Testing\n\n```powershell\n# Basic security testing for Method applications\nfunction Test-MethodSecurityBasics {\n    Write-Host \"Method Security Basic Testing\" -ForegroundColor Green\n    Write-Host \"=============================\" -ForegroundColor Green\n    \n    try {\n        $securityTests = @()\n        \n        # Test URLs for Method services\n        $testUrls = @(\n            @{ Url = \"http://method.local\"; ShouldRedirectHTTPS = $true; Description = \"Main site HTTP to HTTPS redirect\" },\n            @{ Url = \"https://method.local/api\"; ShouldBeAccessible = $true; Description = \"API Gateway accessibility\" },\n            @{ Url = \"https://method.local/accounts\"; RequiresAuth = $true; Description = \"Account management authentication\" },\n            @{ Url = \"https://method.local/config\"; RequiresAuth = $true; Description = \"Configuration service authentication\" },\n            @{ Url = \"https://portal.method.local\"; RequiresAuth = $true; Description = \"Portal authentication\" }\n        )\n        \n        foreach ($test in $testUrls) {\n            Write-Host \"Testing: $($test.Url)\" -ForegroundColor Yellow\n            \n            try {\n                # Test HTTP response\n                $response = Invoke-WebRequest -Uri $test.Url -Method Get -TimeoutSec 10 -UseBasicParsing -MaximumRedirection 0 -ErrorAction SilentlyContinue\n                \n                $testResult = [PSCustomObject]@{\n                    Url = $test.Url\n                    StatusCode = $response.StatusCode\n                    RedirectsToHTTPS = $false\n                    RequiresAuthentication = $false\n                    HasSecurityHeaders = $false\n                    TestResult = \"Unknown\"\n                    Description = $test.Description\n                }\n                \n                # Check for HTTPS redirect\n                if ($test.ShouldRedirectHTTPS -and ($response.StatusCode -eq 301 -or $response.StatusCode -eq 302)) {\n                    $location = $response.Headers.Location\n                    if ($location -and $location.StartsWith(\"https://\")) {\n                        $testResult.RedirectsToHTTPS = $true\n                        $testResult.TestResult = \"✓ Pass - Redirects to HTTPS\"\n                    }\n                }\n                \n                # Check for authentication requirement\n                if ($test.RequiresAuth -and $response.StatusCode -eq 401) {\n                    $testResult.RequiresAuthentication = $true\n                    $testResult.TestResult = \"✓ Pass - Authentication required\"\n                }\n                \n                # Check for successful access\n                if ($test.ShouldBeAccessible -and $response.StatusCode -eq 200) {\n                    $testResult.TestResult = \"✓ Pass - Accessible\"\n                }\n                \n                # Check for security headers\n                $securityHeaders = @(\"X-Frame-Options\", \"X-Content-Type-Options\", \"X-XSS-Protection\")\n                $foundHeaders = 0\n                foreach ($header in $securityHeaders) {\n                    if ($response.Headers.ContainsKey($header)) {\n                        $foundHeaders++\n                    }\n                }\n                $testResult.HasSecurityHeaders = $foundHeaders -gt 0\n                \n                Write-Host \"  Status: $($response.StatusCode) - $($testResult.TestResult)\" -ForegroundColor Gray\n                \n            } catch {\n                $testResult = [PSCustomObject]@{\n                    Url = $test.Url\n                    StatusCode = \"Error\"\n                    RedirectsToHTTPS = $false\n                    RequiresAuthentication = $false\n                    HasSecurityHeaders = $false\n                    TestResult = \"⚠ Error - $($_.Exception.Message.Split('.')[0])\"\n                    Description = $test.Description\n                }\n                \n                Write-Host \"  Error: $($_.Exception.Message.Split('.')[0])\" -ForegroundColor Yellow\n            }\n            \n            $securityTests += $testResult\n        }\n        \n        # Display results\n        Write-Host \"`nSecurity Test Results:\" -ForegroundColor Green\n        Write-Host \"======================\" -ForegroundColor Green\n        $securityTests | Format-Table -Property Url, StatusCode, TestResult, HasSecurityHeaders -AutoSize\n        \n        # Security recommendations\n        Write-Host \"`nSecurity Recommendations:\" -ForegroundColor Yellow\n        Write-Host \"=========================\" -ForegroundColor Yellow\n        Write-Host \"1. Ensure all production services redirect HTTP to HTTPS\" -ForegroundColor Gray\n        Write-Host \"2. Verify authentication is required for sensitive endpoints\" -ForegroundColor Gray\n        Write-Host \"3. Implement comprehensive security headers\" -ForegroundColor Gray\n        Write-Host \"4. Regularly test with security scanning tools\" -ForegroundColor Gray\n        Write-Host \"5. Monitor security logs for suspicious activity\" -ForegroundColor Gray\n        \n        return $securityTests\n        \n    } catch {\n        Write-Host \"✗ Error during security testing: $($_.Exception.Message)\" -ForegroundColor Red\n        return @()\n    }\n}\n\n# Run basic security tests\n$securityTests = Test-MethodSecurityBasics\n```\n\n## Next Steps\n\nAfter configuring IIS security:\n\n1. **Optimize Performance:** [Performance Tuning](./performance-tuning.md)\n2. **Set up GitHub Integration:** [GitHub Setup](../github-setup/README.md)\n3. **Configure Monitoring:** Set up comprehensive security monitoring\n4. **Regular Security Audits:** Schedule periodic security reviews\n\n## Best Practices\n\n### Security Configuration\n\n- **Defense in Depth:** Implement multiple layers of security controls\n- **Principle of Least Privilege:** Grant minimum necessary permissions\n- **Regular Updates:** Keep IIS and Windows updated with security patches\n- **Monitoring and Alerting:** Set up comprehensive security monitoring\n- **Documentation:** Maintain security configuration documentation\n\n### Development vs Production\n\n- **Development Environment:** Balance security with developer productivity\n- **Staging Environment:** Mirror production security settings for testing\n- **Production Environment:** Implement strictest security controls\n- **Regular Testing:** Test security configurations across all environments\n- **Compliance:** Ensure configurations meet organizational security requirements\n\n### Ongoing Security Management\n\n- **Security Audits:** Regular review of security configurations\n- **Vulnerability Assessments:** Periodic security scanning and testing\n- **Incident Response:** Prepare for and respond to security incidents\n- **Training:** Keep development team updated on security best practices\n- **Compliance Monitoring:** Ensure ongoing compliance with security policies\n\n**Back to:** [IIS Configuration](./README.md)\n
