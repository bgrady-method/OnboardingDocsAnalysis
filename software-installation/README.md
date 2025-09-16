# Software Installation

This section covers installing all required software for development.

## Sections

1. [Developer Tools](./developer-tools.md)
2. [Docker Packages](./docker-packages.md)
3. [SQL Server Installation](./sql-server.md)
4. [SQL Aliases Setup](./sql-aliases.md)
5. [SSMS Installation](./ssms.md)
6. [SQL Permissions](./sql-permissions.md)
7. [SQL Backups](./sql-backups.md)

Complete these sections in the order listed to ensure proper setup.

## Overview

This section covers the core software stack needed for Method development:

- **Developer Tools** - Automated installation of development essentials
- **Docker Services** - MongoDB, Redis, Elasticsearch, RabbitMQ containers
- **SQL Server** - Database engine and management tools
- **Database Configuration** - Aliases, permissions, and backup restoration

⏱️ **Estimated Time:** 2-3 hours

📋 **Prerequisites:**
- Administrator access for software installation
- VPN connection for some downloads
- C:\\MethodDev folder created
- Developer Tools repository cloned

⚠️ **Important:** 
- Some installations require multiple PowerShell sessions
- Docker must be running before executing docker package scripts
- SQL Server configuration is critical for database access

## Architecture Components

The software installation covers these key components:

- **Development Environment** - Chocolatey, Node.js, Ruby, Python tools
- **Database Layer** - SQL Server 2019+, SSMS, database aliases
- **Container Services** - Docker Desktop with Method's service containers
- **Build Tools** - MSBuild, NuGet, compilation dependencies

**Next:** [Microsoft Tools](../microsoft-tools/README.md)
