# Active Directory Credentials Storage

This guide covers securely storing and managing Active Directory credentials for Method development tools and services.

## Overview

Method development requires access to various internal services that authenticate against Active Directory. Rather than entering credentials repeatedly, Windows provides secure credential storage mechanisms that integrate with development tools.

## Prerequisites

- Method Active Directory account
- Windows Professional, Enterprise, or Education edition
- Administrator access for credential management
- Valid Method domain credentials

## Windows Credential Manager

### Overview

Windows Credential Manager securely stores credentials for network resources, including:
- **Generic Credentials** - Custom applications and services
- **Windows Credentials** - Domain and network authentication
- **Certificate-Based Credentials** - Smart card and certificate authentication

### Access Credential Manager

#### Through Control Panel
1. **Open Control Panel** → **User Accounts** → **Credential Manager**
2. **Alternative**: Press `Win + R`, type `control keymgr.dll`, press Enter
3. **PowerShell**: `rundll32.exe keymgr.dll,KRShowKeyMgr`

#### Through Windows Settings
1. **Settings** → **Accounts** → **Sign-in options**
2. **Advanced security options** → **Credential Manager**

### Store Method Credentials

#### Add Generic Credentials

```powershell
# Store Method domain credentials using PowerShell
$target = "method.internal"
$username = "domain\username"  # Replace with your Method username
$password = "your-password"    # Replace with your Method password

# Create credential entry
cmdkey /generic:$target /user:$username /pass:$password

# Verify credential is stored
cmdkey /list | Select-String $target
```

#### Manual Entry Through GUI

1. **Open Credential Manager**
2. **Click "Add a generic credential"**
3. **Enter Details:**
   - **Internet or network address:** `method.internal`
   - **User name:** `METHODDOMAIN\yourusername`
   - **Password:** Your Method password
4. **Click "OK"**

### Store Service-Specific Credentials

#### Visual Studio Team Services

```powershell
# Store credentials for Visual Studio Team Services
$vstsTarget = "git:https://dev.azure.com/methodcrm"
$vstsUsername = "your.email@method.com"
$vstsPassword = "your-personal-access-token"

cmdkey /generic:$vstsTarget /user:$vstsUsername /pass:$vstsPassword
```

#### SQL Server Authentication

```powershell
# Store SQL Server credentials (if using SQL authentication)
$sqlTarget = "sql-server.method.internal"
$sqlUsername = "method-dev-user"
$sqlPassword = "your-sql-password"

cmdkey /generic:$sqlTarget /user:$sqlUsername /pass:$sqlPassword
```

#### SharePoint/Office 365

```powershell
# Store Office 365 credentials for SharePoint access
$o365Target = "https://methodcrm.sharepoint.com"
$o365Username = "your.email@method.com"
$o365Password = "your-o365-password"

cmdkey /generic:$o365Target /user:$o365Username /pass:$o365Password
```

## PowerShell Credential Management

### Secure Credential Storage Script

