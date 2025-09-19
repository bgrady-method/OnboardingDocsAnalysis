# Import Certificates

This guide covers importing SSL certificates into the Windows certificate store for Method development.

## Overview

SSL certificates must be imported into the Windows certificate store before they can be used by IIS and other applications. This process involves importing the certificate into both the Personal store and Trusted Root Certification Authorities store.

## Prerequisites

- Administrator access to the development machine
- SSL certificate file (.pfx format) downloaded from [Download Store-Bought Certificate](./download-certificate.md)
- Certificate password: `X7PSmdQEMc4DuMi1rYbl`
- MMC (Microsoft Management Console) access

## Import Process Using MMC

### Step 1: Open Certificate Manager (MMC)

1. **Launch MMC Console**
   - Press `Windows + R`
   - Type `mmc`
   - Press Enter and click "Yes" if prompted by UAC

2. **Add Certificates Snap-in** (if not already present)
   - From "File" menu, select "Add/Remove Snap In"
   - Select "Certificates" from the "Snap-ins" list and click "Add"
   - Select "Computer account" in the next prompt
   - Click "Finish" > "OK"
   - You should now see a list of certificates in the MMC console

### Step 2: Import to Personal Store

1. **Navigate to Personal Certificates**
   - Go to **Certificates > Personal > Certificates**

2. **Start Import Process**
   - Right-click on "Certificates" 
   - Select "All Tasks" > "Import..."

3. **Import Certificate**
   - Click "Next" on the welcome screen
   - Click "Browse" and select your .pfx file
   - Enter the certificate password when prompted
   - Keep all subsequent options as *Default*
   - Click "Finish"
   - If prompted with a security warning, select "Yes"

### Step 3: Import to Trusted Root Store

1. **Navigate to Trusted Root**
   - Go to **Certificates > Trusted Root Certification Authorities > Certificates**

2. **Import Certificate**
   - Right-click on "Certificates" > Click "Import"
   - Click "Next", then "Browse" to the same .pfx file location
   - Enter the certificate password: `X7PSmdQEMc4DuMi1rYbl`
   - Keep the default selection for "Certification Store" and click "Finish"
   - If prompted with a security warning, select "Yes"

## Alternative Import Methods

### Method 1: Double-Click Installation

1. **Simple Installation**
   - Double-click the .pfx certificate file in Windows Explorer
   - Select Store Location: Choose "Local Machine"
   - Enter Password when prompted
   - Choose Certificate Store: Select "Personal" store
   - Complete Installation

2. **Trust the Certificate**
   - The certificate will also need to be added to Trusted Root store manually
   - Follow Step 3 above for trusted root import

### Method 2: PowerShell Import

```powershell
# Import certificate to Personal store
$certPath = "C:\path\to\method-certificate.pfx"
$certPassword = ConvertTo-SecureString -String "X7PSmdQEMc4DuMi1rYbl" -Force -AsPlainText
Import-PfxCertificate -FilePath $certPath -CertStoreLocation Cert:\LocalMachine\My -Password $certPassword

# Import to Trusted Root store
Import-PfxCertificate -FilePath $certPath -CertStoreLocation Cert:\LocalMachine\Root -Password $certPassword
```

## Verification

### Verify Certificate Installation

1. **Check Personal Store**
   - Open Certificate Manager (`mmc`)
   - Navigate to **Personal > Certificates**
   - Look for certificate with subject containing Method domain information
   - Verify valid dates and that private key is present (key icon visible)

2. **Check Certificate Properties**
   - Double-click the certificate to view details
   - **General Tab**: Basic certificate information and validity
   - **Details Tab**: Technical specifications and subject details
   - **Certification Path Tab**: Certificate chain validation

### Test Certificate Chain

```powershell
# Check certificate in store
Get-ChildItem -Path Cert:\LocalMachine\My | Where-Object {$_.Subject -like "*methodlocal*"}

# Verify certificate details
$cert = Get-ChildItem -Path Cert:\LocalMachine\My | Where-Object {$_.Subject -like "*methodlocal*"}
$cert | Format-List Subject, Issuer, NotBefore, NotAfter, Thumbprint
```

## Important Notes

📝 **Certificate Validity**: Current certificate is valid until **December 4, 2025**

🔄 **Annual Renewal**: This certificate needs to be renewed yearly and re-imported

⚠️ **Security Considerations**:
- Only import certificates from trusted sources
- Store certificate files securely after import
- Consider marking keys as non-exportable for production certificates
- Regularly audit installed certificates

## Troubleshooting

### Common Issues

#### Import Fails with Password Error
- Verify password is correct: `X7PSmdQEMc4DuMi1rYbl`
- Check certificate file integrity
- Ensure certificate hasn't expired

#### Certificate Not Visible in IIS
- Import to "Local Machine" store instead of "Current User"
- Restart IIS Manager
- Check certificate permissions

#### Private Key Missing
- Re-import with "Mark this key as exportable" checked
- Verify original certificate file contains private key
- Check certificate was exported properly from source

#### Trust Issues
- Ensure certificate is imported to both Personal and Trusted Root stores
- Verify certificate chain is complete
- Check Windows Update for root certificate updates

## Next Steps

After importing the certificate successfully:
- [Add Certificates to IIS](./iis-certificates.md) - Configure HTTPS bindings for Method applications
- Test HTTPS connectivity to verify proper installation
- Document the certificate thumbprint for future reference
