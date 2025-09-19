# AWS Credentials Setup

This guide covers setting up AWS credentials for Method's Secret Manager integration, based on the Method Developer Machine Setup documentation.

## Overview

As Method is using **AWS Secret Manager** to load all secrets (such as connection strings and encryption keys), you have to add the *AWS Key and Secret* to your local userprofile and home environment folder.

## Prerequisites

- Administrative access for file placement
- Access to Method's AWS credentials file
- Understanding of Windows file system structure

## AWS Credentials File Setup

### Download Credentials File

1. **Download the file listed below:**

   **New (2025-06-09)**
   
   **Download Link:** https://drive.google.com/file/d/112E060eN3cG7oZRQPu93k-APwl32VGIA/view?usp=drive_link

2. **Rename the file to `credentials`** once downloaded (remove any file extension)

### Windows Installation

#### System Profile Location

1. **Go to:** `C:\Windows\System32\config\systemprofile` in File Explorer

2. **Create .aws folder if it doesn't exist:**
   - If .aws folder doesn't exist → **Right-click** on File Explorer → select **"Git Bash Here"** → Run:
   ```bash
   mkdir .aws
   ```

3. **Copy & Paste** the downloaded file **inside the .aws folder**

#### User Profile Location

1. **Go to:** `%SystemDrive%\Users\<username>\.aws` (*replace "username" with yours*)

2. **Create .aws folder if it doesn't exist:**
   - If .aws folder doesn't exist → **Right-click** on File Explorer → select **"Git Bash Here"** → Run:
   ```bash
   mkdir .aws
   ```

3. **Copy & Paste** the downloaded file **inside .aws folder**

### Create Config File

1. **Create another file** in the same folder and name it `config` (no extension)

2. **Paste the following content** in it:
   ```
   [default]
   region = us-west-1
   ```

## Final Directory Structure

After completing the setup, you should have the credentials file in both locations:

### System Profile
```
C:\Windows\System32\config\systemprofile\.aws\
├── credentials
└── config
```

### User Profile
```
C:\Users\<username>\.aws\
├── credentials
└── config
```

## AWS Linux Setup

On any Linux box, put the credential file in **~/user** folder:

```bash
sudo -i
sudo mkdir .aws
cd .aws
# paste the file in this folder
```

## Verification

### Test AWS Access

You can verify the credentials are working by testing AWS CLI (if installed):

```bash
# Test credentials
aws sts get-caller-identity

# Test region configuration
aws configure list
```

### Environment Check

The AWS credentials should now be available to Method applications for accessing Secret Manager.

## Troubleshooting

### File Permissions

- Ensure the credentials file has appropriate read permissions
- Verify both system and user profile locations have the file
- Check that the config file is created with the correct region

### Path Issues

- Double-check that `.aws` folders are created in the correct locations
- Ensure the credentials file is named exactly `credentials` with no extension
- Verify the config file is named exactly `config` with no extension

### Access Issues

- Ensure you downloaded the correct credentials file from the provided link
- Check that the file wasn't corrupted during download
- Verify you have the latest version of the credentials file (2025-06-09)

## Integration with Method

Method applications will automatically use these credentials to:

- Access AWS Secret Manager for configuration values
- Retrieve database connection strings
- Load encryption keys and API credentials
- Connect to other AWS services as needed

## Security Notes

- **Never commit** AWS credentials to source control
- **Keep credentials file secure** and don't share with unauthorized users
- **Use only the provided Method credentials** for development
- **Contact DevOps** if you suspect credentials are compromised

## Next Steps

After setting up AWS credentials:

1. **Restart applications** that use AWS services
2. **Test Method applications** to ensure Secret Manager access works
3. **Continue with setup:** Proceed to local development configuration
4. **Verify integration:** Check that Method apps can load configuration from AWS

**Back to:** [.NET Core Requirements](./README.md)
