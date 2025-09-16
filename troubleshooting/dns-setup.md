# DNS Setup (Optional)

## Local DNS Configuration for Method Development

Setting up local DNS can provide more realistic development environment that mirrors production domain structures.

## Why Use Local DNS?

### Benefits
- **Production-like URLs** - Use actual domain names instead of localhost
- **SSL Certificate Compatibility** - Certificates work with proper domain names
- **Multi-service Testing** - Different subdomains for different services
- **Cross-browser Testing** - Consistent behavior across browsers

### Use Cases
- Testing SSL certificates with real domain names
- Developing with subdomain-based routing
- Simulating production environment locally
- Multi-tenant application development

## DNS Configuration Options

### Option 1: Host File Method (Recommended)
This is the simplest approach and works for most development scenarios.

```bash
# Edit C:\Windows\System32\drivers\etc\hosts
127.0.0.1 local.method.com
127.0.0.1 api.local.method.com
127.0.0.1 portal.local.method.com
127.0.0.1 admin.local.method.com
127.0.0.1 methodlocaldb
```

### Option 2: Local DNS Server
For more advanced scenarios, set up a local DNS server.

#### Using DNS Server on Windows
1. **Install DNS Server Role**
   ```powershell
   # Run in elevated PowerShell
   Install-WindowsFeature -Name DNS -IncludeManagementTools
   ```

2. **Configure Forward Lookup Zone**
   ```powershell
   # Create zone for local.method.com
   Add-DnsServerPrimaryZone -Name "local.method.com" -ZoneFile "local.method.com.dns"
   
   # Add A records
   Add-DnsServerResourceRecordA -ZoneName "local.method.com" -Name "@" -IPv4Address "127.0.0.1"
   Add-DnsServerResourceRecordA -ZoneName "local.method.com" -Name "api" -IPv4Address "127.0.0.1"
   Add-DnsServerResourceRecordA -ZoneName "local.method.com" -Name "portal" -IPv4Address "127.0.0.1"
   ```

3. **Configure Client DNS Settings**
   ```powershell
   # Set DNS server to localhost
   Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses "127.0.0.1", "8.8.8.8"
   ```

### Option 3: Third-Party DNS Tools

#### Acrylic DNS Proxy
Free local DNS proxy with caching and custom rules.

