# NuGet Feeds Setup

This guide covers setting up NuGet package sources for Method development, based on the Method Developer Machine Setup documentation.

## Overview

Method development requires access to both public NuGet packages and Method's internal package feed. This guide covers the basic setup needed to restore Method packages.

## Prerequisites

- Visual Studio installed and configured with MSDN credentials
- Active Directory credentials stored
- VPN connection for accessing internal feeds
- Internet access for public NuGet

## Internet Options Configuration

Before configuring NuGet feeds, you need to add Method's internal sites to your Local Intranet zone:

### Add Sites to Local Intranet

1. **Open Internet Options:**
   - Press the Windows Key and search for **"Internet Options"** in the Start Menu

2. **Go to Local Intranet's Advanced Settings:**
   - Go to **"Security"** Tab
   - Click on **"Local Intranet"**
   - Click on **"Sites"**
   - Click on **"Advanced"** in the popup window

3. **Add the following 3 website URLs to your "Local Intranet" list one-by-one:**
   - `http://tcity`
   - `http://teamcity.method.me`
   - `http://tfs.method.me`

4. **Complete Configuration:**
   - Once they're all added, Click on **"Close"** → **"OK"** → **"OK"**

## Visual Studio NuGet Configuration

### Access Package Manager Settings

1. **Open Visual Studio 2022** (currently 2022)
2. **Sign in** with the login information found under **MSDN Credentials**
3. **Start without code:** Click on **"Continue without code"** found at the bottom of the starting screen
4. **Go to Tools → NuGet Package Manager → Package Manager Settings**

### Configure Package Sources

1. **Verify nuget.org Source:**
   - Select **"Package Sources"** and make sure the NuGet package has the following configuration:
   - **Name:** nuget.org
   - **Source:** https://api.nuget.org/v3/index.json

2. **Add Method Internal Source:**
   - In **"Package Sources"** pane, click on the **"+"** icon
   - Add a new source with the following configuration:
   - **Name:** method-ms
   - **Source:** https://tfs.method.me/tfs/MethodCollection/_packaging/method-ms/nuget/v3/index.json

### Troubleshooting Method NuGet

If you encounter the error:

> Response status code does not indicate success: 402 (Payment Required).

**Solution:**
1. Remove method-ms package source from VS completely
2. Go to console and run the following command:

```cmd
nuget sources add -Name "method-ms" -Source "https://tfs.method.me/tfs/MethodCollection/_packaging/method-ms/nuget/v3/index.json" -username "method\\{{your-AD-account}}" -password "{{your-password}}"
```

Replace `{{your-AD-account}}` and `{{your-password}}` with your actual Active Directory credentials.

## Credential Storage

### Save Active Directory Credentials

To access method-ms nuget feed, you will require your Active Directory Credentials.

**Save your AD credentials** so that nuget package manager can pull down the nuget packages without prompting you to enter a username and password.

**Note:** If you skip this step, your dev setup scripts will fail.

### Windows Credential Manager Setup

1. **Open Credential Manager:**
   - Press the Windows Key and search for **"Manage Windows Credentials"** in Start Menu

2. **Add Windows Credential:**
   - Select **"Windows Credentials"** → click on **"Add a Windows credential"**

3. **Enter the following credentials:**
   - **Internet/network address:** tfs.method.me
   - **Username:** see Method Active Directory Credentials Section
   - **Password:** see Method Active Directory Credentials Section

**Important:** Use your full Method AD credentials including the `method\\` prefix for the username.

## VPN Requirements

- **Connect to VPN** before attempting to restore Method packages
- **NuGet restoration will fail** without VPN connection to Method's internal feed
- **Public packages** from nuget.org work without VPN

## Verification

### Test Package Access

1. **In Visual Studio:**
   - Go to **Tools → NuGet Package Manager → Manage NuGet Packages for Solution**
   - Check that both "nuget.org" and "method-ms" appear in package source dropdown

2. **Test Package Restore:**
   - Try restoring packages in any Method solution
   - Should complete without authentication prompts
   - Check that both public and Method packages restore successfully

### Common Issues

**Authentication Prompts:**
- Verify AD credentials are stored in Windows Credential Manager
- Check that tfs.method.me is added to Local Intranet sites
- Ensure VPN is connected

**Package Restore Failures:**
- Confirm VPN connection is active
- Verify internet access for public packages
- Check that package sources are configured correctly
- Clear NuGet cache if packages appear corrupted

**Visual Studio Issues:**
- Restart Visual Studio after configuration changes
- Sign out and back in with MSDN credentials
- Clear Visual Studio component cache if needed

## Next Steps

After configuring NuGet feeds:

1. **Store Credentials:** [Active Directory Credentials Storage](./ad-credentials.md)
2. **Install Test Framework:** [NUnit Test Adapter](./nunit-adapter.md)
3. **Add IIS Extensions:** [URL Rewrite Installation](./url-rewrite.md)
4. **Test with Method Projects:** Try building a Method solution to verify package access

**Back to:** [Microsoft Tools](./README.md)
