# Developer Machine Setup 3.0

| [Richard Pangborn](mailto:r.pangborn@method.me) | Owner | Status | Draft |
| :---- | :---- | :---- | :---- |
| 2024-06-25 | **Last Modified** |  |  |

# Previous versions: [2.0](https://docs.google.com/document/d/1XwK8jg0V9jWYidg6j_xOb-RZj1h-feAe1a1riAL5VPQ), [1.0](https://docs.google.com/document/d/1Sr8Cf5USohYXMf6PJM2SSsNQkVOm0BsoYnn3Wrclyg8)

# ---

 

# 1\. Prereqs

## 1.1. Update Windows

1) Press *WindowsKey,* and search for “**Check for Updates**” settings

2) In the settings window, click on the **Update** button.R

* Wait for updates to be downloaded and installed

![][image1]

### **Troubleshoot: Awaiting Install… / Red Alert**

If you see [“Awaiting installation”](https://answers.microsoft.com/en-us/windows/forum/windows_10-windows_install/windows-update-stuck-on-awaiting-install/6ae81506-aba8-4242-8a5b-010bf21e47f4), or if you see a red badge:

![][image2]

1. Select “**Troubleshoot**” from the left hand sidebar of the same window.

2. Click on “**Windows Update**” and press “**Run the troubleshooter**”

![][image3]

3. After troubleshooting is complete, **restart** the computer. 

4. **Repeat** steps above (1-3) again and again until you see the following check mark under “**Check for Updates**”  
   ![][image4]

---

## 1.2. VPN

If you’re working remotely, you need to use a VPN to access certain certain files and services

Follow the instructions in **[VPN to MethodCRM](https://docs.google.com/document/d/1ij-WwEIkj1Z8KAPkKYOIwZa1WTUCezSaPmE--XdjFKU/edit)** doc if you have not set-up your VPN yet.

Your credentials for the VPN should be in Passbolt under the Individual folder.

Note: Turn off the VPN if you need to download heavy files that can be accessed directly without a vpn.

---

## 1.3. Create D: drive for logging

The local logging expects an available D: drive. The simplest way to do this is:

1. Windows Key+R \=\> diskmgmt.msc \=\> Select C: \=\> Shrink Volume  
   ![][image5]

2. Set the new unused space to D:

# 2\. Credentials: 

## 2.1. Active Directory Credentials {#2.1.-active-directory-credentials}

You will need your AD credentials for a number of cases such as **Remote Desktop Connection**, access to **Dev-DB Backups**, access to **TFS**, and **Nuget installation**.

Your credentials for the VPN should be in Passbolt under the Individual folder.

You receive it as part of the onboarding process, and it should look like this:

* **Username:** method\\first-initial.last-name   e.g. method\\j.doe

* **Password:** **Confidential.** contact Hossine/Isamel if reset is needed.

**Note \-** You always need the **method\\** prefix in the username

## ---

## 2.2. MSDN Credentials {#2.2.-msdn-credentials}

MSDN credentials are used for downloading MS Tools and Log-in to Visual Studio:

* **EMAIL/ PW:**    \*see passbolt  
* **Type:** Team/Business

# 3\. Github

## 3.1. Account Setup

1) Establish your **Github username**.

* Best practice is to set-up using your *method email* 

* You don’t have an account yet?  Use this link ([Join GitHub · GitHub](https://github.com/join)) to create your GitHub account.

2) Login to your GitHub account.

3) Go to ([Settings \-\> Emails](https://github.com/settings/emails)) 

4) Add your Method Email as a *secondary* email.    
   ![][image6]

5) Email/Slack your Git Administrators (Paul/Hossein/Rich) with information about your **Github username** so that they can add you to the methodcrm organization.

6) Once you get added to the Method repo, go to ([Notification Settings](https://github.com/settings/notifications)) 

7) Under “**Custom Routing**” click the “**Edit**” button and set the notification destination to your *method.me email address*, so all Method repo messages go to your work inbox.   
   ![][image7]

### ---

## 3.2. Git Install

*(Disconnect from the VPN while downloading)*

1) Download the latest version of **Git** from 

* [https://git-scm.com/download/win](https://git-scm.com/download/win)  
2) Run the downloaded Setup file

3) Ensure the following components are selected:

![][image8]  
     “Next”

4) Leave “*Start Menu Folder*” as Git. Click “Next”.

5) Set your editor to *Vim or NotePad++*. Click “Next”.

6) “*Let Git Decide*” what Branch new repos should be on. Click “Next”

7) Go with the *Recommended settings for Adjusting PATH* environment. Click “Next”.

![][image9]

8) If option for “Use bundled openSSH” set as SSH executable  
9) “*Use the OpenSSL Library*” as your SSL/TSL library. Click “Next”

10) Keep Line endings *Windows-style*. Click “Next” 	

 

11)  “*Use MinTTY*” as a terminal emulator.

![][image10]

12)  Git Pull should be set to *Default behaviour*. Click “Next”

![][image11]

13)  Use “*Git Credential Manager Core*” . Click “Next”

14)  Make sure “*Enable File System Caching*” is checked. Click “Next”

![][image12]

15)  Click “Install”

\*\*\* You should now have access to Git Bash \*\*\*

## 3.3. Github SSH

Setup your Github SSH by following the steps below:

1) [Check for existing SSH keys](https://docs.github.com/en/free-pro-team@latest/github/authenticating-to-github/checking-for-existing-ssh-keys)

2) [Generate a new SSH key and add it to the ssh-agent](https://docs.github.com/en/free-pro-team@latest/github/authenticating-to-github/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)

* Note \- Use **empty passphrase** when setting up git ssh  
  ![][image13]

3) [Add the new SSH key to your GitHub account](https://docs.github.com/en/free-pro-team@latest/github/authenticating-to-github/adding-a-new-ssh-key-to-your-github-account)

4) [Test your SSH connection](https://docs.github.com/en/free-pro-team@latest/github/authenticating-to-github/testing-your-ssh-connection)

---

## 3.4. Git Personal Access Token

You need to create a *Personal Access Token* for your Github account in order to setup your projects automatically using Github API

1) Sign in to your Github account and go to Developer Settings ([https://github.com/settings/tokens](https://github.com/settings/tokens))

2) Click on “**Generate New Token**”   
   ![][image14]  
3) It is recommended to select a longer expiration than the default 30 days (e.g. 1 year)  
4) Add the following token configuration:

* For **Note**, enter “MethodDevSetup”    
* Under **Scopes:**  
  * Select  **“repo”**  
  * Select **“admin:org”** \> **“read:org” \*only\***  
  * Leave the rest of options **off**. 

5) Click “**Generate Token**” at the bottom of the page

---

## 3.5. Git Clients

1) Download the latest version **Tortoisegit** from:

    [https://tortoisegit.org/download/](https://tortoisegit.org/download/)

2) Run the downloaded setup file

3) Click “Next” and accept the agreement

4) Use *TortoiseGitPlink  as the SSH client*. Click “Next”![][image15]

5) Leave “*Custom Setup*” as is, and click “Next”

6) Click “Install”,  then “Finish”

\*\*\*\*\*  
You should now be able to right-click inside File Explorer and see TortoiseGit in the menu  
\*\*\*\*\*

**\[ OPTIONAL \]** You can install **Architecture Decision Record(ADR)** cli tool, this can be used to generate ADR templates to fill in for any architectural changes.

