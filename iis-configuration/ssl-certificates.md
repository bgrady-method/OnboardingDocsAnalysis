# SSL Certificates Configuration

This guide covers SSL certificate installation for Method's local development environment, based on the Method Developer Machine Setup documentation.

## Overview

SSL certificates are essential for Method's local development environment to enable HTTPS communication between Method services. This guide covers the basic certificate setup needed for Method development.

## Prerequisites

- IIS installed and configured
- Administrator access for certificate installation
- Method application pools and sites configured

## Method SSL Certificate Setup

### Download Store Bought Certificate

Method provides a store-bought certificate for development:

**Certificate Location:** https://drive.google.com/file/d/1GO7lXtSf9zMKBejZ-ZSd9bsSKOXemyoQ/view?usp=sharing

**Details:**
- **Expires:** Dec 4, 2025
- **Password:** X7PSmdQEMc4DuMi1rYbl

### Certificate Installation Process

#### Step 1: Import to Personal Store

1. **Open Certificate Manager:**
   - Press the Windows Key and search for **"mmc"**
   
2. **Add Certificates Snap-In (if needed):**
   - If your mmc console is empty, go to **File** menu → **"Add/Remove Snap In"**
   - Select **"Certificates"** → click **"Add"**
   - Select **"Computer account"** → Click **"Finish"** → **"Ok"**

3. **Import to Personal Store:**
   - Go to **Certificates → Personal → Certificates**
   - **Right-click on "Certificates"** → Click **"Import"**
   - After selecting **"Import"**, pick the ".pfx" file that you downloaded
   - Keep all subsequent options as **Default**, then click **Finish**
   - If prompted with a security prompt, select **"Yes"**

#### Step 2: Import to Trusted Root Store

1. **Navigate to Trusted Root:**
   - Go to **Certificates → Trusted Root Certification Authorities → Certificates**

2. **Import Certificate:**
   - **Right-click on "Certificates"** → Click **"Import"**
   - Click **"Next"**, then **"Browse"** to the .pfx file location
   - Keep the default selection for "Certification Store" and click **"Finish"**
   - If prompted with a security prompt, select **"Yes"**

**Note:** This certificate should be good for one year. After one year you will need to repeat these steps.

## IIS HTTPS Binding Configuration

### Configure HTTPS Bindings for Method Sites

**Note:** If you have not set up any sites in IIS, complete the IIS application pool setup first.

1. **Open IIS Manager**

2. **Configure HTTPS Bindings:**
   - **Right-click** on every Site in *Connections* tab → **Edit Bindings**
   - Make sure there's a **HTTPS binding** with **port 443**

3. **Add HTTPS Binding (if missing):**
   - If no HTTPS Binding exists → Click on **"Add"**
   - Add these parameters:
     - **Type:** https
     - **Port:** 443  
     - **SSL certificate:** Select the Method certificate (*.methodlocal.com)

4. **Update Existing HTTPS Binding:**
   - If HTTPS Binding exists, click on **"EDIT"**
   - Select **"*.methodlocal.com"** from the SSL certification dropdown
   - Save by clicking **"OK"**

**Important Notes:**
- If you don't have **"*.methodlocal.com"**, ensure you followed the certificate installation steps first
- Once you setup SSL correctly for one site, it most likely gets reflected in every other site with HTTPS binding

5. **Repeat for All Sites:**
   - **Repeat Steps 2-4** for all Method sites in the Connections panel
   - Keep in mind that configuring one site often updates others automatically

## Custom Certificate Creation (Alternative)

If you need to generate custom certificates for development:

### Generate Self-Signed Certificate

1. **Open PowerShell as Administrator**

2. **Create Certificate:**
   ```powershell
   New-SelfSignedCertificate -DnsName *.methodlocal.com -CertStoreLocation cert:\LocalMachine\My
   ```

3. **Copy the Thumbprint** shown in the output

4. **Add to Trusted Root Store:**
   ```powershell
   # Replace <ThumbprintGoesHere> with the actual thumbprint
   $cert = Get-ChildItem -Path cert:\LocalMachine\My\<ThumbprintGoesHere>
   $store = New-Object System.Security.Cryptography.X509Certificates.X509Store("Root","LocalMachine")
   $store.Open("ReadWrite")
   $store.Add($cert)
   $store.Close()
   ```

## Verification

### Test HTTPS Access

After configuring certificates:

1. **Test Method URLs:**
   - Try accessing https://methodlocal.com
   - Try accessing https://api.methodlocal.com
   - Should not show certificate warnings

2. **Check Certificate in Browser:**
   - Click the lock icon in browser address bar
   - Verify the certificate details are correct
   - Should show as trusted/valid

### Troubleshooting

**Certificate Warnings in Browser:**
- Verify certificate is installed in both Personal and Trusted Root stores
- Check that HTTPS binding is using the correct certificate
- Clear browser cache and try again

**Certificate Not Available in IIS:**
- Ensure certificate is installed in Personal store (LocalMachine\My)
- Restart IIS: `iisreset`
- Check certificate has private key associated

**HTTPS Sites Not Accessible:**
- Verify IIS sites are running
- Check Windows Firewall settings for port 443
- Ensure Method hosts file entries are configured

## Integration with Method Setup

The SSL certificate configuration integrates with:

- **Method Sites:** All Method applications require HTTPS
- **Local Development:** Matches production SSL behavior
- **Authentication Services:** Required for secure authentication flow
- **API Communication:** Enables secure API communication between services

## Next Steps

After configuring SSL certificates:

1. **Test Method Applications:** Verify HTTPS access to Method services  
2. **Configure Virtual Directories:** [Virtual Directory Setup](./virtual-directories.md)
3. **Set Security Settings:** [IIS Security Configuration](./security-settings.md)
4. **Optimize Performance:** [Performance Tuning](./performance-tuning.md)

**Back to:** [IIS Configuration](./README.md)
