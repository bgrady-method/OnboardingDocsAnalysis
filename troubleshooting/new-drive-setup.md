# New Hard Drive Setup

## Setting Up Additional Storage for Method Development

Method development can require significant disk space for databases, logs, and application files. This guide covers setting up additional storage.

## Why Additional Storage?

### Disk Space Requirements
- **SQL Server Databases** - Several GB for Method databases
- **Application Logs** - Can grow to GB over time (D:\logs\)
- **Docker Images** - Container images and volumes
- **Build Artifacts** - Compilation outputs and caches
- **Source Code** - Multiple Method repositories

### Recommended Setup
- **C: Drive** - Operating system and core applications
- **D: Drive** - Method application data, logs, and databases
- **E: Drive** (Optional) - Additional storage for archives and backups

## D: Drive Setup Process

### Step 1: Create D: Drive Partition
```powershell
# Check available disks
Get-Disk

# Initialize new disk (if adding physical drive)
Initialize-Disk -Number 1 -PartitionStyle GPT

# Create partition using all available space
New-Partition -DiskNumber 1 -UseMaximumSize -DriveLetter D

# Format the drive
Format-Volume -DriveLetter D -FileSystem NTFS -NewFileSystemLabel "Method Data" -Confirm:$false
```

### Step 2: Create Method Directory Structure
```powershell
# Create main Method directories on D: drive
$directories = @(
    "D:\logs",
    "D:\MethodData",
    "D:\MethodData\MethodIntegrationSQL",
    "D:\MethodData\Backups",
    "D:\MethodData\Archives",
    "D:\temp"
)

foreach ($dir in $directories) {
    New-Item -ItemType Directory -Path $dir -Force
    Write-Host "Created: $dir" -ForegroundColor Green
}
```

### Step 3: Set Permissions
```powershell
# Grant Network Service full control to Method directories
$paths = @("D:\logs", "D:\MethodData")

foreach ($path in $paths) {
    icacls $path /grant "NETWORK SERVICE:(OI)(CI)F"
    icacls $path /grant "IIS_IUSRS:(OI)(CI)M"
    Write-Host "Permissions set for: $path" -ForegroundColor Green
}

# Grant SQL Server service access
icacls "D:\MethodData\MethodIntegrationSQL" /grant "NT SERVICE\MSSQLSERVER:(OI)(CI)F"
```

## Moving Existing Data

### Relocating SQL Server Data Files
```sql
-- Move SQL Server databases to D: drive
USE master;
GO

-- Take database offline
ALTER DATABASE TemplateDevV2 SET OFFLINE;

-- Move files using PowerShell
```

```powershell
# Move database files
$sourcePath = "C:\Program Files\Microsoft SQL Server\MSSQL15.MSSQLSERVER\MSSQL\DATA\"
$destPath = "D:\MethodData\SQL\"

# Create destination directory
New-Item -ItemType Directory -Path $destPath -Force

# Move files
Move-Item "$sourcePath\TemplateDevV2.mdf" "$destPath\TemplateDevV2.mdf"
Move-Item "$sourcePath\TemplateDevV2_Log.ldf" "$destPath\TemplateDevV2_Log.ldf"
```

```sql
-- Update file paths and bring database online
ALTER DATABASE TemplateDevV2 MODIFY FILE (NAME = TemplateDevV2, FILENAME = 'D:\MethodData\SQL\TemplateDevV2.mdf');
ALTER DATABASE TemplateDevV2 MODIFY FILE (NAME = TemplateDevV2_Log, FILENAME = 'D:\MethodData\SQL\TemplateDevV2_Log.ldf');
ALTER DATABASE TemplateDevV2 SET ONLINE;
GO
```

### Relocating Application Logs
```powershell
# Create symbolic link for logs if they're currently on C:
if (Test-Path "C:\logs") {
    # Stop services that write to logs
    Stop-Service -Name "W3SVC" -Force
    
    # Move existing logs
    robocopy "C:\logs" "D:\logs" /MIR /COPYALL
    
    # Remove old directory and create symbolic link
    Remove-Item "C:\logs" -Recurse -Force
    New-Item -ItemType SymbolicLink -Path "C:\logs" -Target "D:\logs"
    
    # Restart services
    Start-Service -Name "W3SVC"
    
    Write-Host "Logs relocated to D:\logs with symbolic link" -ForegroundColor Green
}
```

