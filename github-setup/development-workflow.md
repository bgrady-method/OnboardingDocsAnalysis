# Development Workflow for Method

This guide covers the basic development workflow for Method developers working with GitHub repositories and the development environment.

## Overview

Method development involves working with multiple repositories, using automated setup scripts, and following established development practices. This guide covers the essential workflow for Method developers.

## Prerequisites

- Git installed and configured
- SSH keys set up for GitHub
- Repository access configured
- Method development environment set up
- Understanding of Method's system architecture

## Getting Started

### Initial Repository Setup

1. **Clone DeveloperTools Repository:**
   ```bash
   mkdir C:\MethodDev
   cd C:\MethodDev
   git clone https://github.com/methodcrm/DeveloperTools.git
   ```

2. **Run Automated Setup Scripts:**
   ```powershell
   # Open PowerShell as Administrator
   cd C:\MethodDev\DeveloperTools\DevSetup\build_local
   
   # Run the initialization script
   .\initialize.ps1
   
   # Run the build environment script
   .\build_local_environment.ps1
   ```

### Project Structure

After running the automated setup, your development folder will look like:

```
C:\MethodDev\
├── DeveloperTools\
├── method-platform-ui\
├── runtime-core\
├── ms-gateway-api\
├── ms-authentication-api\
├── [other Method repositories]
└── Blank\  # Required empty folder
```

## Daily Development Workflow

### Starting Your Day

1. **Update All Repositories:**
   ```powershell
   # Navigate to build_local directory
   cd C:\MethodDev\DeveloperTools\DevSetup\build_local
   
   # Run build script to pull latest and rebuild
   .\build_local_environment.ps1
   ```

2. **Check System Services:**
   - Verify IIS Application Pools are running
   - Check SQL Server services
   - Ensure Docker services are running (MongoDB, Redis, Elasticsearch, RabbitMQ)

3. **Connect to VPN** (if working remotely)

### Working on Features

1. **Choose a Repository to Work On:**
   ```bash
   cd C:\MethodDev\[repository-name]
   ```

2. **Create Feature Branch:**
   ```bash
   git checkout main
   git pull origin main
   git checkout -b feature/your-feature-name
   ```

3. **Make Changes and Commit:**
   ```bash
   # Make your changes
   git add .
   git commit -m "Clear description of changes"
   git push origin feature/your-feature-name
   ```

4. **Create Pull Request:**
   - Go to GitHub
   - Create PR from your feature branch to main
   - Add reviewers and description
   - Wait for approval and merge

## Method-Specific Development

### Critical Projects Health Checks

Important projects to monitor and test:

1. **ms-gateway-api**
   - Health Check: https://api.methodlocal.com/v2/health/check
   - Press "play" button with "IIS" selected in Visual Studio

2. **Runtime Project**
   - Open: C:\MethodDev\runtime-core\Runtime.stack.sln
   - Set startup project to Runtime.Core.Api
   - Ensure "Play" profile is "IIS"

3. **ms-authentication-api**
   - Health Check: https://api.methodlocal.com/v2/authentication/health/check

4. **Method UI**
   - Build frontend: `npm run build:dev` in MethodUI folder
   - Health Check: Access https://methodlocal.com

### Building Projects

#### Frontend Projects (method-platform-ui)
```bash
cd C:\MethodDev\method-platform-ui\m-one
npm install
npm run build -ws

cd C:\MethodDev\method-platform-ui\MethodUI
npm install
npm run build:dev
```

#### Backend Projects
```bash
# Open in Visual Studio
# Set startup project appropriately
# Press F5 or use "Start Debugging"
```

## Testing and Validation

### Local Account Setup

Create a local test account:

1. **Prerequisites:**
   - All services running
   - VPN connected
   - Hosts file updated with local URLs

2. **Create Account:**
   - Go to https://signup.methodlocal.com/methodcrm?domain=qbd
   - Enter your information
   - Add organization name to hosts file
   - Complete signup process
   - Use Ctrl+Shift+, to skip QB sync

### Health Checks

Run health checks for all services to verify everything is working:

- Gateway API: https://api.methodlocal.com/v2/health/check
- Authentication: https://api.methodlocal.com/v2/authentication/health/check
- Identity: https://api.methodlocal.com/v2/identity/health/check
- And other service endpoints...

## Troubleshooting

### Common Development Issues

**Build Failures:**
- Ensure VPN is connected for NuGet restoration
- Check that all services are running
- Verify correct Visual Studio startup project

**Service Not Responding:**
- Check IIS Application Pools
- Verify service is built and running
- Check logs in D:\logs\

**Database Issues:**
- Ensure SQL Server is running
- Check connection strings
- Verify database backups are restored

### Log Files

Check logs for errors:
- **Location:** D:\logs\
- **Structure:** Organized by project/service
- **Types:** Debug logs and error logs by date

## Code Review Process

### Creating Pull Requests

1. **Clear Title:** Describe what the PR does
2. **Good Description:** Explain changes and reasoning
3. **Test Your Changes:** Ensure everything works locally
4. **Request Reviewers:** Add relevant team members

### Code Review Guidelines

- **Test locally** before approving
- **Provide constructive feedback**
- **Check for Method coding standards**
- **Verify documentation updates**

## Integration with Method Tools

### Postman Collections

Import Method Postman collections:
1. Open Postman
2. Import from: C:\MethodDev\DeveloperTools\AdHocTools\PostMan
3. Use for API testing and development

### Visual Studio Configuration

- **Sign in** with MSDN credentials
- **Configure NuGet** sources for Method packages
- **Set appropriate startup projects**
- **Use "IIS" profile** for web projects

## Best Practices

### Development Environment

1. **Keep environment updated** - Run build scripts regularly
2. **Monitor service health** - Check health endpoints frequently  
3. **Use VPN when needed** - For NuGet and internal services
4. **Organize work** - Use clear branch names and commit messages

### Code Quality

1. **Test thoroughly** before creating PRs
2. **Follow Method standards** for coding and naming
3. **Keep commits focused** - Single responsibility per commit
4. **Document complex changes** - Add comments and update docs

### Collaboration

1. **Communicate changes** - Let team know about significant work
2. **Review others' code** - Participate in code reviews
3. **Share knowledge** - Help teammates with setup and issues
4. **Ask for help** - Don't hesitate to ask questions

## Next Steps

After mastering the development workflow:

1. **Explore Method Architecture:** Understand the system design
2. **Learn Specific Technologies:** Dive deeper into .NET, React, etc.
3. **Contribute to Projects:** Start working on assigned tasks
4. **Participate in Code Reviews:** Help maintain code quality

## Getting Help

- **Team Members:** Ask colleagues for assistance
- **Documentation:** Reference Method internal docs
- **Logs:** Check D:\logs\ for error details
- **Health Checks:** Use service endpoints to verify status

**Back to:** [GitHub Setup](./README.md)