1. **Download and Install** - [Acrylic DNS Proxy](http://mayakron.altervista.org/wikibase/show.php?id=AcrylicHome)

2. **Configure Hosts File** - Edit AcrylicHosts.txt:
   ```
   127.0.0.1 local.method.com
   127.0.0.1 *.local.method.com
   127.0.0.1 methodlocaldb
   ```

3. **Set DNS Server** - Point network adapter to 127.0.0.1

## Method-Specific DNS Configuration

### Development Domains
```bash
# Core services
127.0.0.1 local.method.com           # Main application
127.0.0.1 api.local.method.com       # API gateway
127.0.0.1 auth.local.method.com      # Authentication service
127.0.0.1 portal.local.method.com    # Customer portal

# Admin interfaces  
127.0.0.1 admin.local.method.com     # Admin portal
127.0.0.1 dev.local.method.com       # Developer tools

# Database and services
127.0.0.1 methodlocaldb             # SQL Server alias
127.0.0.1 mongo.local.method.com    # MongoDB (optional)
127.0.0.1 redis.local.method.com    # Redis (optional)
```

### IIS Bindings Configuration
```powershell
# Add IIS site bindings for custom domains
Import-Module WebAdministration

# Remove default binding
Remove-WebBinding -Name "Default Web Site" -Port 80 -Protocol "http"

# Add Method site bindings
New-WebBinding -Name "Method-UI" -Protocol "http" -Port 80 -HostHeader "local.method.com"
New-WebBinding -Name "Method-UI" -Protocol "https" -Port 443 -HostHeader "local.method.com"
New-WebBinding -Name "Method-API" -Protocol "https" -Port 443 -HostHeader "api.local.method.com"
New-WebBinding -Name "Method-Portal" -Protocol "https" -Port 443 -HostHeader "portal.local.method.com"
```

## SSL Certificate Configuration

### Generate Wildcard Certificate
```powershell
# Create self-signed wildcard certificate
$cert = New-SelfSignedCertificate -DnsName "*.local.method.com", "local.method.com" -CertStoreLocation "cert:\LocalMachine\My"

# Export certificate
$pwd = ConvertTo-SecureString -String "password123" -Force -AsPlainText
Export-PfxCertificate -Cert $cert -FilePath "C:\temp\local.method.com.pfx" -Password $pwd

# Import to Trusted Root
Import-Certificate -FilePath "C:\temp\local.method.com.cer" -CertStoreLocation "cert:\LocalMachine\Root"
```

### Bind Certificate to IIS
```powershell
# Get certificate thumbprint
$cert = Get-ChildItem -Path "cert:\LocalMachine\My" | Where-Object { $_.Subject -like "*local.method.com*" }

# Bind to IIS site
New-WebBinding -Name "Method-UI" -Protocol "https" -Port 443 -HostHeader "local.method.com"
$binding = Get-WebBinding -Name "Method-UI" -Protocol "https"
$binding.AddSslCertificate($cert.Thumbprint, "my")
```

## Testing DNS Configuration

### Verification Commands
```powershell
# Test DNS resolution
nslookup local.method.com
nslookup api.local.method.com

# Test with ping
ping local.method.com
ping api.local.method.com

# Test HTTP connectivity
Invoke-WebRequest -Uri "http://local.method.com" -UseBasicParsing
Invoke-WebRequest -Uri "https://api.local.method.com" -UseBasicParsing
```

### Browser Testing
1. **Open browser and navigate to domains**
2. **Check SSL certificate** - Should show valid for *.local.method.com
3. **Test different subdomains** - Verify routing works correctly
4. **Check developer tools** - Verify no mixed content warnings

## Troubleshooting DNS Issues

### Common Problems

#### DNS Not Resolving
**Symptoms**: Domain names don't resolve to 127.0.0.1
**Solutions**:
```powershell
# Clear DNS cache
ipconfig /flushdns

# Check hosts file syntax
Get-Content C:\Windows\System32\drivers\etc\hosts

# Verify network adapter DNS settings
Get-DnsClientServerAddress
```

#### SSL Certificate Errors
**Symptoms**: Browser shows certificate warnings
**Solutions**:
```powershell
# Verify certificate installation
Get-ChildItem -Path "cert:\LocalMachine\Root" | Where-Object { $_.Subject -like "*local.method.com*" }

# Check certificate binding
netsh http show sslcert

# Re-bind certificate if needed
netsh http add sslcert ipport=0.0.0.0:443 certhash=THUMBPRINT appid={GUID}
```

#### IIS Binding Issues
**Symptoms**: Sites not accessible via custom domains
**Solutions**:
```powershell
# List all bindings
Get-WebBinding

# Test binding response
Test-NetConnection -ComputerName local.method.com -Port 80
Test-NetConnection -ComputerName local.method.com -Port 443

# Check IIS logs
Get-Content "C:\inetpub\logs\LogFiles\W3SVC1\*.log" | Select-String "local.method.com"
```

## PowerShell Scripts

### DNS Setup Script
```powershell
function Set-MethodDNS {
    param(
        [switch]$UseHostsFile = $true,
        [switch]$CreateCertificates = $false
    )
    
    $domains = @(
        "local.method.com",
        "api.local.method.com", 
        "portal.local.method.com",
        "admin.local.method.com",
        "methodlocaldb"
    )
    
    if ($UseHostsFile) {
        Write-Host "Configuring hosts file..." -ForegroundColor Yellow
        
        $hostsFile = "C:\Windows\System32\drivers\etc\hosts"
        $content = Get-Content $hostsFile
        
        # Remove existing Method entries
        $content = $content | Where-Object { $_ -notmatch "method\.com|methodlocaldb" }
        
        # Add new entries
        foreach ($domain in $domains) {
            $content += "127.0.0.1 $domain"
        }
        
        Set-Content -Path $hostsFile -Value $content
        Write-Host "Hosts file updated successfully!" -ForegroundColor Green
    }
    
    if ($CreateCertificates) {
        Write-Host "Creating SSL certificates..." -ForegroundColor Yellow
        
        $cert = New-SelfSignedCertificate -DnsName "*.local.method.com", "local.method.com" -CertStoreLocation "cert:\LocalMachine\My"
        
        # Export to Trusted Root
        $certPath = "cert:\LocalMachine\My\$($cert.Thumbprint)"
        Export-Certificate -Cert $certPath -FilePath "C:\temp\local.method.com.cer"
        Import-Certificate -FilePath "C:\temp\local.method.com.cer" -CertStoreLocation "cert:\LocalMachine\Root"
        
        Write-Host "SSL certificate created and installed!" -ForegroundColor Green
    }
    
    # Flush DNS cache
    ipconfig /flushdns
    
    Write-Host "DNS setup complete!" -ForegroundColor Green
}

# Run setup
Set-MethodDNS -UseHostsFile -CreateCertificates
```

### DNS Health Check
```powershell
function Test-MethodDNS {
    $domains = @(
        "local.method.com",
        "api.local.method.com", 
        "portal.local.method.com"
    )
    
    Write-Host "=== Method DNS Health Check ===" -ForegroundColor Green
    
    foreach ($domain in $domains) {
        try {
            $result = Resolve-DnsName -Name $domain -ErrorAction Stop
            if ($result.IPAddress -eq "127.0.0.1") {
                Write-Host "✓ $domain resolves correctly" -ForegroundColor Green
            } else {
                Write-Host "✗ $domain resolves to $($result.IPAddress)" -ForegroundColor Red
            }
        }
        catch {
            Write-Host "✗ $domain failed to resolve" -ForegroundColor Red
        }
    }
}

Test-MethodDNS
```

## Reverting DNS Changes

### Remove Custom DNS Configuration
```powershell
function Remove-MethodDNS {
    Write-Host "Removing Method DNS configuration..." -ForegroundColor Yellow
    
    # Clean hosts file
    $hostsFile = "C:\Windows\System32\drivers\etc\hosts"
    $content = Get-Content $hostsFile
    $content = $content | Where-Object { $_ -notmatch "method\.com|methodlocaldb" }
    Set-Content -Path $hostsFile -Value $content
    
    # Remove certificates
    Get-ChildItem -Path "cert:\LocalMachine\My" | Where-Object { $_.Subject -like "*local.method.com*" } | Remove-Item
    Get-ChildItem -Path "cert:\LocalMachine\Root" | Where-Object { $_.Subject -like "*local.method.com*" } | Remove-Item
    
    # Flush DNS
    ipconfig /flushdns
    
    Write-Host "Method DNS configuration removed!" -ForegroundColor Green
}
```

**Next**: [New Hard Drive Setup](./new-drive-setup.md)
