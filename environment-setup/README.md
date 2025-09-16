# Environment Setup

This section covers environment configuration and building the local development environment.

## Sections

1. [NPM Path Configuration](./npm-path.md)
2. [Build Local Environment](./build-environment.md)

Complete these steps to prepare your development environment for building and running Method applications.

## Overview

Environment setup involves configuring system paths and building Method's application ecosystem:

- **PATH Configuration** - Adding NPM and Node.js tools to system PATH
- **Build Process** - Automated building of all Method projects and dependencies
- **Project Synchronization** - Pulling latest code and rebuilding entire ecosystem

⏱️ **Estimated Time:** 30 minutes

📋 **Prerequisites:**
- All software installation sections completed
- VPN connection active for builds
- Developer Tools repository available
- Administrator access for environment variables

⚠️ **Critical:** 
- VPN connection required for successful builds and NuGet package restoration
- Build process can take significant time depending on system performance
- Some MSBuild paths may need manual correction for Visual Studio 2022

## Build Process Overview

The environment setup includes:

- **System Configuration** - PATH variables for development tools
- **Code Synchronization** - Git pull for all Method repositories
- **Dependency Resolution** - NuGet package restoration across all projects
- **Compilation** - Building entire Method application ecosystem

**Next:** [Certificates](../certificates/README.md)
