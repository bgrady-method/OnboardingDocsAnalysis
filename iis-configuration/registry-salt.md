# Export the Registry Key - Salt

## 11.2. Export the Registry Key - Salt

This step configures encryption salt values required for Method applications.

### Prerequisites
- Method Platform UI repository cloned to C:\MethodDev\method-platform-ui
- Administrator access to modify registry

### Steps

1) Open *File Explorer* ⇒ go to "**C:\MethodDev\method-platform-ui**"

2) *DoubleClick* to run the **"EncSalt.reg"** file ⇒ say "yes" & "ok" to next prompts

### What This Does

The EncSalt.reg file configures encryption salt values in the Windows Registry that are required for Method's encryption and security functions. These values must match across all Method services to ensure proper data encryption and decryption.

### Verification

After running the registry file:
- Check Windows Event Viewer for any registry-related errors
- Verify that Method services can properly encrypt/decrypt data
- No additional restart is typically required

### Troubleshooting

**Registry file not found:**
- Ensure you have cloned the method-platform-ui repository
- Check if the file exists at: `C:\MethodDev\method-platform-ui\EncSalt.reg`
- Contact your team lead if the file is missing from the repository

**Access denied error:**
- Ensure you're running as Administrator
- Right-click on the .reg file and select "Run as administrator"

**Registry import failed:**
- Check if Windows Defender or antivirus is blocking the operation
- Temporarily disable real-time protection during import
- Manually edit registry if automated import fails

### Security Note

The salt values in this registry file are specifically for local development environments. Never use these values in production environments, as they may be publicly available in development repositories.

**Next:** [Import IIS Sites & App Pools](./application-pools.md)
