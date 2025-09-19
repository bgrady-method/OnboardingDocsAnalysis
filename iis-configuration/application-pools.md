# IIS Application Pool Configuration

This guide covers configuring IIS application pools for Method's development environment, based on the Method Developer Machine Setup documentation.

## Overview

Method's local development environment uses IIS application pools to host various services and applications. This guide covers the basic setup and configuration needed for Method development.

## Prerequisites

- IIS installed with required features enabled
- Method development environment set up
- Administrative access to configure IIS

## Application Pool Setup

### Automated Configuration

Method provides automated scripts to set up IIS application pools and sites:

1. **Open PowerShell as Administrator**
2. **Navigate to:** `C:\MethodDev\DeveloperTools\DevSetup\build_local`
3. **Run:** `import_sites_pools.ps1`

This script will:
- Replace your current IIS setup with standard sites and app pools
- Configure all necessary application pools for Method services
- Set up proper bindings and configurations

### Credentials Required

When prompted for credentials, use:

- **Username:** Your **Local Windows Account Username** (must have Admin access)
- **Password:** Your **Local Windows Account Password**

### Finding Your Username

If you don't know your account username:

**Windows 10+:**
- Press **Windows Key + R**
- Enter: `netplwiz`
- Username should be displayed

**Older Versions:**
- Press Windows Key → Search for "**Computer Management**"
- Expand "**Local Users and Groups**" → Select "**Users**"
- Identify the account with your name
- Right-click → **Properties** → **Member of** Tab
- Make sure your account is a **member of Administrators**

## Method Application Pools

After running the import script, you should have the following application pools configured:

### Core Method Services
- **Method-UI** - Main Method user interface
- **Method-API** - API services
- **Method-Gateway** - API Gateway
- **Method-Authentication** - Authentication services
- **Method-Runtime** - Runtime core services
- **Method-Portal** - Administrative portal

### Application Pool Settings

The automated script configures pools with:
- **.NET Framework Version:** v4.0
- **Pipeline Mode:** Integrated
- **Identity:** ApplicationPoolIdentity (or custom account as specified)
- **Appropriate recycling conditions**
- **Memory limits based on service requirements**

## Manual Configuration (If Needed)

If you need to manually configure application pools:

### Basic Settings
1. **Open IIS Manager**
2. **Navigate to Application Pools**
3. **Right-click** → **Add Application Pool**
4. **Configure:**
   - Name: Method service name
   - .NET CLR Version: .NET CLR Version v4.0
   - Managed pipeline mode: Integrated

### Advanced Settings
- **Identity:** ApplicationPoolIdentity (recommended for development)
- **Idle Timeout:** Adjust based on service type
- **Maximum Worker Processes:** Usually 1 for development
- **Recycling Conditions:** Configure memory and time limits

## SSL Configuration

After setting up application pools, configure SSL certificates:

1. **Download Method Certificate** (expires Dec 4, 2025)
2. **Import to Personal and Trusted Root stores**
3. **Configure HTTPS bindings** for Method sites
4. **Select the Method certificate** for SSL bindings

## Troubleshooting

### Common Issues

**Application Pool Stops:**
- Check event logs for errors
- Verify .NET Framework version compatibility
- Check file permissions for application directories

**Authentication Issues:**
- Verify application pool identity has necessary permissions
- Check that Method directories have appropriate ACLs
- Ensure database connection permissions are correct

**Memory/Performance Issues:**
- Monitor application pool memory usage
- Adjust recycling conditions if needed
- Check for memory leaks in applications

### Verification

To verify application pools are working:

1. **Check IIS Manager:** All pools should show "Started" status
2. **Test Health Endpoints:** Access Method service health check URLs
3. **Monitor Event Logs:** Look for any application pool errors
4. **Check Worker Processes:** Verify processes are running for active pools

## Integration with Method Setup

The application pool configuration integrates with:

- **Method Sites:** Each service has corresponding IIS sites
- **SSL Certificates:** HTTPS bindings use Method certificates
- **Authentication:** Configured for Method's authentication flow
- **Development Scripts:** Works with Method's build and deployment scripts

## Next Steps

After configuring application pools:

1. **Configure Virtual Directories:** [Virtual Directory Setup](./virtual-directories.md)
2. **Set Security Settings:** [IIS Security Configuration](./security-settings.md)
3. **Optimize Performance:** [Performance Tuning](./performance-tuning.md)
4. **Test Method Services:** Verify all health check endpoints

## Best Practices

- **Use automated scripts** when possible for consistency
- **Monitor application pool health** regularly
- **Keep pools isolated** for better debugging and resource management
- **Document any custom configurations** for team reference
- **Regular maintenance** to keep pools running optimally

**Back to:** [IIS Configuration](./README.md)
