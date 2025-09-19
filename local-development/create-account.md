# Create Your Local Account

This guide covers creating and configuring a local Method account for development and testing.

## Overview

After successfully building and verifying Method's critical projects, you need to create a local user account for development and testing. This account will be used to authenticate and access Method applications during local development.

## Prerequisites

Ensure these steps are completed:

- All [prerequisites](./prerequisites.md) verified and passing
- [Critical projects](./critical-projects.md) built successfully and healthy
- All Method services running and accessible
- Database connectivity confirmed

## Method Account System Overview

### Account Architecture

Method uses a multi-tier account system:

```
Method Account Hierarchy
├── System Accounts (Infrastructure)
├── Administrative Accounts (Admin Users)
├── Developer Accounts (Development Team)
├── Test Accounts (QA and Testing)
└── End User Accounts (Customers)
```

### Local Development Account Types

For local development, you'll typically need:

- **Developer Account** - Primary account for development and testing
- **Admin Account** - Administrative privileges for system configuration
- **Test User Account** - Simulate end-user interactions

## Account Creation Process

### Step 1: Database Preparation

```powershell
# Verify Method databases are ready for account creation
function Test-MethodAccountDatabase {
    Write-Host "Verifying Method account database readiness..." -ForegroundColor Green
    
    try {
        # Connect to Method development database
        $connectionString = "Server=MethodSQL;Database=Method_Development;Integrated Security=true;Connection Timeout=10;"
        $connection = New-Object System.Data.SqlClient.SqlConnection($connectionString)
        $connection.Open()
        
        # Check for required account tables
        $requiredTables = @(
            "Users",
            "UserRoles", 
            "Permissions",
            "UserSessions",
            "UserProfiles",
            "Organizations"
        )
        
        $existingTables = @()
        
        foreach ($table in $requiredTables) {
            $command = $connection.CreateCommand()
            $command.CommandText = "SELECT COUNT(*) FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_NAME = '$table'"
            $tableExists = $command.ExecuteScalar()
            
            if ($tableExists -gt 0) {
                Write-Host "✓ Table '$table' exists" -ForegroundColor Green
                $existingTables += $table
            } else {
                Write-Host "✗ Table '$table' missing" -ForegroundColor Red
            }
        }
        
        $connection.Close()
        
        if ($existingTables.Count -eq $requiredTables.Count) {
            Write-Host "✓ All required account tables are present" -ForegroundColor Green
            return $true
        } else {
            Write-Host "✗ Missing $($requiredTables.Count - $existingTables.Count) required tables" -ForegroundColor Red
            return $false
        }
        
    } catch {
        Write-Host "✗ Database connection failed: $($_.Exception.Message)" -ForegroundColor Red
        return $false
    }
}

# Verify database readiness
$dbReady = Test-MethodAccountDatabase
```

### Step 2: Account Management Service

```powershell
# Verify Account Management Service is running
function Test-AccountManagementService {
    Write-Host "Testing Method Account Management Service..." -ForegroundColor Green
    
    $serviceEndpoints = @(
        "https://accounts.method.local/health",
        "https://accounts.method.local/api/status",
        "https://auth.method.local/health"
    )
    
    $serviceResults = @()
    
    foreach ($endpoint in $serviceEndpoints) {
        try {
            $response = Invoke-WebRequest -Uri $endpoint -Method Get -TimeoutSec 10 -UseBasicParsing
            
            if ($response.StatusCode -eq 200) {
                Write-Host "✓ $endpoint is accessible" -ForegroundColor Green
                $serviceResults += [PSCustomObject]@{
                    Endpoint = $endpoint
                    Status = "Available"
                    StatusCode = $response.StatusCode
                }
            } else {
                Write-Host "⚠ $endpoint returned status $($response.StatusCode)" -ForegroundColor Yellow
                $serviceResults += [PSCustomObject]@{
                    Endpoint = $endpoint  
                    Status = "Warning"
                    StatusCode = $response.StatusCode
                }
            }
        } catch {
            Write-Host "✗ $endpoint is not accessible: $($_.Exception.Message)" -ForegroundColor Red
            $serviceResults += [PSCustomObject]@{
                Endpoint = $endpoint
                Status = "Error" 
                StatusCode = "N/A"
            }
        }
    }
    
    # Display results
    $serviceResults | Format-Table -AutoSize
    
    $available = ($serviceResults | Where-Object { $_.Status -eq "Available" }).Count
    return $available -gt 0
}

# Test account management services
$servicesReady = Test-AccountManagementService
```

