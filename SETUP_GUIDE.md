# 📋 Quick Setup Guide

## What You Downloaded

You should have downloaded:
- ✅ **18 JSON files** (st-francis-ch01.json through ch18.json)
- ✅ **README.md** (project overview)
- ✅ **BUILD_INSTRUCTIONS.md** (for Claude Code)
- ✅ **GITHUB_SETUP.md** (GitHub deployment guide)
- ✅ **gitignore.txt** (will become .gitignore)

## Setup Steps

### 1. Create Project Folder Structure

On your computer, create this folder structure:

```
francis-quiz-app/
├── data/
├── public/
└── src/
```

**How to do it:**
- Create a folder called `francis-quiz-app`
- Inside it, create 3 folders: `data`, `public`, `src`

### 2. Place the Files

**JSON files (all 18 chapters):**
- Put ALL the `st-francis-ch**.json` files into the `data/` folder

**Documentation files:**
- Put `README.md` in the root `francis-quiz-app/` folder
- Put `BUILD_INSTRUCTIONS.md` in the root folder
- Put `GITHUB_SETUP.md` in the root folder

**Gitignore file:**
- Rename `gitignore.txt` to `.gitignore` (note the dot at the start!)
- Put it in the root `francis-quiz-app/` folder
- **Important**: On Windows, you may need to rename it in Command Prompt or save it from a text editor as `.gitignore`

### 3. Final Structure

Your folder should look like this:

```
francis-quiz-app/
├── .gitignore                 # ← Renamed from gitignore.txt
├── README.md
├── BUILD_INSTRUCTIONS.md
├── GITHUB_SETUP.md
├── data/
│   ├── st-francis-ch01.json
│   ├── st-francis-ch02.json
│   ├── st-francis-ch03.json
│   ├── st-francis-ch04.json
│   ├── st-francis-ch05.json
│   ├── st-francis-ch06.json
│   ├── st-francis-ch07.json
│   ├── st-francis-ch08.json
│   ├── st-francis-ch09.json
│   ├── st-francis-ch10.json
│   ├── st-francis-ch11.json
│   ├── st-francis-ch12.json
│   ├── st-francis-ch13.json
│   ├── st-francis-ch14.json
│   ├── st-francis-ch15.json
│   ├── st-francis-ch16.json
│   ├── st-francis-ch17.json
│   └── st-francis-ch18.json  # ← All 18 files
├── public/                    # ← Empty for now (Claude Code will add index.html)
└── src/                       # ← Empty (may not be needed)
```

### 4. Initialize Git (Optional but Recommended)

Open a terminal/command prompt in the `francis-quiz-app` folder:

```bash
git init
git branch -m main
git config user.name "Your Name"
git config user.email "your.email@example.com"
git add .
git commit -m "Initial commit: Project setup with question data"
```

### 5. Push to GitHub

Follow the instructions in `GITHUB_SETUP.md` to:
1. Create a new repository on GitHub
2. Push your local project to GitHub
3. Share the repository with Claude Code

### 6. Hand Off to Claude Code

In Claude Code (in the project directory):

```
"Hi! I have a quiz app project ready to build. All the question 
data is in the /data folder (228 questions across 18 chapters). 
Please read BUILD_INSTRUCTIONS.md for complete specifications. 
Your task is to create /public/index.html with the quiz application."
```

## Troubleshooting

**Q: I can't see .gitignore after renaming**
A: That's normal on Mac/Linux - files starting with `.` are hidden. It's there!
   On Windows, you may need to enable "Show hidden files" in File Explorer.

**Q: How do I verify all files are in the right place?**
A: Check that you have:
   - 4 files in the root folder (including .gitignore)
   - 18 JSON files in the data/ folder
   - public/ and src/ folders exist (empty is fine)

**Q: Do I need to install anything?**
A: Not yet! Once Claude Code builds the app, you'll just need a web browser.
   For GitHub, you'll need Git installed.

## What's Next?

1. ✅ Set up the folder structure (you're doing this now!)
2. ⏳ Initialize Git and push to GitHub
3. ⏳ Have Claude Code build the app
4. ⏳ Deploy to GitHub Pages
5. 🎉 Start studying!

---

**Need help?** Read the other documentation files:
- **README.md**: Project overview and features
- **BUILD_INSTRUCTIONS.md**: Complete technical spec (for Claude Code)
- **GITHUB_SETUP.md**: Detailed GitHub setup steps
