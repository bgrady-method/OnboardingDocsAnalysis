# Important Prerequisites

This guide covers the essential prerequisites that must be completed before attempting to run Method applications locally, based on the Method Developer Machine Setup documentation.

## Overview

Before starting Method applications in your local development environment, several critical components must be properly configured and running. This checklist ensures all dependencies are met for successful local development.

## Essential Prerequisites Checklist

### 1. Previous Setup Sections Completed

Ensure you have completed all previous sections of the setup guide:

- [x] **Prerequisites:** Windows updates, VPN, D: drive setup
- [x] **Credentials:** Active Directory and MSDN credentials configured
- [x] **GitHub:** Account setup, Git installation, SSH keys, personal access token
- [x] **Software Installation:** All required software installed via initialize.ps1
- [x] **SQL Server:** Installation, aliases, SSMS, permissions, and database backups
- [x] **Microsoft Tools:** Visual Studio, NuGet feeds, credentials, NUnit, URL Rewrite
- [x] **IIS Configuration:** Features enabled, sites and app pools imported
- [x] **Certificates:** SSL certificates downloaded and imported
- [x] **NetCore Requirements:** Environment variables and AWS credentials

### 2. Service Status Verification

#### Check IIS Application Pools and Sites

1. **Open IIS Manager**
2. **Restart IIS** to ensure fresh state
3. **Verify all Application Pools are running:**
   - Method application pools should show "Started" status
   - If any pools are stopped, start them manually

4. **Verify all Sites/Apps are running:**
   - Check that Method sites are in "Started" state
   - Restart any stopped sites

#### Check Task Manager Services

**Open Task Manager → Services tab**

Verify the following services are running:
- **RabbitMQ** - Message queue service
- **Redis** - Caching service  
- **MongoDB** - Database service
- **MySQL** - Database service (if applicable)

If any services are stopped, start them manually.

#### Check SQL Server Services

**Open SQL Server Configuration Manager → SQL Server Services**

Verify the following are running:
- **SQL Server (MSSQLSERVER)** - Database engine
- **SQL Server Agent** - Job scheduling service
- **SQL Full-text Filter** - Search service

If any services are stopped, start them manually.

### 3. Network Configuration

#### VPN Connection

1. **Connect to Method VPN** (if working remotely)
2. **Verify connection** by accessing internal Method resources

#### Host File Entries

Ensure your hosts file contains Method local URLs:

**File Location:** `C:\Windows\System32\drivers\etc\hosts`

**Required entries** (from the original setup document):
```
127.0.0.1 methodlocaldb
127.0.0.1 methodlocal
127.0.0.1 methodlocal.com
127.0.0.1 www.methodlocal.com
127.0.0.1 gateway.methodlocal.com
127.0.0.1 gateway.methodlocal.int
127.0.0.1 subscriber.methodlocal.com
127.0.0.1 runtime.methodlocal.com
127.0.0.1 microservices.methodlocal.int
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
```

### 4. Required Folder Structure

Create the required blank folder:

**Create folder:** `C:\MethodDev\Blank`

This folder is required for Method applications to function properly.

## Quick Verification Commands

### Check Application Pool Status
```powershell
# Open PowerShell as Administrator
Import-Module WebAdministration
Get-IISAppPool | Where-Object {$_.Name -like "*Method*"} | Select-Object Name, State
```

### Check SQL Server Status
```powershell
# Check SQL Server services
Get-Service | Where-Object {$_.Name -like "*SQL*"} | Select-Object Name, Status
```

### Check Docker Containers
```powershell
# Check Docker containers (if using Docker for services)
docker ps -a
```

## Common Issues and Solutions

### Application Pools Keep Stopping
- **Check application logs** in D:\logs\
- **Verify .NET Framework** versions are correct
- **Check file permissions** for Method directories
- **Restart IIS** completely: `iisreset`

### SQL Server Connection Issues
- **Verify SQL Server services** are running
- **Check SQL aliases** are configured correctly
- **Test connection** using SQL Server Management Studio
- **Verify methodlocaldb** entry in hosts file

### Docker Services Not Running
- **Start Docker Desktop** application
- **Wait for Docker** to fully initialize
- **Run docker commands** to check container status
- **Restart containers** if needed

### VPN Connectivity Issues
- **Disconnect and reconnect** VPN
- **Check VPN credentials** in Passbolt
- **Test access** to internal Method resources
- **Contact IT support** if persistent issues

## Verification Process

Before proceeding to run Method applications:

1. **Complete this checklist** - All items must be verified
2. **Test basic connectivity** - Access a few Method internal URLs
3. **Check service logs** - Look for obvious errors in D:\logs\
4. **Restart services if needed** - Fresh start often resolves issues
5. **Contact team members** if issues persist

## Next Steps

After verifying all prerequisites:

1. **Critical Projects Health Check:** [Critical Projects](./critical-projects.md)
2. **Create Local Account:** [Create Account](./create-account.md)
3. **Optional Services:** [SMS Notifications](./sms-notifications.md)

## Getting Help

If you encounter issues:

- **Check the troubleshooting section** of this guide
- **Review logs** in D:\logs\ for specific error messages
- **Ask team members** for assistance with setup
- **Reference the original setup document** for additional details

**Back to:** [Local Development](./README.md)