### Step 3: Create Developer Account

```powershell
# Create your Method developer account
function New-MethodDeveloperAccount {
    param(
        [Parameter(Mandatory)]
        [string]$FirstName,
        
        [Parameter(Mandatory)]
        [string]$LastName,
        
        [Parameter(Mandatory)]
        [string]$Email,
        
        [Parameter(Mandatory)]
        [string]$Username,
        
        [Parameter(Mandatory)]
        [SecureString]$Password
    )
    
    Write-Host "Creating Method developer account..." -ForegroundColor Green
    
    # Convert secure password for API call
    $plainPassword = [Runtime.InteropServices.Marshal]::PtrToStringAuto(
        [Runtime.InteropServices.Marshal]::SecureStringToBSTR($Password)
    )
    
    # Account data structure
    $accountData = @{
        firstName = $FirstName
        lastName = $LastName
        email = $Email
        username = $Username
        password = $plainPassword
        role = "Developer"
        accountType = "Internal"
        permissions = @(
            "Development.Read",
            "Development.Write", 
            "Testing.Execute",
            "API.Access",
            "Database.Read"
        )
        metadata = @{
            createdBy = "LocalSetup"
            environment = "Development"
            machine = $env:COMPUTERNAME
            createdDate = (Get-Date).ToString("yyyy-MM-dd HH:mm:ss")
        }
    } | ConvertTo-Json -Depth 3
    
    try {
        # Create account via Method Account Management API
        $response = Invoke-RestMethod -Uri "https://accounts.method.local/api/accounts/create" `
                                    -Method Post `
                                    -Body $accountData `
                                    -ContentType "application/json" `
                                    -TimeoutSec 30
        
        if ($response.success) {
            Write-Host "✓ Developer account created successfully" -ForegroundColor Green
            Write-Host "Account ID: $($response.accountId)" -ForegroundColor Gray
            Write-Host "Username: $Username" -ForegroundColor Gray
            Write-Host "Email: $Email" -ForegroundColor Gray
            return $response
        } else {
            Write-Host "✗ Account creation failed: $($response.message)" -ForegroundColor Red
            return $null
        }
        
    } catch {
        Write-Host "✗ Account creation error: $($_.Exception.Message)" -ForegroundColor Red
        
        # Fallback: Direct database creation
        Write-Host "Attempting direct database account creation..." -ForegroundColor Yellow
        return New-MethodAccountDirect -FirstName $FirstName -LastName $LastName -Email $Email -Username $Username -Password $plainPassword
    }
}

# Direct database account creation (fallback)
function New-MethodAccountDirect {
    param(
        [string]$FirstName,
        [string]$LastName, 
        [string]$Email,
        [string]$Username,
        [string]$Password
    )
    
    try {
        $connectionString = "Server=MethodSQL;Database=Method_Development;Integrated Security=true;"
        $connection = New-Object System.Data.SqlClient.SqlConnection($connectionString)
        $connection.Open()
        
        # Hash password (simplified for development)
        $passwordHash = [System.Convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes($Password + "METHOD_SALT"))
        
        # Insert user record
        $command = $connection.CreateCommand()
        $command.CommandText = @"
            INSERT INTO Users (Username, Email, FirstName, LastName, PasswordHash, Role, Status, CreatedDate)
            VALUES (@Username, @Email, @FirstName, @LastName, @PasswordHash, 'Developer', 'Active', GETDATE());
            SELECT SCOPE_IDENTITY();
"@
        $command.Parameters.AddWithValue("@Username", $Username)
        $command.Parameters.AddWithValue("@Email", $Email) 
        $command.Parameters.AddWithValue("@FirstName", $FirstName)
        $command.Parameters.AddWithValue("@LastName", $LastName)
        $command.Parameters.AddWithValue("@PasswordHash", $passwordHash)
        
        $userId = $command.ExecuteScalar()
        
        $connection.Close()
        
        Write-Host "✓ Account created directly in database" -ForegroundColor Green
        Write-Host "User ID: $userId" -ForegroundColor Gray
        
        return @{ accountId = $userId; success = $true }
        
    } catch {
        Write-Host "✗ Direct database creation failed: $($_.Exception.Message)" -ForegroundColor Red
        return $null
    }
}

# Prompt for account details
Write-Host "=== Method Developer Account Creation ===" -ForegroundColor Green
$firstName = Read-Host "Enter your first name"
$lastName = Read-Host "Enter your last name"  
$email = Read-Host "Enter your Method email address"
$username = Read-Host "Enter desired username (or press Enter to use email)"

if ([string]::IsNullOrWhiteSpace($username)) {
    $username = $email
}

$password = Read-Host "Enter password for your account" -AsSecureString

# Create the account
$accountResult = New-MethodDeveloperAccount -FirstName $firstName -LastName $lastName -Email $email -Username $username -Password $password
```

