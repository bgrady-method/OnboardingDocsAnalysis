# SQL Aliases Setup

## 4.4. Setup SQL Aliases

1) Press the *Windows Key* and search for "**SQL Server {2019} Configuration Manager**" via the *Start Menu*.

![][image21]

2) If you cannot find the app try:  
   1) Navigating to the file location: C:\\Windows\\SysWOW64\\SQLServerManager16.msc (for your version).   
   2) Click SQLServerManager16.msc to open the Configuration Manager. You can also right-click SQLServerManager16.msc to pin the Configuration Manager to the Start Page or Task Bar.  

3) Expand **"SQL Server Network Configuration**" ⇒ select "**Protocols for MSSQLSERVER**"

⇒ Double-click on "**TCP/IP**" and make sure:

* "**Enabled**" is set to "**Yes**"   
* "**Listen All**" is set to "**Yes**".

⇒ Click "OK"

![][image22]

4) Expand "**SQL Native Client 11.0 Configuration**" ⇒ Right-Click on "**Aliases**" ⇒ Select "**New Alias**"

![][image23]

5) In the pop-up window, enter the following:

* **Alias Name**: `methodlocaldb`  
* **Port No**: `1433`  
* **Protocol**: `TCP/IP`  
* **Server:** `localhost`

![][image24]

Click "OK".

6) Repeat **Step 3** with "**SQL Server Network Configuration (32bit)**"  

And, under "*client protocols*" check "*TCP/IP*" to see if it's Enabled.

**Note -** on some machines with 64-bit SQL Server, "*SQL Server Network Configuration (32bit)*" is empty - just skip it.

If new alias is greyed out use the cliconfg.exe in both the 32-bit (folder SysWOW64) and 64-bit (folder System32)

7) Repeat **Step 4/5** with "**SQL Native Client 11.0 Configuration (32bit)**".  

And, under "*client protocols*" check "*TCP/IP*" to see if it's enabled.

8) Click "**SQL Server Services**" …

   1) Right click on "**SQL Server (MSSQLSERVER)**"

   2) Select "**Restart**" from the menu

![][image25]

## Add methodlocaldb to Host Files

9) Add methodlocaldb to host files by:

1. Open Windows PowerShell in 'Admin mode' by **Right-clicking** on the *Start Menu Button* ⇒ select "**Windows PowerShell (Admin)**"  
   ![][image26]

2. Execute this command to open host files: `notepad drivers\\etc\\hosts`

3. **Add a line at the bottom** of the *hosts file* that reads: `127.0.0.1 methodlocaldb`

4. **Add the rest of the hosts from the troubleshooting section** (this will make things easier in later steps)

5. **Save the file**

## Why Aliases Are Important

SQL aliases provide:

- **Consistent Connection Strings** - Applications can use the alias name instead of server names
- **Environment Flexibility** - Easy switching between development, staging, and production
- **Local Development** - Maps Method's expected database names to your local SQL Server

**Next:** [SSMS Installation](./ssms.md)
