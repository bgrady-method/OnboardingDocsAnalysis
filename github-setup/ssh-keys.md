# SSH Keys Setup for GitHub

This guide covers setting up SSH keys for GitHub access, based on the Method Developer Machine Setup documentation.

## Overview

SSH keys provide secure authentication to GitHub without needing to enter your username and password each time. This is essential for Method's development workflow.

## Prerequisites

- Git installed and configured
- GitHub account set up with Method email
- Access to GitHub account settings

## SSH Key Setup Process

Follow the GitHub documentation steps in order:

### 1. Check for Existing SSH Keys

First, check if you already have SSH keys:

1. Follow the GitHub guide: [Check for existing SSH keys](https://docs.github.com/en/free-pro-team@latest/github/authenticating-to-github/checking-for-existing-ssh-keys)

### 2. Generate New SSH Key

If you don't have existing keys or need new ones:

1. Follow the GitHub guide: [Generate a new SSH key and add it to the ssh-agent](https://docs.github.com/en/free-pro-team@latest/github/authenticating-to-github/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)

**Important Note:** Use **empty passphrase** when setting up Git SSH keys for Method development.

### 3. Add SSH Key to GitHub Account

Add your public key to your GitHub account:

1. Follow the GitHub guide: [Add the new SSH key to your GitHub account](https://docs.github.com/en/free-pro-team@latest/github/authenticating-to-github/adding-a-new-ssh-key-to-your-github-account)

### 4. Test SSH Connection

Verify that your SSH connection works:

1. Follow the GitHub guide: [Test your SSH connection](https://docs.github.com/en/free-pro-team@latest/github/authenticating-to-github/testing-your-ssh-connection)

## Quick Setup Commands

If you prefer command-line setup, here are the essential commands:

### Generate SSH Key

```bash
# Open Git Bash and generate SSH key (use your Method email)
ssh-keygen -t ed25519 -C "your.email@method.me"

# When prompted for file location, press Enter for default
# When prompted for passphrase, press Enter (leave empty) twice
```

### Start SSH Agent and Add Key

```bash
# Start SSH agent
eval "$(ssh-agent -s)"

# Add your SSH key to the agent
ssh-add ~/.ssh/id_ed25519
```

### Copy Public Key

```bash
# Copy the public key to clipboard
clip < ~/.ssh/id_ed25519.pub
```

### Add to GitHub

1. Go to [GitHub SSH Keys Settings](https://github.com/settings/ssh/new)
2. Paste the key into the "Key" field
3. Give it a descriptive title (e.g., "Method Dev Machine")
4. Click "Add SSH key"

### Test Connection

```bash
# Test SSH connection to GitHub
ssh -T git@github.com

# You should see a message like:
# "Hi username! You've successfully authenticated, but GitHub does not provide shell access."
```

## Troubleshooting

### Common Issues

**Permission Denied (publickey)**
- Ensure your SSH key was added to the SSH agent
- Verify the public key was correctly added to your GitHub account
- Check that you're using the correct key file

**SSH Agent Not Running**
```bash
# Start SSH agent if not running
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

**Wrong SSH Key Type**
- Method recommends using Ed25519 keys for better security
- If you have RSA keys, they should work but Ed25519 is preferred

### Verify SSH Configuration

```bash
# Check SSH configuration
ssh -T git@github.com -v

# List keys added to SSH agent
ssh-add -l

# Check SSH key files
ls -la ~/.ssh/
```

## Multiple SSH Keys

If you need multiple SSH keys for different purposes:

### SSH Config File

Create or edit `~/.ssh/config`:

```
# Method GitHub account
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519

# Personal GitHub account (if needed)
Host github-personal
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_personal
```

## Security Best Practices

1. **Use Ed25519 keys** when possible (more secure than RSA)
2. **Keep private keys secure** - never share your private key
3. **Use empty passphrase** for Method development (as specified in setup guide)
4. **Regularly rotate keys** if security requires it
5. **Remove old keys** from GitHub when no longer needed

## Next Steps

After setting up SSH keys:

1. **Test repository cloning:** Try cloning a Method repository
2. **Configure Git settings:** [Git Configuration](./git-configuration.md)
3. **Learn branching strategy:** [Branching Strategy](./branching-strategy.md)
4. **Set up development workflow:** [Development Workflow](./development-workflow.md)

## Verification Checklist

- [ ] SSH key generated successfully
- [ ] SSH key added to SSH agent  
- [ ] Public key added to GitHub account
- [ ] SSH connection test passes
- [ ] Can clone Method repositories using SSH URLs

**Back to:** [GitHub Setup](./README.md)
