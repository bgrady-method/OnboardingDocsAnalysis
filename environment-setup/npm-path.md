# NPM Path Configuration

This guide covers configuring system PATH variables for NPM and Node.js development tools in the Method development environment.

## Overview

Proper PATH configuration ensures that NPM, Node.js, and related development tools are accessible from any command line interface, enabling efficient development workflow for Method applications.

## Prerequisites

- Node.js and NPM installed (via initialize.ps1 script from DeveloperTools)
- Administrator access for environment variable modification
- Understanding of Windows PATH environment variables

## PATH Configuration Process

Based on the Method Developer Machine Setup, you need to add NPM to the system PATH manually after the automated installation.

### Step 1: Open Environment Variables

1. **Access System Environment Variables**
   - Press Windows Key
   - Type "env"
   - Windows will suggest "Edit the System Environment Variables"
   - Click that option

2. **Open PATH Editor**
   - In the System Properties window, click "Environment Variables"
   - Under "System variables", highlight the "Path" variable
   - Click "Edit"

### Step 2: Add NPM Path

1. **Add NPM Binary Path**
   - In the "Edit environment variable" window, click "New"
   - Paste the following path:
     ```
     C:\Program Files\nodejs\node_modules\npm\bin
     ```

2. **Save Changes**
   - Press "OK" to close the environment variable editor
   - Press "OK" again to close System Properties

### Step 3: Verify Installation

Open a new Command Prompt or PowerShell window and test:

```cmd
# Test Node.js
node --version

# Test NPM
npm --version

# Test NPM path resolution
where npm

# Test global NPM packages location
npm config get prefix
```

## Additional NPM Configuration

### Global Package Configuration

Configure NPM for Method development needs:

```cmd
# Set NPM cache directory
npm config set cache "C:\npm-cache"

# Configure global packages directory (optional)
npm config set prefix "C:\npm-global"

# Add custom global directory to PATH if used
# Add C:\npm-global to PATH using the same process above
```

### Method-Specific NPM Settings

```cmd
# Install global packages needed for Method development
npm install -g grunt-cli
npm install -g typescript
npm install -g @angular/cli

# Verify global installations
npm list -g --depth=0
```

## Troubleshooting PATH Issues

### Common Issues from Method Setup

Based on the Method setup document, you may encounter these issues:

#### Issue 1: NPM Commands Not Recognized

**Symptoms**: `'npm' is not recognized as an internal or external command`

**Solutions**:
```cmd
# Verify Node.js installation
where node

# Check if npm exists
where npm

# Restart command prompt after PATH changes
# If still failing, reinstall Node.js
```

#### Issue 2: Global Package Installation Failures

**Symptoms**: Permission errors when installing global packages

**Solutions**:
```cmd
# Use custom global directory
mkdir C:\npm-global
npm config set prefix "C:\npm-global"

# Add custom directory to PATH
# Follow Step 2 above to add C:\npm-global to PATH
```

#### Issue 3: Grunt-CLI Not Found

From the Method setup, you may need to reinstall grunt-cli:

```cmd
# Install grunt-cli globally
npm install -g grunt-cli

# Verify installation
grunt --version
```

## Method Build Environment Integration

### NPM for Method Projects

The Method platform requires specific NPM configurations:

```cmd
# Navigate to Method UI project
cd C:\MethodDev\method-platform-ui\MethodUI

# Install project dependencies
npm install

# If you face errors, try legacy peer deps
npm install --legacy-peer-deps

# Install build tools
npm install -g grunt-cli

# Run Method-specific build commands
npm run build:dev
```

### Build Script Configuration

For Method projects, ensure your PATH supports build scripts:

```json
{
  "scripts": {
    "build:dev": "npm run prettify && npm run lint && npm run build:sass && npx mix"
  }
}
```

### Memory Configuration for Large Builds

For Method's large projects, you may need to increase Node.js memory:

```cmd
# Set Node.js memory options
set NODE_OPTIONS=--max-old-space-size=4096

# Or add to system environment variables permanently
setx NODE_OPTIONS "--max-old-space-size=4096"
```

## Advanced Configuration

### PowerShell Profile Setup

Add to PowerShell profile for persistent configuration:

```powershell
# Edit PowerShell profile
notepad $PROFILE

# Add these lines to profile:
$env:PATH += ";C:\Program Files\nodejs\node_modules\npm\bin"
$env:NODE_OPTIONS = "--max-old-space-size=4096"

# NPM aliases for common commands
Set-Alias -Name ni -Value "npm install"
Set-Alias -Name nb -Value "npm run build"
Set-Alias -Name ns -Value "npm start"
```

