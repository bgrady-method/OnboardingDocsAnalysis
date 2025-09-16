# SQL Permissions

## 4.6. Set Permissions

1. Press the *Windows Key* and search for "**SQL Server Management Studio**" via the Start Menu.

![][image28]

2. On the "*Connect to Server*" dialog, click "**Connect**"

### **4.6.1. Update SQL Authentication Mode**

In the "**Object Explorer**" pane on the left hand side, RightClick on your SQL Server instance ⇒ "**properties**" ⇒ "**Security**" ⇒ **Select** "**SQL Server and Windows Authentication Mode**"  
![][image29]  
![][image30]

### **4.6.2. Add "NT AUTHORITY\\NETWORK SYSTEM" to "Logins"**

1. In the "**Object Explorer**" pane on, Expand the tree ⇒ "**Security**" ⇒ "**Logins**" ⇒ *Right-click* on "**Logins**" ⇒ select "**New Login…**"  
   ![][image31]

2. In the "**Login - New**" popup window, Click "**Search**"  
   ![][image32]

3. Enter Network Service in the textarea ⇒ click "OK"  
   ![][image33]

4. You should now be seeing "NT AUTHORITY\\NETWORK SERVICE" as the "**Login name**" 

5. Select "**Server Roles**" from the menu on the left ⇒ ***check all the roles***  
   ![][image34]

6. Click "**OK**" ⇒ you should now see "NT AUTHORITY\\NETWORK SERVICE" under Logins

### **4.6.3. Add "runtime-core-engine" to "Logins"**

1. *Right-click* on "**Logins**" ⇒ select "**New Login…**"  
   ![][image31]

2. In the "**Login - New**" popup window

* **Login name:** **runtime-core-engine**

* **Select** "**SQL Server authentication**"  

* **Password:** **local**

* **Uncheck** "**Enforce password policy**"![][image35]

3. Select "**Server Roles**" from the menu on the left ⇒ Check "**public**" and "**sysadmin**"  
   ![][image36]

4. Click "**OK**" ⇒ you should now see runtime-core-engine under Logins

### **4.6.4. Fix bcpreader issue**

There is a [known issue](https://methodme.slack.com/archives/CDJNL2ZBJ/p1680726161304529) that is fixed by running this SQL within your local server:

```sql
If not Exists (select loginname from master.dbo.syslogins where name = 'bcpreader')
Begin
    CREATE LOGIN bcpreader WITH PASSWORD = 'Gravity4fRee'
END
```

## **IMPORTANT: Restart SQL Server**

After adding new users above, you must **restart your SQL Server** by:

1) Open "**SQL Server {2019} Configuration Manager**" from the Start Menu

2) Click "**SQL Server Services**" ⇒ Right Click "**SQL Server**" ⇒ **Restart**  
   ![][image37]

3) Also make sure "**SQL Server Agent**" is running

## Why These Permissions Are Needed

- **NT AUTHORITY\\NETWORK SERVICE** - Required for IIS application pools to access SQL Server
- **runtime-core-engine** - Method's core business logic engine requires database access
- **bcpreader** - Used by Method's data import/export processes
- **Mixed Authentication Mode** - Allows both Windows and SQL Server authentication methods

**Next:** [SQL Backups](./sql-backups.md)