* [https://github.com/npryce/adr-tools/blob/master/INSTALL.md](https://github.com/npryce/adr-tools/blob/master/INSTALL.md) 

**\[ OPTIONAL \]** Download & install **Github Desktop** to have a better visual experience while performing simple Git operations

* [https://desktop.github.com/](https://desktop.github.com/) 

---

⚠️  **NOTE** \- When installing a new software, check if the version is still compatible or a newer one can be installed.

---

# 4\. Install required software

## 4.1 Download developer tools

1. Create the C:\\MethodDev folder if you don’t have it already.

2. Pull the the repo [https://github.com/methodcrm/DeveloperTools](https://github.com/methodcrm/DeveloperTools) to get access to all the scripts you need to help setup and maintain your local development environment.

3. Open powershell as administrator and navigate to folder C:\\MethodDev\\DeveloperTools\\DevSetup\\build\_local

4. Run the command initialize.ps1 to install most of the software you need and setup needed features.

* Troubleshooting/Notes:  
  * Might have to redo last 2 commands:  
    * gem install sass  
    * npm install \-g grunt-cli  
    * This is because you need to Close and Reopen powershell in Admin mode for Gem and Npm to load  
  * If the “refreshenv” command doesn’t work:  
    * Run *Import-Module $env:ChocolateyInstall\\helpers\\chocolateyProfile.psm1*  
    * Then rerun refreshenv  
  * MSBuild is now installed in a folder under each version of Visual Studio. For example, \_C:\\Program Files (x86)\\Microsoft Visual Studio\\2017\\Enterprise\\MSBuild\_. You will likely need to update your script to reference the correct MSBuild path  
  * If you get an error saying that “Containers” does not exist:  
    Enable-WindowsOptionalFeature : Feature name Containers is unknown.  
    Try enabling Windows Hypervisor Platform in Windows Features (not sure if it works or not):  
    ![][image16]  
  * When you try to run: .\\initialize.ps1 you might get an error message: “..\\initialize.ps1 cannot be loaded because running scripts is disabled on this system.   
    Solution: run this command to enable execution of scripts:  
    Set-ExecutionPolicy \-ExecutionPolicy RemoteSigned  
  * If you encounter an error stating “WMC is not a recognized command”, open Windows ‘Optional Features’ and install it there. WMC has apparently been defaulted to not installed in versions of Windows 11 since 24H2  
      
      
      
      
    

---

## 4.2 Download docker packages

1\. Run the command pull\_docker\_packages.ps1 to pull down and configure the needed docker packages.  This will setup the following pieces of software

* MongoDB  
  * Redis  
  * Elasticsearch  
  * rabbitmq

Note: Make sure Docker is running before executing pull\_docker\_packages.ps1 

---

## sql4.3. SQL Server Installation

1) Download **SQL Server 2019 (or latest) Developer Edition** from here and run it: ([https://go.microsoft.com/fwlink/?linkid=866662](https://go.microsoft.com/fwlink/?linkid=866662))

2) Choose “**Basic**” under *Installation Type*, and continue.

![][image17]

3) Once Finished click on “**Customize**” button

![][image18]

4) Click **Next** until you reach the “*Installation Type*” step. 

   ⇒ Select “**Add features to an existing instance of SQL Server**” from radio options  
   ⇒ click **Next**.

   Note: Keep the instance as “**MSSQLSERVER**”

   ![][image19]

5) Under the “*Feature Selection”* step, ensure “**Database Engine Services**” & “**Full-Text and Semantic Extractions for Search**” are selected ⇒ click “Next”  
   ![][image20]

6) Continue pressing *Next* & *Install* in the following steps until you reach “**Complete**”

You should see a summary of installed items with a status of *‘**Succeeded**’*

*\*\*\** You can now close the Installation Wizard. \*\*\*

## ---

4.4. Setup SQL Aliases

1) Press the *Windows Key*  and search for “**SQL Server {2019} Configuration Manager**” via the *Start Menu*.

![][image21]

2) If you cannot find the app try:  
   1)  Navigating to the file location: C:\\Windows\\SysWOW64\\SQLServerManager16.msc (for your version).   
   2) Click SQLServerManager16.msc to open the Configuration Manager. You can also right-click SQLServerManager16.msc to pin the Configuration Manager to the Start Page or Task Bar.  
3) Expand **“SQL Server Network Configuration**” ⇒ select “**Protocols for MSSQLSERVER**”

⇒ Double-click on “**TCP/IP**” and make sure:

* “**Enabled**” is set to “**Yes**”   
* “**Listen All**” is set to “**Yes**”.

⇒ Click “OK*”*

# ![][image22]

4) Expand “**SQL Native Client 11.0 Configuration**” ⇒  Right-Click on “**Aliases**” ⇒ Select  “**New Alias**”

![][image23]

5) In the pop-up window, enter the following:

* **Alias Name**: `methodlocaldb`  
* **Port No**: `1433`  
* **Protocol**: `TCP/IP`  
* **Server:**    `localhost`

![][image24]

Click “OK”.

6) Repeat **Step 3** with “**SQL Server Network Configuration (32bit)**”  

And, under “*client protocols*” check “*TCP/IP*” to see if it’s Enabled.

**Note \-** on some machines with 64-bit SQL Server, “*SQL Server Network Configuration (32bit)*” is empty \-  just skip it.

If new alias is greyed out use the cliconfg.exe in both the 32-bit (folder SysWOW64) and 64-bit (folder System32

7) Repeat **Step 4/5** with “**SQL Native Client 11.0 Configuration (32bit)**”.  

And, under “*client protocols*” check “*TCP/IP*” to see if it’s enabled.

8) Click “**SQL Server Services**” …

   1) Right click on  “**SQL Server (MSSQLSERVER)**”

   2) Select “**Restart**” from the menu

![][image25]

9) Add methodlocaldb to host files by:

1. Open Windows PowerShell in ‘Admin mode’ by **Right-clicking** on the *Start Menu Button* ⇒ select “**Windows PowerShell (Admin)**”  
   ![][image26]

2. Execute this command to go host files:  notepad drivers\\etc\\hosts 

3. **Add a line at the bottom** of the *hosts file* that reads:  127.0.0.1 methodlocaldb

4. **Add the rest of the hosts from step 18.1 (this will make things easier in later steps)**

5. **Save the file**

## ---

## 4.5. SSMS Installation

To install **SQL Server Management Studio**, open the link below and download the latest version available:

[Download SQL Server Management Studio (SSMS) \- SQL Server Management Studio (SSMS)](https://docs.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms)

Run the setup file and click “**Install**”

![][image27]

---

## 4.6. Set Permissions

1. Press the *Windows Key*  and search for “**SQL Server Management Studio**” via the Start Menu.

![][image28]

2. On the “*Connect to Server*” dialog, click “**Connect**”

### **4.6.1. Update SQL Authentication Mode**

In the “**Object Explorer**” pane on the left hand side, RightClick on your SQL Server instance ⇒  “**properties**” ⇒  “**Security**” ⇒ **Select** “**SQL Server and Windows Authentication Mode**”  
![][image29]  
![][image30]

### **4.6.2. Add  “NT AUTHORITY\\NETWORK SYSTEM” to “Logins”**

1. In the “**Object Explorer**” pane on, Expand the tree ⇒  “**Security**” ⇒  “**Logins**”  ⇒ *Right-click* on “**Logins**” ⇒  select “**New Login…**”  
   ![][image31]

2. In the “**Login \- New**” popup window, Click “**Search**”  
   ![][image32]

3. Enter Network Service in the textarea ⇒ click “OK”  
   ![][image33]

4. You should now be seeing “NT AUTHORITY\\NETWORK SERVICE” as the “**Login name**” 

5. Select “**Server Roles**” from the menu on the left ⇒  ***check all the roles***  
   ![][image34]

6. Click “**OK**”   ⇒   you should now see “NT AUTHORITY\\NETWORK SERVICE”  under Logins

### **2d  “runtime-core-engine” to “Logins”**

1. *Right-click* on “**Logins**” ⇒  select “**New Login…**”  
   ![][image31]

2. In the “**Login \- New**” popup window

* **Login name:** **runtime-core-engine**

* **Select** “**SQL Server authentication**”  

* **Password:** **local**

* **Uncheck**  “**Enforce password policy**”![][image35]

7. Select “**Server Roles**” from the menu on the left ⇒ Check “**public**” and “**sysadmin**”  
   ![][image36]

8. Click “**OK**”   ⇒  you should now see runtime-core-engine under Logins

**NOTE\!**  

After adding new users above, you must **restart your SQL Server** by:

1) Open “**SQL Server {2019} Configuration Manager**” from the Start Menu

2) Click “**SQL Server Services**” ⇒ Right Click “**SQL Server**” ⇒ **Restart**  
   ![][image37]

3) Also make sure “**SQL Server Agent**” is running

### **4.6.4. Fix bcpreader issue**

There is a [known issue](https://methodme.slack.com/archives/CDJNL2ZBJ/p1680726161304529) that is fixed by running this sql within your local server

*If not Exists (select loginname from master.dbo.syslogins where name \= 'bcpreader')*

 *Begin*

*CREATE LOGIN bcpreader WITH PASSWORD \= 'Gravity4fRee'*

  *END*

---

## 4.7. SQL Backup: QBO & QBDT

1) Open File Explorer and **create** these folders in C:\\ drive:

“C:\\Methoddata\\MethodIntegrationSQL”

2) Give Network Service user full access to the C:\\Methoddata\\MethodIntegrationSQL folder (will be needed for some App Pools in IIS):  
   1) right click in the MethodData folder  
   2) go to Properties  
   3) go to Security tab  
   4) add NETWORK SERVICE user to Group and User names, and then select it if it is not selected.  
   5) give “Full Control” to NETWORK SERVICE in the “Permissions for NETWORK SERVICE” pane  
