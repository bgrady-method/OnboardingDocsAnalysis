# SSMS Installation

## 4.5. SSMS Installation

To install **SQL Server Management Studio**, open the link below and download the latest version available:

[Download SQL Server Management Studio (SSMS) - SQL Server Management Studio (SSMS)](https://docs.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms)

Run the setup file and click "**Install**"

![][image27]

## What is SSMS?

SQL Server Management Studio (SSMS) is a free IDE for managing SQL Server infrastructure. It provides:

- **Database Management** - Create, modify, and manage databases
- **Query Editor** - Write and execute T-SQL queries
- **Object Explorer** - Browse database objects and schemas
- **Backup/Restore** - Database backup and restore operations
- **Security Management** - User accounts, roles, and permissions
- **Performance Monitoring** - Query performance and server statistics

## Installation Notes

- **System Requirements** - Requires .NET Framework 4.7.2 or later
- **Installation Size** - Approximately 500MB download
- **Prerequisites** - No additional software required
- **Compatibility** - Works with all supported SQL Server versions

## First Launch

After installation:

1. **Launch SSMS** from Start Menu
2. **Connect to Server** using these settings:
   - Server type: Database Engine
   - Server name: localhost (or methodlocaldb)
   - Authentication: Windows Authentication
3. **Verify Connection** - You should see your SQL Server instance in Object Explorer

## Common Features You'll Use

During Method development, you'll frequently use:

- **Object Explorer** - Navigate databases and tables
- **New Query** - Write SQL commands and scripts
- **Database Restore** - Restore Method's database backups
- **Security Settings** - Configure user permissions
- **Activity Monitor** - Monitor SQL Server performance

**Next:** [SQL Permissions](./sql-permissions.md)
