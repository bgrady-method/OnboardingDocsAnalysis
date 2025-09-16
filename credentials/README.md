# Credentials

This section covers the various credentials needed for development setup.

## Sections

1. [Active Directory Credentials](./active-directory.md)
2. [MSDN Credentials](./msdn.md)

These credentials are essential for accessing various services and tools throughout the development environment.

## Overview

You'll need several different credentials throughout the setup process:

- **Active Directory (AD)** - For accessing Method's internal services, TFS, and database backups
- **MSDN** - For downloading Microsoft development tools and Visual Studio licensing

🔐 **Security Note:** All credentials should be stored securely in Passbolt

⏱️ **Estimated Time:** 15 minutes

📋 **Prerequisites:** Access to Method's Passbolt instance

## Where to Find Credentials

**Passbolt Location:**
- Log into Method's Passbolt system
- Navigate to "Individual" folder for personal credentials
- AD credentials should be clearly labeled
- MSDN credentials marked as "Team/Business" type

## Credential Usage Throughout Setup

Your credentials will be needed for:

1. **VPN Access** - Initial connection to Method's network
2. **Database Downloads** - Accessing dev-setup.methodintegration.com
3. **NuGet Feeds** - Method's internal package repository
4. **Visual Studio Licensing** - MSDN subscription activation
5. **TFS/Azure DevOps** - Source control access
6. **Build Processes** - Internal service authentication

⚠️ **Important:** Keep credentials secure and never store in plain text

**Next:** [GitHub Setup](../github-setup/README.md)
