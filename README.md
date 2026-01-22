# St. Francis Quiz App

A web-based quiz application for studying "St. Francis of Assisi: Passion, Poverty and the Man who Transformed the Catholic Church" for Academic Decathlon 2026.

## Features

- **Multiple Choice Questions**: 228 questions across 18 chapters
- **Dynamic Distractors**: Each question has 5-6 wrong answers; app randomly selects 3 each time
- **Quiz Modes**: 
  - Practice by chapter
  - Mix questions from all chapters
  - Random selection (10, 25, 50 questions)
- **Immediate Feedback**: See correct/incorrect answers right away
- **Results Tracking**: View your score and review missed questions
- **Mobile Responsive**: Works on laptop and phone

## Project Structure

```
francis-quiz-app/
├── data/                    # Question data (18 JSON files)
│   ├── st-francis-ch01.json
│   ├── st-francis-ch02.json
│   └── ...
├── public/                  # Static assets
│   └── index.html
├── src/                     # Source code
│   ├── App.jsx             # Main React component
│   ├── components/         # React components
│   └── utils/              # Helper functions
├── README.md
└── package.json
```

## Technology Stack

- **React** (via CDN - no build process needed)
- **Vanilla JavaScript** for quiz logic
- **CSS** for styling
- **JSON** for question data

## Data Format

Each chapter file (`st-francis-ch01.json`) contains:
- Chapter number and name
- Array of questions with:
  - Unique ID
  - Question text
  - Correct answer
  - 5-6 distractors (wrong answers)
  - Page reference from the book

## Getting Started

See BUILD_INSTRUCTIONS.md for development setup with Claude Code.

## Deployment

The app can be deployed to:
- GitHub Pages (free, easy)
- Netlify (free, drag-and-drop)
- Vercel (free, Git integration)
- Local file system (just open index.html)

## Future Enhancements

- Add other Academic Decathlon subjects
- Progress tracking over time
- "Weak areas" identification
- Study mode vs. test mode toggle
- "All of the above" / "A and C" style questions

## License

Educational use only - Academic Decathlon 2026
