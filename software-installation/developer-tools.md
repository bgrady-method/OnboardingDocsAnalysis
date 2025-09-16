# Developer Tools Installation

## 4.1 Download developer tools

1. Create the C:\\MethodDev folder if you don't have it already.

2. Pull the the repo [https://github.com/methodcrm/DeveloperTools](https://github.com/methodcrm/DeveloperTools) to get access to all the scripts you need to help setup and maintain your local development environment.

3. Open powershell as administrator and navigate to folder C:\\MethodDev\\DeveloperTools\\DevSetup\\build_local

4. Run the command initialize.ps1 to install most of the software you need and setup needed features.

## Troubleshooting/Notes:

* Might have to redo last 2 commands:  
  * gem install sass  
  * npm install -g grunt-cli  
  * This is because you need to Close and Reopen powershell in Admin mode for Gem and Npm to load  

* If the "refreshenv" command doesn't work:  
  * Run *Import-Module $env:ChocolateyInstall\\helpers\\chocolateyProfile.psm1*  
  * Then rerun refreshenv  

* MSBuild is now installed in a folder under each version of Visual Studio. For example, _C:\\Program Files (x86)\\Microsoft Visual Studio\\2017\\Enterprise\\MSBuild_. You will likely need to update your script to reference the correct MSBuild path  

* If you get an error saying that "Containers" does not exist:  
  Enable-WindowsOptionalFeature : Feature name Containers is unknown.  
  Try enabling Windows Hypervisor Platform in Windows Features (not sure if it works or not):  
  ![][image16]  

* When you try to run: .\\initialize.ps1 you might get an error message: "..\\initialize.ps1 cannot be loaded because running scripts is disabled on this system.   
  Solution: run this command to enable execution of scripts:  
  Set-ExecutionPolicy -ExecutionPolicy RemoteSigned  

* If you encounter an error stating "WMC is not a recognized command", open Windows 'Optional Features' and install it there. WMC has apparently been defaulted to not installed in versions of Windows 11 since 24H2

## What Gets Installed

The initialize.ps1 script installs:

- **Chocolatey** - Package manager for Windows
- **Node.js and NPM** - JavaScript runtime and package manager
- **Ruby and Gems** - Ruby runtime for Sass compilation
- **Git** - Version control system
- **Docker Desktop** - Container platform
- **Visual Studio Build Tools** - Compilation tools
- **Python** - For various build scripts
- **Other Development Essentials** - Various utilities and tools

**Next:** [Docker Packages](./docker-packages.md)
