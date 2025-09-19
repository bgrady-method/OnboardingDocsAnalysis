# Build Local Environment

This guide covers the automated building and synchronization of the complete Method development environment, including all projects and dependencies.

## Overview

The Method build environment process synchronizes all repositories, restores dependencies, and builds the entire application ecosystem to create a fully functional local development environment using the `build_local_environment.ps1` script.

## Prerequisites

- All software installation sections completed (initialize.ps1 executed)
- VPN connection active and stable for NuGet restoration  
- NPM PATH configuration completed ([NPM Path Configuration](./npm-path.md))
- Developer Tools repository available at `C:\MethodDev\DeveloperTools`
- Administrator access for build processes
- Visual Studio 2022 Enterprise installed with appropriate workloads

## Build Process Overview

### What the Build Process Does

The `build_local_environment.ps1` script performs these operations:

1. **Repository Synchronization**: Changes to master branch and pulls latest changes from all Method repositories
2. **Dependency Restoration**: Downloads and installs NuGet packages for all projects
3. **Project Compilation**: Builds all .NET Framework and .NET Core solutions
4. **Frontend Assets**: Compiles JavaScript, CSS, and other frontend resources

### Important Pre-Configuration

Before running the build script, you must update the projects.yaml file:

#### Update MSBuild Paths for Visual Studio 2022

1. **Navigate to Configuration File**
   ```cmd
   cd C:\MethodDev\DeveloperTools\DevSetup\build_local
   notepad projects.yaml
   ```

2. **Update internal-attribution-api Section**
   - Search for 'internal-attribution-api' in the file
   - Replace 'Professional' with 'Enterprise' in restore and build command paths
   
   **Update restore command to**:
   ```yaml
   restore: C:\'Program Files'\'Microsoft Visual Studio'\2022\Enterprise\MSBuild\Current\Bin\MSBuild.exe AttributionSrv.sln -t:restore -p:RestorePackagesConfig=true
   ```
   
   **Update build command to**:
   ```yaml
   build: C:\'Program Files'\'Microsoft Visual Studio'\2022\Enterprise\MSBuild\Current\Bin\MSBuild.exe AttributionSrv.sln
   ```

## Executing the Build Process

### Step 1: Prepare Environment

1. **Ensure VPN Connection**
   ```cmd
   # Test VPN connectivity to Method resources
   ping tfs.method.me
   nslookup method.me
   ```

2. **Navigate to Build Directory**
   ```cmd
   # Open PowerShell as Administrator
   cd C:\MethodDev\DeveloperTools\DevSetup\build_local
   ```

3. **Verify Prerequisites**
   ```cmd
   # Check disk space (build requires significant space)
   dir C:\ /-c
   
   # Verify git is accessible
   git --version
   
   # Check Visual Studio MSBuild
   dir "C:\Program Files\Microsoft Visual Studio\2022\Enterprise\MSBuild\Current\Bin\MSBuild.exe"
   ```

### Step 2: Execute Build Script

#### Run the Automated Build

```powershell
# Navigate to build directory
cd C:\MethodDev\DeveloperTools\DevSetup\build_local

# Execute the build script
.\build_local_environment.ps1
```

#### Monitor Build Progress

The script will:
- Switch all repositories to master branch
- Pull latest changes from GitHub
- Restore NuGet packages for each project
- Build solutions in dependency order
- Report success/failure for each step

Expected output indicators:
- "Switching to master branch for [repository]"
- "Pulling latest changes for [repository]"  
- "Restoring packages for [solution]"
- "Building [solution]"
- "Build completed successfully" or error messages

### Step 3: Build Process Details

#### Repository Updates

The script updates these key Method repositories:
- `method-platform-ui` - Main Method application UI
- `runtime-core` - Core runtime services
- `ms-gateway-api` - API gateway service
- `ms-authentication-api` - Authentication microservice
- `ms-identity-api` - Identity management service
- Additional microservices and supporting projects