3) Go to:  [**`https://dev-setup.methodintegration.com/dbs/`**](https://dev-setup.methodintegration.com/dbs/)** 

* If prompted to sign in, Enter your [Method Active Directory Credentials](#2.1.-active-directory-credentials)

4) Download the latest version of the following files and move them into C:\\Methoddata\\ MethodIntegrationSQL:

* [TemplateMethodNewQBO.bak](https://dev-setup.methodintegration.com/dbs/templatemethodnewqbo/sql/FULL/)  (this db is used to generate the offline in postman accountmanagement, in order to signup)

* [TemplateDevV2.bak](https://dev-setup.methodintegration.com/dbs/templatedevv2/sql/FULL/)  (this db is used to develop apps and push to appstore)

---

## 4.8. SQL Backups

1. Open powershell as administrator and navigate to folder C:\\MethodDev\\DeveloperTools\\DevSetup\\build\_local 

   #### Run the command restore\_dbs.ps1 to download and restore the needed system sql and mongodb databases.

   1. When the console asks if there are any user dbs to be restored:

      1. If you don’t want any user dbs (from builder) to be copied, then type ‘N’ and proceed

      2. If there are any user dbs you want to copy, then create a file: user\_dbs.yaml, and include the user db names in the file. And type ‘Y’ when you run restore\_dbs.ps1 (Make sure the user’s .bak file is present in the builder env)

   2. If you create a user\_dbs.yaml file in that folder it will download any database you list in there as well. ([readme](https://github.com/methodcrm/DeveloperTools/tree/master/DevSetup/build_local#readme))

* When prompted for credentials, Enter your [Method Active Directory Credentials](#2.1.-active-directory-credentials)  
* If you see an error you may need to install sqlserver for powershell to run the commands  
  * Using "Install-Module \-Name SqlServer"  
  * If it doesnt work: might have to use \-AllowClobber parameter [https://dba.stackexchange.com/questions/174704/is-it-normal-for-install-module-sqlserver-to-clobber-existing-modules](https://dba.stackexchange.com/questions/174704/is-it-normal-for-install-module-sqlserver-to-clobber-existing-modules)

**Full List of Databases (make sure in SSMS these are present):**  
    **SQL Databases:**

* AlocetSystem  
* Method App Store  
* MethodNotifications  
* MethodOAuthSecurity  
* MethodShortURLRedirects	  
* TemplateDevV2  
* TemplateMethodNewQBOy

    **MongoDBs:**

* Accounts  
* AlocetSystem  
* Authentication  
* Methodappstore  
* methodmobilenotifications  
* oauth\_token  
* Preferences  
* TemplateDevV2


  

To view your MongoDBs you should have Robo 3T installed from a previous step.  
To view your SQL Databases open Microsoft SQL Server Management Studio.

**Troubleshooting restore\_dbs.ps1**

* If you’re running SQL server 2019, you will see the following error:  
  * Invoke-Sqlcmd : A parameter cannot be found that matches parameter name 'TrustServerCertificate'.  
    You can fix this by removing this parameter from the script  
  * You can manually restore .bak files in SSMS if any script fails:  
    Open SQL Management Studio and connect to the local server. Once connected, right-click on the "Databases" folder and select "Restore Database". Then, follow these steps and choose the path that contains the backup file for your database.

    

### ---

# 7\. Microsoft Tools

## 7.1. Visual Studio Installation

1)  Go to **MSDN Downloads** ([https://my.visualstudio.com/Downloads](https://my.visualstudio.com/Downloads))  

2) Enter the login credentials found under [**MSDN Credentials**](#2.2.-msdn-credentials)

3) Download the Latest Edition of **Visual Studio Enterprise 20XX ( currently 2022\)**

    *This will download the installer for you*	

4) Run the Visual Studio Installer that you just downloaded \- You should see a single-screen wizard with multiple tabs at the top.

5) From the “**Workloads**” Tab, **select** the following modules:

    

![][image38]

6) Make sure The **ASP.NET and development tools** in the screenshot below are **Selected**

Note: If "Development time IIS support" is not in the list, IIS may need to be enabled first in Windows. Search "Turn Windows features on or off". In the resulting window, select the item "Internet Information Service". Select OK, let the installer run, and restart if necessary. The Visual Studio installer may need to be restarted before the item is visible.  
![][image39]

* You might need to install dotnet framework 4.6.2 manually, double check this.

7) Select the .NetCore Runtime Bundles list in the screenshot below:  
   ![][image40]

Note: The .net runtime bundles are located under individual components in the Visual Studio Installer, 2.2 runtime is not available from here.

8) Leave all other options in the installer as is in their *default state*.

9) Select “**Download All, then Install**” , and Click “**Install**”.  
   ![][image41]

10)  **Restart** your machine once installation is *completed* 

11)  **Download and Install** this specific *.NETCore Version* found here:  
    [https://dotnet.microsoft.com/download/dotnet-core/thank-you/sdk-2.2.207-windows-x64-installer](https://dotnet.microsoft.com/download/dotnet-core/thank-you/sdk-2.2.207-windows-x64-installer) 

## 7.2. Setup Nuget Feeds

1) Press the *Windows Key* and search for “**Internet Options**” in the *Start Menu*.  
   ![][image42]

2) Go to Local Intranet’s “**Advanced Settings**” using the steps below:  
   ![][image43]![][image44]

* Go to “**Security**” Tab

* Click on “**Local Intranet**”

* Click on “**Sites**”

* Click on “**Advanced**” in the popup window

3) Add the 3 website urls below to your “**Local Intranet**” list 1-by-1.

* http://tcity

* http://teamcity.method.me

*  http://tfs.method.me

![][image45]

4) Once they’re all *added*, Click on “**Close**” ⇒  “**OK**” ⇒  “**OK**”

5) Press the Windows Key and search for “**Visual Studio 20XX” (currently 2022\)**  
   ![][image46]

6) Click “**Sign in**” on the ‘*Welcome\!*’ prompt and enter the login information found under [MSDN Credentials](#2.2.-msdn-credentials)![][image47]

7) *Once signed in*, click on “**Continue without code**” found at the bottom of the starting screen  
   ![][image48]

8) Go to Tools \> NuGet Package Manager \> Package Manager Settings  
   ![][image49]

9) Select “**Package Sources**” and make sure the *Nuget package* has the following configuration:

* **Name:** nuget.org

* **Source:** https://api.nuget.org/v3/index.json 	

![][image50]

10)  In “**Package Sources**” pane, click on the “**\+**” icon, and *add a new source* with the following configuration:

* **Name:** method-ms  
* **Source:**  
  https://tfs.method.me/tfs/MethodCollection/\_packaging/method-ms/nuget/v3/index.json 

![][image51]

**Troubleshooting adding method Nuget**

If you encounter the error below:

| Response status code does not indicate success: 402 (Payment Required). |
| :---- |

Try to remove method-ms package source from VS completely then go to console and run the following command:

| nuget sources add \-Name "method-ms" \-Source "https://tfs.method.me/tfs/MethodCollection/\_packaging/method-ms/nuget/v3/index.json" \-username "method\\{{ur-AD-account}}" \-password "{{your-password}}" |
| :---- |

---

## 7.3. Save ‘Active Directory’ Credentials

To access method-ms nuget feed, you will require your Active Directory Credentials. 

***Save your AD credentials*** so that nuget package manager can pull down the nuget packages without prompting you to enter a username and password. 

Note \- If you skip this step, your dev setup scripts will fail.

1) Press the Windows Key and search for “**Manage Windows Credentials**” in Start Menu: 

![][image52]

2) Select “**Windows Credentials**”  ⇒  click on “**Add a Windows credential**”

![][image53]

3) Enter the following credentials in the Add window:

* **Internet/network address:**  tfs.method.me

* **Username:**  see [Method Active Directory Credentials Section](#2.1.-active-directory-credentials)

* **Password:**  see [Method Active Directory Credentials Section](#2.1.-active-directory-credentials)


![][image54]

---

## 7.4. Install Nunit Test Adapter

Note: The NUnit extension no longer seems to be an option, but can still be added at a project level.

1) Open Visual Studio and go to “**Manage Extensions**”  
   ![][image55]

2) Search for “**Nunit 3**”  ⇒  click “**Download**” on “**Nunit 3 Test Adapter**”

![][image56]

3) Close “*Manage Extensions*” ⇒  Close “Visual Studio” to *start installation*  
4) Click “**Modify**” on the installation prompt  
   ![][image57]

5) Once installation is finished you can click “Close”

Note: NUnit 3 Test Adapter is deprecated and not available from VS 2019\. Install NUnit Adapter package as mentioned in the link: 

[NUnit 3 Test Adapter \- Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=NUnitDevelopers.NUnit3TestAdapter)

nuget : [NuGet Gallery | NU	nit3TestAdapter 4.5.0](https://www.nuget.org/packages/NUnit3TestAdapter/)

---

## 7.5. Install URL Rewrite

1) Download the extension ( [https://www.iis.net/downloads/microsoft/url-rewrite](https://www.iis.net/downloads/microsoft/url-rewrite) )

2) Make sure IIS is enabled in (Turn Windows features on or off)- follow Appendix A to enable IIS features

3) Run the installer ⇒  click “Install” (might take some time)

4) If you’re prompted with “**Windows Platform Installer**” after the setup, click “**Exit**” 

---

# 8\. Add NPM to PATH

