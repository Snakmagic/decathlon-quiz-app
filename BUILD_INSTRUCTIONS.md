# BUILD INSTRUCTIONS FOR CLAUDE CODE

## Project Overview
Build a web-based quiz application for studying St. Francis of Assisi (Academic Decathlon 2026). The app uses React (via CDN) - no build process needed, just a single HTML file that can be opened in a browser or deployed online.

## What Already Exists
✅ **Data files**: 18 chapter JSON files in `/data/` folder (228 total questions)
✅ **Project structure**: Folders set up and ready
✅ **Design approach**: Documented below

## Your Task
Build the quiz application following these specifications.

---

## Technical Approach

### Architecture Decision: Single HTML File with React via CDN
- Use React loaded from CDN (no build process, no npm install needed)
- Everything in one `public/index.html` file
- Inline CSS in `<style>` tags
- Inline JavaScript/React in `<script type="text/babel">` tags
- Load Babel standalone for JSX compilation

### Why This Approach?
- ✅ No build process - just open in browser
- ✅ Easy to deploy anywhere
- ✅ Still get React's benefits (state management, components)
- ✅ Simple for user to understand and modify

---

## Core Functionality Requirements

### 1. Data Loading
**Load all 18 chapter files on app startup:**
```javascript
// Load all chapters from /data/ folder
const chapters = [];
for (let i = 1; i <= 18; i++) {
  const response = await fetch(`./data/st-francis-ch${i.toString().padStart(2, '0')}.json`);
  const chapter = await response.json();
  chapters.push(chapter);
}
```

### 2. Quiz Modes
User can select:
- **By Chapter**: Practice one specific chapter
- **All Chapters**: Mix questions from entire book
- **Random**: Choose 10, 25, or 50 questions randomly

### 3. Question Display Logic
**Critical: Randomize distractors each time**
```javascript
// For each question:
// 1. Randomly select 3 distractors from the 5-6 available
// 2. Combine with correct answer (4 choices total)
// 3. Shuffle the 4 choices so correct answer isn't always in same position
// 4. Display to user
```

### 4. User Flow
```
Start Screen
  ↓
Select Quiz Mode (Chapter / All / Random)
  ↓
Quiz Screen
  - Show question counter (5/50)
  - Display question
  - Show 4 answer choices (A, B, C, D)
  - Submit button
  ↓
After answering:
  - Show if correct/incorrect
  - If wrong, show correct answer
  - "Next Question" button
  ↓
Results Screen
  - Score: 42/50 (84%)
  - List of missed questions
  - Options:
    * Review missed questions
    * Try again
    * Return to menu
```

### 5. State Management
Use React state to track:
```javascript
const [currentMode, setCurrentMode] = useState('menu'); // 'menu', 'quiz', 'results'
const [questions, setQuestions] = useState([]); // Current quiz questions
const [currentQuestionIndex, setCurrentQuestionIndex] = useState(0);
const [selectedAnswer, setSelectedAnswer] = useState(null);
const [userAnswers, setUserAnswers] = useState([]); // Track all answers
const [showFeedback, setShowFeedback] = useState(false);
```

---

## Design & Styling Requirements

