# SQL Backups

## 4.7. SQL Backup: QBO & QBDT

1) Open File Explorer and **create** these folders in C:\\ drive:

"C:\\Methoddata\\MethodIntegrationSQL"

2) Give Network Service user full access to the C:\\Methoddata\\MethodIntegrationSQL folder (will be needed for some App Pools in IIS):  
   1) right click in the MethodData folder  
   2) go to Properties  
   3) go to Security tab  
   4) add NETWORK SERVICE user to Group and User names, and then select it if it is not selected.  
   5) give "Full Control" to NETWORK SERVICE in the "Permissions for NETWORK SERVICE" pane  

3) Go to: [**`https://dev-setup.methodintegration.com/dbs/`**](https://dev-setup.methodintegration.com/dbs/)

* If prompted to sign in, Enter your [Method Active Directory Credentials](../credentials/active-directory.md)

4) Download the latest version of the following files and move them into C:\\Methoddata\\MethodIntegrationSQL:

* [TemplateMethodNewQBO.bak](https://dev-setup.methodintegration.com/dbs/templatemethodnewqbo/sql/FULL/) (this db is used to generate the offline in postman accountmanagement, in order to signup)

* [TemplateDevV2.bak](https://dev-setup.methodintegration.com/dbs/templatedevv2/sql/FULL/) (this db is used to develop apps and push to appstore)

## 4.8. SQL Backups Restoration

1. Open powershell as administrator and navigate to folder C:\\MethodDev\\DeveloperTools\\DevSetup\\build_local 

2. Run the command **restore_dbs.ps1** to download and restore the needed system sql and mongodb databases.

   1. When the console asks if there are any user dbs to be restored:

      1. If you don't want any user dbs (from builder) to be copied, then type 'N' and proceed

      2. If there are any user dbs you want to copy, then create a file: user_dbs.yaml, and include the user db names in the file. And type 'Y' when you run restore_dbs.ps1 (Make sure the user's .bak file is present in the builder env)

   2. If you create a user_dbs.yaml file in that folder it will download any database you list in there as well. ([readme](https://github.com/methodcrm/DeveloperTools/tree/master/DevSetup/build_local#readme))

* When prompted for credentials, Enter your [Method Active Directory Credentials](../credentials/active-directory.md)
* If you see an error you may need to install sqlserver for powershell to run the commands  
  * Using "Install-Module -Name SqlServer"  
  * If it doesnt work: might have to use -AllowClobber parameter [https://dba.stackexchange.com/questions/174704/is-it-normal-for-install-module-sqlserver-to-clobber-existing-modules](https://dba.stackexchange.com/questions/174704/is-it-normal-for-install-module-sqlserver-to-clobber-existing-modules)

## Full List of Databases

Make sure in SSMS these databases are present:

**SQL Databases:**
* AlocetSystem  
* Method App Store  
* MethodNotifications  
* MethodOAuthSecurity  
* MethodShortURLRedirects  
* TemplateDevV2  
* TemplateMethodNewQBO

**MongoDBs:**
* Accounts  
* AlocetSystem  
* Authentication  
* Methodappstore  
* methodmobilenotifications  
* oauth_token  
* Preferences  
* TemplateDevV2

To view your MongoDBs you should have Robo 3T installed from a previous step.  
To view your SQL Databases open Microsoft SQL Server Management Studio.

## Troubleshooting restore_dbs.ps1

* If you're running SQL server 2019, you will see the following error:  
  * Invoke-Sqlcmd : A parameter cannot be found that matches parameter name 'TrustServerCertificate'.  
    You can fix this by removing this parameter from the script  
  * You can manually restore .bak files in SSMS if any script fails:  
    Open SQL Management Studio and connect to the local server. Once connected, right-click on the "Databases" folder and select "Restore Database". Then, follow these steps and choose the path that contains the backup file for your database.

## What These Databases Contain

- **AlocetSystem** - Core Method system database
- **TemplateDevV2** - Template database for app development  
- **TemplateMethodNewQBO** - QuickBooks Online integration template
- **Method App Store** - Application marketplace data
- **MethodNotifications** - Notification system data
- **MethodOAuthSecurity** - OAuth authentication data

**Next:** [Microsoft Tools](../microsoft-tools/README.md)