### Step 4: Configure Account Permissions

```powershell
# Configure developer account permissions
function Set-MethodDeveloperPermissions {
    param(
        [Parameter(Mandatory)]
        [string]$AccountId,
        [string[]]$AdditionalPermissions = @()
    )
    
    Write-Host "Configuring developer permissions..." -ForegroundColor Green
    
    # Standard developer permissions
    $standardPermissions = @(
        "Application.Read",
        "Application.Write",
        "API.Gateway.Access", 
        "Database.Development.Read",
        "Database.Development.Write",
        "Logs.Read",
        "Configuration.Read",
        "Testing.Execute",
        "Debug.Attach",
        "Performance.Monitor"
    )
    
    $allPermissions = $standardPermissions + $AdditionalPermissions
    
    try {
        $permissionData = @{
            accountId = $AccountId
            permissions = $allPermissions
            role = "Developer"
            environment = "Development"
        } | ConvertTo-Json
        
        $response = Invoke-RestMethod -Uri "https://accounts.method.local/api/permissions/assign" `
                                    -Method Post `
                                    -Body $permissionData `
                                    -ContentType "application/json"
        
        if ($response.success) {
            Write-Host "✓ Developer permissions configured" -ForegroundColor Green
            Write-Host "Assigned permissions:" -ForegroundColor Gray
            $allPermissions | ForEach-Object { Write-Host "  - $_" -ForegroundColor Gray }
        } else {
            Write-Host "⚠ Permission assignment partially failed" -ForegroundColor Yellow
        }
        
    } catch {
        Write-Host "✗ Permission configuration failed: $($_.Exception.Message)" -ForegroundColor Red
    }
}

