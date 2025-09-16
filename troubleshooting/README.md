# Troubleshooting

This section contains troubleshooting guides and reference information.

## Sections

1. [General Troubleshooting](./general.md)
2. [Log Files](./log-files.md)
3. [Debugging .NET Core Projects](./debug-netcore.md)
4. [Debugging .NET Framework Projects](./debug-netframework.md)
5. [App Pools & Services](./app-pools-services.md)
6. [Visual Studio NuGet Issues](./nuget-issues.md)
7. [Host File List](./host-file-list.md)
8. [DNS Setup (Optional)](./dns-setup.md)
9. [New Hard Drive Setup](./new-drive-setup.md)
10. [Download and Restore Databases](./database-restore.md)
11. [Appendix - IIS Features](./appendix-iis.md)

Use these guides when encountering issues during setup or development.

## Overview

Troubleshooting resources for common development environment issues:

- **General Guidelines** - Systematic approach to problem solving
- **Log Analysis** - Understanding Method's logging infrastructure
- **Debugging Tools** - Visual Studio debugging for different project types
- **Service Management** - IIS, SQL Server, and Docker service issues
- **Network Configuration** - Host files, DNS, and local development URLs
- **Database Issues** - Backup, restore, and connection problems

⏱️ **When Needed:** Reference material for problem resolution

📋 **Key Resources:**
- D:\\logs\\ directory for application logs
- Host file configuration for local development
- Service management for IIS and SQL Server

## Quick Reference

**Most Common Issues:**
1. **Service not running** - Check IIS app pools and SQL Server services
2. **Database connection** - Verify SQL aliases and permissions
3. **Build failures** - Check VPN connection and NuGet feeds
4. **Local URLs not working** - Verify host file entries
5. **SSL certificate errors** - Check certificate installation and bindings

**Log Locations:**
- Application logs: `D:\\logs\\`
- IIS logs: `C:\\inetpub\\logs\\LogFiles\\`
- SQL Server logs: SQL Server Management Studio → Management → SQL Server Logs

**Health Check Priority:**
1. SQL Server and Agent running
2. Docker containers operational (MongoDB, Redis, Elasticsearch, RabbitMQ)
3. IIS application pools started
4. Method microservices responding to health checks

## Problem Resolution Process

1. **Identify the Issue** - Note error messages and symptoms
2. **Check Logs** - Review relevant log files for detailed errors
3. **Verify Services** - Ensure all required services are running
4. **Test Components** - Use health checks to isolate problems
5. **Consult Documentation** - Reference specific troubleshooting guides
6. **Seek Help** - Contact team via Slack channels if needed

**Back to:** [Documentation Home](../README.md)