## Disk Space Management

### Monitoring Disk Usage
```powershell
function Get-MethodDiskUsage {
    Write-Host "=== Method Development Disk Usage ===" -ForegroundColor Green
    
    # Overall disk space
    Get-WmiObject -Class Win32_LogicalDisk | Where-Object { $_.DriveType -eq 3 } | ForEach-Object {
        $size = [math]::Round($_.Size / 1GB, 2)
        $free = [math]::Round($_.FreeSpace / 1GB, 2)
        $used = $size - $free
        $percentFree = [math]::Round(($free / $size) * 100, 1)
        
        Write-Host "`n$($_.DeviceID) Drive:" -ForegroundColor Yellow
        Write-Host "  Total: ${size} GB" 
        Write-Host "  Used:  ${used} GB"
        Write-Host "  Free:  ${free} GB ($percentFree%)"
    }
    
    # Method-specific directories
    $methodPaths = @(
        "D:\logs",
        "D:\MethodData",
        "C:\MethodDev"
    )
    
    Write-Host "`nMethod Directory Sizes:" -ForegroundColor Yellow
    foreach ($path in $methodPaths) {
        if (Test-Path $path) {
            $size = (Get-ChildItem -Path $path -Recurse -ErrorAction SilentlyContinue | 
                    Measure-Object -Property Length -Sum).Sum / 1GB
            Write-Host "  $path : $([math]::Round($size, 2)) GB"
        }
    }
}

Get-MethodDiskUsage
```

### Automated Cleanup Script
```powershell
function Start-MethodCleanup {
    param(
        [int]$LogRetentionDays = 30,
        [int]$TempFileAgeDays = 7,
        [switch]$DryRun = $false
    )
    
    Write-Host "=== Method Cleanup Process ===" -ForegroundColor Green
    
    if ($DryRun) {
        Write-Host "DRY RUN MODE - No files will be deleted" -ForegroundColor Yellow
    }
    
    # Clean old log files
    $logPath = "D:\logs"
    if (Test-Path $logPath) {
        $oldLogs = Get-ChildItem -Path $logPath -Recurse -File | 
                   Where-Object { $_.LastWriteTime -lt (Get-Date).AddDays(-$LogRetentionDays) }
        
        $logSize = ($oldLogs | Measure-Object -Property Length -Sum).Sum / 1MB
        Write-Host "Found $($oldLogs.Count) old log files ($([math]::Round($logSize, 2)) MB)"
        
        if (-not $DryRun -and $oldLogs.Count -gt 0) {
            $oldLogs | Remove-Item -Force
            Write-Host "Deleted old log files" -ForegroundColor Green
        }
    }
    
    # Clean temp files
    $tempPaths = @("D:\temp", "C:\temp")
    foreach ($tempPath in $tempPaths) {
        if (Test-Path $tempPath) {
            $oldTemp = Get-ChildItem -Path $tempPath -Recurse -File | 
                      Where-Object { $_.LastWriteTime -lt (Get-Date).AddDays(-$TempFileAgeDays) }
            
            if ($oldTemp.Count -gt 0) {
                $tempSize = ($oldTemp | Measure-Object -Property Length -Sum).Sum / 1MB
                Write-Host "Found $($oldTemp.Count) old temp files in $tempPath ($([math]::Round($tempSize, 2)) MB)"
                
                if (-not $DryRun) {
                    $oldTemp | Remove-Item -Force -ErrorAction SilentlyContinue
                    Write-Host "Deleted old temp files from $tempPath" -ForegroundColor Green
                }
            }
        }
    }
    
    # Clean NuGet cache (optional)
    $nugetCache = "$env:USERPROFILE\.nuget\packages"
    if (Test-Path $nugetCache) {
        $cacheSize = (Get-ChildItem -Path $nugetCache -Recurse | 
                     Measure-Object -Property Length -Sum).Sum / 1GB
        Write-Host "NuGet cache size: $([math]::Round($cacheSize, 2)) GB"
        
        if ($cacheSize -gt 5) {  # If cache is larger than 5GB
            Write-Host "Consider running: dotnet nuget locals all --clear" -ForegroundColor Yellow
        }
    }
    
    Write-Host "Cleanup process completed!" -ForegroundColor Green
}

