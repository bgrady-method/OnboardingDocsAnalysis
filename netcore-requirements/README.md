# .NET Core Requirements

This section covers .NET Core specific environment setup.

## Sections

1. [ASP.NET Core Environment Variables](./aspnetcore-env.md)
2. [AWS Credentials Setup](./aws-credentials.md)

These configurations are required for .NET Core applications to run properly in the development environment.

## Overview

.NET Core applications in Method's ecosystem require specific environment configuration:

- **Environment Variables** - ASP.NET Core and .NET runtime settings
- **AWS Integration** - Secret Manager credentials for configuration and secrets
- **Development Configuration** - Local development environment settings

⏱️ **Estimated Time:** 30 minutes

📋 **Prerequisites:**
- .NET Core runtime installed (from Software Installation section)
- Administrator access for system environment variables
- Access to Method's AWS credentials file

⚠️ **Important:**
- Environment variables must be set at system level, not user level
- AWS credentials are required for all Method microservices
- Incorrect configuration will prevent Method applications from starting

## Configuration Components

This section configures:

- **ASPNETCORE_ENVIRONMENT** - Set to "Development" for local development
- **DOTNET_ENVIRONMENT** - .NET runtime environment configuration  
- **DOTNET_HOST_PATH** - Path to .NET runtime executable
- **AWS Credentials** - Access keys for Secret Manager integration

## AWS Secret Manager Integration

Method uses AWS Secret Manager for:
- Database connection strings
- API keys and external service credentials
- Encryption keys and security tokens
- Environment-specific configuration values

**Next:** [Local Development](../local-development/README.md)
