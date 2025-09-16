# Download and Restore Databases

## Method Database Backup and Restoration Process

This guide covers downloading and restoring Method's database backups for local development.

## Database Overview

### Required Method Databases

**SQL Server Databases:**
- **AlocetSystem** - Core Method system database
- **Method App Store** - Application marketplace data
- **MethodNotifications** - Notification system data
- **MethodOAuthSecurity** - OAuth authentication data
- **MethodShortURLRedirects** - URL shortening service
- **TemplateDevV2** - Template database for app development
- **TemplateMethodNewQBO** - QuickBooks Online integration template

**MongoDB Databases:**
- **Accounts** - User account data
- **AlocetSystem** - System configuration data
- **Authentication** - Authentication tokens
- **Methodappstore** - App store content
- **methodmobilenotifications** - Mobile notification data
- **oauth_token** - OAuth token storage
- **Preferences** - User preferences
- **TemplateDevV2** - Development template data

## Prerequisites

- VPN connection to Method's network
- Method Active Directory credentials
- SQL Server and MongoDB installed and running
- C:\Methoddata\MethodIntegrationSQL directory created
- Administrator access for database operations

## Manual Database Download

### Step 1: Access Database Repository
1. **Connect to VPN** - Ensure VPN connection is active
2. **Navigate to**: [https://dev-setup.methodintegration.com/dbs/](https://dev-setup.methodintegration.com/dbs/)
3. **Login** - Use your Method Active Directory credentials

### Step 2: Download Required Databases
```powershell
# Create download directory
New-Item -ItemType Directory -Path "C:\Methoddata\MethodIntegrationSQL" -Force

# Navigate to download location
Set-Location "C:\Methoddata\MethodIntegrationSQL"
```

**Download these key databases:**
- [TemplateMethodNewQBO.bak](https://dev-setup.methodintegration.com/dbs/templatemethodnewqbo/sql/FULL/)
- [TemplateDevV2.bak](https://dev-setup.methodintegration.com/dbs/templatedevv2/sql/FULL/)

## Automated Database Restoration

### Using restore_dbs.ps1 Script

```powershell
# Navigate to developer tools
Set-Location "C:\MethodDev\DeveloperTools\DevSetup\build_local"

# Run the restoration script
.\restore_dbs.ps1
```

**Interactive Prompts:**
1. **User databases restoration** - Type 'N' unless you need specific user databases
2. **Credentials** - Enter your Method Active Directory credentials when prompted
3. **Custom databases** - Create user_dbs.yaml if you need additional databases

### Custom Database List (user_dbs.yaml)
```yaml
# Create this file if you need additional user databases
databases:
  - database_name_1
  - database_name_2
  - specific_customer_db
```

## Manual Database Restoration

### SQL Server Database Restoration

```sql
-- Restore TemplateDevV2 database
USE master;
GO

RESTORE DATABASE TemplateDevV2 
FROM DISK = 'C:\Methoddata\MethodIntegrationSQL\TemplateDevV2.bak'
WITH 
    FILE = 1,
    MOVE 'TemplateDevV2' TO 'C:\Methoddata\MethodIntegrationSQL\TemplateDevV2.mdf',
    MOVE 'TemplateDevV2_Log' TO 'C:\Methoddata\MethodIntegrationSQL\TemplateDevV2_Log.ldf',
    NOUNLOAD,
    REPLACE,
    STATS = 5;
GO

-- Restore TemplateMethodNewQBO database
RESTORE DATABASE TemplateMethodNewQBO 
FROM DISK = 'C:\Methoddata\MethodIntegrationSQL\TemplateMethodNewQBO.bak'
WITH 
    FILE = 1,
    MOVE 'TemplateMethodNewQBO' TO 'C:\Methoddata\MethodIntegrationSQL\TemplateMethodNewQBO.mdf',
    MOVE 'TemplateMethodNewQBO_Log' TO 'C:\Methoddata\MethodIntegrationSQL\TemplateMethodNewQBO_Log.ldf',
    NOUNLOAD,
    REPLACE,
    STATS = 5;
GO
```

### PowerShell Database Restoration
```powershell
# Import SQL Server module
Import-Module SqlServer

# Define connection parameters
$ServerInstance = "localhost"
$BackupPath = "C:\Methoddata\MethodIntegrationSQL"

# Restore TemplateDevV2
$RestoreParams = @{
    ServerInstance = $ServerInstance
    Database = "TemplateDevV2"
    BackupFile = "$BackupPath\TemplateDevV2.bak"
    RelocateFile = @{
        'TemplateDevV2' = "$BackupPath\TemplateDevV2.mdf"
        'TemplateDevV2_Log' = "$BackupPath\TemplateDevV2_Log.ldf"
    }
    ReplaceDatabase = $true
}

Restore-SqlDatabase @RestoreParams

Write-Host "TemplateDevV2 restored successfully!" -ForegroundColor Green
```

## MongoDB Database Restoration

### Using MongoDB Tools
```powershell
# Ensure MongoDB is running
Start-Service MongoDB

# Restore MongoDB databases (if backup files are available)
$mongoBackupPath = "C:\Methoddata\MongoDB\Backups"

# Restore Accounts database
mongorestore --db Accounts "$mongoBackupPath\Accounts"

# Restore Authentication database
mongorestore --db Authentication "$mongoBackupPath\Authentication"

# Verify restoration
mongo --eval "db.adminCommand('listDatabases')"
```

## Troubleshooting Database Restoration

### Common Issues and Solutions

#### SQL Server Permission Errors
**Symptoms**: Access denied during restoration
**Solutions**:
```powershell
# Grant SQL Server service access to backup location
icacls "C:\Methoddata\MethodIntegrationSQL" /grant "NT SERVICE\MSSQLSERVER:(OI)(CI)F"

# Ensure SQL Server Agent is running
Start-Service SQLSERVERAGENT
```

#### PowerShell SQL Module Issues
**Symptoms**: SqlServer module not found
**Solutions**:
```powershell
# Install SQL Server PowerShell module
Install-Module -Name SqlServer -AllowClobber -Force

# Alternative if above fails
Install-Module -Name SqlServer -AllowClobber -Parameter @{ Force = $true }
```

#### TrustServerCertificate Parameter Error
**Symptoms**: Parameter not found error in SQL Server 2019+
**Solutions**:
```powershell
# Remove TrustServerCertificate parameter from script
# Or update to newer SQL PowerShell module
Update-Module SqlServer
```

#### Large Database Timeout Issues
**Symptoms**: Restoration times out for large databases
**Solutions**:
```sql
-- Increase timeout for large database restoration
RESTORE DATABASE LargeDatabase 
FROM DISK = 'C:\Methoddata\MethodIntegrationSQL\LargeDatabase.bak'
WITH 
    REPLACE,
    STATS = 10,
    BUFFERCOUNT = 100,
    MAXTRANSFERSIZE = 4194304;
```

### Manual SSMS Restoration
If scripts fail, use SQL Server Management Studio:

1. **Open SSMS** and connect to localhost
2. **Right-click Databases** → Restore Database
3. **Select Device** → Browse to backup file
4. **Options tab** → Check "Overwrite the existing database"
5. **Files tab** → Verify file paths are correct
6. **Click OK** to start restoration

## Database Verification

### Health Check Script
```powershell
function Test-MethodDatabases {
    Write-Host "=== Method Database Health Check ===" -ForegroundColor Green
    
    # Check SQL Server databases
    $sqlDatabases = @(
        'AlocetSystem',
        'Method App Store',
        'MethodNotifications', 
        'MethodOAuthSecurity',
        'MethodShortURLRedirects',
        'TemplateDevV2',
        'TemplateMethodNewQBO'
    )
    
    Write-Host "`nSQL Server Databases:" -ForegroundColor Yellow
    foreach ($db in $sqlDatabases) {
        try {
            $result = Invoke-Sqlcmd -ServerInstance "localhost" -Database $db -Query "SELECT 1 as Test" -ErrorAction Stop
            if ($result.Test -eq 1) {
                Write-Host "✓ $db - Accessible" -ForegroundColor Green
            }
        }
        catch {
            Write-Host "✗ $db - Error: $($_.Exception.Message)" -ForegroundColor Red
        }
    }
    
    # Check MongoDB databases
    Write-Host "`nMongoDB Databases:" -ForegroundColor Yellow
    try {
        $mongoDbs = mongo --quiet --eval "db.adminCommand('listDatabases').databases.forEach(function(db) { print(db.name); })"
        
        $expectedMongoDbs = @('Accounts', 'AlocetSystem', 'Authentication', 'Methodappstore', 'methodmobilenotifications', 'oauth_token', 'Preferences', 'TemplateDevV2')
        
        foreach ($db in $expectedMongoDbs) {
            if ($mongoDbs -contains $db) {
                Write-Host "✓ $db - Present" -ForegroundColor Green
            } else {
                Write-Host "✗ $db - Missing" -ForegroundColor Red
            }
        }
    }
    catch {
        Write-Host "✗ MongoDB - Connection failed" -ForegroundColor Red
    }
}

Test-MethodDatabases
```

### Database Size Check
```sql
-- Check database sizes
SELECT 
    name AS DatabaseName,
    size/128.0 AS CurrentSizeMB,
    size/128.0 - CAST(FILEPROPERTY(name, 'SpaceUsed') AS INT)/128.0 AS FreeSpaceMB
FROM sys.database_files
WHERE type IN (0,1);

-- Check all databases
EXEC sp_helpdb;
```

## Maintenance and Updates

### Regular Backup Updates
```powershell
# Schedule weekly database updates
function Update-MethodDatabases {
    Write-Host "Checking for database updates..." -ForegroundColor Yellow
    
    # Check modification dates on dev-setup server
    $latestBackups = @(
        @{Name="TemplateDevV2"; Url="https://dev-setup.methodintegration.com/dbs/templatedevv2/sql/FULL/"},
        @{Name="TemplateMethodNewQBO"; Url="https://dev-setup.methodintegration.com/dbs/templatemethodnewqbo/sql/FULL/"}
    )
    
    foreach ($backup in $latestBackups) {
        $localFile = "C:\Methoddata\MethodIntegrationSQL\$($backup.Name).bak"
        
        if (Test-Path $localFile) {
            $localDate = (Get-Item $localFile).LastWriteTime
            Write-Host "$($backup.Name) local backup: $localDate"
            
            # Compare with server version if needed
            # Download newer version if available
        } else {
            Write-Host "$($backup.Name) - No local backup found" -ForegroundColor Yellow
        }
    }
}
```

### Database Cleanup
```sql
-- Shrink database logs after restoration
USE TemplateDevV2;
GO
DBCC SHRINKFILE (N'TemplateDevV2_Log' , 100);
GO

-- Update database compatibility level
ALTER DATABASE TemplateDevV2 SET COMPATIBILITY_LEVEL = 150;
GO

-- Update statistics
EXEC sp_updatestats;
GO
```

## Backup Strategy

### Local Backup Creation
```powershell
function Backup-MethodDatabases {
    $backupDate = Get-Date -Format "yyyy-MM-dd"
    $backupPath = "D:\MethodData\Backups\SQL\$backupDate"
    
    New-Item -ItemType Directory -Path $backupPath -Force
    
    $databases = @('TemplateDevV2', 'TemplateMethodNewQBO', 'AlocetSystem')
    
    foreach ($db in $databases) {
        $backupFile = "$backupPath\$db.bak"
        
        Backup-SqlDatabase -ServerInstance "localhost" -Database $db -BackupFile $backupFile
        
        Write-Host "Backed up $db to $backupFile" -ForegroundColor Green
    }
}
```

### Automated Restoration Script
```powershell
# Complete automated restoration
function Start-MethodDatabaseSetup {
    param(
        [switch]$DownloadLatest = $false,
        [switch]$RestoreAll = $true
    )
    
    Write-Host "=== Method Database Setup ===" -ForegroundColor Green
    
    if ($DownloadLatest) {
        Write-Host "Downloading latest database backups..." -ForegroundColor Yellow
        # Call download script
        & "C:\MethodDev\DeveloperTools\DevSetup\build_local\restore_dbs.ps1"
    }
    
    if ($RestoreAll) {
        Write-Host "Restoring databases..." -ForegroundColor Yellow
        
        # Restore critical databases
        $databases = @(
            @{Name="TemplateDevV2"; File="TemplateDevV2.bak"},
            @{Name="TemplateMethodNewQBO"; File="TemplateMethodNewQBO.bak"}
        )
        
        foreach ($db in $databases) {
            try {
                $backupFile = "C:\Methoddata\MethodIntegrationSQL\$($db.File)"
                
                if (Test-Path $backupFile) {
                    Restore-SqlDatabase -ServerInstance "localhost" -Database $db.Name -BackupFile $backupFile -ReplaceDatabase
                    Write-Host "✓ $($db.Name) restored successfully" -ForegroundColor Green
                } else {
                    Write-Host "✗ Backup file not found: $backupFile" -ForegroundColor Red
                }
            }
            catch {
                Write-Host "✗ Failed to restore $($db.Name): $($_.Exception.Message)" -ForegroundColor Red
            }
        }
    }
    
    # Verify restoration
    Test-MethodDatabases
}

# Run complete setup
Start-MethodDatabaseSetup -DownloadLatest -RestoreAll
```

**Next**: [Appendix - IIS Features](./appendix-iis.md)
