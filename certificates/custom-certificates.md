# Generate Custom Certificates

This guide covers creating custom SSL certificates for Method development when commercial certificates are not available or for specialized testing scenarios.

## Overview

Custom certificates are useful for development environments, testing specific scenarios, or when commercial certificates are not accessible. The Developer Machine Setup 3.0 document provides specific PowerShell commands for generating custom certificates for Method development.

## Prerequisites

- PowerShell with Administrator privileges
- Basic understanding of PKI concepts
- MMC (Microsoft Management Console) access for certificate management

## Method 1: PowerShell Self-Signed Certificate (Recommended)

This method creates a wildcard certificate for `*.methodlocal.com` domains using PowerShell.

### Generate Custom Certificate

1. **Open PowerShell as Administrator**
   - Right-click Start Menu → "Windows PowerShell (Admin)"

2. **Create Self-Signed Certificate**
   ```powershell
   # Generate wildcard certificate for Method development
   New-SelfSignedCertificate -DnsName *.methodlocal.com -CertStoreLocation cert:\LocalMachine\My
   ```

3. **Note the Thumbprint**
   - Copy the thumbprint shown in the output
   - You'll need this for the next step
   - Example output: `A1B2C3D4E5F6789...`

### Trust the Certificate

After creating the certificate, you need to add it to the trusted root store:

```powershell
# Replace <ThumbprintGoesHere> with the actual thumbprint from previous step
$thumbprint = "<ThumbprintGoesHere>"

# Get the certificate from Personal store
$cert = Get-ChildItem -Path Cert:\LocalMachine\My\$thumbprint

# Add certificate to Trusted Root store
$rootStore = Get-Item "cert:\LocalMachine\Root"
$rootStore.Open("ReadWrite")
$rootStore.Add($cert)
$rootStore.Close()

Write-Host "Certificate added to Trusted Root store successfully"
```

### Complete Example

```powershell
# Complete script to create and trust Method certificate
$cert = New-SelfSignedCertificate -DnsName *.methodlocal.com -CertStoreLocation cert:\LocalMachine\My

# Trust the certificate
$rootStore = Get-Item "cert:\LocalMachine\Root"
$rootStore.Open("ReadWrite")
$rootStore.Add($cert)
$rootStore.Close()

Write-Host "Method development certificate created and trusted"
Write-Host "Thumbprint: $($cert.Thumbprint)"
```

## Method 2: Enhanced PowerShell Certificate

For more control over certificate properties:

```powershell
# Create certificate with additional configuration
$cert = New-SelfSignedCertificate `
    -DnsName "*.methodlocal.com", "methodlocal.com", "localhost" `
    -CertStoreLocation "cert:\LocalMachine\My" `
    -KeyUsage KeyEncipherment, DigitalSignature `
    -KeyAlgorithm RSA `
    -KeyLength 2048 `
    -NotAfter (Get-Date).AddYears(2) `
    -Subject "CN=Method Development Certificate"

# Export certificate for backup/sharing
$certPath = "C:\MethodDev\method-dev-cert.pfx"
$certPassword = ConvertTo-SecureString -String "MethodDev123!" -Force -AsPlainText
Export-PfxCertificate -Cert $cert -FilePath $certPath -Password $certPassword

Write-Host "Certificate created and exported to: $certPath"
Write-Host "Password: MethodDev123!"
```

## Method 3: OpenSSL Custom Certificates

If you have OpenSSL available (included with Git for Windows):

### Generate Private Key and Certificate

```bash
# Navigate to Git Bash or OpenSSL environment
cd /c/MethodDev/certificates

# Generate private key
openssl genrsa -out method-dev.key 2048

# Create certificate signing request
openssl req -new -key method-dev.key -out method-dev.csr -subj "/C=CA/ST=Ontario/L=Toronto/O=Method Financial/CN=*.methodlocal.com"

# Generate self-signed certificate
openssl x509 -req -days 365 -in method-dev.csr -signkey method-dev.key -out method-dev.crt

