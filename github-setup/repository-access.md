# Repository Access for Method Development

This guide covers accessing Method repositories on GitHub for development work.

## Overview

Once your GitHub account is set up and SSH keys are configured, you'll need access to Method's GitHub repositories to begin development work.

## Prerequisites

- GitHub account set up with Method email
- SSH keys configured and working
- Added to Method GitHub organization
- Git installed and configured

## Getting Repository Access

### Organization Access

1. **Contact Git Administrators** with your GitHub username:
   - Paul
   - Hossein  
   - Rich

2. **Provide Information:**
   - Your GitHub username
   - Method email address
   - Which repositories you need access to

3. **Wait for Organization Invite:**
   - You'll receive an email invitation to join the methodcrm organization
   - Accept the invitation to gain access

## Key Method Repositories

### Essential Repositories

The main repositories you'll work with for Method development:

- **DeveloperTools** - Setup scripts and development utilities
- **method-platform-ui** - Main Method UI application
- **runtime-core** - Core runtime services
- **ms-gateway-api** - API Gateway service
- **ms-authentication-api** - Authentication services
- **Various microservices** - Individual service repositories

### Repository Structure

Method uses multiple GitHub organizations:
- **methodcrm** - Main organization for core repositories
- **method-inc** - Internal tools and utilities  
- **method-public** - Public-facing repositories

## Setting Up Local Development

### Clone Developer Tools

The first repository to clone is DeveloperTools:

```bash
# Create Method development directory
mkdir C:\MethodDev
cd C:\MethodDev

# Clone the DeveloperTools repository
git clone https://github.com/methodcrm/DeveloperTools.git
```

### Repository Organization

Organize your local repositories under `C:\MethodDev\`:

```
C:\MethodDev\
├── DeveloperTools\
├── method-platform-ui\
├── runtime-core\
├── ms-gateway-api\
├── ms-authentication-api\
└── [other repositories]
```

## Repository Cloning

### Using SSH (Recommended)

```bash
# Clone using SSH (requires SSH keys setup)
git clone git@github.com:methodcrm/repository-name.git
```

### Using HTTPS (Alternative)

```bash
# Clone using HTTPS (will prompt for credentials)
git clone https://github.com/methodcrm/repository-name.git
```

## Automated Setup

### Using DeveloperTools Scripts

After cloning DeveloperTools, you can use automated scripts:

```powershell
# Navigate to build_local directory
cd C:\MethodDev\DeveloperTools\DevSetup\build_local

# Run build_local_environment.ps1 to clone and build all projects
.\build_local_environment.ps1
```

This script will:
- Clone all required Method repositories
- Switch to master branch
- Build all projects
- Set up dependencies

## Working with Method Repositories

### Branch Management

- **Main branch:** Usually `main` or `master`
- **Development:** Create feature branches from main
- **Pull requests:** Required for all changes

### Common Git Operations

```bash
# Update repository
git pull origin main

# Create feature branch
git checkout -b feature/your-feature-name

# Push changes
git push origin feature/your-feature-name
```

## Repository Health Check

### Verify Repository Access

Test that you can access and work with Method repositories:

```bash
# Test cloning a repository
git clone git@github.com:methodcrm/DeveloperTools.git test-clone

# Test SSH connection
ssh -T git@github.com

# Remove test clone
rm -rf test-clone
```

## Troubleshooting

### Common Access Issues

**Repository Not Found**
- Verify you have access to the methodcrm organization
- Check that the repository name is correct
- Ensure you're using the right GitHub account

**Permission Denied**
- Check SSH key setup
- Verify SSH agent is running
- Test SSH connection to GitHub

**Clone Failures**
- Try HTTPS instead of SSH temporarily
- Check network/VPN connection
- Verify repository exists and you have access

### Getting Help

If you can't access repositories:

1. **Verify organization membership** at https://github.com/methodcrm
2. **Contact administrators:** Paul, Hossein, or Rich
3. **Check SSH setup** following the SSH keys guide
4. **Test with public repositories** first

## Security Notes

- **Never commit credentials** to repositories
- **Use SSH keys** for authentication when possible
- **Keep access tokens secure** and rotate them regularly
- **Follow Method's security policies** for repository access

## Next Steps

After gaining repository access:

1. **Clone DeveloperTools:** Essential for setup scripts
2. **Run automated setup:** Use build_local_environment.ps1
3. **Learn branching strategy:** [Branching Strategy](./branching-strategy.md)
4. **Understand workflow:** [Development Workflow](./development-workflow.md)

**Back to:** [GitHub Setup](./README.md)
