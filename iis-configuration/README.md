# IIS Configuration

This section covers complete Internet Information Services (IIS) setup.

## Sections

1. [Enable IIS Features](../troubleshooting/appendix-iis.md)
2. [Export Registry Key - Salt](./registry-salt.md) ⚠️ *Coming Soon*
3. [Import IIS Sites & App Pools](./application-pools.md)
4. [Legacy Sync Service API Configuration](./legacy-sync-service.md) ⚠️ *Coming Soon*
5. [HTTPS Binding & SSL Certificates](./ssl-certificates.md)

### Available Advanced Configuration

- [Application Pools Configuration](./application-pools.md) - Comprehensive PowerShell automation
- [Performance Tuning](./performance-tuning.md) - Enterprise performance optimization  
- [Security Settings](./security-settings.md) - Advanced security configuration
- [SSL Certificates](./ssl-certificates.md) - SSL/TLS certificate management
- [Virtual Directories](./virtual-directories.md) - Directory configuration

IIS configuration is critical for hosting Method web applications locally.

## Overview

Internet Information Services (IIS) is the web server platform used to host Method's web applications locally. This section covers:

- **Feature Installation** - Enabling required IIS components and features
- **Security Configuration** - Registry keys and encryption settings
- **Site Management** - Importing pre-configured sites and application pools
- **Legacy Support** - Configuration for older sync service components
- **SSL Setup** - HTTPS bindings and certificate configuration

⏱️ **Estimated Time:** 1 hour

📋 **Prerequisites:**
- Windows Professional, Enterprise, or Education edition
- Administrator access for Windows features and IIS management
- SSL certificates configured (see Certificates section)
- Method developer tools repository cloned

⚠️ **Important:** IIS must be properly configured before attempting to run Method web applications locally

🔧 **Tools Required:**
- IIS Manager
- PowerShell (Administrator mode)
- Method's automated configuration scripts
