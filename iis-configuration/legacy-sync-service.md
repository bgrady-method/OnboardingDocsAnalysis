# Legacy Sync Service API Configuration

## 11.4. Legacy-syncservice-api project configuration

This step configures the legacy sync service API project for compatibility with older Method integrations.

### Prerequisites
- Legacy sync service API repository cloned to C:\MethodDev\legacy-syncservice-api
- Method Platform UI repository cloned to C:\MethodDev\method-platform-ui
- IIS sites and application pools already configured

### Configuration Steps

1. **Check Configuration Directory**
   - Navigate to: `C:\MethodDev\legacy-syncservice-api\SiteConfig`
   - Verify the directory exists and contains configuration files

2. **Copy Method.config File**
   - Source location: `C:\MethodDev\method-platform-ui\SiteConfig\Debug\Method.config`
   - Destination: `C:\MethodDev\legacy-syncservice-api\SiteConfig\Method.config`
   - Copy the Method.config file from MethodUI to the legacy sync service directory

### What This Configuration Does

The Method.config file contains:
- Database connection strings for legacy integrations
- API endpoint configurations
- Authentication settings for legacy services
- Logging and debugging parameters

### Verification

After configuration:
1. Check that the Method.config file exists in the legacy sync service SiteConfig directory
2. Verify the legacy sync service API starts without configuration errors
3. Test legacy sync endpoints are accessible

### Troubleshooting

**Configuration directory not found:**
- Ensure the legacy-syncservice-api repository is properly cloned
- Check if the project structure has changed in recent versions
- Contact your team lead if the directory structure is unexpected

**Method.config file missing from source:**
- Verify the method-platform-ui repository is fully cloned
- Check the Debug folder exists: `C:\MethodDev\method-platform-ui\SiteConfig\Debug\`
- Build the method-platform-ui project if the Debug folder is missing

**Legacy service fails to start:**
- Check IIS application pool is running for the legacy service
- Verify database connections in Method.config are correct
- Review error logs in D:\logs for specific configuration issues

**Access permission errors:**
- Ensure the IIS application pool identity has read access to the config file
- Check that NETWORK SERVICE has appropriate permissions
- Run IIS with elevated privileges if needed

### Legacy Service Context

The legacy sync service API provides backward compatibility for:
- Older QuickBooks integrations
- Legacy mobile applications
- Third-party systems using deprecated API endpoints
- Historical data synchronization processes

### Security Note

This legacy service should only be enabled in development environments. Production environments should use the modern API endpoints whenever possible for better security and performance.

**Next:** [HTTPS Binding & SSL Certificates](./ssl-certificates.md)