1) Press Windows Key, type “**env**”, Windows will suggest "**Edit the System Environment Variables**", click that.

![][image58]

2) Highlight the "**Path**" variable, click "**Edit**"

   ![][image59]

   

3) This will bring up the "Edit environment variable" window, click "**New**", and Paste the path below in the new line:

* C:\\Program Files\\nodejs\\node\_modules\\npm\\bin

![][image60]

4) Press "OK", "OK"

---

# 9\. Build local environment

1. Open powershell as administrator and navigate to folder C:\\MethodDev\\DeveloperTools\\DevSetup\\build\_local

2. Run the command build\_local\_environment.ps1 to change to and pull down the master branch of all projects and rebuild them

NOTE: Before running this script, follow these steps:

- Go to the projects.yaml file in this folder: C:\\MethodDev\\DeveloperTools\\DevSetup\\build\_local  
- Search for ‘internal-attribution-api’ in that file and navigate to that section	  
- For the ‘restore’ and ‘build’ command paths, replace the word ‘Professional’ to ‘Enterprise', since we have installed Visual Studio Enterprise. The restore command path should look similar to this: C:\\'Program Files'\\'Microsoft Visual Studio'\\2022\\Enterprise\\MSBuild\\Current\\Bin\\MSBuild.exe AttributionSrv.sln \-t:restore \-p:RestorePackagesConfig=true and build command path should look like this: C:\\'Program Files'\\'Microsoft Visual Studio'\\2022\\Enterprise\\MSBuild\\Current\\Bin\\MSBuild.exe AttributionSrv.sln

* You can run this script many times to rebuild/pull projects from git

**Important Notes:**

* Make sure you’re **connected to the VPN** for builds/nuget restoration to work successfully.

---

# 10\. Certificates {#10.-certificates}

## 10.1. Download Store Bought Certificate

One is available here:

[https://drive.google.com/file/d/1GO7lXtSf9zMKBejZ-ZSd9bsSKOXemyoQ/view?usp=sharing](https://drive.google.com/file/d/1GO7lXtSf9zMKBejZ-ZSd9bsSKOXemyoQ/view?usp=sharing) 

Expires Dec 4, 2025

**Password:** X7PSmdQEMc4DuMi1rYbl

---

## 10.2. Import Certificates

1) Press the Windows Key and search for “**mmc**”  
   ![][image61]

2) If your **mmc** console is empty and you don’t see a list of certificates, follow the steps below to **add a “Snap In”** for certificates first. **If not, skip to step 3**.

1. From “**File**” menu, select “**Add/Remove Snap In**”  
   ![][image62]

2. Select “**Certificates**” from the “**Snap-ins**” prompt and click “**Add**”  
   ![][image63]

3. Select “**Computer account**” in the next prompt. Click “**Finish” \> “Ok**”

   ![][image64]

4. You should now see a list of your certificates in mmc console

   

3) Next, go to **Certificates \> Personal \> Certificates**

   

   After selecting “**Import**”, pick the “.pfx” file that you just downloaded earlier. 

   Keep all subsequent options as *Default*, then click Finish. If you’re given a security prompt, select “Yes”.

4) Next, go to **Certificates \> Trusted Root Certification Authorities \> Certificates**  
   ![][image65]

5) 

6) **Right-click on “Certificates”** \> Click “**Import**”  
   ![][image66]  
   Click “**Next**”. Then, “**Browse**” to C:\\ drive where the  .pfx  file is located  
   ![][image67]  
   Keep the default selection for “Certification Store” and click “**Finish**”.  
   If prompted with a security prompt, select “Yes”. 

**Note** \- This certificate should be good for one year. After one year you will need to repeat these steps.

---

## 10.3. Add Certificates to IIS

**NOTE** \- If you have not set up any sites in IIS, complete the other sections below first.

Next, in **IIS**, go to the **https bindings** for one of your sites and add this certificate. This should update all bindings. If prompted to update all certificates, click Yes.

This is also handled at the end of [IIS section](#11.-iis-configuration).

---

## Aside: BrowserStack

Browser Stack is the tool we use for cross-platform testing. 

**Note \-** This is a shared account, so please be mindful of your usage, and ensure you’re not kicking anyone out of their session

To get BrowserStack working with your local, you need two things:

1. Setup SSL Certificates as instructed above.  
2. Enable local testing by following the steps outlined here: [https://www.browserstack.com/docs/live/local-testing](https://www.browserstack.com/docs/live/local-testing)

Once you have everything setup, you should be able to login to BrowserStack and get started.

You can sign-in here: [https://www.browserstack.com/users/sign\_in](https://www.browserstack.com/users/sign_in?source=bstacklocal-3.2.1)

* **Email:** qa@method.me  
* **PW:**   Admin00+

---

## Aside: How to Generate Custom Certificates

In order to utilize https, you will need to **create a trusted certificate for methodlocal** and **add it to your https bindings in IIS**. 

A new certificate can be generated by executing the following command in Powershell:

1) Open **PowerShell in Admin Mode**

2) Enter the command below to put the cert in *certlm.msc \> Certificates \> Personal \> Certificates* (Should be \*.methodlocal.com), though it won't be trusted yet:

| New-SelfSignedCertificate \-DnsName \*.methodlocal.com \-CertStoreLocation cert:\\LocalMachine\\My |
| :---- |

3) **Copy the Thumbprint** shown in the output.  
   ![][image68]

4) **Execute** the Powershell commands below to add the personal cert above to your list of trusted certs ( **Note** \- You should **replace** *\<ThumbprintGoesHere\>* with the thumbprint retrieved from the **previous step** ) 

#  11\. IIS Configuration {#11.-iis-configuration}

## 11.1. Enable IIS Features

1) Press *WindowsKey* ⇒ search for “**Turn Windows features on or off**”  
   ![][image69]

2) **Turn ON** the following features if they’re unchecked:

* **ASP.NET 4.7**  
  ![][image70]

* **IIS Management Console**  
  ![][image71]

* The following **4 features** under *“Common HTTP Features*”  
  ![][image72]

* Most of the “*Application Development Features*” **excluding CGI & .NET3.5** features  
  ![][image73]

3) Click “**OK**” to install 

4) **Restart your PC**

---

## 11.2. Export the Registry Key \- Salt {#11.2.-export-the-registry-key---salt}

1) Open *File Explorer* ⇒ go to “**C:\\MethodDev\\method-platform-ui**”

2) *DoubleClick* to run the  “**EncSalt.reg”**  file ⇒ say “yes” & “ok” to next prompts

---

## 11.3. Import IIS Sites & App Pools

1) Open powershell as administrator and navigate to folder C:\\MethodDev\\DeveloperTools\\DevSetup\\build\_local

2) Run the command import\_sites\_pools.ps1 to replace your current IIS setup with the standard sites and app pools

3) When prompted for credentials use:

   **User Name:** Your **Local Windows Account Username** (must have Admin access)  
   **Password:** Your **Local Windows Account Password**

Troubleshooting Custom Account App Pool  **git** 

* **Don’t know your account username?**   
  * **Win 10+:**  
* **Press windows key \+ r**

* **Enter: netplwiz**

* **Username should be displayed**

  * **Older:**  
* Press WindowsKey  ⇒  Search for “**Computer Management**” from the Start Menu

* Expand “**Local Users and Groups**”  ⇒  Select “**Users**”  ⇒ Identify the account with your name

* RightClick on the **account name**  ⇒ select **Properties**  ⇒ select **Member of** **Tab**  \---  Make sure your Account is a ***member of Administrators***. 

  If you don’t have Admin permissions, contact your Team Lead

* **Don’t remember your password?**  
  Your password is usually the one used for ***unlocking your Windows*** when the machine boots, or if you use the PIN to unlock, the password is the one you use to *login to your Windows Account* ( Open “**Settings**” from the Start Menu ⇒  **Accounts** ⇒  Find the Email Address used for *logging into Windows* ⇒  **Use its password**)

  If you still can’t remember your password, *you might have to reset it*. Please Contact Your Team Lead before doing so. See this article on how to [create/reset passwords](https://www.isunshare.com/windows-10/3-ways-to-create-password-for-user-account-in-windows-10.html).

---

## 11.4 Legacy-syncservice-api project configuration

1. Check C:\\MethodDev\\legacy-syncservice-api\\SyncServices Service\\SiteConfig  
   NOTE: you can copy-paste the one from MethodUI: has a Method.config in this folder: C:\\MethodDev\\method-platform-ui\\SiteConfig\\Debug\\Method.config

---

## 11.5. HTTPS Binding & SSL Certificates

Almost every site in Method needs to have an https binding for SSL

1) *RightClick* on every Site in *Connections* tab ⇒ **Edit Bindings**  
   ![][image74]

2) Make sure there’s a **HTTPS binding** with **port 443**

* No HTTPS Binding?  ⇒ Click on “Add” in the same window ⇒  Add the parameters outlined in the screenshot below:  
  ![][image75]