```powershell
# Method Credential Storage Script
Write-Host "Method Active Directory Credential Storage" -ForegroundColor Green

# Function to securely store credentials
function Store-MethodCredential {
    param(
        [string]$Target,
        [string]$Username,
        [string]$Description = ""
    )
    
    Write-Host "Storing credentials for: $Target" -ForegroundColor Yellow
    
    # Prompt for password securely
    $securePassword = Read-Host "Enter password for $Username" -AsSecureString
    $password = [Runtime.InteropServices.Marshal]::PtrToStringAuto(
        [Runtime.InteropServices.Marshal]::SecureStringToBSTR($securePassword)
    )
    
    try {
        # Store credential
        cmdkey /generic:$Target /user:$Username /pass:$password
        Write-Host "✓ Credential stored successfully for $Target" -ForegroundColor Green
        
        # Clear password from memory
        $password = $null
        [System.GC]::Collect()
        
    } catch {
        Write-Host "✗ Failed to store credential for $Target" -ForegroundColor Red
        Write-Host $_.Exception.Message -ForegroundColor Red
    }
}

# Store common Method credentials
$credentials = @(
    @{
        Target = "method.internal"
        Username = "METHODDOMAIN\$env:USERNAME"
        Description = "Method domain authentication"
    },
    @{
        Target = "git:https://dev.azure.com/methodcrm"
        Username = "$env:USERNAME@method.com"
        Description = "Azure DevOps Git authentication"
    },
    @{
        Target = "nuget:https://dev.azure.com/methodcrm"
        Username = "$env:USERNAME@method.com" 
        Description = "NuGet package feed authentication"
    }
)

foreach ($cred in $credentials) {
    $response = Read-Host "Store credential for $($cred.Target)? (y/n)"
    if ($response -eq 'y' -or $response -eq 'Y') {
        Store-MethodCredential -Target $cred.Target -Username $cred.Username -Description $cred.Description
    }
}

Write-Host "`nCredential storage completed!" -ForegroundColor Green
```

### Retrieve Stored Credentials

```powershell
# Function to retrieve stored credentials
function Get-StoredCredential {
    param([string]$Target)
    
    try {
        # Query credential manager
        $output = cmdkey /list:$Target 2>&1
        
        if ($output -match "Target: $Target") {
            Write-Host "✓ Credential found for $Target" -ForegroundColor Green
            return $true
        } else {
            Write-Host "✗ No credential found for $Target" -ForegroundColor Red
            return $false
        }
    } catch {
        Write-Host "✗ Error checking credential for $Target" -ForegroundColor Red
        return $false
    }
}

# Check common Method credentials
$targets = @(
    "method.internal",
    "git:https://dev.azure.com/methodcrm",
    "nuget:https://dev.azure.com/methodcrm"
)

Write-Host "Checking stored credentials..." -ForegroundColor Green
foreach ($target in $targets) {
    Get-StoredCredential -Target $target
}
```

## Visual Studio Integration

### Automatic Credential Usage

Visual Studio automatically uses stored credentials for:
- **Git operations** (clone, push, pull)
- **NuGet package restoration**
- **Azure DevOps integration**
- **Team Foundation Server** connections

### Configure Visual Studio Authentication

#### Git Credential Manager

```powershell
# Configure Git Credential Manager for Windows
git config --global credential.helper manager-core

# Set credential storage to Windows Credential Manager
git config --global credential.useHttpPath true
git config --global credential.writeable true
```

#### Azure DevOps Integration

1. **Tools** → **Options** → **Source Control** → **Git Global Settings**
2. **Set email:** your.email@method.com
3. **Enable credential caching**
4. **Configure authentication helper**

## SQL Server Management Studio

### Windows Authentication

SSMS automatically uses your domain credentials when configured for Windows Authentication:

```sql
-- Connection string example using Windows Authentication
Server=sql-server.method.internal;Database=MethodDev;Trusted_Connection=true;
```

### SQL Server Authentication

For SQL Server authentication, store credentials securely:

```powershell
# Store SQL Server credentials
$sqlServer = "sql-server.method.internal"
$sqlUsername = "method-dev-user"

cmdkey /generic:$sqlServer /user:$sqlUsername /pass:$(Read-Host "SQL Password" -AsSecureString | ConvertFrom-SecureString)
```

## PowerShell Profile Configuration

### Automatic Credential Loading

Add to your PowerShell profile (`$PROFILE`) for automatic credential management:

```powershell
# Method PowerShell Profile - Credential Management
# Location: $env:USERPROFILE\Documents\WindowsPowerShell\Microsoft.PowerShell_profile.ps1

# Function to check and load Method credentials
function Initialize-MethodCredentials {
    $requiredCredentials = @(
        "method.internal",
        "git:https://dev.azure.com/methodcrm"
    )
    
    $missingCredentials = @()
    
    foreach ($target in $requiredCredentials) {
        $output = cmdkey /list:$target 2>&1
        if ($output -notmatch "Target: $target") {
            $missingCredentials += $target
        }
    }
    
    if ($missingCredentials.Count -gt 0) {
        Write-Host "Missing credentials detected:" -ForegroundColor Yellow
        foreach ($missing in $missingCredentials) {
            Write-Host "  - $missing" -ForegroundColor Red
        }
        Write-Host "Run Initialize-MethodCredentialStorage to set up missing credentials" -ForegroundColor Yellow
    } else {
        Write-Host "✓ All Method credentials are configured" -ForegroundColor Green
    }
}