# Create PFX file for Windows
openssl pkcs12 -export -out method-dev.pfx -inkey method-dev.key -in method-dev.crt -password pass:MethodDev123!
```

### Import OpenSSL Certificate

```powershell
# Import the PFX certificate
$certPath = "C:\MethodDev\certificates\method-dev.pfx"
$certPassword = ConvertTo-SecureString -String "MethodDev123!" -Force -AsPlainText
Import-PfxCertificate -FilePath $certPath -CertStoreLocation Cert:\LocalMachine\My -Password $certPassword
Import-PfxCertificate -FilePath $certPath -CertStoreLocation Cert:\LocalMachine\Root -Password $certPassword
```

## Certificate Configuration

### Enhanced Configuration with Subject Alternative Names

For comprehensive domain coverage:

```powershell
# Create certificate with multiple SANs
$cert = New-SelfSignedCertificate `
    -DnsName @(
        "*.methodlocal.com",
        "methodlocal.com", 
        "localhost",
        "api.methodlocal.com",
        "auth.methodlocal.com",
        "signup.methodlocal.com",
        "signin.methodlocal.com"
    ) `
    -CertStoreLocation "cert:\LocalMachine\My" `
    -KeyUsage KeyEncipherment, DigitalSignature `
    -KeyAlgorithm RSA `
    -KeyLength 2048 `
    -NotAfter (Get-Date).AddYears(2) `
    -Subject "CN=Method Development Wildcard Certificate"
```

## Installation and IIS Configuration

### Import Custom Certificate to IIS

After generating the certificate, configure it in IIS:

1. **Open IIS Manager**
   - Launch IIS Manager (`inetmgr`)

2. **Configure HTTPS Bindings**
   - For each Method site, edit bindings
   - Add/update HTTPS binding
   - Select your custom certificate from the dropdown

3. **Verify Certificate Assignment**
   ```powershell
   # Check IIS bindings
   Get-WebBinding -Name "*method*" | Format-Table
   
   # Verify certificate in store
   Get-ChildItem -Path Cert:\LocalMachine\My | Where-Object {$_.Subject -like "*methodlocal*"}
   ```

## Validation and Testing

### Verify Certificate Installation

```powershell
# Check certificate details
$cert = Get-ChildItem -Path Cert:\LocalMachine\My | Where-Object {$_.Subject -like "*methodlocal*"}
$cert | Format-List Subject, Issuer, NotBefore, NotAfter, Thumbprint

# Test certificate binding
Test-Certificate -Cert $cert -Policy SSL
```

### Browser Testing

1. **Chrome Certificate Management**
   - Navigate to `chrome://settings/certificates`
   - Check if certificate appears in "Authorities" tab

2. **Test HTTPS Connection**
   ```powershell
   # Test local HTTPS connection
   try {
       $response = Invoke-WebRequest -Uri "https://methodlocal.com" -UseBasicParsing
       Write-Host "HTTPS connection successful: $($response.StatusCode)"
   } catch {
       Write-Host "HTTPS connection failed: $($_.Exception.Message)"
   }
   ```

3. **Browser Verification**
   - Navigate to `https://methodlocal.com` in browser
   - Check for certificate warnings
   - Verify certificate details show your custom certificate

## Common Use Cases

### Development Environment Setup

1. **Initial Development**: Create certificates for new developer setup
2. **Team Consistency**: Share certificate files across development team
3. **Offline Development**: Work without internet access to commercial certificates

### Testing Scenarios