3) If there’s HTTPS Binding, click on “**EDIT**” ⇒ Select  **\*.methodlocal.com**  from the SSL certification dropdown ⇒ Save by clicking “**OK**”

* Note 1 \- If you don’t have   **\*.methodlocal.com**,  you should follow steps in [**Certificates Section**](#10.-certificates) first

* Note 2 \- Once you setup your SSL correctly for one of the Site services, it most likely gets reflected in every other site with HTTPS binding

4) **Repeat Steps 1 \- 3** for the remainder of Sites in the Connections panel, but keep in mind the “Note 2” under Step-3 above. 

---

# 12\. NetCore Requirements

## 12.1. Add ASPNETCore Env Variables

1) Go to **Control Panel\\System and Security\\System** ⇒ Click “**Advanced System Settings**” ⇒ Click “**Environment Variables**”

2) Click on “**New**” under SYSTEM VARIABLES

3) Add the following parameters:

* **Variable Name: ASPNETCORE\_ENVIRONMENT**

* **Variable Value:**  **Development**

* **Variable Name: DOTNET\_ENVIRONMENT**

* **Variable Value:**  **Development**

4) Click “OK”

5) Click on “**New**”  under SYSTEM VARIABLES again

6) Add the following parameters:

* **Variable Name: DOTNET\_HOST\_PATH**

* **Variable Value:**  **%ProgramFiles%\\dotnet\\dotnet.exe**   (double check this path)  
7) Click “OK” ⇒ “Ok”

---

## 12.2. Add AWS Credentials

As we are using **AWS Secret Manager** to load all secrets (such as connection strings and encryption keys), you have to add the *AWS Key and Secret* to your local userprofile and home environment folder.

Windows

1) **Download** the file listed below:

   New (2025-06-09)

   Rename the file to credentials once downloaded.

   [https://drive.google.com/file/d/112E060eN3cG7oZRQPu93k-APwl32VGIA/view?usp=drive\_link](https://drive.google.com/file/d/112E060eN3cG7oZRQPu93k-APwl32VGIA/view?usp=drive_link) 

2) Go to  **C:\\Windows\\System32\\config\\systemprofile** in File Explorer**.** If .aws folder doesn’t exist ⇒ **RightClick** on File Explorer ⇒  select “**Git Bash Here**” ⇒ Run:	  
    mkdir .aws  

3) **Copy & Paste** the downloaded file *inside the **.aws** folder.*  
   *![][image76]*

4) Go to **%SystemDrive%\\Users\\\<username\>\\.aws** (*replace “username” with yours*), If .aws folder doesn’t exist ⇒ **RightClick** on File Explorer ⇒  select “**Git Bash Here**” ⇒ Run:  mkdir .aws

5) **Copy & Paste** the downloaded file *inside **.aws** folder.*  
   *![][image77]*

6) Create another file in the same folder and name it \`config\` and paste the following content in it

| \[default\] region \= us-west-1 |
| :---- |

   ![][image78]

AWSLinux  
On any linux box just put the credential file in **\~/user** folder.

| sudo \-i sudo mkdir .aws cd .aws  // paste the file in this folder |
| :---- |

---

# 13\. Running Projects on Local Account (WIP)

## 13.1. Important Prerequisites {#13.1.-important-prerequisites}

1) Check to see you have everything outlined in the sections prior to this correctly setup

2) Open *IIS Manager* ⇒ Restart ⇒ Ensure all *Application Pools* and *Sites/Apps* are **running**

3) Open “*Task Manager*” ⇒ “*Services*” tab ⇒ Check to see if the following services are running:  
* RabbitMQ  
* Redis  
* MongoDB  
* MySQL

4) Open “*SQL Server {2019} Configuration Manager*” ⇒ click on “*SQL Services”* ⇒ check to see if the following are running:  
* SQL Server   
* SQL Server Agent  
* SQL Full-text Filter

5) Connect to the **VPN**

6) Add local [URLs](#18.-host-file-list) to your hosts file in: "C:\\Windows\\System32\\drivers\\etc\\hosts"

7) Create a folder called “blank” in “C:\\MethodDev”. Full path will look like this “C:\\MethodDev\\Blank”.

---

## 13.2. Critical Projects (Health Check & Rebuild)

**Important Notes:**

* Make sure you’re **connected to the VPN** for builds/nuget restoration

* Health Checks usually should have **No “Fail” keyword** in them. If you see Errors during build or “Fails” in health check, contact your team (one exception: if “File Storage” dependency failing, you can ignore that)

* If you get a Network Request Error such as  400, 500, 503, etc \- see **Troubleshooting section** right after

1.  Rebuild **ms-gateway-api**  
   Health Check: Press the “play” button with “IIS” selected  
   Health Check (alternative): [https://api.methodlocal.com/v2/health/check](https://api.methodlocal.com/v2/health/check) 

2.  Configure and Rebuild **Runtime Project**  
1) In *Visual Studio* ⇒ Open “**C:\\MethodDev\\runtime-core\\Runtime.stack.sln”**  
2) Make sure “*Startup Project*” is set to **Runtime.Core.Api** and “*Play*” profile is “**IIS**“  
   ![][image79]

3) Rebuild **Runtime.Core.Engine**

    

3. Rebuild **ms-authentication-api**   
   Health Check: Press the “play” button with “IIS” selected   
   Health Check (alternative): [https://api.methodlocal.com/v2/authentication/health/check](https://api.methodlocal.com/v2/authentication/health/check)  
4. Rebuild **ms-authentication-api-oauth2**   
   Health Check: Press the “play” button with “IIS” selected   
   Health Check (alternative): [https://auth.methodlocal.com/health/check](https://auth.methodlocal.com/health/check)    
* **Reference**: [auth.methodlocal.com (identityserver)](https://docs.google.com/document/d/1XwK8jg0V9jWYidg6j_xOb-RZj1h-feAe1a1riAL5VPQ/edit#heading=h.bf98lfn328fq)  
* Troubleshoot: If you are not able to access the service, then probably the IIS is pointing to incorrect path. 

  The old path/project is : C:/MethodDev/ms-authentication-oauth2 

  The new project is: oauth2

  Download this from github to C:/MethodDev/ and rebuild it:

  [methodcrm/oauth2 (github.com)](https://github.com/methodcrm/oauth2)

  Go to IIS, go to advanced settings for auth.methodlocal.com, and change the path to C:\\MethodDev\\oauth2\\authentication\\API:


5. Rebuild **ms-identity-api**  
   Health Check: Press the “play” button with “IIS” selected  
   Health Check (alternative): [https://api.methodlocal.com/v2/identity/health/check](https://api.methodlocal.com/v2/identity/health/check)

6. Rebuild **ms-preferences-api**  
   Health Check: Press the “play” button with “IIS” selected  
   Health Check (alternative): [https://api.methodlocal.com/v2/preferences/health/check](https://api.methodlocal.com/v2/preferences/health/check) 

7. Rebuild **ms-email-api**  
   Health Check: Press the “play” button with “IIS” selected  
   Health Check (alternative): [https://api.methodlocal.com/v2/email/health/check](https://api.methodlocal.com/v2/email/health/check)

8. Rebuild **ms-account-api**  
   Health Check: [http://microservices.methodlocal.int/account/health/check](http://microservices.methodlocal.int/account/health/check)

     

### First-Run and Troubleshooting

An offline template database needs to be generated for new local test accounts to be based on. In order to ensure the template you’re creating new accounts from is up-to-date, be sure to go through the following steps. 

Additionally, should you find you’re missing important migrations on new test accounts in the future, or you encounter an error "*missing C:\\\\MethodData\\\\MethodIntegrationSQL\\\\TemplateMethodNewOffline.mdf*", you can perform these steps again.

1. Open Postman  
2. Use Account Management \-\> v3 \-\> Offlines \-\> internal endpoints  
3. Call below GET endpoints in an order;  
   1. Get Clean Generation State \- Internal; [http://microservices.methodlocal.int/account/v3/offline/internal/GetCleanGenerationState](http://microservices.methodlocal.int/account/v3/offline/internal/GetCleanGenerationState)  
   2. Get Unified Template Generated \- Internal; [http://microservices.methodlocal.int/account/v3/offline/internal/GenerateUnifiedTemplate](http://microservices.methodlocal.int/account/v3/offline/internal/GenerateUnifiedTemplate)  
   3. SetOfflineUnifiedTemplate \- Internal; [http://microservices.methodlocal.int/account/v3/offline/internal/SetOfflineUnifiedTemplate](http://microservices.methodlocal.int/account/v3/offline/internal/SetOfflineUnifiedTemplate)   
4. Verify no errors appear in D:\\logs\\microservices\\api\\account and [http://microservices.methodlocal.int/account/health/check](http://microservices.methodlocal.int/account/health/check) returns success  
5. TemplateMethodNewOffline\_Log.ldf file was missing: manually copied ldf provided by others into C:\\MethodData\\MethodIntegrationSQL

9. Rebuild **ms-apps-api**  
   Health Check: [http://microservices.methodlocal.int/apps/health/check](http://microservices.methodlocal.int/apps/health/check)

10. Rebuild **ms-tables-fields-api**  
    Health Check: [http://microservices.methodlocal.int/tables-fields/health/check](http://microservices.methodlocal.int/tables-fields/health/check)

Troubleshoot:

- Make sure you have run registry: [Export the Registry Key \- Salt](#11.2.-export-the-registry-key---salt)

11. Rebuild **legacy-authentication-api**   
    Health Check: [http://services.methodlocal.com/authservice/health/check](http://services.methodlocal.com/authservice/health/check) 

12.  Rebuild **method-signin-ui**

Health Check: [https://signin.methodlocal.com/health/check](https://signin.methodlocal.com/health/check) 

13. Rebuild **method-signup-ui**

Health Check: [https://signup.methodlocal.com/health/check](https://signup.methodlocal.com/health/check)

14. Rebuild **internal-migration-api**

Health  Check: [https://migration.methodlocal.com/health/check](https://migration.methodlocal.com/health/check)

Endpoints: [https://migration.methodlocal.com/help](https://signup.methodlocal.com/health/check)

**Run migrations (migrateall):**

1. Open Postman  
2. Use Migrations/All Companies  
3. Url \=\> [https://migration.methodlocal.com/migration/MigrateAllAccounts?stopOnError=false](https://migration.methodlocal.com/migration/MigrateAllAccounts?stopOnError=false)  
4. Run  
5. Verify no errors appear in D:\\logs\\MigrationService

14\) Configure and Rebuild **MethodUI project**

1) Open *Command Prompt in Admin mode*

2)  Go to **“C:\\MethodDev\\method-platform-ui\\m-one”**

3)  Install our frontend npm dependencies by executing:  npm install

4)  Build the monorep executing:  npm run build \-ws

5) Go to **“C:\\MethodDev\\method-platform-ui\\MethodUI”**

6) Install our frontend npm dependencies by executing:  npm install

- if you face any errors, try this: npm i \--legacy-peer-deps

  Note: Some errors here were previously resolved by installing the Windows 10 SDK, however that should *not* be required, please ask for assistance if issues with this step.

7) You could face npm gyp errors with message: 

   You need to install the latest version of Visual Studio

   find VS including the "Desktop development with C++" workload.

   Missing any windows SDK

   Solution: install "Desktop development with C++" workload.

   Also go to individual components and install the latest version of Windows 10 SDK (Windows 11 SDK doesn’t seem to work)

8) Install grunt-cli for compiling sass files by executing:  
    npm install \-g grunt-cli

9) Build all legacy, react and sass-foundation files at once by executing:  
   npm run build:dev 

   1) If you run into out of memory errors when running the above command, you can allocate additional memory to Node by opening **C:\\MethodDev\\method-platform-ui\\methodui\\package.json** and updating the line titled "build:dev" under “scripts” to the text below and try again:  
        

"set NODE\_OPTIONS=--max-old-space-size=4096 npm run prettify && npm run lint && npm run build:sass && npx mix",

![][image80]

*If the action editor designer or dashboard does not load properly or the layout is out of whack run the above command (npm run build:dev) again and restart your IIS.* 

![][image81]  
*You should be seeing screenshots above*

10) In *Visual Studio* ⇒ Open “**C:\\MethodDev\\method-platform-ui\\MethodPhoenix.sln”**

11) Go to MethodUI properties \> Web 

![][image82]

1. Select “Local IIS”  
2. Project URL: [`http://methodlocal.com/apps`](http://methodlocal.com/apps)  
3. Save All files