# Function to initialize credential storage
function Initialize-MethodCredentialStorage {
    # Implementation of credential storage script here
    & "C:\MethodDev\Scripts\Setup-Credentials.ps1"
}

# Check credentials on profile load
Initialize-MethodCredentials
```

## Network Authentication

### Single Sign-On (SSO)

Method services support SSO through Active Directory:

```powershell
# Configure SSO for Method web applications
$ssoTargets = @(
    "https://portal.method.com",
    "https://internal.method.com", 
    "https://methodcrm.sharepoint.com"
)

foreach ($target in $ssoTargets) {
    # Add to IE trusted sites for seamless SSO
    $reg = "HKCU:\Software\Microsoft\Windows\CurrentVersion\Internet Settings\ZoneMap\Domains"
    $domain = ([uri]$target).Host
    
    if (!(Test-Path "$reg\$domain")) {
        New-Item -Path "$reg\$domain" -Force
        Set-ItemProperty -Path "$reg\$domain" -Name "https" -Value 1
        Write-Host "✓ Added $domain to trusted sites for SSO" -ForegroundColor Green
    }
}
```

### Kerberos Authentication

For Kerberos-enabled services:

```powershell
# Configure Kerberos authentication
setspn -L $env:USERNAME

# Test Kerberos authentication
klist tickets
```

## Security Best Practices

### Credential Hygiene

- **Regular Updates:** Change passwords according to Method policy
- **Unique Passwords:** Use different passwords for different services
- **Multi-Factor Authentication:** Enable MFA where available
- **Password Complexity:** Follow Method password complexity requirements

### Secure Storage

```powershell
# Backup credential store (for migration to new machine)
function Backup-CredentialStore {
    $backupPath = "C:\temp\credential-backup-$(Get-Date -Format 'yyyyMMdd').txt"
    
    Write-Host "Creating credential backup..." -ForegroundColor Yellow
    cmdkey /list | Out-File $backupPath
    
    Write-Host "Credential list backed up to: $backupPath" -ForegroundColor Green
    Write-Host "Note: This backup contains target names only, not passwords" -ForegroundColor Yellow
}

# Clear sensitive credentials when needed
function Clear-MethodCredentials {
    $targets = @(
        "method.internal",
        "git:https://dev.azure.com/methodcrm",
        "nuget:https://dev.azure.com/methodcrm"
    )
    
    foreach ($target in $targets) {
        try {
            cmdkey /delete:$target
            Write-Host "✓ Cleared credential for $target" -ForegroundColor Green
        } catch {
            Write-Host "⚠ Could not clear credential for $target" -ForegroundColor Yellow
        }
    }
}
```

## Troubleshooting

### Common Issues

#### Credentials Not Working

```powershell
# Test credential access
function Test-CredentialAccess {
    param([string]$Target)
    
    $output = cmdkey /list:$Target 2>&1
    
    if ($output -match "Target: $Target") {
        Write-Host "✓ Credential exists for $Target" -ForegroundColor Green
        
        # Test if credential is accessible
        try {
            $credential = Get-StoredCredential -Target $Target
            Write-Host "✓ Credential is accessible" -ForegroundColor Green
        } catch {
            Write-Host "✗ Credential exists but is not accessible" -ForegroundColor Red
        }
    } else {
        Write-Host "✗ No credential found for $Target" -ForegroundColor Red
    }
}
```

#### Git Authentication Failures

```powershell
# Reset Git credential manager
git config --global --unset credential.helper
git config --global credential.helper manager-core

