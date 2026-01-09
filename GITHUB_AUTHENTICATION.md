# GitHub Authentication Setup

GitHub no longer accepts passwords for Git operations. You need to use either:
1. **Personal Access Token (PAT)** - Recommended, easier to set up
2. **SSH Keys** - More secure, requires setup

## Option 1: Personal Access Token (PAT) - RECOMMENDED

### Step 1: Create a Personal Access Token

1. Go to GitHub: https://github.com/settings/tokens
2. Click **"Generate new token"** → **"Generate new token (classic)"**
3. Token settings:
   - **Note**: `WsnLocalization - Local Development`
   - **Expiration**: Choose duration (90 days, 1 year, or no expiration)
   - **Scopes**: Check `repo` (this gives full control of private repositories)
     - This includes: `repo:status`, `repo_deployment`, `public_repo`, `repo:invite`, `security_events`
4. Click **"Generate token"**
5. **IMPORTANT**: Copy the token immediately - you won't be able to see it again!
   - It will look like: `ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`

### Step 2: Use the Token for Push

When you run `git push`, it will ask for credentials:

**Username**: `hsmazumdar`  
**Password**: Paste your Personal Access Token (not your GitHub password!)

### Alternative: Store Token in Git Credential Manager

You can store the token so you don't have to enter it each time:

**Windows (Git Credential Manager):**

```bash
git config --global credential.helper manager-core
```

Then when you push, enter:
- Username: `hsmazumdar`
- Password: [Your PAT token]

---

## Option 2: SSH Authentication - More Secure

### Step 1: Check if you have SSH keys

```bash
ls ~/.ssh
```

If you see `id_rsa.pub` or `id_ed25519.pub`, you already have SSH keys.

### Step 2: Generate SSH Key (if needed)

```bash
ssh-keygen -t ed25519 -C "hsmazumdar@users.noreply.github.com"
```

Press Enter to accept default file location.  
Enter a passphrase (optional but recommended for security).

### Step 3: Copy Public Key

```bash
cat ~/.ssh/id_ed25519.pub
```

Copy the entire output (starts with `ssh-ed25519`...)

### Step 4: Add SSH Key to GitHub

1. Go to: https://github.com/settings/keys
2. Click **"New SSH key"**
3. **Title**: `WsnLocalization - Development`
4. **Key**: Paste your public key
5. Click **"Add SSH key"**

### Step 5: Change Remote URL to SSH

```bash
cd "d:\_January2026\WsnQukMap"
git remote set-url origin git@github.com:hsmazumdar/WsnLocalization.git
```

### Step 6: Test SSH Connection

```bash
ssh -T git@github.com
```

You should see: `Hi hsmazumdar! You've successfully authenticated...`

### Step 7: Push Using SSH

```bash
git push -u origin main
```

---

## Quick Fix: Use PAT Now

If you just want to push quickly, create a PAT and use it:

1. Create PAT at: https://github.com/settings/tokens/new (classic)
2. Check `repo` scope
3. Copy the token
4. Run: `git push -u origin main`
5. When prompted:
   - Username: `hsmazumdar`
   - Password: [Paste your PAT token]

---

## Troubleshooting

### If you get "Permission denied" with SSH:
- Make sure your SSH key is added to GitHub
- Test connection: `ssh -T git@github.com`

### If you get "Authentication failed" with HTTPS:
- Make sure you're using PAT token, not password
- Check that the token has `repo` scope
- Try: `git config --global credential.helper manager-core` to cache credentials

### If you want to remove cached credentials:
```bash
git credential-manager-core erase
# Then enter: host=github.com
```
