# SQL Server Installation

## 4.3. SQL Server Installation

1) Download **SQL Server 2022 (or latest) Developer Edition** from here and run it: ([https://go.microsoft.com/fwlink/?linkid=866662](https://go.microsoft.com/fwlink/?linkid=866662))

> **Note:** SQL Server 2022 is recommended. It is fully backward compatible with SQL Server 2019 databases and backup files. The Method application databases restore and function correctly on SQL Server 2022.

2) Choose "**Basic**" under *Installation Type*, and continue.

![][image17]

3) Once Finished click on "**Customize**" button

![][image18]

4) Click **Next** until you reach the "*Installation Type*" step. 

   ⇒ Select "**Add features to an existing instance of SQL Server**" from radio options  
   ⇒ click **Next**.

   Note: Keep the instance as "**MSSQLSERVER**"

   ![][image19]

5) Under the "*Feature Selection"* step, ensure "**Database Engine Services**" & "**Full-Text and Semantic Extractions for Search**" are selected ⇒ click "Next"  
   ![][image20]

6) Continue pressing *Next* & *Install* in the following steps until you reach "**Complete**"

You should see a summary of installed items with a status of *'**Succeeded**'*

*** You can now close the Installation Wizard. ***

## Important Features to Install

Make sure these components are selected:

- **Database Engine Services** - Core SQL Server functionality
- **Full-Text and Semantic Extractions for Search** - Required for Method's search features
- **SQL Server Replication** - For data synchronization features
- **Client Tools Connectivity** - For connecting applications to SQL Server

## Instance Configuration

- **Instance Name**: Use the default "MSSQLSERVER"
- **Authentication Mode**: Will be configured later in SQL Permissions section
- **Data Directories**: Use default locations unless specific requirements

## Post-Installation

After installation completes:

1. **Verify Installation** - SQL Server should appear in Services
2. **Check Service Status** - SQL Server service should be running
3. **Test Connection** - Will be verified when setting up SQL aliases

**Next:** [SQL Aliases Setup](./sql-aliases.md)