# Clear cached Git credentials
git credential-manager-core erase <<< "protocol=https\nhost=dev.azure.com\npath=/methodcrm"
```

#### Visual Studio Authentication Issues

1. **Clear Visual Studio cache:**
   ```powershell
   Remove-Item "$env:LOCALAPPDATA\Microsoft\VisualStudio\*\ComponentModelCache" -Recurse -Force
   ```

2. **Reset Azure DevOps connection:**
   - Tools → Options → Source Control → Git Global Settings
   - Clear cached credentials
   - Re-authenticate

#### Credential Manager Corruption

```powershell
# Reset Windows Credential Manager (use with caution)
function Reset-CredentialManager {
    Write-Host "⚠ This will clear ALL stored credentials!" -ForegroundColor Red
    $confirm = Read-Host "Are you sure? Type 'YES' to continue"
    
    if ($confirm -eq "YES") {
        # This is a destructive operation
        rundll32.exe keymgr.dll,KRShowKeyMgr
        Write-Host "Please manually clear credentials in the opened dialog" -ForegroundColor Yellow
    }
}
```

## Verification

### Test Credential Storage

```powershell
# Comprehensive credential verification
function Test-MethodCredentials {
    Write-Host "Method Credential Verification" -ForegroundColor Green
    
    $testTargets = @(
        @{Target="method.internal"; Description="Domain authentication"},
        @{Target="git:https://dev.azure.com/methodcrm"; Description="Git/Azure DevOps"},
        @{Target="nuget:https://dev.azure.com/methodcrm"; Description="NuGet packages"}
    )
    
    $results = @()
    
    foreach ($test in $testTargets) {
        $exists = $false
        try {
            $output = cmdkey /list:$test.Target 2>&1
            $exists = $output -match "Target: $($test.Target)"
        } catch {
            $exists = $false
        }
        
        $results += [PSCustomObject]@{
            Target = $test.Target
            Description = $test.Description
            Status = if ($exists) { "✓ Configured" } else { "✗ Missing" }
            Color = if ($exists) { "Green" } else { "Red" }
        }
    }
    
    $results | ForEach-Object {
        Write-Host "$($_.Status) $($_.Description) ($($_.Target))" -ForegroundColor $_.Color
    }
    
    $missing = ($results | Where-Object { $_.Status -like "*Missing*" }).Count
    if ($missing -eq 0) {
        Write-Host "`n✓ All credentials are properly configured!" -ForegroundColor Green
    } else {
        Write-Host "`n⚠ $missing credential(s) need to be configured" -ForegroundColor Yellow
    }
}

# Run verification
Test-MethodCredentials
```

## Migration and Backup

### Moving to New Machine

```powershell
# Export credential targets for documentation
function Export-CredentialTargets {
    $exportPath = "C:\temp\method-credentials-$(Get-Date -Format 'yyyyMMdd').txt"
    
    $credentialInfo = @"
Method Development Environment - Credential Targets
Generated: $(Get-Date)

Required credentials for Method development:

1. Domain Authentication
   Target: method.internal
   Username: METHODDOMAIN\yourusername
   Description: Active Directory domain authentication

2. Azure DevOps Git
   Target: git:https://dev.azure.com/methodcrm
   Username: your.email@method.com
   Description: Git operations and Azure DevOps access

3. NuGet Package Feed
   Target: nuget:https://dev.azure.com/methodcrm
   Username: your.email@method.com
   Description: Access to Method internal NuGet packages

Setup Instructions:
1. Use Control Panel > Credential Manager
2. Add Generic Credentials for each target
3. Test access through Visual Studio and Git operations

"@
    
    Set-Content -Path $exportPath -Value $credentialInfo
    Write-Host "Credential setup guide exported to: $exportPath" -ForegroundColor Green
}
```

## Next Steps

After setting up credential storage:

1. **Install Testing Framework:** [NUnit Test Adapter](./nunit-adapter.md)
2. **Add IIS Components:** [URL Rewrite Installation](./url-rewrite.md)
3. **Test Authentication:** Verify Git operations and NuGet package access
4. **Configure Environment:** [Environment Setup](../environment-setup/README.md)

## Additional Resources

- **Windows Credential Manager Documentation:** Microsoft Docs
- **Git Credential Manager:** https://github.com/GitCredentialManager/git-credential-manager
- **Method IT Support:** Contact for password reset and account issues
- **Azure DevOps Personal Access Tokens:** https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens

**Back to:** [Microsoft Tools](./README.md)