12) Rebuild **MethodPhoenix**  
    *Health Check: You can run this once there’s an account to work with*

Troubleshooting (Work In Progress)

- missing ssl:   
- Not built service: HTTP Error 500.0 \- ANCM In-Process Handler Load Failure

- Incorrect netcore config  
- Stopped IIS app pool: HTTP Error 503\. The service is unavailable.  
- ![][image83]

- HTTP Error 502.5 (Below SS)  
  ![][image84]  
- If the health checks show that rabbitmq is unhealthy (They are not able to access rabbitmq running in docker), then try installing rabbitmq directly using this powershell command:  
  choco install rabbitmq

---

## 13.3. Create your Local Account

**Prerequisites**

Check areas you just checked at the beginning of the process in the [“Important Prerequisites” section](#13.1.-important-prerequisites) above. Some services stop after builds \- best to double-check everything is running. 

* **Run the AppUpdate Agent**  
  **NOTE \-** *This can take a while*. Make sure it reaches completion and leave it open for now.

1) In *Visual Studio* ⇒ Open & Rebuild “**C:\\MethodDev\\runtime-core\\Runtime.AppUpdate.Agent.sln”**

2) In *File Explorer*  ⇒ go to:  
   **“C:\\MethodDev\\runtime-core\\Runtime.AppUpdate.Agent\\bin\\Debug\\net7.0”**

   1. DoubleClick to run this file:  “**Runtime.AppUpdate.Agent.exe**”   
      ![][image85]

* Open hosts file (**C:\\Windows\\System32\\drivers\\etc**) in Admin mode ⇒ **Copy** the sites [**listed here**](https://docs.google.com/document/d/1XwK8jg0V9jWYidg6j_xOb-RZj1h-feAe1a1riAL5VPQ/edit#heading=h.15we4ktzp3ja) ⇒ **Paste into hosts file and save**. ⇒ Leave hosts file open

* Resolve any issues with backups being out of sync. The issue faced here was a record already existing in mongo and not in sql (sync times for local db backups were different). To check if this is the case:

  1. Find the last record in the SQL via SSMS like: 

     SELECT TOP 1 RecordId

     FROM \[AlocetSystem\].\[dbo\].\[CustomerMethodAccount\]

     ORDER BY RecordID DESC

  2. Connect to mongo via robomongo (Studio T)

  3. connect to localhost

  4. In **Accounts-\> Collections \-\> Accounts** delete the document with the next ID after the ID found in step 1 (e.g. if SQL last ID was 123, delete the 124 document)

  5. This should create space in the mongo db to allow for the creation of the new account

* Open Chrome **inCognito** mode

* Go to “[https://signup.methodlocal.com/methodcrm?domain=qbd](https://signup.methodlocal.com/methodcrm?domain=qbd)”

  ![][image86]

* Enter your regular email then “Continue”

  ![][image87]

* Enter the following information, but **DO NOT** press “Start Free Trial” yet\!\!

  1. Organization name: any unique name you want

     *Organization name may start with m18 or m11 to be a local dev account, but it does not necessarily need to start with those*.

  2. First and last name: any name you want

  3. Password:  anything you like or, Admin01\!

* Copy “**Organization Name**” you entered above

* Open  **hosts**  file in Admin mode from:  **“C:\\Windows\\System32\\drivers\\etc”**

* Add your *Organization Name* to the bottom like so:

  1. 127.0.0.1 \<Organization Name\>.methodlocal.com

* **Save** the hosts file ⇒ go back to Sign up page in Chrome

* Click “**Lets Go**”

    
  ![][image88]

*  In the next Sync screen, Press **`[CTRL]` \+ \[ SHIFT \] \+ `[,]`** button to skip the QB sync

  Note: Shortcut can take some time, however if shortcut keys do not work, please restart computer and try again  
  ![][image89]

**You should now be logged in 🎉**

---

## 

## 13.4. SMS Notifications (WIP)

You’ll need to set up the SMS Notifications project to create/edit invoices and estimates. This may become unnecessary in the future.

1\.	Add new application pool in IIS Manager called mobilenotifications  
![][image90]

2\. 	Once created go to advanced settings through the right panel. Then scroll down till you see **Identity** found under the **Process Model** section. You will want to edit this to be the current user you sign in to your laptop with.  
![][image91]

3\.	Next look for microservices.methodlocal.int under sites, right click on the site and select **add application**. Here we will set this to be the Method.Mobile.Notifications API. Here is the path for that `C:\MethodDev\runtime-core\Method.Mobile.Notifications\Method.Mobile.Notifications.Api`  
In the end it should look something like this:  
![][image92]

4\.	Go to `C:\MethodDev\runtime-core\Method.Mobile.Notifications` and open the solution in Visual Studio as administrator. Rebuild the solution. 

5\.	You should now be able to create/edit invoices in Method.  
---

# 14\. Postman (WIP)

1) If you hit the Postman login page, click the Skip Sign-in button hidden at the bottom  
   ![][image93]  
2) Import Method Postman Collections by:  
   1) Click “Import” at the top \=\> “Folders” tab ⇒ “Choose Folder…”  
      ![][image94]

   2) Select the “PostMan” folder from: C:\\MethodDev\\DeveloperTools\\AdHocTools\\PostMan  
   3) Click “Import”  ( Note \- if there’s a duplicate, import as copy)  
      ![][image95]