### Batch Script for Team Setup

Create `setup-npm-path.bat` for consistent team configuration:

```batch
@echo off
echo Configuring NPM PATH for Method development...

REM Add NPM to PATH
setx PATH "%PATH%;C:\Program Files\nodejs\node_modules\npm\bin" /M

REM Configure NPM
npm config set cache "C:\npm-cache"
mkdir C:\npm-cache

REM Install global packages for Method
npm install -g grunt-cli
npm install -g typescript

echo NPM PATH configuration complete!
echo Please restart your command prompt.
pause
```

## Integration with Method Development Tools

### DeveloperTools Integration

The Method DeveloperTools initialize.ps1 script installs Node.js and NPM, but you must manually add to PATH:

1. **After running initialize.ps1**:
   - Follow the PATH configuration steps above
   - Verify NPM is accessible

2. **Install additional Method dependencies**:
   ```cmd
   # These may need to be reinstalled after restarting PowerShell
   gem install sass
   npm install -g grunt-cli
   ```

### Build Local Environment Integration

When running build_local_environment.ps1, ensure NPM is properly configured:

```cmd
# Verify NPM is available before building
npm --version

# Navigate to DeveloperTools
cd C:\MethodDev\DeveloperTools\DevSetup\build_local

# Run build with properly configured PATH
.\build_local_environment.ps1
```

## Verification Commands

### Complete PATH Verification

```cmd
# Check PATH contains Node.js paths
echo %PATH% | findstr nodejs

# Verify NPM configuration
npm config list

# Test global package installation
npm install -g --dry-run typescript

# Check global packages location
npm root -g
```

### Method-Specific Testing

```cmd
# Test Method project NPM commands
cd C:\MethodDev\method-platform-ui\MethodUI
npm run build:dev

# Verify grunt-cli works
grunt --version

# Test TypeScript compilation (if used)
tsc --version
```

## Best Practices

### PATH Management

1. **Order Matters**: Ensure Node.js paths come before conflicting installations
2. **Clean Paths**: Remove duplicate or obsolete PATH entries
3. **Documentation**: Keep record of PATH modifications for team consistency
4. **Testing**: Always test in new command prompt after PATH changes

### NPM Configuration

1. **Cache Management**: Regularly clean NPM cache with `npm cache clean --force`
2. **Global vs Local**: Prefer local package installations for projects
3. **Version Consistency**: Ensure team uses same Node.js/NPM versions
4. **Security**: Regularly audit global packages with `npm audit`

### Team Consistency

1. **Standardized Setup**: Use shared scripts for PATH configuration
2. **Version Alignment**: Document required Node.js/NPM versions
3. **Configuration Sharing**: Share .npmrc files for project-specific settings
4. **Documentation**: Maintain setup instructions for new team members

## Environment Variables Summary

Essential environment variables for Method development:

```cmd
# Required PATH additions
C:\Program Files\nodejs\node_modules\npm\bin

# Optional but recommended
NODE_OPTIONS=--max-old-space-size=4096
NPM_CONFIG_CACHE=C:\npm-cache
NPM_CONFIG_PREFIX=C:\npm-global (if using custom global directory)
```

## Troubleshooting Reference

### Diagnostic Commands

```cmd
# Check current PATH
echo %PATH%

# Find Node.js installation
where node

# Find NPM installation  
where npm

# Check NPM configuration
npm config list

# Test NPM functionality
npm doctor

# Check Node.js and NPM versions
node --version && npm --version
```

### Reset NPM Configuration

If NPM configuration becomes corrupted:

```cmd
# Clear NPM cache
npm cache clean --force

# Reset NPM configuration
npm config delete prefix
npm config delete cache

# Reinstall global packages
npm install -g grunt-cli
```

## Next Steps

After configuring NPM PATH:
- [Build Local Environment](./build-environment.md) - Set up complete Method development environment
- Test Method application builds with proper NPM access
- Configure additional development tools that depend on NPM
- Set up team development standards and scripts

## Important Notes

🔧 **Manual Configuration Required**: Unlike other components, NPM PATH must be configured manually after the automated setup

⚡ **Restart Required**: Always restart command prompt/PowerShell after PATH changes

📝 **Team Consistency**: Ensure all team members follow the same PATH configuration process

🔄 **Build Integration**: Proper NPM PATH is essential for Method's build processes to work correctly
