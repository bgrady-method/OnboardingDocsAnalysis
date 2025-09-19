# Visual Studio Installation

This guide covers installing and configuring Visual Studio Enterprise for Method development, based on the Method Developer Machine Setup documentation.

## Overview

Visual Studio Enterprise is the primary IDE for Method development. This guide covers the installation process and basic configuration needed for Method development.

## Prerequisites

- MSDN credentials from Passbolt
- Administrator access for software installation
- At least 20GB free disk space

## Download Visual Studio Enterprise

### Access MSDN Portal

1. **Get MSDN Credentials:**
   - Check Passbolt for MSDN credentials entry
   - Use the provided email and password

2. **Download from MSDN:**
   - Go to **MSDN Downloads:** https://my.visualstudio.com/Downloads
   - Enter the login credentials found under **MSDN Credentials**
   - Download the Latest Edition of **Visual Studio Enterprise 2022** (currently 2022)
   - This will download the installer for you

## Installation Process

### Run the Installer

1. **Run the Visual Studio Installer** that you just downloaded
2. You should see a single-screen wizard with multiple tabs at the top

### Required Workloads

From the **"Workloads"** Tab, **select** the following modules:

#### Essential Workloads
- **ASP.NET and web development**
- **.NET desktop development** 
- **Data storage and processing**

### Required Components

Make sure The **ASP.NET and development tools** are **Selected**:

**Note:** If "Development time IIS support" is not in the list, IIS may need to be enabled first in Windows. Search "Turn Windows features on or off". In the resulting window, select the item "Internet Information Service". Select OK, let the installer run, and restart if necessary. The Visual Studio installer may need to be restarted before the item is visible.

**Important:** You might need to install dotnet framework 4.6.2 manually, double check this.

### .NET Core Runtime Bundles

Select the .NetCore Runtime Bundles from the individual components:

**Note:** The .net runtime bundles are located under individual components in the Visual Studio Installer, 2.2 runtime is not available from here.

### Complete Installation

1. Leave all other options in the installer as is in their **default state**
2. Select **"Download All, then Install"**, and Click **"Install"**
3. **Restart** your machine once installation is **completed**

### Additional .NET Core Installation

**Download and Install** this specific *.NETCore Version*:
- https://dotnet.microsoft.com/download/dotnet-core/thank-you/sdk-2.2.207-windows-x64-installer

## Initial Configuration

### First Launch Setup

1. **Sign In:**
   - Press the Windows Key and search for **"Visual Studio 2022"** (currently 2022)
   - Click **"Sign in"** on the '*Welcome!*' prompt 
   - Enter the login information found under **MSDN Credentials**

2. **Start Development:**
   - Once signed in, click on **"Continue without code"** found at the bottom of the starting screen

## Next Steps

After Visual Studio installation:

1. **Configure NuGet Feeds:** [NuGet Feeds Setup](./nuget-feeds.md)
2. **Set Up Credentials:** [Active Directory Credentials Storage](./ad-credentials.md)
3. **Install Testing Tools:** [NUnit Test Adapter](./nunit-adapter.md)
4. **Add IIS Components:** [URL Rewrite Installation](./url-rewrite.md)

## Troubleshooting

### Common Issues

**Installation Failures:**
- Ensure you have enough disk space (minimum 20GB)
- Run installer as Administrator
- Temporarily disable antivirus during installation

**License Issues:**
- Verify MSDN subscription is active
- Sign out and sign back in to Visual Studio
- Contact IT for MSDN subscription support

**Missing Components:**
- Re-run installer and add missing workloads
- Verify all required components are selected
- Check that IIS features are enabled if needed

**Back to:** [Microsoft Tools](./README.md)
