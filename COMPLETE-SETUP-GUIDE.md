# Complete Method CRM Local Development Environment Setup Guide

## Overview

This guide provides comprehensive, step-by-step instructions for setting up a Windows laptop as a fully functional local development environment for Method CRM. This document captures all lessons learned, troubleshooting knowledge, and fixes discovered during actual setup.

**Last Updated:** 2026-01-21

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Phase 1: System Prerequisites](#phase-1-system-prerequisites)
3. [Phase 2: Development Tools](#phase-2-development-tools)
4. [Phase 3: Method-CLI Installation](#phase-3-method-cli-installation)
5. [Phase 4: IIS Configuration](#phase-4-iis-configuration)
6. [Phase 5: Network Configuration](#phase-5-network-configuration)
7. [Phase 6: SQL Server Configuration](#phase-6-sql-server-configuration)
8. [Phase 7: Docker Services](#phase-7-docker-services)
9. [Phase 8: Source Code](#phase-8-source-code)
10. [Phase 9: NuGet Configuration](#phase-9-nuget-configuration)
11. [Phase 10: Build Projects](#phase-10-build-projects)
12. [Phase 11: SSL Certificates](#phase-11-ssl-certificates)
13. [Phase 12: AWS Credentials](#phase-12-aws-credentials)
14. [Phase 13: Database Restoration](#phase-13-database-restoration)
15. [Phase 14: Registry Configuration](#phase-14-registry-configuration)
16. [Phase 15: Validation](#phase-15-validation)
17. [Troubleshooting](#troubleshooting)
18. [Known Issues](#known-issues)
19. [Scripts Reference](#scripts-reference)
20. [Quick Reference Commands](#quick-reference-commands)

---

## Prerequisites

Before starting, ensure you have:

- [ ] Windows 10/11 Pro or Enterprise (for Hyper-V/Docker support)
- [ ] Administrator access to the machine
- [ ] Method VPN access credentials
- [ ] GitHub account with access to methodcrm organization
- [ ] Azure DevOps credentials for NuGet feed (tfs.method.me)
- [ ] SSL certificate file: `STAR.methodlocal.com.pfx` and its password
- [ ] Database server credentials for `dev-setup.methodintegration.com`
- [ ] AWS credentials file (download from Confluence)

---

## Phase 1: System Prerequisites

### 1.1 Install Chocolatey Package Manager

Open PowerShell as Administrator:

```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force
[System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072
Invoke-Expression ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))

# Refresh environment
refreshenv

# Enable global confirmation
choco feature enable -n allowGlobalConfirmation
```

### 1.2 Install Core Development Tools

```powershell
# Source Control
choco install git -y
choco install tortoisegit -y
choco install gh -y  # GitHub CLI

# .NET SDKs and Runtimes (ALL versions needed)
choco install dotnet-sdk -y
choco install dotnet-5.0-sdk -y
choco install dotnet-6.0-sdk -y
choco install dotnet-7.0-sdk -y
choco install dotnet-8.0-sdk -y
choco install dotnet-9.0-sdk -y

# .NET Framework (for legacy projects)
choco install dotnet4.5 -y
choco install dotnet4.6 -y
choco install dotnet4.6.1 -y
choco install dotnet4.6.2 -y
choco install dotnet4.7 -y

# Build Tools
choco install nodejs-lts -y
choco install python -y
choco install ruby -y

# Install Visual Studio 2022 Build Tools (REQUIRED)
choco install visualstudio2022buildtools --package-parameters "--allWorkloads --includeRecommended --includeOptional --passive --locale en-US" -y
```

### 1.3 Install Database Tools

```powershell
# SQL Server 2022 Developer Edition
choco install sql-server-2022 -y

# SQL Server Management Studio
choco install sql-server-management-studio -y

# MongoDB Database Tools (for mongorestore)
choco install mongodb-database-tools -y

# Docker Desktop
choco install docker-desktop -y
```

### 1.4 Install Additional Applications

```powershell
# Code Editors
choco install vscode -y
choco install notepadplusplus -y

# API Testing
choco install postman -y

# Database GUI
choco install mongodb-compass -y
choco install mongodb-shell -y

# AWS Tools
choco install awscli -y
choco install aws-iam-authenticator -y

# URL Rewrite for IIS
choco install urlrewrite -y
```

### 1.5 Install .NET Hosting Bundles (CRITICAL for IIS)

```powershell
# These are required for IIS to host ASP.NET Core applications
winget install Microsoft.DotNet.HostingBundle.8 --accept-package-agreements --accept-source-agreements
winget install Microsoft.DotNet.HostingBundle.7 --accept-package-agreements --accept-source-agreements
winget install Microsoft.DotNet.HostingBundle.6 --accept-package-agreements --accept-source-agreements

# Restart IIS to load the new modules
net stop was /y
net start w3svc
```

### 1.6 Install PowerShell Modules

```powershell
# YAML parsing (required for restore scripts)
Install-Module powershell-yaml -Force -Scope CurrentUser

# SQL Server module (required for database restore)
Install-Module -Name SqlServer -AllowClobber -Force -Scope CurrentUser
```

### 1.7 Install Node.js Build Tools

```powershell
# Grunt CLI
npm install -g grunt-cli

# SASS (via Ruby)
gem install sass
```

### 1.8 Create Required Directories

```powershell
New-Item -ItemType Directory -Path "C:\MethodDev" -Force
New-Item -ItemType Directory -Path "C:\MethodDev\blank" -Force
New-Item -ItemType Directory -Path "C:\MethodData" -Force
New-Item -ItemType Directory -Path "C:\MethodData\MethodIntegrationSQL" -Force
New-Item -ItemType Directory -Path "C:\MethodData\MethodIntegrationSQL\TEMPBACKUP" -Force
New-Item -ItemType Directory -Path "C:\MongoDump" -Force
New-Item -ItemType Directory -Path "C:\SQLDump" -Force
New-Item -ItemType Directory -Path "C:\Temp" -Force
New-Item -ItemType Directory -Path "D:\logs" -Force

# Create blank folder web.config for IIS
Set-Content -Path "C:\MethodDev\blank\web.config" -Value @"
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <system.webServer>
        <httpErrors errorMode="Detailed" />
    </system.webServer>
</configuration>
"@

# Grant NETWORK SERVICE full control to MethodData folder
$acl = Get-Acl "C:\MethodData"
$rule = New-Object System.Security.AccessControl.FileSystemAccessRule("NETWORK SERVICE", "FullControl", "ContainerInherit,ObjectInherit", "None", "Allow")
$acl.SetAccessRule($rule)
Set-Acl "C:\MethodData" $acl
```

### 1.9 Reboot if Required

Check for pending reboots:
```powershell
Get-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Session Manager' -Name 'PendingFileRenameOperations' -ErrorAction SilentlyContinue
```

If found, reboot before continuing. To clear without reboot (not recommended):
```powershell
Remove-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Session Manager' -Name 'PendingFileRenameOperations' -ErrorAction SilentlyContinue
```

---

## Phase 2: Development Tools

### 2.1 Configure Git

```powershell
# Set up Git identity
git config --global user.name "Your Name"
git config --global user.email "your.email@method.me"

# Configure for HTTPS (if using GitHub CLI auth)
gh auth login
gh auth setup-git
git config --global url."https://github.com/".insteadOf "git@github.com:"
```

### 2.2 Install nvm for Node.js Version Management

```powershell
choco install nvm -y
refreshenv

# Install required Node.js version
nvm install lts
nvm use lts
```

---

## Phase 3: Method-CLI Installation

### 3.1 Clone DeveloperTools Repository

```powershell
cd C:\MethodDev
git clone https://github.com/methodcrm/DeveloperTools.git
```

### 3.2 Install Method-CLI

```powershell
cd C:\MethodDev\DeveloperTools\Method-CLI
.\install.ps1
```

### 3.3 Verify Installation

```powershell
mtd --version
mtd --help
```

---

## Phase 4: IIS Configuration

### 4.1 Enable IIS Features

Run in elevated PowerShell:

```powershell
# Core IIS
Enable-WindowsOptionalFeature -Online -FeatureName IIS-WebServerRole -All
Enable-WindowsOptionalFeature -Online -FeatureName IIS-WebServer -All
Enable-WindowsOptionalFeature -Online -FeatureName IIS-CommonHttpFeatures -All
Enable-WindowsOptionalFeature -Online -FeatureName IIS-ApplicationDevelopment -All

# ASP.NET Support
Enable-WindowsOptionalFeature -Online -FeatureName IIS-ASPNET45 -All
Enable-WindowsOptionalFeature -Online -FeatureName IIS-NetFxExtensibility45 -All
Enable-WindowsOptionalFeature -Online -FeatureName NetFx4Extended-ASPNET45 -All

# ISAPI
Enable-WindowsOptionalFeature -Online -FeatureName IIS-ISAPIExtensions -All
Enable-WindowsOptionalFeature -Online -FeatureName IIS-ISAPIFilter -All

# Management
Enable-WindowsOptionalFeature -Online -FeatureName IIS-ManagementConsole -All
Enable-WindowsOptionalFeature -Online -FeatureName IIS-ManagementScriptingTools -All

# Security
Enable-WindowsOptionalFeature -Online -FeatureName IIS-BasicAuthentication -All
Enable-WindowsOptionalFeature -Online -FeatureName IIS-WindowsAuthentication -All
Enable-WindowsOptionalFeature -Online -FeatureName IIS-RequestFiltering -All

# Performance
Enable-WindowsOptionalFeature -Online -FeatureName IIS-HttpCompressionStatic -All
Enable-WindowsOptionalFeature -Online -FeatureName IIS-Performance -All

# Application Initialization
Enable-WindowsOptionalFeature -Online -FeatureName IIS-ApplicationInit -All

# WebSockets
Enable-WindowsOptionalFeature -Online -FeatureName IIS-WebSockets -All

# Additional Features
Enable-WindowsOptionalFeature -Online -FeatureName IIS-DefaultDocument -All
Enable-WindowsOptionalFeature -Online -FeatureName IIS-DirectoryBrowsing -All
Enable-WindowsOptionalFeature -Online -FeatureName IIS-HttpErrors -All
Enable-WindowsOptionalFeature -Online -FeatureName IIS-HttpLogging -All
Enable-WindowsOptionalFeature -Online -FeatureName IIS-HttpRedirect -All
Enable-WindowsOptionalFeature -Online -FeatureName IIS-HttpTracing -All
Enable-WindowsOptionalFeature -Online -FeatureName IIS-StaticContent -All
Enable-WindowsOptionalFeature -Online -FeatureName IIS-RequestMonitor -All
Enable-WindowsOptionalFeature -Online -FeatureName IIS-HealthAndDiagnostics -All
Enable-WindowsOptionalFeature -Online -FeatureName IIS-LoggingLibraries -All
Enable-WindowsOptionalFeature -Online -FeatureName IIS-Metabase -All
Enable-WindowsOptionalFeature -Online -FeatureName IIS-IIS6ManagementCompatibility -All
Enable-WindowsOptionalFeature -Online -FeatureName IIS-HostableWebCore -All
Enable-WindowsOptionalFeature -Online -FeatureName IIS-ServerSideIncludes -All
Enable-WindowsOptionalFeature -Online -FeatureName NetFx4-AdvSrvs -All
```

### 4.2 Import IIS Sites and App Pools

```powershell
cd C:\MethodDev\DeveloperTools\DevSetup\build_local
powershell -ExecutionPolicy Bypass -Command ".\import_sites_pools.ps1 -importOnly $true"
```

### 4.3 Verify IIS Configuration

```powershell
mtd iis sites status
mtd iis pool status
```

---

## Phase 5: Network Configuration

### 5.1 Configure Hosts File

Add to `C:\Windows\System32\drivers\etc\hosts`:

```
##################################
# Method Local Development
##################################
127.0.0.1 methodlocaldb
127.0.0.1 methodlocal
127.0.0.1 methodlocal.com
127.0.0.1 www.methodlocal.com
127.0.0.1 method.methodlocal.com
127.0.0.1 gateway.methodlocal.com
127.0.0.1 gateway.methodlocal.int
127.0.0.1 subscriber.methodlocal.com
127.0.0.1 runtime.methodlocal.com
127.0.0.1 microservices.methodlocal.int
127.0.0.1 microservices.methodlocal.com
127.0.0.1 auth.methodlocal.com
127.0.0.1 emailgadget.methodlocal.com
127.0.0.1 oauth.methodlocal.com
127.0.0.1 identityserver.methodlocal.com
127.0.0.1 eda.methodlocal.int
127.0.0.1 templatedevv2.methodlocal.com
127.0.0.1 alocetsystem.methodlocal.com
127.0.0.1 miurl.methodlocal.com
127.0.0.1 api.methodlocal.com
127.0.0.1 api.methodlocal.int
127.0.0.1 client.methodlocal.com
127.0.0.1 proxy.methodlocal.com
127.0.0.1 signup.methodlocal.com
127.0.0.1 signin.methodlocal.com
127.0.0.1 services.methodlocal.com
127.0.0.1 migration.methodlocal.com
127.0.0.1 sync.methodlocal.com
127.0.0.1 notify.methodlocal.com
127.0.0.1 intercom.methodlocal.com
127.0.0.1 billing.methodlocal.com
127.0.0.1 outlookgadget.methodlocal.com
127.0.0.1 appdirect.methodlocal.com
127.0.0.1 syncservices.methodlocal.com
127.0.0.1 designer.methodlocal.com
127.0.0.1 notifications.methodlocal.com
127.0.0.1 px.methodlocal.com
127.0.0.1 signin.methodportallocal.com
127.0.0.1 dataminer.methodlocal.com
127.0.0.1 methodportallocal.com
127.0.0.1 restapi.methodlocal.com
127.0.0.1 openid.methodlocal.com
127.0.0.1 webhooks.methodlocal.com
127.0.0.1 mailchimp.methodlocal.com
##################################
```

PowerShell command to add all entries:
```powershell
$hostsContent = @"

##################################
# Method Local Development
##################################
127.0.0.1 methodlocaldb
127.0.0.1 methodlocal
127.0.0.1 methodlocal.com
127.0.0.1 www.methodlocal.com
127.0.0.1 method.methodlocal.com
127.0.0.1 gateway.methodlocal.com
127.0.0.1 gateway.methodlocal.int
127.0.0.1 subscriber.methodlocal.com
127.0.0.1 runtime.methodlocal.com
127.0.0.1 microservices.methodlocal.int
127.0.0.1 microservices.methodlocal.com
127.0.0.1 auth.methodlocal.com
127.0.0.1 emailgadget.methodlocal.com
127.0.0.1 oauth.methodlocal.com
127.0.0.1 identityserver.methodlocal.com
127.0.0.1 eda.methodlocal.int
127.0.0.1 templatedevv2.methodlocal.com
127.0.0.1 alocetsystem.methodlocal.com
127.0.0.1 miurl.methodlocal.com
127.0.0.1 api.methodlocal.com
127.0.0.1 api.methodlocal.int
127.0.0.1 client.methodlocal.com
127.0.0.1 proxy.methodlocal.com
127.0.0.1 signup.methodlocal.com
127.0.0.1 signin.methodlocal.com
127.0.0.1 services.methodlocal.com
127.0.0.1 migration.methodlocal.com
127.0.0.1 sync.methodlocal.com
127.0.0.1 notify.methodlocal.com
127.0.0.1 intercom.methodlocal.com
127.0.0.1 billing.methodlocal.com
127.0.0.1 outlookgadget.methodlocal.com
127.0.0.1 appdirect.methodlocal.com
127.0.0.1 syncservices.methodlocal.com
127.0.0.1 designer.methodlocal.com
127.0.0.1 notifications.methodlocal.com
127.0.0.1 px.methodlocal.com
127.0.0.1 signin.methodportallocal.com
127.0.0.1 dataminer.methodlocal.com
127.0.0.1 methodportallocal.com
127.0.0.1 restapi.methodlocal.com
127.0.0.1 openid.methodlocal.com
127.0.0.1 webhooks.methodlocal.com
127.0.0.1 mailchimp.methodlocal.com
##################################
"@
Add-Content -Path 'C:\Windows\System32\drivers\etc\hosts' -Value $hostsContent
```

---

## Phase 6: SQL Server Configuration

### 6.1 Enable TCP/IP Protocol

1. Open SQL Server Configuration Manager
2. Navigate to: SQL Server Network Configuration > Protocols for MSSQLSERVER
3. Right-click TCP/IP > Enable
4. Right-click TCP/IP > Properties > IP Addresses tab
5. Set TCP Port to 1433 for IPAll
6. Restart SQL Server service

### 6.2 Create SQL Alias

```powershell
# Create registry paths if they don't exist
$paths = @(
    "HKLM:\SOFTWARE\Microsoft\MSSQLServer\Client\ConnectTo",
    "HKLM:\SOFTWARE\WOW6432Node\Microsoft\MSSQLServer\Client\ConnectTo"
)
foreach ($path in $paths) {
    if (-not (Test-Path $path)) {
        New-Item -Path $path -Force | Out-Null
    }
}

# Create 32-bit alias
New-ItemProperty -Path "HKLM:\SOFTWARE\WOW6432Node\Microsoft\MSSQLServer\Client\ConnectTo" -Name "methodlocaldb" -Value "DBMSSOCN,localhost,1433" -PropertyType String -Force

# Create 64-bit alias
New-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\MSSQLServer\Client\ConnectTo" -Name "methodlocaldb" -Value "DBMSSOCN,localhost,1433" -PropertyType String -Force
```

### 6.3 Enable Mixed Mode Authentication and Create Logins

Connect to SQL Server and run:

```sql
-- Enable mixed mode authentication
EXEC xp_instance_regwrite N'HKEY_LOCAL_MACHINE',
    N'Software\Microsoft\MSSQLServer\MSSQLServer',
    N'LoginMode', REG_DWORD, 2;
GO

-- Create runtime-core-engine login (used by runtime services)
IF NOT EXISTS (SELECT * FROM sys.server_principals WHERE name = 'runtime-core-engine')
BEGIN
    CREATE LOGIN [runtime-core-engine] WITH PASSWORD = 'local', CHECK_POLICY = OFF;
    ALTER SERVER ROLE [sysadmin] ADD MEMBER [runtime-core-engine];
END
GO

-- Create bcpreader login (used for bulk operations)
IF NOT EXISTS (SELECT * FROM sys.server_principals WHERE name = 'bcpreader')
BEGIN
    CREATE LOGIN [bcpreader] WITH PASSWORD = 'Gravity4fRee', CHECK_POLICY = OFF;
    ALTER SERVER ROLE [sysadmin] ADD MEMBER [bcpreader];
END
GO

-- Grant NT AUTHORITY\NETWORK SERVICE access (for IIS app pools)
IF NOT EXISTS (SELECT * FROM sys.server_principals WHERE name = 'NT AUTHORITY\NETWORK SERVICE')
BEGIN
    CREATE LOGIN [NT AUTHORITY\NETWORK SERVICE] FROM WINDOWS;
    ALTER SERVER ROLE [sysadmin] ADD MEMBER [NT AUTHORITY\NETWORK SERVICE];
END
GO
```

### 6.4 Create IIS App Pool Logins

```sql
-- Grant SQL Server access to IIS Application Pool identities
USE master;
GO

DECLARE @pools TABLE (name NVARCHAR(128));
INSERT INTO @pools VALUES
    ('ms-account'), ('authentication'), ('gateway'), ('gateway-v2'),
    ('identity'), ('preferences'), ('documents'), ('tags'), ('email'),
    ('apps'), ('runtime-core'), ('signin'), ('auth.methodlocal.com');

DECLARE @poolName NVARCHAR(128);
DECLARE @sql NVARCHAR(MAX);

DECLARE pool_cursor CURSOR FOR SELECT name FROM @pools;
OPEN pool_cursor;
FETCH NEXT FROM pool_cursor INTO @poolName;

WHILE @@FETCH_STATUS = 0
BEGIN
    SET @sql = 'IF NOT EXISTS (SELECT * FROM sys.server_principals WHERE name = ''IIS APPPOOL\' + @poolName + ''')
    BEGIN
        CREATE LOGIN [IIS APPPOOL\' + @poolName + '] FROM WINDOWS;
        EXEC sp_addsrvrolemember ''IIS APPPOOL\' + @poolName + ''', ''sysadmin'';
    END';
    EXEC sp_executesql @sql;
    FETCH NEXT FROM pool_cursor INTO @poolName;
END

CLOSE pool_cursor;
DEALLOCATE pool_cursor;
GO
```

### 6.5 Restart SQL Server

```powershell
Restart-Service MSSQLSERVER
```

---

## Phase 7: Docker Services

### 7.1 Start Docker Desktop

Ensure Docker Desktop is running and configured for Linux containers.

### 7.2 Pull and Start Required Containers

```powershell
cd C:\MethodDev\DeveloperTools\DevSetup\build_local
powershell -ExecutionPolicy Bypass -File .\pull_docker_packages.ps1
```

This starts:
- **Redis** - Caching layer (port 6379)
- **Redis Commander** - Redis GUI (port 8081)
- **Elasticsearch** - Search engine (ports 9200, 9300)
- **MongoDB** - Document database (port 27017)
- **RabbitMQ** - Message queue (ports 5672, 15672)

### 7.3 Verify Containers

```powershell
docker ps
```

Expected output should show all containers running.

---

## Phase 8: Source Code

### 8.1 Clone All Repositories

**IMPORTANT:** Connect to Method VPN before cloning.

```powershell
mtd git clone -a
```

This clones ~84 repositories to C:\MethodDev.

---

## Phase 9: NuGet Configuration

### 9.1 Add Private NuGet Feed

**IMPORTANT:** Connect to Method VPN before this step.

```powershell
# Add the method-ms NuGet source
dotnet nuget add source "https://tfs.method.me/tfs/MethodCollection/_packaging/method-ms/nuget/v3/index.json" -n "method-ms"

# Configure credentials (replace with your Azure DevOps credentials)
dotnet nuget update source "method-ms" --username "YOUR_USERNAME" --password "YOUR_PASSWORD" --store-password-in-clear-text
```

### 9.2 Fix HTTP NuGet Sources (CRITICAL)

Many repositories have local NuGet.Config files with HTTP sources that must be changed to HTTPS.

Run this script **BEFORE** building:

```powershell
# Fix ALL NuGet.Config files that have HTTP instead of HTTPS
$configFiles = Get-ChildItem -Path 'C:\MethodDev' -Recurse -Filter 'NuGet.Config' -ErrorAction SilentlyContinue
foreach ($file in $configFiles) {
    $content = Get-Content $file.FullName -Raw
    if ($content -match 'http://tfs\.method\.me') {
        $newContent = $content -replace 'http://tfs\.method\.me', 'https://tfs.method.me'
        Set-Content -Path $file.FullName -Value $newContent -NoNewline
        Write-Host "Fixed: $($file.FullName)"
    }
}
Write-Host "Done!"
```

### 9.3 Disable TeamCity NuGet Source

Some repos have an old TeamCity source (http://tcity) that causes errors. Add disabled section to affected NuGet.Config files:

```xml
<disabledPackageSources>
    <add key="Method" value="true" />
</disabledPackageSources>
```

---

## Phase 10: Build Projects

### 10.1 Set Environment Variables

```powershell
[System.Environment]::SetEnvironmentVariable('ASPNETCORE_ENVIRONMENT', 'Development', 'Machine')
[System.Environment]::SetEnvironmentVariable('DOTNET_ENVIRONMENT', 'Development', 'Machine')
[System.Environment]::SetEnvironmentVariable('DOTNET_HOST_PATH', 'C:\Program Files\dotnet\dotnet.exe', 'Machine')
```

### 10.2 Build All Projects

**IMPORTANT:** Connect to Method VPN before building.

```powershell
mtd build -a --restore
```

### 10.3 Expected Build Results

- **Successful:** ~41 projects
- **Failed:** ~8 projects (known issues documented in troubleshooting section)

---

## Phase 11: SSL Certificates

### 11.1 Import Certificate

```powershell
$certPath = 'C:\Users\YOUR_USERNAME\Downloads\STAR.methodlocal.com.pfx'
$password = Read-Host -AsSecureString "Enter PFX password"

# Import to Personal store
Import-PfxCertificate -FilePath $certPath -CertStoreLocation 'Cert:\LocalMachine\My' -Password $password

# Import to Trusted Root (for self-signed trust)
Import-PfxCertificate -FilePath $certPath -CertStoreLocation 'Cert:\LocalMachine\Root' -Password $password
```

### 11.2 Bind Certificate to IIS Sites (CRITICAL)

The certificate must be bound to all HTTPS sites:

```powershell
$thumbprint = (Get-ChildItem -Path "Cert:\LocalMachine\My" | Where-Object { $_.Subject -like "*methodlocal*" }).Thumbprint
Import-Module WebAdministration

$bindings = Get-WebBinding -Protocol https
foreach ($binding in $bindings) {
    $siteName = $binding.ItemXPath -replace ".*\[@name='([^']*)'.*", '$1'
    Write-Host "Binding certificate to: $siteName"
    try {
        $binding.AddSslCertificate($thumbprint, "My")
        Write-Host "  Success!" -ForegroundColor Green
    } catch {
        Write-Host "  Error: $_" -ForegroundColor Yellow
    }
}
```

---

## Phase 12: AWS Credentials

### 12.1 Download AWS Credentials

Download the `localDev_YYYYMMDD` credentials file from:
- Confluence: [AWS Credentials Setup](https://method.atlassian.net/wiki/spaces/SD/pages/187007423/AWS+Credentials+Setup)
- Or Google Drive link provided in onboarding

### 12.2 Set Up User Profile Credentials

```powershell
# Create .aws folder
New-Item -ItemType Directory -Path "$env:USERPROFILE\.aws" -Force

# Copy credentials file (rename to just 'credentials')
Copy-Item "C:\Users\$env:USERNAME\Downloads\localDev_*" "$env:USERPROFILE\.aws\credentials"

# Create config file
Set-Content -Path "$env:USERPROFILE\.aws\config" -Value @"
[default]
region = us-west-1
output = json
"@
```

### 12.3 Set Up System Profile Credentials (for IIS)

```powershell
# Create .aws folder for system profile
New-Item -ItemType Directory -Path "C:\Windows\System32\config\systemprofile\.aws" -Force

# Copy credentials
Copy-Item "$env:USERPROFILE\.aws\credentials" "C:\Windows\System32\config\systemprofile\.aws\credentials"
Copy-Item "$env:USERPROFILE\.aws\config" "C:\Windows\System32\config\systemprofile\.aws\config"
```

### 12.4 Set Machine-Level Environment Variables (for .NET Framework apps)

```powershell
# Get AWS credentials from file
$creds = Get-Content "$env:USERPROFILE\.aws\credentials"
$accessKey = ($creds | Select-String "aws_access_key_id").ToString().Split("=")[1].Trim()
$secretKey = ($creds | Select-String "aws_secret_access_key").ToString().Split("=")[1].Trim()

# Set machine-level environment variables
[Environment]::SetEnvironmentVariable("AWS_ACCESS_KEY_ID", $accessKey, [EnvironmentVariableTarget]::Machine)
[Environment]::SetEnvironmentVariable("AWS_SECRET_ACCESS_KEY", $secretKey, [EnvironmentVariableTarget]::Machine)
[Environment]::SetEnvironmentVariable("AWS_REGION", "us-west-1", [EnvironmentVariableTarget]::Machine)
[Environment]::SetEnvironmentVariable("AWS_DEFAULT_REGION", "us-west-1", [EnvironmentVariableTarget]::Machine)

# Restart IIS to apply
iisreset /restart
```

---

## Phase 13: Database Restoration

### 13.1 Prerequisites

- VPN connected
- Credentials for `dev-setup.methodintegration.com`
- MongoDB running (in Docker)
- SQL Server running

### 13.2 Restore All Databases

```powershell
cd C:\MethodDev\DeveloperTools\DevSetup\build_local
powershell -ExecutionPolicy Bypass -File .\restore_dbs.ps1
```

### 13.3 Required Databases

| Database | Type | Purpose |
|----------|------|---------|
| templatedevv2 | SQL + MongoDB | Core template database |
| methodappstore | SQL + MongoDB | App store data |
| MethodOAuthSecurity | SQL | OAuth/Authentication |
| alocetsystem | MongoDB | Core system data |
| Accounts | MongoDB | User accounts |
| Preferences | MongoDB | User preferences |
| Authentication | MongoDB | Auth data |
| MethodNotifications | SQL | Notifications |

### 13.4 Manual MongoDB Restore

If automated restore fails:

```powershell
# Navigate to MongoDB tools
cd "C:\Program Files\MongoDB\Tools\100\bin"

# Restore a database (replace with actual values)
.\mongorestore.exe --host="localhost" --port=27017 --db=DATABASE_NAME --archive="C:\MongoDump\DATABASE_NAME.bak" --gzip --drop --noIndexRestore
```

### 13.5 Manual SQL Restore

```sql
RESTORE DATABASE [DatabaseName]
FROM DISK = N'C:\SQLDump\DatabaseName.bak'
WITH FILE = 1,
MOVE N'DatabaseName' TO N'C:\Program Files\Microsoft SQL Server\MSSQL16.MSSQLSERVER\MSSQL\DATA\DatabaseName.mdf',
MOVE N'DatabaseName_log' TO N'C:\Program Files\Microsoft SQL Server\MSSQL16.MSSQLSERVER\MSSQL\DATA\DatabaseName_log.ldf',
NOUNLOAD, REPLACE, STATS = 5;
```

---

## Phase 14: Registry Configuration

### 14.1 Create EncSalt Registry Key

```powershell
$registryPath = "HKLM:\SOFTWARE\Method"
if (-not (Test-Path $registryPath)) {
    New-Item -Path $registryPath -Force | Out-Null
}
Set-ItemProperty -Path $registryPath -Name "EncSalt" -Value "MethodLocalDevelopmentSalt2024"
```

Or import the registry file:
```powershell
reg import "C:\MethodDev\method-platform-ui\EncSalt.reg"
```

### 14.2 Restart IIS

```powershell
iisreset /restart
```

---

## Phase 15: Validation

### 15.1 Run Health Checks

```powershell
mtd healthcheck run -a
```

### 15.2 Expected Healthy Services

After database restore, these services should be healthy:
- internal-migration-api
- legacy-authentication-api (Services)
- legacy-billingsubscription-api (Billing)
- legacy-miurl-api
- legacy-openid-api
- method-notifications-api
- qbo-sync-api (Sync)

### 15.3 Check Logs

```powershell
# View recent logs
Get-ChildItem D:\logs -Recurse | Sort-Object LastWriteTime -Descending | Select-Object -First 20
```

---

## Troubleshooting

### IIS Application Pools Crashing on Startup

**Symptom:** Event log shows "The identity of application pool X is invalid"

**Fix:** Set app pools to use ApplicationPoolIdentity:
```powershell
Import-Module WebAdministration
$pools = Get-ChildItem IIS:\AppPools
foreach ($pool in $pools) {
    $identityType = $pool.processModel.identityType
    $userName = $pool.processModel.userName
    if ($identityType -eq "SpecificUser" -and [string]::IsNullOrEmpty($userName)) {
        Set-ItemProperty "IIS:\AppPools\$($pool.Name)" -Name processModel.identityType -Value 4
        Write-Output "Fixed: $($pool.Name)"
    }
}
```

### HTTP 500.19 - Configuration Error (0x8007000d)

**Symptom:** Services return HTTP 500.19 with error code `0x8007000d`

**Cause:** ASP.NET Core Module V2 (ANCM) not installed

**Fix:**
```powershell
winget install Microsoft.DotNet.HostingBundle.8 --accept-package-agreements --accept-source-agreements
winget install Microsoft.DotNet.HostingBundle.7 --accept-package-agreements --accept-source-agreements
net stop was /y
net start w3svc
```

### HTTP 500.35 - Multiple Apps in Same App Pool

**Symptom:** "ASP.NET Core does not support multiple apps in the same app pool"

**Fix:** Create separate app pools:
```powershell
Import-Module WebAdministration
New-WebAppPool -Name "gateway-v2"
Set-ItemProperty "IIS:\AppPools\gateway-v2" -Name processModel.identityType -Value 4
Set-ItemProperty "IIS:\Sites\api.methodlocal.com\v2" -Name applicationPool -Value "gateway-v2"
iisreset /restart
```

### Missing C:\MethodDev\blank folder

**Fix:**
```powershell
New-Item -ItemType Directory -Path "C:\MethodDev\blank" -Force
Set-Content -Path "C:\MethodDev\blank\web.config" -Value @"
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <system.webServer>
        <httpErrors errorMode="Detailed" />
    </system.webServer>
</configuration>
"@
```

### AWS Region Not Configured

**Symptom:** "AmazonClientException: No RegionEndpoint or ServiceURL configured"

**Fix:** Set AWS environment variables in web.config or machine-level:
```powershell
[Environment]::SetEnvironmentVariable("AWS_REGION", "us-west-1", [EnvironmentVariableTarget]::Machine)
[Environment]::SetEnvironmentVariable("AWS_DEFAULT_REGION", "us-west-1", [EnvironmentVariableTarget]::Machine)
iisreset /restart
```

### EncSalt NullReferenceException

**Symptom:** NullReferenceException in MD5.MD5SaltedHashEncryption

**Fix:**
```powershell
$registryPath = "HKLM:\SOFTWARE\Method"
if (-not (Test-Path $registryPath)) { New-Item -Path $registryPath -Force | Out-Null }
Set-ItemProperty -Path $registryPath -Name "EncSalt" -Value "MethodLocalDevelopmentSalt2024"
iisreset /restart
```

### Missing Offline Template Files

**Symptom:** "missing C:\MethodData\MethodIntegrationSQL\TemplateMethodNewOffline.mdf"

**Fix:**
```powershell
New-Item -Path "C:\MethodData\MethodIntegrationSQL\TemplateMethodNewOffline.mdf" -ItemType File -Force
New-Item -Path "C:\MethodData\MethodIntegrationSQL\TemplateMethodNewOffline_Log.ldf" -ItemType File -Force
New-Item -ItemType Directory -Path "C:\MethodData\MethodIntegrationSQL\TEMPBACKUP" -Force
```

### SQL Access Denied for IIS App Pools

**Symptom:** Health checks fail for sql_c1, sql_c2

**Fix:** Create SQL logins for IIS app pool identities (see Phase 6.4)

### NuGet HTTP Source Error

**Fix:** Run the HTTP to HTTPS fix script (see Phase 9.2)

### IIS Path Mismatches

**Symptom:** 500.19 pointing to paths that don't exist

**Common mismatches:**
- `ms-analytics-api\analytics\API` → should be `ms-analytics-api\MethodAnalytics.Api`
- `ms-archive\archive\API` → should be `ms-archive-api\archive\API`

**Fix:**
```powershell
Import-Module WebAdministration
Set-ItemProperty "IIS:\Sites\microservices.methodlocal.int\analytics" -Name PhysicalPath -Value "C:\MethodDev\ms-analytics-api\MethodAnalytics.Api"
```

---

## Known Issues

| Issue | Solution |
|-------|----------|
| Pending reboot blocking installs | Clear: `Remove-ItemProperty -Path 'HKLM:\...\Session Manager' -Name 'PendingFileRenameOperations'` |
| MSBuild.exe not found | Install VS Build Tools; Method-CLI auto-detects editions |
| NuGet 401 Unauthorized | Configure credentials with `--store-password-in-clear-text` |
| NuGet HTTP source error | Run HTTP→HTTPS fix script |
| HTTP 500.19 code 0x8007000d | Install .NET Hosting Bundle |
| App pool identity invalid | Set identityType to 4 |
| HTTP 500.35 multiple apps | Create separate app pools |
| SQL access denied | Create IIS APPPOOL logins |
| AWS region error | Set machine-level env vars |
| EncSalt NullReferenceException | Create registry key |
| Missing offline files | Create placeholder files |
| TeamCity NuGet (http://tcity) | Add disabledPackageSources |
| MethodOAuthSecurity backup missing | Trigger via Database Copy API |
| mongorestore archive error | Use `--gzip` flag |
| sqlcmd not in PATH | Use full path |
| SqlServer module not found | Install with `Install-Module -Name SqlServer` |
| URL Rewrite missing | `choco install urlrewrite` |
| SyncIppGuid missing | Add to web.config appSettings |

---

## Scripts Reference

| Script | Location | Purpose |
|--------|----------|---------|
| initialize.ps1 | DevSetup\build_local\ | Install dev tools |
| pull_docker_packages.ps1 | DevSetup\build_local\ | Start Docker services |
| restore_dbs.ps1 | DevSetup\build_local\ | Restore databases |
| import_sites_pools.ps1 | DevSetup\build_local\ | Import IIS configuration |
| build_local_environment.ps1 | DevSetup\build_local\ | Build all projects |

---

## Quick Reference Commands

```powershell
# Health check
mtd healthcheck run -a

# Build all
mtd build -a --restore

# IIS status
mtd iis sites status
mtd iis pool status

# Restart IIS
iisreset

# Docker status
docker ps

# View logs
Get-ChildItem D:\logs -Recurse | Sort-Object LastWriteTime -Descending | Select-Object -First 10

# Check pending reboots
Get-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Session Manager' -Name 'PendingFileRenameOperations' -ErrorAction SilentlyContinue

# Check installed .NET runtimes
dotnet --list-runtimes

# Check ASP.NET Core Module
Import-Module WebAdministration
Get-WebGlobalModule | Where-Object { $_.Name -like '*AspNetCore*' }

# Check stopped app pools
Import-Module WebAdministration
Get-ChildItem IIS:\AppPools | Where-Object { $_.State -ne 'Started' } | Select-Object Name, State

# Test SQL connection
sqlcmd -S methodlocaldb -E -Q "SELECT 1"

# Verify AWS credentials
aws sts get-caller-identity
```

---

## Database Availability Status

### SQL Server Databases with Backups Available:
- templatedevv2 (FULL backup available)
- methodappstore (FULL backup available)

### SQL Server Databases WITHOUT Backups:
- MethodOAuthSecurity (may need to trigger copy)
- MethodNotifications (may need to trigger copy)

### MongoDB Databases Available:
- templatedevv2
- methodappstore
- alocetsystem
- Accounts
- Preferences
- Authentication

---

## Generating Missing Database Backups

If a database backup is not available, trigger a copy using the Method API:

1. Install Postman: `choco install postman -y`
2. Import Method API Postman collection (from Confluence)
3. Use Database Copy endpoint with method.me support credentials
4. Wait for backup generation
5. Re-run restore scripts

---

## Contact

For issues not covered in this guide, contact the DevOps team or post in the #dev-help Slack channel.