1. **Certificate Expiration Testing**
   ```powershell
   # Create short-lived certificate for expiration testing
   $testCert = New-SelfSignedCertificate `
       -DnsName "*.methodlocal.com" `
       -CertStoreLocation "cert:\LocalMachine\My" `
       -NotAfter (Get-Date).AddDays(1)
   ```

2. **Certificate Mismatch Testing**
   ```powershell
   # Create certificate with wrong domain for testing
   $mismatchCert = New-SelfSignedCertificate `
       -DnsName "*.wrongdomain.com" `
       -CertStoreLocation "cert:\LocalMachine\My"
   ```

3. **Chain Validation Testing**: Test intermediate certificate scenarios

## Security Considerations

⚠️ **Important Security Notes**

### Development Only Usage
- Custom certificates should **only** be used in development
- **Never** use custom certificates in production environments
- Remove custom root CAs from production systems

### Certificate Management Best Practices
1. **Strong Passwords**: Use complex passwords for certificate files
2. **Secure Storage**: Store private keys securely
3. **Regular Rotation**: Rotate development certificates regularly
4. **Documentation**: Document certificate usage and expiration dates

### Trust Boundaries
- Only trust custom CAs on development machines
- Don't distribute custom root certificates to end users
- Use proper commercial certificates for staging/production

## Troubleshooting

### Common Issues

#### Certificate Not Trusted
```powershell
# Verify certificate is in both stores
Get-ChildItem -Path Cert:\LocalMachine\My | Where-Object {$_.Subject -like "*methodlocal*"}
Get-ChildItem -Path Cert:\LocalMachine\Root | Where-Object {$_.Subject -like "*methodlocal*"}

# Re-add to trusted root if missing
$cert = Get-ChildItem -Path Cert:\LocalMachine\My | Where-Object {$_.Subject -like "*methodlocal*"}
$rootStore = Get-Item "cert:\LocalMachine\Root"
$rootStore.Open("ReadWrite")
$rootStore.Add($cert)
$rootStore.Close()
```

#### Browser Still Shows Warnings
- Clear browser cache and restart browser
- Check certificate hasn't expired
- Verify certificate includes correct domain names
- Restart IIS and application pools

#### IIS Binding Issues
```powershell
# Check certificate thumbprint
$cert = Get-ChildItem -Path Cert:\LocalMachine\My | Where-Object {$_.Subject -like "*methodlocal*"}
Write-Host "Certificate Thumbprint: $($cert.Thumbprint)"

# Verify IIS can access certificate
Get-WebBinding -Name "Default Web Site" | Format-Table
```

## Automation and Team Sharing

### Team Setup Script

Create a shared script for consistent team setup:

```powershell
# team-cert-setup.ps1
param(
    [string]$CertPassword = "MethodDev123!"
)

Write-Host "Setting up Method development certificates..."

# Check if certificate already exists
$existingCert = Get-ChildItem -Path Cert:\LocalMachine\My | Where-Object {$_.Subject -like "*methodlocal*"}

if ($existingCert) {
    Write-Host "Method certificate already exists. Thumbprint: $($existingCert.Thumbprint)"
} else {
    # Create new certificate
    $cert = New-SelfSignedCertificate -DnsName *.methodlocal.com -CertStoreLocation cert:\LocalMachine\My
    
    # Trust the certificate
    $rootStore = Get-Item "cert:\LocalMachine\Root"
    $rootStore.Open("ReadWrite")
    $rootStore.Add($cert)
    $rootStore.Close()
    
    Write-Host "New Method certificate created and trusted. Thumbprint: $($cert.Thumbprint)"
}

Write-Host "Method certificate setup complete!"
```

### CI/CD Integration

```yaml
# Example CI/CD step for certificate setup
- name: Setup Development Certificates
  run: |
    powershell -ExecutionPolicy Bypass -File scripts/team-cert-setup.ps1
  if: runner.os == 'Windows'
```

## Next Steps

After generating custom certificates:
- Configure Method applications to use HTTPS with custom certificates
- Test all Method services with SSL
- Document certificate management procedures for the team
- Set up certificate renewal reminders
- Consider automation for certificate lifecycle management

## Integration with Method Development

### Method-Specific Configuration

1. **Host File Setup**: Ensure all Method domains are in hosts file
2. **IIS Configuration**: Apply certificates to all Method sites
3. **Application Testing**: Test Method applications with custom certificates
4. **Team Documentation**: Share certificate setup procedures

### Maintenance Tasks

- **Monthly**: Check certificate expiration dates
- **Quarterly**: Review and update certificate configurations
- **Annually**: Regenerate certificates and update team procedures