# Run cleanup (add -DryRun to preview)
Start-MethodCleanup -LogRetentionDays 30 -TempFileAgeDays 7
```

## Backup and Archive Strategy

### Creating Backup Locations
```powershell
# Set up backup directory structure
$backupPaths = @(
    "D:\MethodData\Backups\SQL",
    "D:\MethodData\Backups\Logs", 
    "D:\MethodData\Backups\Config",
    "D:\MethodData\Archives\Monthly",
    "D:\MethodData\Archives\Projects"
)

foreach ($path in $backupPaths) {
    New-Item -ItemType Directory -Path $path -Force
}
```

### Automated Backup Script
```powershell
function Start-MethodBackup {
    $backupDate = Get-Date -Format "yyyy-MM-dd"
    $backupRoot = "D:\MethodData\Backups"
    
    Write-Host "Starting Method backup for $backupDate" -ForegroundColor Green
    
    # Backup SQL databases
    $sqlBackupPath = "$backupRoot\SQL\$backupDate"
    New-Item -ItemType Directory -Path $sqlBackupPath -Force
    
    # Critical configuration files
    $configFiles = @(
        "C:\Windows\System32\drivers\etc\hosts",
        "$env:USERPROFILE\.gitconfig",
        "$env:APPDATA\NuGet\NuGet.Config"
    )
    
    $configBackupPath = "$backupRoot\Config\$backupDate"
    New-Item -ItemType Directory -Path $configBackupPath -Force
    
    foreach ($file in $configFiles) {
        if (Test-Path $file) {
            $fileName = Split-Path $file -Leaf
            Copy-Item $file "$configBackupPath\$fileName" -Force
        }
    }
    
    # Compress old backups
    $oldBackups = Get-ChildItem "$backupRoot" -Directory | 
                  Where-Object { $_.Name -match "\d{4}-\d{2}-\d{2}" -and 
                                $_.Name -lt (Get-Date).AddDays(-7).ToString("yyyy-MM-dd") }
    
    foreach ($backup in $oldBackups) {
        $zipPath = "$($backup.FullName).zip"
        if (-not (Test-Path $zipPath)) {
            Compress-Archive -Path $backup.FullName -DestinationPath $zipPath
            Remove-Item $backup.FullName -Recurse -Force
            Write-Host "Compressed and removed: $($backup.Name)" -ForegroundColor Yellow
        }
    }
    
    Write-Host "Backup completed successfully!" -ForegroundColor Green
}
```

## Performance Optimization

### SSD vs HDD Considerations
```powershell
# Check drive types
Get-PhysicalDisk | Select-Object DeviceID, FriendlyName, MediaType, Size

# Optimize based on drive type
function Optimize-MethodDrives {
    $drives = Get-WmiObject -Class Win32_LogicalDisk | Where-Object { $_.DriveType -eq 3 }
    
    foreach ($drive in $drives) {
        $driveLetter = $drive.DeviceID.Substring(0,1)
        
        # Check if SSD
        $physicalDisk = Get-PhysicalDisk | Where-Object { 
            (Get-Partition -DriveLetter $driveLetter).DiskNumber -eq $_.DeviceID 
        }
        
        if ($physicalDisk.MediaType -eq "SSD") {
            Write-Host "$($drive.DeviceID) is SSD - Optimizing for SSD" -ForegroundColor Green
            
            # Disable indexing for data drives (keep for C:)
            if ($driveLetter -ne "C") {
                fsutil behavior set DisableLastAccess 1
            }
            
            # Enable TRIM
            fsutil behavior set DisableDeleteNotify 0
        }
        else {
            Write-Host "$($drive.DeviceID) is HDD - Optimizing for HDD" -ForegroundColor Yellow
            
            # Enable prefetching
            fsutil behavior set DisableLastAccess 0
        }
    }
}
```

### RAID Configuration (Optional)
For high-performance setups with multiple drives:

```powershell
# Check for RAID capabilities
Get-StoragePool
Get-VirtualDisk