Import the collections and environments from C:\\MethodDev\\DeveloperTools\\AdHocTools\\PostMan into postman

# 15\. Portal & Public Pages (WIP)

## 15.1. Portal Setup ( PortalUI & Signin UI) 

- method-portal-ui  
- portal-signin-ui

[https://github.com/methodcrm/portal-signin-ui/blob/master/README.md](https://github.com/methodcrm/portal-signin-ui/blob/master/README.md)

## 15.2. Public Pages Setup

method-platform-ui

You will need an iis binding, and host entries. Point “/apps” to the regular method-platform-ui\\MethodUI folder. The base (non-”apps”) site can point to a blank folder.

\*\*To have methodportallocal running correctly, ensure that the following is present in IIS:

- Add a ‘runtime’ app manually under the [methodportallocal.com](http://methodportallocal.com) site if it doesn’t exist and make sure the advanced setting configurations are as follows:  
  ![][image96]

![][image97]

---

# 16\. Run audittrail projects

## 16.1. Setup audittrail-subscriber (.NET Core based agent)

1. Build Solution [https://github.com/methodcrm/runtime-core/blob/master/Events.Subscribers.AuditTrail.sln](https://github.com/methodcrm/runtime-core/blob/master/Events.Subscribers.AuditTrail.sln)  
2.   
3.   
4. CD to folder MethodDev\\runtime-core\\Events.Subscribers.AuditTrail\\bin\\Debug\\netcoreapp3.1 in Powershell   
5. Run dotnet audittrail-subscriber.dll

---

# 17\. Elastic Search

## 17.1. Setup Index Template and Pipeline

1. Create Template, import following in postman and run it

| curl \--location \--request POST 'localhost:9200/\_template/audittrail\_template' \\ \--header 'Host: localhost:9200' \\ \--header 'Content-Type: application/json' \\ \--header 'User-Agent: PostmanRuntime/7.19.0' \\ \--header 'Accept: /' \\ \--header 'Cache-Control: no-cache' \\ \--header 'Postman-Token: 4b76880b-d5ad-4cc7-8774-7e3de9eb4c19,6846d959-1369-43f5-ab87-57916eac9f77' \\ \--header 'Host: localhost:9200' \\ \--header 'Accept-Encoding: gzip, deflate' \\ \--header 'Connection: keep-alive' \\ \--header 'cache-control: no-cache' \\ |
| :---- |

   

   \<JSON BODY from [https://github.com/methodcrm/runtime-core/blob/master/Events.Subscribers.AuditTrail/elasticsearch\_template.json](https://github.com/methodcrm/runtime-core/blob/master/Events.Subscribers.AuditTrail/elasticsearch_template.json)\>

2. Create Pipeline, import following in postman and run it  
   Import into Postman with this format:

| curl \--location \--request PUT 'localhost:9200/\_ingest/pipeline/audittrail\_pipeline' \\ \--header 'Host: localhost:9200' \\ \--header 'Content-Type: application/json' \\ \--header 'cache-control: no-cache' \\ \--header 'Postman-Token: a8ab738d-8a72-4cf0-bb39-d3d1fcc42668' \\ |
| :---- |

   

   \<JSON BODY from [https://github.com/methodcrm/runtime-core/blob/master/Events.Subscribers.AuditTrail/audittrail\_pipeline.json](https://github.com/methodcrm/runtime-core/blob/master/Events.Subscribers.AuditTrail/audittrail_pipeline.json)\>

---

# 18\. Host File List {#18.-host-file-list}

| \#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\# 127.0.0.1 methodlocaldb 127.0.0.1 methodlocal 127.0.0.1 methodlocal.com 127.0.0.1 www.methodlocal.com 127.0.0.1 gateway.methodlocal.com 127.0.0.1 gateway.methodlocal.int 127.0.0.1 subscriber.methodlocal.com 127.0.0.1 runtime.methodlocal.com 127.0.0.1 microservices.methodlocal.int 127.0.0.1 auth.methodlocal.com 127.0.0.1 emailgadget.methodlocal.com 127.0.0.1 oauth.methodlocal.com 127.0.0.1 identityserver.methodlocal.com 127.0.0.1 eda.methodlocal.int 127.0.0.1 templatedevv2.methodlocal.com 127.0.0.1 alocetsystem.methodlocal.com 127.0.0.1 miurl.methodlocal.com 127.0.0.1 api.methodlocal.com 127.0.0.1 api.methodlocal.int 127.0.0.1 client.methodlocal.com 127.0.0.1 proxy.methodlocal.com 127.0.0.1 signup.methodlocal.com 127.0.0.1 signin.methodlocal.com 127.0.0.1 services.methodlocal.com 127.0.0.1 migration.methodlocal.com 127.0.0.1 sync.methodlocal.com 127.0.0.1 notify.methodlocal.com 127.0.0.1 intercom.methodlocal.com 127.0.0.1 billing.methodlocal.com 127.0.0.1 outlookgadget.methodlocal.com 127.0.0.1 appdirect.methodlocal.com 127.0.0.1 syncservices.methodlocal.com 127.0.0.1 designer.methodlocal.com 127.0.0.1 notifications.methodlocal.com 127.0.0.1 px.methodlocal.com 127.0.0.1 signin.methodportallocal.com 127.0.0.1 dataminer.methodlocal.com \#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\# |
| :---- |

# ---

19\. General Troubleshooting

If you are encountering any errors not covered by this document, here are some general guidelines you can use to help you determine the problem. Remember to ask your co-workers if you get stuck/are unsure how to proceed.

## 19.1 Log Files

If you are having issues with your local machine during setup and beyond, one of the first things you can reference are the log files which are written to your D: Drive.

Note: If you do not currently have a D: Drive, you can [shrink](https://docs.microsoft.com/en-us/windows-server/storage/disk-management/shrink-a-basic-volume) your current drive and [partition](https://support.microsoft.com/en-us/windows/create-and-format-a-hard-disk-partition-bbb8e185-1bda-ecd1-3465-c9728f7d7d2e) it (see 1.4 for details).

![][image98]

Logs are separated by project with each set of logs existing in their own folder. Typically there will be one debug and one error log per date. If you are encountering an error, you should reference the error log for today’s date to get the call stack for the error which will reference the exact line of code which threw the error.

If you are unsure which project could be throwing the error, you can attempt to delete/move the existing logs and reproduce the error. This should return only the logs relevant to the given action.

## 

## 19.2 Debugging .NET Core Projects

If your error exists in a project build using the .NET Core Framework, you can attach directly to the process using Visual Studio. Once you have set appropriate breakpoints for the process that you are debugging (refer to the call stack in the logs if you need help picking relevant areas of code to debug), simply select ‘Start Debugging’ from the ‘Debug’ toolbar (the green ‘play’ button) or pressing F5.

![][image99]

## 19.3 Debugging .NET Framework Projects

To debug an older project that was built using .NET Framework, you will first need to determine which worker process/application pool it is tied to. You can find this information in IIS.

In IIS, locate the site/application you are trying to debug, then select ‘Advanced Settings’ (either by right clicking the item on the left-hand menu or selecting it and choosing the option from the right-hand menu). Note which **Application Pool** is listed.

Next, select the top level option from the Connections menu (left-hand side of IIS, should be the name of your computer), then select Worker Processes from the main menu. This should bring up a list of your active processes. Note the **process id** that corresponds to the application pool you are interested in attaching to.

![][image100]

Finally, open up your project in Visual Studio (you will likely need to launch Visual Studio as an Administrator). From the Debug menu, select ‘Attach to Process’ (or press Ctrl+Alt+P). Locate the process that corresponds to the process id you retrieved from IIS (if you do not see it in the list, you may need to check off ‘Show processes from all users’. The process name should be w3wp.exe. After attaching to this process, you should be able to debug your .NET Framework project\!

## 19.4 App Pools & Services

Occasionally, issues on your local machine could just be a case of a required component not running. If services are unreachable and you are unable to debug them, you should verify whether the appropriate application pool or service is actually running.

For Application Pools, you can check this in IIS. Select the list of Application Pools from the left-hand menu and ensure that no application pools are currently stopped. If a pool is stopped, it can be restarted via the right-hand menu.

If your error log references one of our database services being unreachable, you should verify whether it is currently running. Access the ‘Services’ app by searching on your local.  
Scroll through the list until you find the appropriate service (ex. MongoDB Server, SQL Server) and ensure it is running. If it is not, select it and start the service.

## 19.5. Visual Studio is failing to pull down Method NuGets

First make sure that your VPN is on. Our NuGet Feed is hosted on our TFS server which is behind the firewall.

If you have recently changed your AD credentials, the previous credentials are cached in Windows Credential Manager and you need to delete those first.

Use the Windows key and search for “Credential Manager” or access it from Control Panel \> User Accounts \> Credential Manager:

Select the Windows Credential and find the entries that start with tfs and delete them. 

![][image101]

Then close the Visual Studio and reopen it. Right mouse click on the solution and choose the “Restore NuGet Packages” open. This time it should ask you for your new credentials. Remember your username should start with “Method\\” 

## 19.6. 

# 20\. DNS Setup (optional)

| Troubleshooting Note:  If your local hangs and is stuck in a loading state because JS/CSS assets won’t fetch, you might want to remove these DNS settings, as you might not need them. |
| :---- |

Some internal addresses (such as anything in the “**method.local**” domain) will only be available after using these DNS settings:

1. Navigate to the Advanced TCP/IP Settings panel by:

   1. Press *WindowsKey* and search for “**Control Panel**”

   2. Go to “**Network and Internet**”  ⇒  “**Network and Sharing Center**” ⇒  “**Change adapter settings**”

   3. **Right-Click** your ethernet or wifi connection ⇒ select “**Properties”**  
      ![][image102]

   4. Select “**Internet Protocol Version 4 (TCP/IPv4)**” ⇒  click “**Properties**” ⇒ click “**Advanced**”  
      ![][image103]

   5. Add the following DNS servers and suffixes **in this order:**

* **Our DNS server:      192.168.0.21**  
* **Beanfield DNS (1):   66.207.192.6**  
* **Beanfield DNS (2):   206.223.173.7**  
* **Google DNS:            8.8.8.8**

* **DNS Suffix:** **method.local**

  **![][image104]**

---

# 21\. New Hard Drive Setup

If you’re not cloning your hard drive and aiming for a clean install, follow the steps below to get it started:

1. Find out what **Version of Windows** you are running \- professional, home, etc.

2. Ask Dev Management for a copy of a  **Windows ISO Image** that’s specific to your Windows version

3. Get a **USB Flash Drive** with enough capacity to hold the ISO image

* Make sure to format the usb drive to “FAT32”

4. Download [**Rufus Image Burner**](https://rufus.ie/) and launch it

   1. Set the “Device” to your USB Flash Drive

   2. Set the “Boot Selection” to your Windows ISO image

   3. Check the other options to ensure they’re okay

   4. Click “Start” to burn the image onto the drive.

5. (\!)  Make sure to get your Windows **Product Key** ( License Key) so you can activate your windows again after the installation.

   1. Open Command Prompt as an Administrator.

   2. Run this command:   
      wmic path SoftwareLicensingService get OA3xOriginalProductKey

   3. Your product key should be revealed \- Make sure to write it down.

6.  **Replace** your old hard drive with the new one ( [Example tutorial](https://www.youtube.com/watch?v=oFCZQLKYHAU) )

7. **Reboot** your computer from the flash drive and start the installation process.

* Watch the [instructional video here](https://www.youtube.com/watch?v=aJCkI14Lcd4) for a complete walkthrough of how you can install windows from a usb.

* As part of the installation, you can partition your new drive

* Make sure to choose “Setup for personal use” when given the option

8. After the installation, **activate your new Windows** using the Product Key that you jotted down in *Step 5*:

   1. Open *Windows Explorer*

   2. In the left-hand pane, Right-Click on “This PC” and select “Properties” 

   3. Click on “Change Product Key” at the bottom of the window to enter yours  
      ![][image105]

---

# 22\. Download and Restore SQL/Mongo DBs

1. Run copy tool api from postman or use alocetsystem tools to queue the copy job for the database you like to download.  
   Here is an example:

| curl \--location 'https://api.method.me/support/db/copy' \\ \--header 'Content-Type: application/json' \\ \--header 'method-client: auth-legacy' \\ \--header 'Authorization: Bearer {{YOUR-TOKEN}}' \\ \--data '{    "Name": "{{DB-NAME}}",   "TargetEnvironment": "{{DESTINATION-ENV}}",   "OverwriteTarget": true,   "RequestLiveData": false,   "CopyFilesOnly": false } ' |
| :---- |

   

2. You can check [support-tools](https://methodme.slack.com/archives/C9L8RPKH8) slack channel to see the job progress  
3. Open the website [https://dev-setup.methodintegration.com](https://dev-setup.methodintegration.com/) and log in using your AD credentials. After logging in, select your database from the list. Inside the database, there will be two folders: "SQL" and "Mongo". Download all the files contained in both folders.  
   * **SQL**  
     Open SQL Management Studio and connect to the local server. Once connected, right-click on the "Databases" folder and select "Restore Database". Then, follow these steps and choose the path that contains the backup file for your database.

     ![][image44]

   * **MongoDB**  
     Make sure you have MongoDB tools installed on your machine, you can download the latest version from here:

     [Download MongoDB Command Line Database Tools | MongoDB](https://www.mongodb.com/try/download/bi-connector)

     ![][image106]

     

     Once you have downloaded the MongoDB bak file for your database to your local machine (as shown in the screenshot above, the file is named m11pakbaz2.bak). Navigate to the folder where you have placed the MongoDB tools. You should be able to locate the mongorestore executable. Open the terminal and type the following command:

     .\\mongorestore \--host="localhost" \--port=27017 \--db={{db-name}} \--archive={{path\_to\_bak\_file}} \--gzip

     **{{db-name}}:** is your dbname

     **{{paht\_to\_bak\_file}}:** the path to where you have downloaded bak file

     

     **Example:**   
     If you have downloaded a database called "m11pakbaz2.bak" and you have stored it in the "D:\\mongo-backups" directory, Therefore, your \`mongorestore\` command will be as follows:

     .\\mongorestore.exe \--host="localhost" \--port=27017 \--db=m11pakbaz2 \--archive=D:\\mongo-backups\\m11pakbaz2.bak \--gzip

# 23\. Run healthCheck/ Warm up application

    1\.	Build Solution C:\\MethodDev\\DeveloperTools\\Method-HealthChecks.  
    2\.	Publish a single file version of the application **HealthCheck.App**.   
    ![][image107]  
    3\.	Open the publish folder and run the application.  
    4\.	Review all errors and warning to indicate which services need to be   
examined.   
This information will also be stored in the log files located at   
D:\\logs\\HealthCheckApp  
5\.	If all services are working correctly you, Method UI will be warmed up   
and ready for use. 

PS	The HealthCheckService.Json file in the publish folder can be edited to add/remove services as necessary.  
![][image108]

# 24\. Setting Up Elasticsearch and Kibana

Previously, Method used NLog to log errors; for local environments, we logged to the D Drive. We are in the process of migrating from NLog to Serilog to provide more useful contexts and additional information to the logs, and using Elasticsearch as a sink. Thus, we need to set up Elasticsearch on local machines to see these new logs. Additionally, we will set up Kibana, a data visualization software for Elasticsearch, allowing us to easily visualize, query, and create dashboards for our logs\! Way better than seeing logs in a Windows folder, right?

* We are using Docker to automatically pull down the images for Elasticsearch and Kibana, and they will be automatically set up in the docker-compose script.  
* On AMIs, Elasticsearch and Kibana will automatically run on startup; however, it can take from 5 to 10 minutes to be up and running. On local machines, it should take less than 30 seconds

Here are the steps to setup Elasticsearch and Kibana on local machines:

1. First, ensure you have Docker Desktop installed (this should’ve been done in a previous step).   
   Note: If you already have a container for elasticsearch, delete it (as it will be setup by the docker-compose script)  
2. Open a terminal and run the following command to cd into the folder:  
   cd C:\\MethodDev\\DeveloperTools\\DevSetup\\elasticsearch-kibana  
3. Then, run the following command to run the docker-compose file:  
   docker-compose up \-d  
4. Verify that elasticsearch is working at [http://localhost:9200/](http://localhost:9200/). It should look something like this:  
   ![][image109]  
5. Verify that Kibana is working at [http://localhost:5601/](http://localhost:5601/). It should look something like this:  
   ![][image110]  
6. Now, Elasticsearch and Kibana should have been fully set up. Refer to this page to create index patterns to start seeing your logs\! To view all logs, simply make an index pattern with a wildcard (\*).  
   [https://www.elastic.co/guide/en/kibana/7.17/index-patterns.html](https://www.elastic.co/guide/en/kibana/7.17/index-patterns.html) 

 


# Appendix

## A Enable IIS Features

1) Press *WindowsKey* ⇒ search for “**Turn Windows features on or off**”  
   ![][image111]

2) **Turn ON** the following features if they’re unchecked:

* **ASP.NET 4.7**  
  ![][image112]

* **IIS Management Console**  
  ![][image113]

* The following **4 features** under *“Common HTTP Features*”  
  ![][image114]

* Most of the “*Application Development Features*” **excluding CGI & .NET3.5** features  
  ![][image115]

3) Click “**OK**” to install 

4) **Restart your PC**