#### Dependency Restoration

For each project, the script:
- Restores NuGet packages using MSBuild
- Downloads required .NET dependencies
- Resolves package version conflicts
- Configures package sources (including method-ms feed)

#### Build Compilation

Solutions are built in dependency order:
- Core libraries first
- Microservices
- Web applications
- Frontend assets

## Time and Resource Requirements

⏱️ **Estimated Time**: 30-60 minutes (depending on system performance and network speed)  
📦 **Disk Space**: 10-15 GB for complete build  
🌐 **Network**: VPN required for NuGet package restoration  
💾 **Memory**: 8GB+ RAM recommended for optimal build performance  
🔄 **Repeatability**: Script can be run multiple times to rebuild/pull projects

## Post-Build Verification

### Verify Build Success

1. **Check Build Output**
   - Review console output for any "FAILED" messages
   - Ensure all solutions built successfully
   - Note any warnings for investigation

2. **Verify Key Directories**
   ```cmd
   # Check for compiled binaries
   dir C:\MethodDev\method-platform-ui\MethodUI\bin\
   dir C:\MethodDev\runtime-core\Runtime.Core.Api\bin\
   dir C:\MethodDev\ms-gateway-api\bin\
   ```

3. **Test Core Services**
   ```cmd
   # These will be tested during critical projects setup
   # Verification happens in subsequent setup steps
   ```

## Common Build Issues and Solutions

### Issue 1: VPN Connection Problems

**Symptoms**: NuGet package restoration failures, authentication errors

**Solutions**:
```cmd
# Verify VPN connectivity
ping tfs.method.me

# Test NuGet source access
nuget sources list

# Check Windows credentials for TFS
# Go to Control Panel > Credential Manager
# Verify tfs.method.me credentials are current
```

### Issue 2: MSBuild Path Issues

**Symptoms**: "MSBuild not found" errors, build failures

**Solutions**:
```cmd
# Verify MSBuild location
dir "C:\Program Files\Microsoft Visual Studio\2022\Enterprise\MSBuild\Current\Bin\MSBuild.exe"

# If path differs, update projects.yaml
# Edit the file to match your Visual Studio installation path
```

### Issue 3: Git Repository Issues

**Symptoms**: Git pull failures, repository access denied

**Solutions**:
```cmd
# Check git configuration
git config --list

# Verify SSH key authentication
ssh -T git@github.com

# Check repository access
cd C:\MethodDev\method-platform-ui
git status
git remote -v
```

### Issue 4: Memory Issues During Build

**Symptoms**: Out of memory errors, build timeouts

**Solutions**:
```cmd
# Increase Node.js memory for frontend builds
set NODE_OPTIONS=--max-old-space-size=4096

# Close unnecessary applications
# Ensure sufficient available RAM

# Consider building projects individually if issues persist
```

## Manual Build Process (If Automated Script Fails)

If the automated script encounters issues, you can build manually:

### Manual Repository Updates

```cmd
# Navigate to each repository and update
cd C:\MethodDev

# For each project directory:
cd method-platform-ui
git checkout master
git pull origin master

cd ..\runtime-core  
git checkout master
git pull origin master

# Repeat for other repositories
```

### Manual NuGet Restoration

```cmd
# Restore packages for each solution
cd C:\MethodDev\method-platform-ui
nuget restore MethodPhoenix.sln

cd C:\MethodDev\runtime-core
nuget restore Runtime.stack.sln

# Continue for other solutions
```

### Manual Build Compilation

```cmd
# Build solutions using MSBuild
"C:\Program Files\Microsoft Visual Studio\2022\Enterprise\MSBuild\Current\Bin\MSBuild.exe" MethodPhoenix.sln /p:Configuration=Debug

"C:\Program Files\Microsoft Visual Studio\2022\Enterprise\MSBuild\Current\Bin\MSBuild.exe" Runtime.stack.sln /p:Configuration=Debug
```

