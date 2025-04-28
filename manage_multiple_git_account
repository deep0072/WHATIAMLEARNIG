# Setting Up Multiple GitHub Accounts on One Machine

This comprehensive guide walks you through the process of setting up and using multiple GitHub accounts (such as personal and work accounts) on the same machine.

## Prerequisites
- Git installed on your machine
- Terminal/Command Line access

## Step 1: Create SSH Keys for Each Account

First, navigate to your SSH directory:
```bash
cd ~/.ssh
```

Generate an SSH key for your first account (e.g., work account):
```bash
ssh-keygen -t rsa -C "your-work-email@company.com"
```

When prompted for a file name, specify a custom name to avoid overwriting existing keys:
```
Enter file in which to save the key: ~/.ssh/id_rsa_work
```

Follow the same process for your second account (e.g., personal account):
```bash
ssh-keygen -t rsa -C "your-personal-email@example.com"
```

Name this key differently:
```
Enter file in which to save the key: ~/.ssh/id_rsa_personal
```

## Step 2: Add SSH Keys to GitHub Accounts

For each GitHub account:

1. Copy the public key content:
   ```bash
   # For work account
   cat ~/.ssh/id_rsa_work.pub
   
   # For personal account
   cat ~/.ssh/id_rsa_personal.pub
   ```

2. Log in to the respective GitHub account
3. Go to Settings → SSH and GPG keys → New SSH key
4. Give the key a descriptive title (e.g., "Work Laptop" or "Personal Laptop")
5. Paste the public key content and click "Add SSH key"

## Step 3: Create SSH Config File

Create or edit the config file in your SSH directory:
```bash
touch ~/.ssh/config
```

Add the following configurations:

```
# Work GitHub account
Host github.com-work
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_rsa_work

# Personal GitHub account
Host github.com-personal
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_rsa_personal
```

## Step 4: Using Different Accounts

### Cloning Repositories

When cloning repositories, use the appropriate host:

For work repositories:
```bash
# Original URL: git@github.com:WorkOrg/repo-name.git
git clone git@github.com-work:WorkOrg/repo-name.git
```

For personal repositories:
```bash
# Original URL: git@github.com:YourUsername/repo-name.git
git clone git@github.com-personal:YourUsername/repo-name.git
```

### Setting Up User Configuration for Repositories

After cloning a repository, set the local Git configuration:

For work repositories:
```bash
cd work-repository
git config user.name "Your Work Name"
git config user.email "your-work-email@company.com"
```

For personal repositories:
```bash
cd personal-repository
git config user.name "Your Personal Name"
git config user.email "your-personal-email@example.com"
```

### Adding Remote to Existing Repository

For work repositories:
```bash
git remote add origin git@github.com-work:WorkOrg/repo-name.git
```

For personal repositories:
```bash
git remote add origin git@github.com-personal:YourUsername/repo-name.git
```

## Step 5: Managing SSH Agent

Start the SSH agent:
```bash
eval $(ssh-agent -s)
```

Remove existing identities:
```bash
ssh-add -D
```

Add the specific identity you want to use:
```bash
# For work
ssh-add ~/.ssh/id_rsa_work

# For personal
ssh-add ~/.ssh/id_rsa_personal
```

## Troubleshooting

If you encounter authentication issues:

1. Verify your SSH agent has the correct key loaded:
   ```bash
   ssh-add -l
   ```

2. Test your SSH connection:
   ```bash
   # For work
   ssh -T git@github.com-work
   
   # For personal
   ssh -T git@github.com-personal
   ```

3. Check your remote URL:
   ```bash
   git remote -v
   ```

4. Update remote if necessary:
   ```bash
   git remote set-url origin git@github.com-work:WorkOrg/repo-name.git
   ```

## Example Workflow

### Switching Between Accounts

#### Working with Work GitHub:

```bash
# Start SSH agent
eval $(ssh-agent -s)

# Remove existing identities
ssh-add -D

# Add work key
ssh-add ~/.ssh/id_rsa_work

# Clone a work repository
git clone git@github.com-work:WorkOrg/project.git

# Set local repo config
cd project
git config user.name "Your Work Name"
git config user.email "your-work-email@company.com"

# Make changes and push
git add .
git commit -m "Work-related changes"
git push origin main
```

#### Switching to Personal GitHub:

```bash
# Start SSH agent (or use existing)
eval $(ssh-agent -s)

# Remove existing identities
ssh-add -D

# Add personal key
ssh-add ~/.ssh/id_rsa_personal

# Clone a personal repository
git clone git@github.com-personal:YourUsername/personal-project.git

# Set local repo config
cd personal-project
git config user.name "Your Personal Name"
git config user.email "your-personal-email@example.com"

# Make changes and push
git add .
git commit -m "Personal project update"
git push origin main
```

By following this guide, you can seamlessly switch between multiple GitHub accounts on the same machine without authentication conflicts.
