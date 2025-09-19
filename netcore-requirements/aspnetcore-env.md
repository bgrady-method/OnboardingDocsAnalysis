# ASP.NET Core Environment Variables

This guide covers setting up the required environment variables for ASP.NET Core applications in Method's development environment, based on the Method Developer Machine Setup documentation.

## Overview

ASP.NET Core applications require specific environment variables to function properly in the local development environment. This guide covers the essential variables needed for Method development.

## Required Environment Variables

### Setting Environment Variables

1. **Open Environment Variables:**
   - Go to **Control Panel\System and Security\System** 
   - Click "**Advanced System Settings**" 
   - Click "**Environment Variables**"

2. **Add System Variables:**
   - Click on "**New**" under SYSTEM VARIABLES

### ASPNETCORE_ENVIRONMENT

**Variable Name:** `ASPNETCORE_ENVIRONMENT`
**Variable Value:** `Development`

1. Click on "**New**" under SYSTEM VARIABLES
2. Add the parameters above
3. Click "OK"

### DOTNET_ENVIRONMENT

**Variable Name:** `DOTNET_ENVIRONMENT`
**Variable Value:** `Development`

1. Click on "**New**" under SYSTEM VARIABLES again
2. Add the parameters above
3. Click "OK"

### DOTNET_HOST_PATH

**Variable Name:** `DOTNET_HOST_PATH`
**Variable Value:** `%ProgramFiles%\dotnet\dotnet.exe` (double check this path)

1. Click on "**New**" under SYSTEM VARIABLES
2. Add the parameters above
3. Click "OK" → "Ok"

## Verification

### Check Variables are Set

Open a new PowerShell window and check that variables are set:

```powershell
# Check ASPNETCORE_ENVIRONMENT
$env:ASPNETCORE_ENVIRONMENT

# Check DOTNET_ENVIRONMENT  
$env:DOTNET_ENVIRONMENT

# Check DOTNET_HOST_PATH
$env:DOTNET_HOST_PATH
```

All should return the values you set.

### Test .NET Installation

```powershell
# Verify .NET is working
dotnet --version

# Check installed SDKs
dotnet --list-sdks

# Check installed runtimes
dotnet --list-runtimes
```

## Troubleshooting

### Environment Variables Not Taking Effect

- **Restart applications** after setting environment variables
- **Close and reopen** PowerShell/Command Prompt windows
- **Restart Visual Studio** for changes to take effect

### Path Issues

- **Verify the dotnet.exe path** exists at the location specified
- **Update the path** if .NET is installed in a different location
- **Check both Program Files locations** (x86 and regular)

### Common Locations

.NET is typically installed in:
- `C:\Program Files\dotnet\dotnet.exe` (64-bit)
- `C:\Program Files (x86)\dotnet\dotnet.exe` (32-bit - older)

## Next Steps

After setting environment variables:

1. **Restart development tools** (Visual Studio, PowerShell)
2. **Configure AWS Credentials:** [AWS Credentials Setup](./aws-credentials.md)
3. **Test Method applications** to ensure proper environment detection

## Integration with Method Setup

These environment variables work with:

- **ASP.NET Core applications** - Control development/production behavior
- **Configuration files** - Enable loading of appsettings.Development.json
- **Visual Studio** - Development-time features and debugging
- **Method services** - Proper environment detection for local development

**Back to:** [.NET Core Requirements](./README.md)