## Frontend Build Requirements

### Method Platform UI Build

After the main build script, the Method UI requires additional frontend setup:

```cmd
# Navigate to Method UI
cd C:\MethodDev\method-platform-ui\MethodUI

# Install NPM dependencies
npm install

# If errors occur, try:
npm install --legacy-peer-deps

# Install grunt-cli globally
npm install -g grunt-cli

# Build frontend assets
npm run build:dev
```

### Handle Memory Issues

If you encounter memory errors during frontend builds:

```json
// Update package.json build:dev script to:
"build:dev": "set NODE_OPTIONS=--max-old-space-size=4096 && npm run prettify && npm run lint && npm run build:sass && npx mix"
```

## Integration with Development Workflow

### Daily Development Routine

```cmd
# Quick update and rebuild
cd C:\MethodDev\DeveloperTools\DevSetup\build_local
.\build_local_environment.ps1

# This pulls latest changes and rebuilds all projects
```

### Selective Project Building

```cmd
# Build specific project instead of full environment
cd C:\MethodDev\method-platform-ui
"C:\Program Files\Microsoft Visual Studio\2022\Enterprise\MSBuild\Current\Bin\MSBuild.exe" MethodPhoenix.sln
```

### Build Performance Optimization

1. **Use SSD Storage**: Ensure C:\MethodDev is on SSD for faster I/O
2. **Sufficient RAM**: 16GB+ recommended for large solution builds  
3. **Close Applications**: Free up system resources during builds
4. **Incremental Builds**: Use individual project builds when possible

## Environment Configuration

### Required Environment Variables

Ensure these are set before building:
```cmd
# Set during .NET Core setup
ASPNETCORE_ENVIRONMENT=Development
DOTNET_ENVIRONMENT=Development
DOTNET_HOST_PATH=%ProgramFiles%\dotnet\dotnet.exe

# NPM memory configuration
NODE_OPTIONS=--max-old-space-size=4096
```

### Build Dependencies

The build process requires:
- Visual Studio 2022 Enterprise
- .NET Core SDK 2.2.207 (specific version)
- NuGet command line tools
- Git for Windows
- Node.js and NPM (properly configured in PATH)

## Success Indicators

### Build Completion Signs

✅ **Successful Build Indicators**:
- All repository pulls complete without errors
- NuGet package restoration succeeds for all projects
- MSBuild compilation completes without errors
- No "FAILED" messages in console output
- All project bin directories contain compiled assemblies

### Ready for Next Steps

After successful build completion, you're ready for:
- IIS configuration and site setup
- Database restoration and configuration  
- Certificate installation and HTTPS setup
- Critical projects health checks

## Troubleshooting Commands

```cmd
# Check git status across repositories
cd C:\MethodDev
for /d %i in (*) do (cd "%i" && if exist .git (echo === %i === && git status --short) && cd ..)

# Verify NuGet sources
nuget sources list

# Test MSBuild availability
"C:\Program Files\Microsoft Visual Studio\2022\Enterprise\MSBuild\Current\Bin\MSBuild.exe" /?

# Check NPM global packages
npm list -g --depth=0
```

## Next Steps

After successful build environment setup:
- Configure IIS sites and application pools (import_sites_pools.ps1)
- Install and configure SSL certificates
- Restore SQL and MongoDB databases
- Set up and test critical Method projects
- Create local Method account for testing

## Maintenance

### Regular Maintenance Tasks

- **Daily**: Pull latest changes and rebuild as needed
- **Weekly**: Full clean build to ensure environment consistency  
- **Monthly**: Update dependencies and clean build caches
- **Quarterly**: Review and update build scripts and documentation

### Build Environment Health

```cmd
# Quick health check
cd C:\MethodDev\DeveloperTools\DevSetup\build_local
dir projects.yaml
git status

# Verify core tools
node --version
npm --version
git --version
dotnet --version
```

This completes the build environment setup process, providing a foundation for Method application development and testing.
