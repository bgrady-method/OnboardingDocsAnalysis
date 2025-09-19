# Add Certificates to IIS

This guide covers configuring SSL certificates in Internet Information Services (IIS) for Method applications.

## Overview

After importing certificates into the Windows certificate store, they must be bound to websites in IIS to enable HTTPS functionality. Almost every site in Method requires HTTPS binding for SSL.

## Prerequisites

- SSL certificate imported into Windows certificate store ([Import Certificates](./import-certificates.md))
- IIS Manager access with administrative privileges
- Method applications configured in IIS (completed via import_sites_pools.ps1 script)
- IIS features enabled and URL Rewrite module installed

## Adding HTTPS Bindings

### Step 1: Open IIS Manager

1. **Launch IIS Manager**
   - Press `Windows + R`
   - Type `inetmgr`
   - Press Enter

2. **Navigate to Sites**
   - Expand the server node in the left panel
   - Click on "Sites" to view all configured websites

### Step 2: Configure HTTPS Bindings for Each Site

For each Method site in IIS, you need to add or update HTTPS bindings:

1. **Select Method Site**
   - Right-click on the Method application site
   - Select "Edit Bindings..."

2. **Add/Edit HTTPS Binding**
   - If no HTTPS binding exists: Click "Add..." button
   - If HTTPS binding exists: Select it and click "Edit..."
   
3. **Configure Binding Settings**
   - Set **Type**: `https`
   - Set **Port**: `443` (standard HTTPS port)
   - Set **Host name**: Appropriate Method domain (e.g., `methodlocal.com`, `api.methodlocal.com`)
   - Select **SSL certificate**: Choose `*.methodlocal.com` from dropdown
   - Click "OK"

## Method Sites Requiring HTTPS Bindings

Configure HTTPS bindings for these Method sites (as configured by import_sites_pools.ps1):

- **methodlocal.com** - Main Method application
- **api.methodlocal.com** - API gateway
- **auth.methodlocal.com** - Authentication service
- **signup.methodlocal.com** - Signup application
- **signin.methodlocal.com** - Signin application
- **services.methodlocal.com** - Legacy services
- **migration.methodlocal.com** - Migration service
- **microservices.methodlocal.int** - Internal microservices
- **Other Method sites** as configured in your environment

## Automated Certificate Assignment

**Important Note**: Once you set up your SSL certificate correctly for one site, it often gets reflected automatically in other sites with HTTPS bindings. However, you should verify each site individually.

### Bulk Update Process

1. **Configure First Site**
   - Start with the main `methodlocal.com` site
   - Add HTTPS binding with `*.methodlocal.com` certificate

2. **Check Other Sites**
   - Review each Method site's bindings
   - Verify the certificate is properly assigned
   - Update any sites that didn't get automatically updated

3. **IIS Prompt for Global Update**
   - If prompted "Update all certificates?" click "Yes"
   - This will apply the certificate to all compatible bindings

## SSL Settings Configuration

### Configure SSL Requirements

For each Method site:

1. **Open SSL Settings**
   - Select the Method site in IIS Manager
   - Double-click "SSL Settings" in the main panel

2. **Configure SSL Options**
   - Check "Require SSL"
   - Select appropriate client certificate requirement:
     - **"Ignore"** for development environments
     - **"Accept"** or **"Require"** for enhanced security

## Verification

### Test HTTPS Connectivity

1. **Browser Testing**
   - Navigate to `https://methodlocal.com`
   - Verify certificate is trusted (no browser warnings)
   - Check certificate details in browser (should show *.methodlocal.com)

2. **Test Multiple Sites**
   ```cmd
   # Test various Method sites
   curl -k https://methodlocal.com
   curl -k https://api.methodlocal.com
   curl -k https://auth.methodlocal.com
   ```

3. **PowerShell Testing**
   ```powershell
   # Test SSL connection
   $request = [System.Net.WebRequest]::Create("https://methodlocal.com")
   $response = $request.GetResponse()
   $response.StatusCode
   ```

### Certificate Binding Verification

```powershell
# List all SSL bindings
netsh http show sslcert

# Check specific binding
netsh http show sslcert ipport=0.0.0.0:443

# Get IIS site bindings
Get-WebBinding -Name "*method*"
```

## Common Issues and Solutions

### Certificate Not Available in Dropdown

**Problem**: Certificate doesn't appear in SSL certificate dropdown

**Solutions**:
- Verify certificate is in Local Machine store (not Current User)
- Check certificate has private key (key icon visible in MMC)
- Restart IIS Manager
- Verify certificate hasn't expired
- Confirm certificate is properly imported to Personal store

### SSL Handshake Failures

**Problem**: Browser shows SSL errors or connection failures

**Solutions**:
- Check certificate chain validation in MMC
- Verify intermediate certificates are installed
- Confirm certificate subject matches host name
- Ensure certificate is in Trusted Root store

### Mixed Content Warnings

**Problem**: Pages load but show mixed content warnings

**Solutions**:
- Ensure all resources load over HTTPS
- Update hardcoded HTTP links in applications
- Configure HTTP to HTTPS redirects

## URL Rewrite for HTTPS Redirect

Add this rule to web.config for automatic HTTPS redirect:

```xml
<system.webServer>
  <rewrite>
    <rules>
      <rule name="Redirect to HTTPS" stopProcessing="true">
        <match url="(.*)" />
        <conditions>
          <add input="{HTTPS}" pattern="off" ignoreCase="true" />
        </conditions>
        <action type="Redirect" url="https://{HTTP_HOST}/{R:1}" 
                redirectType="Permanent" />
      </rule>
    </rules>
  </rewrite>
</system.webServer>
```

## Best Practices

### Security Configuration

⚠️ **Important Security Settings:**
- Always require SSL for Method applications
- Disable weak SSL/TLS protocols (SSLv2, SSLv3)
- Use strong cipher suites
- Implement HTTP Strict Transport Security (HSTS) headers
- Regularly update certificates before expiration

### Certificate Management

1. **Documentation**: Keep record of certificate assignments and expiration dates
2. **Monitoring**: Set up alerts for certificate expiration
3. **Testing**: Regularly test HTTPS connectivity across all Method sites
4. **Backup**: Maintain secure backup of certificate files

## Troubleshooting Commands

```cmd
# Restart IIS to apply changes
iisreset

# Check IIS application pools status
%windir%\system32\inetsrv\appcmd list apppools

# List all sites and their status
%windir%\system32\inetsrv\appcmd list sites

# Test SSL connectivity
telnet methodlocal.com 443
```

```powershell
# Check certificate details
$cert = Get-ChildItem -Path Cert:\LocalMachine\My | Where-Object {$_.Subject -like "*methodlocal*"}
$cert | Format-List Subject, Issuer, NotBefore, NotAfter, Thumbprint

# Test Method site connectivity
Test-NetConnection -ComputerName methodlocal.com -Port 443
```

## Next Steps

After configuring IIS certificates:
- [BrowserStack Setup](./browserstack.md) for cross-browser testing
- Test Method applications with HTTPS
- Configure any additional security headers
- Verify all Method services are accessible via HTTPS

## Important Reminders

📅 **Certificate Expiration**: Current certificate expires December 4, 2025
🔧 **Maintenance**: Regularly verify all HTTPS bindings are working
🔒 **Security**: Monitor for SSL/TLS vulnerabilities and update configurations
⚡ **Performance**: Test SSL performance across different Method applications
