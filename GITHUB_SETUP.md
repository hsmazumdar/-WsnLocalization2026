# GitHub Repository Setup Guide

This guide will help you publish this codebase to your GitHub account at https://github.com/hsmazumdar/

## Step 1: Create Repository on GitHub

1. Go to https://github.com/new
2. Sign in to your GitHub account (hsmazumdar)
3. Repository settings:
   - **Repository name**: `WsnLocalization` (or your preferred name)
   - **Description**: "Wireless Sensor Network (WSN) Localization Algorithm - C# Application for anchor-free, range-free node localization"
   - **Visibility**: Choose Public or Private (recommend Public for academic/research)
   - **DO NOT** initialize with README, .gitignore, or license (we already have these)
4. Click **Create repository**

## Step 2: Add Remote and Push

After creating the repository, GitHub will show you commands. Use these commands in your terminal:

```bash
cd "d:\_January2026\WsnQukMap"
git remote add origin https://github.com/hsmazumdar/WsnLocalization.git
git branch -M main
git push -u origin main
```

**Note**: If you chose a different repository name, replace `WsnLocalization` with your chosen name.

## Alternative: Using SSH (if you have SSH keys set up)

If you prefer using SSH instead of HTTPS:

```bash
cd "d:\_January2026\WsnQukMap"
git remote add origin git@github.com:hsmazumdar/WsnLocalization.git
git branch -M main
git push -u origin main
```

## Authentication

When you push for the first time, GitHub will prompt for authentication:
- **For HTTPS**: You may need to use a Personal Access Token instead of password
- **For SSH**: Make sure your SSH key is added to your GitHub account

## What's Already Done

✅ Git repository initialized  
✅ .gitignore file created (excludes build artifacts, Visual Studio files, etc.)  
✅ All source files added  
✅ Initial commit created with message: "Initial commit: WSN Localization Application - Mazumdar"  
✅ Git user configured as: H.S.Mazumdar <hsmazumdar@users.noreply.github.com>

## Recommended Repository Name

Based on your existing GitHub repositories:
- ✅ `WsnLocalization` - Recommended (matches your work on WSN localization)
- `WsnMap` - Alternative (matches your solution name)
- `WsnLocalizerCSharp` - Alternative (if you want to distinguish from Python version)

## Next Steps After Pushing

1. Add repository description and topics on GitHub
2. Update README.md if needed with more details
3. Consider adding a LICENSE file (MIT, Apache, or Academic License)
4. Add repository to your profile's pinned repositories if desired