### Visual Design
- **Clean, minimal interface**
- **Easy to read on laptop and phone**
- **Color scheme**: 
  - Primary: Deep blue (#2c5282)
  - Accent: Gold/amber (#d97706) 
  - Success: Green (#059669)
  - Error: Red (#dc2626)
  - Background: Light gray (#f9fafb)

### Component Layout

**Start Screen:**
```
┌─────────────────────────────────┐
│   St. Francis Quiz              │
│   Academic Decathlon 2026       │
│                                 │
│   [Select Chapter ▼]            │
│   [All Chapters]                │
│   [Random 10 Questions]         │
│   [Random 25 Questions]         │
│   [Random 50 Questions]         │
└─────────────────────────────────┘
```

**Quiz Screen:**
```
┌─────────────────────────────────┐
│ Question 5 of 50                │
│ Chapter 3: From War to Spoleto  │
│                                 │
│ Where did the voice tell        │
│ Francis to go?                  │
│                                 │
│ ○ A) To Rome to see the Pope   │
│ ○ B) Back home                  │
│ ○ C) To Jerusalem               │
│ ○ D) To a monastery             │
│                                 │
│          [Submit Answer]        │
└─────────────────────────────────┘
```

**After Answer (Correct):**
```
┌─────────────────────────────────┐
│ ✓ Correct!                      │
│                                 │
│ The answer is: Back home        │
│ (Page reference: 27)            │
│                                 │
│          [Next Question]        │
└─────────────────────────────────┘
```

**Results Screen:**
```
┌─────────────────────────────────┐
│ Quiz Complete!                  │
│                                 │
│ Your Score: 42/50 (84%)         │
│                                 │
│ Questions Missed: 8             │
│ • Question 3 (Chapter 1)        │
│ • Question 12 (Chapter 2)       │
│ ...                             │
│                                 │
│ [Review Missed Questions]       │
│ [Try Again]                     │
│ [Return to Menu]                │
└─────────────────────────────────┘
```

### Responsive Design
- **Desktop (>768px)**: Max width 800px, centered
- **Mobile (<768px)**: Full width with padding
- **Touch-friendly**: Buttons at least 44px height

---

## Implementation Guide

### Step 1: Create the HTML Shell
Create `/public/index.html` with:
- React, ReactDOM, Babel from CDN
- Basic meta tags for mobile
- Container div with id="root"

### Step 2: Build Core Components
```javascript
// Main App component
function App() {
  // State management
  // Component rendering
}

// StartScreen component
function StartScreen({ onStartQuiz }) {
  // Quiz mode selection
}

// QuizScreen component  
function QuizScreen({ questions, onComplete }) {
  // Question display
  // Answer selection
  // Progress tracking
}

// ResultsScreen component
function ResultsScreen({ score, totalQuestions, missedQuestions, onRestart }) {
  // Score display
  // Missed questions list
  // Action buttons
}
```

### Step 3: Implement Quiz Logic
```javascript
// Utility functions needed:
function loadAllChapters() { /* Fetch all JSON files */ }
function selectQuestions(mode, chapterNumber, count) { /* Return question array */ }
function shuffleArray(array) { /* Fisher-Yates shuffle */ }
function selectRandomDistractors(distractors, count = 3) { /* Pick 3 random */ }
function prepareQuestion(question) { 
  /* Combine correct answer + 3 random distractors, shuffle */
}
```

### Step 4: Add Styling
Inline CSS with:
- Flexbox for layouts
- Mobile-first responsive design
- Button hover states
- Smooth transitions
- Accessible color contrast

### Step 5: Test & Refine
- Test on laptop browser
- Test on mobile (or use browser dev tools mobile view)
- Verify all 18 chapters load correctly
- Test all quiz modes
- Verify randomization works

---

## File Structure to Create

```
francis-quiz-app/
├── public/
│   └── index.html          ← CREATE THIS (entire app in one file)
└── data/                   ← ALREADY EXISTS
    ├── st-francis-ch01.json
    ├── st-francis-ch02.json
    └── ... (all 18 chapters)
```

---

## Testing Checklist

After building, verify:
- ✅ App loads without errors
- ✅ All 18 chapters load successfully
- ✅ Each quiz mode works (Chapter, All, Random)
- ✅ Questions display correctly
- ✅ Answers are randomized (not always in same position)
- ✅ Distractors change each time (check by retaking same chapter)
- ✅ Scoring works correctly
- ✅ Results screen shows missed questions
- ✅ Can navigate back to menu and start new quiz
- ✅ Responsive on mobile viewport
- ✅ No console errors

---

## Deployment Instructions (for later)

### Option 1: GitHub Pages
```bash
# In the repository:
git add .
git commit -m "Add quiz app"
git push origin main

# Enable GitHub Pages:
# Settings > Pages > Source: main branch > /public folder
# Your app will be at: https://username.github.io/francis-quiz-app/
```

### Option 2: Local Use
Simply open `/public/index.html` in a web browser. That's it!

---

## Important Notes

1. **Keep it simple**: One HTML file, no external dependencies except CDN React
2. **Focus on functionality**: Get it working before making it pretty
3. **Test the randomization**: This is the key feature - verify distractors change
4. **Mobile matters**: User will primarily use on laptop but also phone
5. **Error handling**: Add try/catch for file loading, show friendly error messages

---

## Questions to Answer During Development

If you encounter these scenarios:

**Q: Should I use a package.json and npm?**
A: No. This is a CDN-based React app, no build process needed.

**Q: Should I split into multiple JS files?**
A: No. Keep everything in one HTML file for simplicity.

**Q: What about browser compatibility?**
A: Modern browsers only (Chrome, Firefox, Safari, Edge). No IE11 support needed.

**Q: How should I handle data loading errors?**
A: Show friendly error message: "Could not load questions. Please check that all chapter files are in the /data folder."

---

## Success Criteria

The app is complete when:
1. ✅ A user can open index.html in a browser and see the start screen
2. ✅ They can select any quiz mode and take a quiz
3. ✅ Questions display with 4 randomized choices
4. ✅ They get immediate feedback on correct/incorrect
5. ✅ They see their final score and missed questions
6. ✅ They can retake the quiz with different distractor combinations
7. ✅ It works on both desktop and mobile

---

## Ready to Build!

You have:
- ✅ All 228 questions in JSON format
- ✅ Clear technical specification
- ✅ Design mockups
- ✅ Implementation guide
- ✅ Testing checklist

**Your task**: Create `/public/index.html` with the complete working quiz application.

Good luck! 🚀