# Configure permissions if account was created successfully
if ($accountResult -and $accountResult.success) {
    Set-MethodDeveloperPermissions -AccountId $accountResult.accountId
}
```

### Step 5: Test Account Authentication

```powershell
# Test the newly created account
function Test-MethodAccountAuth {
    param(
        [Parameter(Mandatory)]
        [string]$Username,
        [Parameter(Mandatory)]
        [SecureString]$Password
    )
    
    Write-Host "Testing account authentication..." -ForegroundColor Green
    
    # Convert secure password
    $plainPassword = [Runtime.InteropServices.Marshal]::PtrToStringAuto(
        [Runtime.InteropServices.Marshal]::SecureStringToBSTR($Password)
    )
    
    # Test authentication
    $authData = @{
        username = $Username
        password = $plainPassword
        environment = "Development"
    } | ConvertTo-Json
    
    try {
        $response = Invoke-RestMethod -Uri "https://auth.method.local/api/authenticate" `
                                    -Method Post `
                                    -Body $authData `
                                    -ContentType "application/json"
        
        if ($response.success) {
            Write-Host "✓ Authentication successful" -ForegroundColor Green
            Write-Host "Access Token: $($response.token.Substring(0, 20))..." -ForegroundColor Gray
            Write-Host "Token expires: $($response.expiresAt)" -ForegroundColor Gray
            
            # Test API access with token
            $headers = @{ "Authorization" = "Bearer $($response.token)" }
            
            try {
                $userInfo = Invoke-RestMethod -Uri "https://api.method.local/user/profile" `
                                            -Method Get `
                                            -Headers $headers
                
                Write-Host "✓ API access verified" -ForegroundColor Green
                Write-Host "User profile retrieved: $($userInfo.name)" -ForegroundColor Gray
                
            } catch {
                Write-Host "⚠ Authentication succeeded but API access failed" -ForegroundColor Yellow
            }
            
            return $response.token
            
        } else {
            Write-Host "✗ Authentication failed: $($response.message)" -ForegroundColor Red
            return $null
        }
        
    } catch {
        Write-Host "✗ Authentication error: $($_.Exception.Message)" -ForegroundColor Red
        return $null
    }
}

# Test authentication with created account
$authToken = Test-MethodAccountAuth -Username $username -Password $password
```

## Create Additional Test Accounts

### Test User Account

```powershell
# Create additional test accounts for development scenarios
function New-MethodTestAccounts {
    Write-Host "Creating additional test accounts..." -ForegroundColor Green
    
    $testAccounts = @(
        @{
            Username = "test.user@method.local"
            FirstName = "Test"
            LastName = "User"
            Email = "test.user@method.local"
            Role = "EndUser"
            Permissions = @("Application.Read", "Profile.Manage")
        },
        @{
            Username = "admin.test@method.local"
            FirstName = "Admin"
            LastName = "Test"
            Email = "admin.test@method.local" 
            Role = "Administrator"
            Permissions = @("Application.Admin", "User.Manage", "System.Configure")
        },
        @{
            Username = "api.test@method.local"
            FirstName = "API"
            LastName = "Test"
            Email = "api.test@method.local"
            Role = "ServiceAccount" 
            Permissions = @("API.Full", "Integration.Execute")
        }
    )
    
    $createdAccounts = @()
    
    foreach ($account in $testAccounts) {
        try {
            Write-Host "Creating account: $($account.Username)" -ForegroundColor Yellow
            
            # Use a standard test password for these accounts
            $testPassword = ConvertTo-SecureString "TestPassword123!" -AsPlainText -Force
            
            $result = New-MethodDeveloperAccount -FirstName $account.FirstName `
                                               -LastName $account.LastName `
                                               -Email $account.Email `
                                               -Username $account.Username `
                                               -Password $testPassword
            
            if ($result -and $result.success) {
                $createdAccounts += [PSCustomObject]@{
                    Username = $account.Username
                    Role = $account.Role
                    AccountId = $result.accountId
                    Password = "TestPassword123!"
                }
                
                # Set permissions for this account
                Set-MethodDeveloperPermissions -AccountId $result.accountId -AdditionalPermissions $account.Permissions
                
                Write-Host "✓ $($account.Username) created successfully" -ForegroundColor Green
            } else {
                Write-Host "✗ Failed to create $($account.Username)" -ForegroundColor Red
            }
            
        } catch {
            Write-Host "✗ Error creating $($account.Username): $($_.Exception.Message)" -ForegroundColor Red
        }
    }
    
    if ($createdAccounts.Count -gt 0) {
        Write-Host "`nTest Accounts Summary:" -ForegroundColor Green
        Write-Host "======================" -ForegroundColor Green
        $createdAccounts | Format-Table -AutoSize
        
        # Save account info to file
        $accountsPath = "C:\\temp\\Method-Test-Accounts-$(Get-Date -Format 'yyyyMMdd').txt"
        $accountInfo = @()
        $accountInfo += "Method Development Test Accounts"
        $accountInfo += "Generated: $(Get-Date)"
        $accountInfo += "=" * 40
        $accountInfo += ""
        
        foreach ($acc in $createdAccounts) {
            $accountInfo += "Username: $($acc.Username)"
            $accountInfo += "Role: $($acc.Role)"
            $accountInfo += "Password: $($acc.Password)"
            $accountInfo += "Account ID: $($acc.AccountId)"
            $accountInfo += "-" * 20
        }
        
        $accountInfo | Out-File -FilePath $accountsPath -Encoding UTF8
        Write-Host "Account information saved to: $accountsPath" -ForegroundColor Gray
    }
    
    return $createdAccounts
}

