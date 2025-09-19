# Download Store-Bought Certificate

This guide covers downloading and obtaining the commercial SSL certificate for Method development.

## Overview

Method uses a store-bought SSL certificate valid for one year that must be downloaded and installed for HTTPS development.

## Certificate Location

The current Method SSL certificate is available here:

**Download Link:** [Google Drive Certificate](https://drive.google.com/file/d/1GO7lXtSf9zMKBejZ-ZSd9bsSKOXemyoQ/view?usp=sharing)

**Expiration:** December 4, 2025  
**Password:** X7PSmdQEMc4DuMi1rYbl

## Steps

### 1. Access Certificate File

1. **Download the Certificate**
   - Use the Google Drive link provided above
   - Save the .pfx file to a secure location on your machine
   - Recommended location: `C:\MethodDev\certificates\`

2. **Verify Download**
   - Confirm the file downloaded completely
   - Note the file size and verify integrity
   - Keep the password secure and accessible

### 2. Certificate Details

The Method SSL certificate includes:
- **Certificate Type**: Commercial SSL certificate (.pfx format)
- **Validity Period**: 1 year (expires Dec 4, 2025)
- **Coverage**: `*.methodlocal.com` wildcard certificate
- **Format**: .pfx file with embedded private key
- **Password Protection**: Yes (see password above)

### 3. Security Considerations

⚠️ **Important Security Notes:**
- Certificate files contain private keys and must be handled securely
- Do not store certificates in version control or unsecured locations
- Certificate passwords should be obtained through secure channels
- Store the certificate file in a secure location with appropriate access controls

## Certificate Renewal

🔄 **Annual Renewal Process:**
- Certificates expire annually and must be renewed
- Monitor expiration dates to prevent service interruptions
- New certificates require re-import and IIS binding updates
- Contact Method IT or development team for renewal procedures

## Troubleshooting

### Certificate Download Issues
- **File Access Problems**: Verify you have access to the Google Drive link
- **Corporate Firewall**: May need to download from a personal network
- **File Corruption**: Re-download if file appears corrupted

### Password Issues
- **Incorrect Password**: Double-check the password case sensitivity
- **Password Changes**: Contact Method IT if password has been updated
- **Authentication Failures**: Verify you have the current certificate version

## Next Steps

After downloading the certificate, proceed to:
- [Import Certificates](./import-certificates.md) - Install the certificate into Windows certificate store
- [Add Certificates to IIS](./iis-certificates.md) - Configure HTTPS bindings

## Important Notes

📅 **Expiration Reminder**: Current certificate expires **December 4, 2025**
🔐 **Password Security**: Store the certificate password securely
⚡ **Quick Setup**: Certificate is ready for immediate use after download
🌐 **Domain Coverage**: Supports all `*.methodlocal.com` subdomains for development
