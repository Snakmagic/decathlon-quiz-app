# GitHub Setup Guide

## Overview
Your project is ready to be pushed to GitHub! Follow these steps to get it online.

## Current Status
✅ Git repository initialized
✅ All files ready to commit
✅ Project structure complete
⏳ Needs: GitHub repository creation and initial push

## Steps to Set Up GitHub

### 1. Configure Git (One-time setup)
```bash
cd /home/claude/francis-quiz-app
git config user.name "Your Name"
git config user.email "your.email@example.com"
```

### 2. Create Initial Commit
```bash
git add .
git commit -m "Initial commit: Project setup with all 18 chapter question files"
```

### 3. Create GitHub Repository
1. Go to https://github.com/new
2. Repository name: `francis-quiz-app` (or whatever you prefer)
3. Description: "Academic Decathlon 2026 - St. Francis Quiz App"
4. **Important**: Keep it **Public** (required for GitHub Pages free hosting)
5. **Don't** initialize with README (we already have one)
6. Click "Create repository"

### 4. Connect Local Repo to GitHub
GitHub will show you commands, but here they are:

```bash
cd /home/claude/francis-quiz-app
git remote add origin https://github.com/YOUR-USERNAME/francis-quiz-app.git
git push -u origin main
```

Replace `YOUR-USERNAME` with your GitHub username.

### 5. Enable GitHub Pages (After Claude Code builds the app)
Once the app is built:
1. Go to repository Settings
2. Click "Pages" in left sidebar
3. Under "Source":
   - Branch: `main`
   - Folder: `/public`
4. Click "Save"
5. Wait 1-2 minutes
6. Your app will be live at: `https://YOUR-USERNAME.github.io/francis-quiz-app/`

## Project Directory Structure

```
francis-quiz-app/
├── .git/                    # Git repository (initialized ✅)
├── .gitignore              # Files to ignore
├── README.md               # Project documentation
├── BUILD_INSTRUCTIONS.md   # Instructions for Claude Code
├── data/                   # Question data (18 JSON files ✅)
│   ├── st-francis-ch01.json
│   ├── st-francis-ch02.json
│   └── ... (all 18 chapters)
├── public/                 # Will contain the built app
│   └── index.html         # Claude Code will create this
└── src/                    # Empty for now (may not need)
```

## What's Next?

### For You (Human):
1. Follow steps 1-4 above to push to GitHub
2. Share the repository with Claude Code

### For Claude Code:
1. Clone the repository
2. Read BUILD_INSTRUCTIONS.md
3. Create the quiz application in `/public/index.html`
4. Test it works
5. Commit and push

### After Claude Code Builds:
1. Enable GitHub Pages (step 5 above)
2. Share the URL with your son
3. He can start studying!

## Troubleshooting

**Q: I get "permission denied" when pushing**
A: You may need to set up SSH keys or use a personal access token. See: https://docs.github.com/en/authentication

**Q: How do I update the app after changes?**
A: 
```bash
git add .
git commit -m "Description of changes"
git push
```

**Q: Can I make the repo private?**
A: Yes, but GitHub Pages for private repos requires GitHub Pro. Keep it public for free hosting.

**Q: What if I want to add more subjects later?**
A: Just add new JSON files to `/data/` folder with the same format. The app will automatically detect them (future enhancement).

## Files Ready for GitHub

All these files are staged and ready to commit:
- ✅ .gitignore
- ✅ README.md
- ✅ BUILD_INSTRUCTIONS.md
- ✅ data/st-francis-ch01.json through ch18.json (18 files)

**Total: 228 questions ready to go!** 🎉