# Create test accounts
$testAccounts = New-MethodTestAccounts
```

## Account Configuration Storage

### Save Account Configuration

```powershell
# Save account configuration for development use
function Save-MethodDevelopmentConfig {
    param(
        [string]$PrimaryUsername,
        [string]$AuthToken,
        [array]$TestAccounts
    )
    
    Write-Host "Saving Method development configuration..." -ForegroundColor Green
    
    $config = @{
        Environment = "Development"
        CreatedDate = Get-Date
        Machine = $env:COMPUTERNAME
        User = $env:USERNAME
        
        PrimaryAccount = @{
            Username = $PrimaryUsername
            AuthToken = $AuthToken
            TokenExpiry = (Get-Date).AddHours(8)  # Typical token expiry
        }
        
        TestAccounts = $TestAccounts
        
        APIEndpoints = @{
            Gateway = "https://api.method.local"
            Authentication = "https://auth.method.local"
            AccountManagement = "https://accounts.method.local"
            Configuration = "https://config.method.local"
        }
        
        DatabaseConnections = @{
            Primary = "Server=MethodSQL;Database=Method_Development;Integrated Security=true;"
            Logs = "Server=MethodSQL;Database=Method_Logs;Integrated Security=true;"
            Cache = "Server=MethodSQL;Database=Method_Cache;Integrated Security=true;"
        }
    }
    
    # Save to JSON file for easy access
    $configPath = "C:\\MethodDev\\local-dev-config.json"
    $config | ConvertTo-Json -Depth 4 | Out-File -FilePath $configPath -Encoding UTF8
    
    # Also save as PowerShell data file
    $psConfigPath = "C:\\MethodDev\\local-dev-config.ps1"
    $psConfig = @"
# Method Local Development Configuration
# Generated: $(Get-Date)

`$MethodDevConfig = @{
    PrimaryAccount = @{
        Username = '$PrimaryUsername'
        AuthToken = '$AuthToken'
    }
    
    APIEndpoints = @{
        Gateway = 'https://api.method.local'
        Auth = 'https://auth.method.local'
        Accounts = 'https://accounts.method.local'
    }
    
    TestAccounts = @(
"@
    
    foreach ($acc in $TestAccounts) {
        $psConfig += @"
        @{ Username = '$($acc.Username)'; Password = '$($acc.Password)'; Role = '$($acc.Role)' },
"@
    }
    
    $psConfig += @"
    )
}

# Export for use in other scripts
Export-ModuleMember -Variable MethodDevConfig
"@
    
    $psConfig | Out-File -FilePath $psConfigPath -Encoding UTF8
    
    Write-Host "✓ Configuration saved to:" -ForegroundColor Green
    Write-Host "  JSON: $configPath" -ForegroundColor Gray
    Write-Host "  PowerShell: $psConfigPath" -ForegroundColor Gray
}

# Save configuration
if ($accountResult -and $accountResult.success) {
    Save-MethodDevelopmentConfig -PrimaryUsername $username -AuthToken $authToken -TestAccounts $testAccounts
}
```

## Method Application Access

### Test Application Access

```powershell
# Test access to Method applications with the new account
function Test-MethodApplicationAccess {
    param(
        [string]$AuthToken
    )
    
    Write-Host "Testing Method application access..." -ForegroundColor Green
    
    if (!$AuthToken) {
        Write-Host "⚠ No authentication token available, skipping application tests" -ForegroundColor Yellow
        return
    }
    
    $headers = @{ "Authorization" = "Bearer $AuthToken" }
    
    $appEndpoints = @(
        @{ Name = "User Profile"; Url = "https://api.method.local/user/profile" },
        @{ Name = "Account Settings"; Url = "https://api.method.local/account/settings" },
        @{ Name = "Method Dashboard"; Url = "https://method.local/dashboard" },
        @{ Name = "Development Tools"; Url = "https://api.method.local/dev/tools" },
        @{ Name = "System Status"; Url = "https://api.method.local/system/status" }
    )
    
    $accessResults = @()
    
    foreach ($endpoint in $appEndpoints) {
        try {
            $response = Invoke-WebRequest -Uri $endpoint.Url -Headers $headers -Method Get -TimeoutSec 10 -UseBasicParsing
            
            if ($response.StatusCode -eq 200) {
                Write-Host "✓ $($endpoint.Name): Accessible" -ForegroundColor Green
                $accessResults += [PSCustomObject]@{
                    Application = $endpoint.Name
                    Status = "Accessible"
                    StatusCode = $response.StatusCode
                    ResponseSize = $response.Content.Length
                }
            } else {
                Write-Host "⚠ $($endpoint.Name): HTTP $($response.StatusCode)" -ForegroundColor Yellow
                $accessResults += [PSCustomObject]@{
                    Application = $endpoint.Name
                    Status = "Warning"
                    StatusCode = $response.StatusCode
                    ResponseSize = 0
                }
            }
            
        } catch {
            Write-Host "✗ $($endpoint.Name): Not accessible" -ForegroundColor Red
            $accessResults += [PSCustomObject]@{
                Application = $endpoint.Name
                Status = "Error"
                StatusCode = "N/A"
                ResponseSize = 0
            }
        }
    }
    
    Write-Host "`nApplication Access Summary:" -ForegroundColor Green
    Write-Host "===========================" -ForegroundColor Green
    $accessResults | Format-Table -AutoSize
    
    $accessible = ($accessResults | Where-Object { $_.Status -eq "Accessible" }).Count
    $total = $accessResults.Count
    
    Write-Host "Access Results: $accessible/$total applications accessible" -ForegroundColor $(if ($accessible -eq $total) { "Green" } else { "Yellow" })
}