# Create storage pool and virtual disk (example)
# New-StoragePool -FriendlyName "MethodPool" -StorageSubSystemFriendlyName "Storage Spaces*" -PhysicalDisks (Get-PhysicalDisk -CanPool $true)
# New-VirtualDisk -StoragePoolFriendlyName "MethodPool" -FriendlyName "MethodVirtualDisk" -ResiliencySettingName Mirror -UseMaximumSize
```

## Troubleshooting Drive Issues

### Common Problems

#### Drive Not Accessible
```powershell
# Check drive status
Get-Disk
Get-Volume

# Check for errors
chkdsk D: /f /r

# Check disk health
Get-StorageReliabilityCounter -PhysicalDisk (Get-PhysicalDisk)
```

#### Permission Issues
```powershell
# Reset permissions on Method directories
$paths = @("D:\logs", "D:\MethodData")
foreach ($path in $paths) {
    takeown /f $path /r /d y
    icacls $path /reset /t
    icacls $path /grant "NETWORK SERVICE:(OI)(CI)F"
    icacls $path /grant "IIS_IUSRS:(OI)(CI)M"
}
```

#### Performance Issues
```powershell
# Check disk performance
Get-Counter "\PhysicalDisk(*)\Avg. Disk Queue Length" -Continuous

# Check fragmentation
Optimize-Volume -DriveLetter D -Analyze
Optimize-Volume -DriveLetter D -Defrag
```

## Maintenance Tasks

### Weekly Maintenance
```powershell
# Schedule weekly maintenance
$action = New-ScheduledTaskAction -Execute "PowerShell.exe" -Argument "-File C:\Scripts\MethodMaintenance.ps1"
$trigger = New-ScheduledTaskTrigger -Weekly -DaysOfWeek Sunday -At 2AM
$settings = New-ScheduledTaskSettingsSet -AllowStartIfOnBatteries -DontStopIfGoingOnBatteries
Register-ScheduledTask -Action $action -Trigger $trigger -Settings $settings -TaskName "Method Weekly Maintenance" -Description "Weekly maintenance for Method development environment"
```

### Health Monitoring
```powershell
function Test-MethodDriveHealth {
    Write-Host "=== Method Drive Health Check ===" -ForegroundColor Green
    
    # Check available space
    $drives = @("C:", "D:")
    foreach ($drive in $drives) {
        $disk = Get-WmiObject -Class Win32_LogicalDisk -Filter "DeviceID='$drive'"
        $freeSpaceGB = [math]::Round($disk.FreeSpace / 1GB, 2)
        $totalSpaceGB = [math]::Round($disk.Size / 1GB, 2)
        $percentFree = [math]::Round(($disk.FreeSpace / $disk.Size) * 100, 1)
        
        if ($percentFree -lt 10) {
            Write-Host "$drive Low space warning: $freeSpaceGB GB free ($percentFree%)" -ForegroundColor Red
        } elseif ($percentFree -lt 20) {
            Write-Host "$drive Space concern: $freeSpaceGB GB free ($percentFree%)" -ForegroundColor Yellow
        } else {
            Write-Host "$drive Space OK: $freeSpaceGB GB free ($percentFree%)" -ForegroundColor Green
        }
    }
    
    # Check critical directories
    $criticalPaths = @("D:\logs", "D:\MethodData\MethodIntegrationSQL")
    foreach ($path in $criticalPaths) {
        if (Test-Path $path) {
            Write-Host "✓ $path - Accessible" -ForegroundColor Green
        } else {
            Write-Host "✗ $path - Not accessible" -ForegroundColor Red
        }
    }
}

Test-MethodDriveHealth
```

**Next**: [Download and Restore Databases](./database-restore.md)
