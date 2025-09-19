# Git Configuration and Installation

This guide covers Git installation and configuration for Method's development environment, based on the Method Developer Machine Setup documentation.

## Overview

Git is essential for Method's development workflow. This guide covers installing Git for Windows with the correct settings and configuring it for Method development.

## Prerequisites

- Windows 10/11 updated
- Administrative access to install software
- Method email address established

## Git Installation

### Download and Install Git

**Note:** Disconnect from VPN while downloading to improve download speed.

1. Download the latest version of **Git** from: https://git-scm.com/download/win

2. Run the downloaded Setup file

3. Ensure the following components are selected:
   - Git Bash Here
   - Git GUI Here  
   - Git LFS (Large File Support)
   - Associate .git* configuration files with the default text editor
   - Associate .sh files to be run with Bash

4. Leave "Start Menu Folder" as Git. Click "Next".

5. Set your editor to **Vim or NotePad++**. Click "Next".

6. "Let Git Decide" what Branch new repos should be on. Click "Next"

7. Go with the **Recommended settings for Adjusting PATH** environment. Click "Next".

8. If option for "Use bundled openSSH" set as SSH executable

9. "Use the OpenSSL Library" as your SSL/TSL library. Click "Next"

10. Keep Line endings **Windows-style**. Click "Next"

11. "Use MinTTY" as a terminal emulator.

12. Git Pull should be set to **Default behaviour**. Click "Next"

13. Use "Git Credential Manager Core". Click "Next"

14. Make sure "Enable File System Caching" is checked. Click "Next"

15. Click "Install"

**You should now have access to Git Bash**

## Git Configuration

### Basic Configuration

After installing Git, configure it with your Method credentials:

```bash
# Open Git Bash and configure user information
git config --global user.name "Your Name"
git config --global user.email "your.email@method.me"

# Set default branch name
git config --global init.defaultBranch main

# Configure line endings for Windows
git config --global core.autocrlf true

# Set default editor (if using VS Code)
git config --global core.editor "code --wait"
```

### Method-Specific Configuration

```bash
# Configure useful aliases for Method development
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.st status
git config --global alias.unstage 'reset HEAD --'
git config --global alias.last 'log -1 HEAD'
git config --global alias.visual '!gitk'
```

## Next Steps

After Git installation and configuration:

1. **Set up SSH Keys:** [SSH Keys Setup](./ssh-keys.md)
2. **Configure GitHub Account:** [GitHub Account Setup](#github-account-setup)
3. **Set up Repository Access:** [Repository Access](./repository-access.md)

## GitHub Account Setup

### Account Setup

1. Establish your **GitHub username**
   - Best practice is to set up using your **Method email**
   - Don't have an account? Use [Join GitHub](https://github.com/join) to create your account

2. Login to your GitHub account

3. Go to [Settings > Emails](https://github.com/settings/emails)

4. Add your Method Email as a **secondary** email

5. Email/Slack your Git Administrators (Paul/Hossein/Rich) with information about your **GitHub username** so they can add you to the methodcrm organization

6. Once added to the Method repo, go to [Notification Settings](https://github.com/settings/notifications)

7. Under "Custom Routing" click the "Edit" button and set the notification destination to your **method.me email address**, so all Method repo messages go to your work inbox

### Personal Access Token

You need to create a Personal Access Token for your GitHub account for automated project setup using GitHub API:

1. Sign in to GitHub and go to [Developer Settings](https://github.com/settings/tokens)

2. Click on "Generate New Token"

3. Recommended to select a longer expiration than default 30 days (e.g. 1 year)

4. Add the following token configuration:
   - **Note:** Enter "MethodDevSetup"
   - **Scopes:**
     - Select "repo"
     - Select "admin:org" > "read:org" **only**
     - Leave other options **off**

5. Click "Generate Token" at the bottom of the page

**Important:** Save this token securely as you won't be able to see it again.

## Git Clients

### TortoiseGit Installation

1. Download the latest version **TortoiseGit** from: https://tortoisegit.org/download/

2. Run the downloaded setup file

3. Click "Next" and accept the agreement

4. Use **TortoiseGitPlink as the SSH client**. Click "Next"

5. Leave "Custom Setup" as is, and click "Next"

6. Click "Install", then "Finish"

**You should now be able to right-click inside File Explorer and see TortoiseGit in the menu**

### Optional Git Tools

**[OPTIONAL]** You can install **Architecture Decision Record (ADR)** CLI tool for generating ADR templates:
- https://github.com/npryce/adr-tools/blob/master/INSTALL.md

**[OPTIONAL]** Download & install **GitHub Desktop** for a visual Git experience:
- https://desktop.github.com/

## Verification

To verify Git is properly installed and configured:

```bash
# Check Git version
git --version

# Check configuration
git config --global --list

# Test Git functionality
git init test-repo
cd test-repo
echo "Test file" > test.txt
git add test.txt
git commit -m "Initial commit"
cd ..
rm -rf test-repo
```

## Next Steps

After completing Git configuration:

1. **Set up SSH Keys:** [SSH Keys Setup](./ssh-keys.md)
2. **Learn Branching Strategy:** [Branching Strategy](./branching-strategy.md)
3. **Understand Development Workflow:** [Development Workflow](./development-workflow.md)
4. **Access Method Repositories:** [Repository Access](./repository-access.md)

**Back to:** [GitHub Setup](./README.md)