# Test application access
Test-MethodApplicationAccess -AuthToken $authToken
```

## Account Validation and Testing

### Comprehensive Account Validation

```powershell
# Perform comprehensive account validation
function Test-MethodAccountSetup {
    Write-Host "Method Account Setup Validation" -ForegroundColor Green
    Write-Host "===============================" -ForegroundColor Green
    
    $validationResults = @()
    
    # Test 1: Database connectivity
    $dbTest = Test-MethodAccountDatabase
    $validationResults += [PSCustomObject]@{
        Test = "Database Connectivity"
        Status = if ($dbTest) { "✓ Pass" } else { "✗ Fail" }
        Details = if ($dbTest) { "All required tables present" } else { "Missing tables or connection failed" }
    }
    
    # Test 2: Account services
    $serviceTest = Test-AccountManagementService
    $validationResults += [PSCustomObject]@{
        Test = "Account Services"
        Status = if ($serviceTest) { "✓ Pass" } else { "✗ Fail" }
        Details = if ($serviceTest) { "Account management services accessible" } else { "Services not accessible" }
    }
    
    # Test 3: Primary account creation
    $accountExists = $accountResult -and $accountResult.success
    $validationResults += [PSCustomObject]@{
        Test = "Primary Account Creation"
        Status = if ($accountExists) { "✓ Pass" } else { "✗ Fail" }
        Details = if ($accountExists) { "Developer account created successfully" } else { "Account creation failed" }
    }
    
    # Test 4: Authentication
    $authWorking = $authToken -ne $null
    $validationResults += [PSCustomObject]@{
        Test = "Authentication"
        Status = if ($authWorking) { "✓ Pass" } else { "✗ Fail" }
        Details = if ($authWorking) { "Account authentication successful" } else { "Authentication failed" }
    }
    
    # Test 5: Test accounts
    $testAccountsCreated = $testAccounts -and $testAccounts.Count -gt 0
    $validationResults += [PSCustomObject]@{
        Test = "Test Accounts"
        Status = if ($testAccountsCreated) { "✓ Pass" } else { "⚠ Partial" }
        Details = if ($testAccountsCreated) { "$($testAccounts.Count) test accounts created" } else { "Test accounts not created" }
    }
    
    # Display results
    $validationResults | Format-Table -AutoSize
    
    # Summary
    $passed = ($validationResults | Where-Object { $_.Status -like "*Pass*" }).Count
    $total = $validationResults.Count
    $percentage = [math]::Round(($passed / $total) * 100, 1)
    
    Write-Host "`nValidation Summary: $passed/$total tests passed ($percentage%)" -ForegroundColor $(if ($passed -eq $total) { "Green" } elseif ($percentage -ge 80) { "Yellow" } else { "Red" })
    
    if ($passed -eq $total) {
        Write-Host "✓ Account setup completed successfully!" -ForegroundColor Green
        Write-Host "You can now proceed with Method local development." -ForegroundColor Green
    } else {
        Write-Host "⚠ Some account setup steps need attention." -ForegroundColor Yellow
        Write-Host "Review the failed tests and retry as needed." -ForegroundColor Yellow
    }
    
    return $passed -eq $total
}

# Run comprehensive validation
$setupComplete = Test-MethodAccountSetup
```

## Account Management Tips

### Daily Development Workflow

```powershell
# Helper functions for daily development workflow
function Get-MethodAuthToken {
    param([string]$Username, [SecureString]$Password)
    
    Write-Host "Authenticating with Method..." -ForegroundColor Yellow
    
    $plainPassword = [Runtime.InteropServices.Marshal]::PtrToStringAuto(
        [Runtime.InteropServices.Marshal]::SecureStringToBSTR($Password)
    )
    
    $authData = @{
        username = $Username
        password = $plainPassword
        environment = "Development"
    } | ConvertTo-Json
    
    try {
        $response = Invoke-RestMethod -Uri "https://auth.method.local/api/authenticate" `
                                    -Method Post `
                                    -Body $authData `
                                    -ContentType "application/json"
        
        if ($response.success) {
            Write-Host "✓ Authentication successful" -ForegroundColor Green
            return $response.token
        } else {
            Write-Host "✗ Authentication failed" -ForegroundColor Red
            return $null
        }
    } catch {
        Write-Host "✗ Authentication error: $($_.Exception.Message)" -ForegroundColor Red
        return $null
    }
}

function Test-MethodToken {
    param([string]$Token)
    
    if (!$Token) {
        Write-Host "No token provided" -ForegroundColor Red
        return $false
    }
    
    try {
        $headers = @{ "Authorization" = "Bearer $Token" }
        $response = Invoke-RestMethod -Uri "https://api.method.local/user/profile" -Headers $headers
        Write-Host "✓ Token is valid" -ForegroundColor Green
        return $true
    } catch {
        Write-Host "✗ Token is invalid or expired" -ForegroundColor Red
        return $false
    }
}

# Quick authentication function for daily use
function Connect-MethodDev {
    Write-Host "Method Development Authentication" -ForegroundColor Green
    
    # Try to load saved config
    $configPath = "C:\\MethodDev\\local-dev-config.json"
    if (Test-Path $configPath) {
        $config = Get-Content $configPath | ConvertFrom-Json
        
        Write-Host "Found saved configuration for: $($config.PrimaryAccount.Username)" -ForegroundColor Gray
        
        # Test if token is still valid
        if (Test-MethodToken -Token $config.PrimaryAccount.AuthToken) {
            Write-Host "✓ Using saved authentication token" -ForegroundColor Green
            return $config.PrimaryAccount.AuthToken
        }
    }
    
    # Request fresh authentication
    $username = Read-Host "Username"
    $password = Read-Host "Password" -AsSecureString
    
    return Get-MethodAuthToken -Username $username -Password $password
}

Write-Host "`nDaily Development Functions Available:" -ForegroundColor Green
Write-Host "- Connect-MethodDev: Quick authentication" -ForegroundColor Gray
Write-Host "- Test-MethodToken: Validate auth token" -ForegroundColor Gray
Write-Host "- Get-MethodAuthToken: Get new auth token" -ForegroundColor Gray
```

## Security Best Practices

### Account Security Guidelines

- **Use strong passwords** for all development accounts
- **Rotate credentials** regularly (monthly recommended)
- **Limit permissions** to only what's needed for development
- **Monitor account usage** through Method's audit logs
- **Never commit credentials** to version control
- **Use environment variables** for sensitive configuration

### Credential Storage

```powershell
# Secure credential storage for development
function Set-MethodDevCredential {
    param(
        [string]$Username,
        [SecureString]$Password
    )
    
    # Store in Windows Credential Manager
    $target = "Method-Development-$Username"
    $plainPassword = [Runtime.InteropServices.Marshal]::PtrToStringAuto(
        [Runtime.InteropServices.Marshal]::SecureStringToBSTR($Password)
    )
    
    cmdkey /generic:$target /user:$Username /pass:$plainPassword
    Write-Host "✓ Credentials stored securely for $Username" -ForegroundColor Green
}

function Get-MethodDevCredential {
    param([string]$Username)
    
    $target = "Method-Development-$Username"
    
    try {
        # Note: This is a simplified example
        # In practice, you'd retrieve from Windows Credential Manager
        Write-Host "Retrieving credentials for $Username..." -ForegroundColor Yellow
        
        # Return credential object
        return @{
            Username = $Username
            SecurePassword = Read-Host "Enter password for $Username" -AsSecureString
        }
    } catch {
        Write-Host "✗ Could not retrieve credentials for $Username" -ForegroundColor Red
        return $null
    }
}

Write-Host "`nCredential Management Functions:" -ForegroundColor Green
Write-Host "- Set-MethodDevCredential: Store credentials securely" -ForegroundColor Gray
Write-Host "- Get-MethodDevCredential: Retrieve stored credentials" -ForegroundColor Gray
```

## Next Steps

After successfully creating your Method account:

1. **Configure SMS Notifications:** [SMS Notifications Setup](./sms-notifications.md) (optional)
2. **Start Development:** Begin working with Method applications
3. **Explore APIs:** Use your authentication token to test Method APIs
4. **Set up IDE Integration:** Configure your development tools with the new account

## Troubleshooting

### Common Account Creation Issues

#### Database Connection Problems
- Verify SQL Server is running and accessible
- Check SQL aliases are configured correctly
- Ensure Windows Authentication is working

#### Service Unavailable
- Confirm all Method services are running
- Check IIS application pools are started
- Verify host file entries for local domains

#### Authentication Failures
- Verify account was created successfully in database
- Check password complexity requirements
- Ensure authentication service is accessible

#### Permission Denied
- Confirm account has appropriate role assignments
- Check permission configuration in database
- Verify API endpoints are accessible

## Maintenance

### Regular Account Maintenance

```powershell
# Weekly account maintenance script
function Start-MethodAccountMaintenance {
    Write-Host "Method Account Maintenance" -ForegroundColor Green
    
    # 1. Test account authentication
    Write-Host "Testing account authentication..." -ForegroundColor Yellow
    $config = Get-Content "C:\\MethodDev\\local-dev-config.json" | ConvertFrom-Json
    $tokenValid = Test-MethodToken -Token $config.PrimaryAccount.AuthToken
    
    if (!$tokenValid) {
        Write-Host "⚠ Authentication token expired, refresh required" -ForegroundColor Yellow
    }
    
    # 2. Check account permissions
    Write-Host "Checking account permissions..." -ForegroundColor Yellow
    # Implementation depends on Method's permission API
    
    # 3. Validate database access
    Write-Host "Validating database access..." -ForegroundColor Yellow
    Test-MethodAccountDatabase
    
    # 4. Clean up expired sessions
    Write-Host "Cleaning up expired sessions..." -ForegroundColor Yellow
    # Implementation depends on session management
    
    Write-Host "✓ Account maintenance completed" -ForegroundColor Green
}

# Run maintenance
# Start-MethodAccountMaintenance
```

**Back to:** [Local Development](./README.md)
