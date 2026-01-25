# Implementation Guide for Vocabulary & "Accept Any" Improvements

## REVISED VERSION - WITH FIREBASE TRACKING

## Overview

This document provides complete instructions for Claude Code to implement two improvements:

1. **Vocabulary Quiz Mode** - Test vocabulary terms separately from review questions (with Firebase tracking)
2. **"Accept Any" Answer Rotation** - For questions with multiple acceptable answers, rotate which one is "correct"

---

## Part 1: Vocabulary Implementation

### Data Structure

**Vocabulary files are already created** in separate JSON files.

**Location:** `/docs/data/vocab/`

- `vocab-ch01.json` through `vocab-ch18.json`

**Format:**

```json
{
  "chapterNumber": 1,
  "chapterName": "Nativity",
  "vocabulary": [
    {
      "id": "ch1_vocab1",
      "term": "expectant",
      "definition": "expecting the birth of a child",
      "distractors": [
        "practicing strict self-denial",
        "a person who repents of sin",
        "one who conveys news",
        "impressive because of grandeur",
        "a prediction of something to come"
      ],
      "pageReference": "1"
    }
  ]
}
```

**Distractor Strategy:**

- Distractors are definitions from OTHER vocab terms in the same chapter
- Makes it harder - student must know the specific term
- Ensures distractors are plausible

### App Changes

**1. Add Vocabulary Mode to Quiz Selection:**

```
Start Screen:
━━━━━━━━━━━━━━━━━━━━━━━━
Regular Questions:
- Select Chapter (existing)
- All Chapters (existing)
- Random Questions (existing)

Vocabulary:                ← NEW SECTION
- Vocabulary - By Chapter
- Vocabulary - All Chapters
━━━━━━━━━━━━━━━━━━━━━━━━
```

**2. Vocabulary Question Display:**

- Show: **Definition** (as the question)
- Ask: "Which term matches this definition?"
- 4 choices: 1 correct term + 3 distractor terms
- Randomize order

**Example:**

```
Vocabulary Quiz - Chapter 1
Question 3 of 11

Which term matches this definition?
"expecting the birth of a child"

○ A) herald
○ B) expectant  ← correct
○ C) penitent
○ D) prophecy

[Submit Answer]
```

**3. Loading:**

- Load vocab files same way as question files
- `fetch('./data/vocab/vocab-ch01.json')`
- Use same loading pattern as existing quiz questions

**4. Scoring:**

- Same scoring system as regular questions
- Show correct term if answer is wrong
- Track which terms were missed

---

## Part 2: "Accept Any" Answer Rotation

### Problem

Some questions have multiple acceptable answers listed as one long string separated by semicolons.

**Example (current format):**

```json
{
  "correctAnswer": "his heart came alive with a burning, consuming love; the sense of separation from God and others began to leave him; he began to feel a oneness with God..."
}
```

### Solution: Split and Rotate

**1. Questions That Need Converting:**

The parent will manually convert these questions to the new format:

| Chapter | Question ID | Description                                        |
| ------- | ----------- | -------------------------------------------------- |
| Ch 3    | ch3_q9      | Why turning back was distressing                   |
| Ch 4    | ch4_q10     | How Francis was transformed (7 acceptable answers) |
| Ch 8    | ch8_q20     | Frederico Spadalunga question                      |
| Ch 10   | ch10_q4     | Scripture passages (3 passages)                    |
| Ch 14   | ch14_q5     | Pilgrimage symbols                                 |
| Ch 15   | ch15_q18    | What sultan realized about Francis                 |
| Ch 17   | ch17_q4     | Ways health was failing                            |
| Ch 17   | ch17_q8     | Seeing crosses in life                             |
| Ch 18   | ch18_q10    | Final admonition                                   |

**2. New Data Structure:**

Parent will convert these to:

```json
{
  "id": "ch4_q10",
  "question": "Through Francis' encounter with God in the crucifix at San Damiano how was he being transformed?",
  "acceptableAnswers": [
    "His heart came alive with a burning, consuming love",
    "The sense of separation from God and others began to leave him",
    "He began to feel a oneness with God",
    "The sense of inner shame and sinfulness began to leave him",
    "His awareness of sin caused him to place all his faith and reliance on God",
    "He was filling and reforming his sinful soul with God's grace",
    "He began to experience grace and peace"
  ],
  "distractors": [
    "He became more disciplined in following religious rules",
    "He developed a fear of sin and punishment",
    "He gained knowledge of theology and scripture",
    "He became determined to prove his worthiness to God",
    "He learned to control his emotions through willpower",
    "He started attending more church services"
  ],
  "pageReference": "38-39",
  "type": "accept-any"
}
```

**3. Quiz Logic:**

When encountering a question with `"type": "accept-any"`:

```javascript
function prepareQuestion(question) {
  if (question.type === "accept-any") {
    // Pick ONE random acceptable answer as the "correct" one for this attempt
    const correctAnswer = randomChoice(question.acceptableAnswers);

    // Pick 3 random distractors
    const wrongAnswers = randomChoices(question.distractors, 3);

    // Combine and shuffle all 4 choices
    const allChoices = shuffle([correctAnswer, ...wrongAnswers]);

    // Return prepared question
    return {
      id: question.id,
      question: question.question,
      choices: allChoices,
      correctAnswer: correctAnswer, // This specific attempt's correct answer
      pageReference: question.pageReference,
      type: "accept-any",
    };
  } else {
    // Handle normal questions (existing logic)
    // ...
  }
}
```

**Result:**

- Attempt 1: Correct answer might be "His heart came alive with burning love"
- Attempt 2: Correct answer might be "He began to feel oneness with God"
- Different each time while still being correct!

---

## Part 3: Firebase Integration (CRITICAL!)

### Overview

Both regular questions AND vocabulary quizzes must log to Firebase. The parent dashboard must show both types of attempts.

### Data Structure for Firebase

**Collection:** `quizAttempts`

**Document structure for regular questions:**

```json
{
  "studentName": "Alex",
  "timestamp": "2026-01-23T14:30:00Z",
  "quizMode": "Chapter 5", // or "All Chapters", "Random 25"
  "score": 8,
  "total": 10,
  "percentage": 80,
  "timeSpent": "5 min 23 sec",
  "attemptType": "questions", // ← NEW FIELD
  "missedQuestions": [
    {
      "questionId": "ch5_q3",
      "question": "What did Medieval people believe...",
      "studentAnswer": "They were contagious",
      "correctAnswer": "They deserved it because of sin",
      "pageReference": "48"
    }
  ]
}
```

**Document structure for vocabulary:**

```json
{
  "studentName": "Alex",
  "timestamp": "2026-01-23T15:45:00Z",
  "quizMode": "Vocabulary - Chapter 5", // or "Vocabulary - All Chapters"
  "score": 10,
  "total": 12,
  "percentage": 83,
  "timeSpent": "3 min 45 sec",
  "attemptType": "vocabulary", // ← NEW FIELD
  "missedTerms": [
    // ← Different field name for vocab
    {
      "termId": "ch5_vocab8",
      "term": "solidarity",
      "definition": "a moral virtue that calls us to recognize that all people are part of one human family",
      "studentAnswer": "marginalized",
      "correctAnswer": "solidarity",
      "pageReference": "54"
    }
  ]
}
```

### Implementation Requirements

**1. Track Both Quiz Types:**

- Regular questions: `attemptType: "questions"`, use `missedQuestions` field
- Vocabulary: `attemptType: "vocabulary"`, use `missedTerms` field

**2. Consistent Logging:**

```javascript
async function logQuizAttempt(attemptData) {
  try {
    await firebase
      .firestore()
      .collection("quizAttempts")
      .add({
        studentName: attemptData.studentName,
        timestamp: new Date().toISOString(),
        quizMode: attemptData.quizMode,
        score: attemptData.score,
        total: attemptData.total,
        percentage: Math.round((attemptData.score / attemptData.total) * 100),
        timeSpent: attemptData.timeSpent,
        attemptType: attemptData.attemptType, // "questions" or "vocabulary"
        [attemptData.attemptType === "vocabulary"
          ? "missedTerms"
          : "missedQuestions"]: attemptData.missedItems,
      });
    console.log("Quiz attempt logged successfully");
  } catch (error) {
    console.error("Error logging quiz attempt:", error);
  }
}
```

**3. Quiz Mode Naming Convention:**

- Regular questions: `"Chapter 5"`, `"All Chapters"`, `"Random 25"`
- Vocabulary: `"Vocabulary - Chapter 5"`, `"Vocabulary - All Chapters"`

This makes it easy to filter in the dashboard.

---

## Part 4: Parent Dashboard Updates

### Requirements

**1. Show All Attempts Together:**
The dashboard must display both regular question attempts and vocabulary attempts in the timeline.

**Example timeline:**

```
Recent Activity:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Jan 23, 3:45 PM - Vocabulary - Chapter 5
Score: 10/12 (83%)
Time: 3 min 45 sec

Jan 23, 2:30 PM - Chapter 5
Score: 8/10 (80%)
Time: 5 min 23 sec

Jan 22, 4:15 PM - All Chapters
Score: 42/50 (84%)
Time: 18 min 12 sec

Jan 22, 3:00 PM - Vocabulary - All Chapters
Score: 155/193 (80%)
Time: 35 min 8 sec
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**2. Filtering Options:**

Add filter buttons/dropdown to dashboard:

```
View: [All Attempts ▼]
      - All Attempts
      - Questions Only
      - Vocabulary Only
```

**3. Separate Statistics:**

Show overall stats broken down by type:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Overall Statistics:

Regular Questions:
  Total Quizzes: 23
  Average Score: 82%
  Most Recent: Chapter 5 (80%)

Vocabulary:
  Total Quizzes: 15
  Average Score: 78%
  Most Recent: Chapter 5 (83%)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**4. Chapter Breakdown:**

For each chapter, show both:

```
Chapter 5: Leprosy and Minority
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Questions:
  Attempts: 3
  Average: 85%
  Last: 80%

Vocabulary:
  Attempts: 2
  Average: 81%
  Last: 83%
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**5. Weak Areas:**

Show frequently missed items from both types:

```
Frequently Missed:

Questions:
  • Ch5 Q3: Medieval beliefs about leprosy (missed 2/3 times)
  • Ch3 Q8: Where the voice told Francis to go (missed 2/2 times)

Vocabulary:
  • solidarity (Ch5) - missed 3/4 times
  • metanoia (Ch6) - missed 2/3 times
```

### Dashboard Implementation Details

**Fetching Data:**

```javascript
async function fetchAllAttempts() {
  const snapshot = await firebase
    .firestore()
    .collection("quizAttempts")
    .where("studentName", "==", currentStudent)
    .orderBy("timestamp", "desc")
    .get();

  return snapshot.docs.map((doc) => ({
    id: doc.id,
    ...doc.data(),
  }));
}

async function fetchAttemptsByType(type) {
  const snapshot = await firebase
    .firestore()
    .collection("quizAttempts")
    .where("studentName", "==", currentStudent)
    .where("attemptType", "==", type) // "questions" or "vocabulary"
    .orderBy("timestamp", "desc")
    .get();

  return snapshot.docs.map((doc) => ({
    id: doc.id,
    ...doc.data(),
  }));
}
```

**Calculating Stats:**

```javascript
function calculateStats(attempts, type) {
  const filteredAttempts = type
    ? attempts.filter((a) => a.attemptType === type)
    : attempts;

  return {
    totalQuizzes: filteredAttempts.length,
    averageScore: Math.round(
      filteredAttempts.reduce((sum, a) => sum + a.percentage, 0) /
        filteredAttempts.length,
    ),
    mostRecent: filteredAttempts[0], // Already sorted by timestamp desc
  };
}
```

---

## Implementation Steps for Claude Code

### Step 1: Vocabulary Quiz Mode

**Files to modify:** `/docs/index.html`

1. **Add vocabulary mode options to start screen**

   - Add "Vocabulary" section with chapter selector
   - Add "Vocabulary - All Chapters" option

2. **Add vocab file loading**

   - Load from `./data/vocab/vocab-chXX.json`
   - Same loading pattern as question files

3. **Add vocab question display logic**

   - Show definition as question text
   - Display terms as answer choices (1 correct + 3 distractors)
   - Randomize order

4. **Add Firebase logging for vocab**
   - Log with `attemptType: "vocabulary"`
   - Use `missedTerms` field instead of `missedQuestions`
   - Quiz mode: `"Vocabulary - Chapter X"` or `"Vocabulary - All Chapters"`

### Step 2: "Accept Any" Question Logic

**Files to modify:** `/docs/index.html`

1. **Add detection for "accept any" questions**

   - Check if question has `type: "accept-any"` field

2. **Implement rotation logic**

   - Pick ONE random answer from `acceptableAnswers` array
   - Pick 3 random distractors
   - Combine and shuffle
   - Display as normal multiple choice

3. **Handle feedback**
   - Show which answer was correct THIS attempt
   - Could optionally show "Other acceptable answers" in feedback

### Step 3: Update Parent Dashboard

**Files to modify:** `/docs/dashboard.html` (or wherever dashboard code is)

1. **Update data fetching**

   - Fetch all attempts (both types)
   - Add filtering by `attemptType`

2. **Update timeline display**

   - Show both question and vocab attempts
   - Visual distinction (icon, color, label)

3. **Add filter dropdown**

   - All Attempts
   - Questions Only
   - Vocabulary Only

4. **Update statistics calculations**

   - Overall stats by type
   - Chapter breakdown by type
   - Weak areas from both types

5. **Update visualizations**
   - Line graph: show both types (different colors/lines)
   - Bar chart by chapter: side-by-side bars for questions vs vocab

### Step 4: Testing

**Test vocabulary mode:**

- [ ] Vocab quiz option appears
- [ ] All 18 chapters load
- [ ] Definitions display as questions
- [ ] Terms are answer choices
- [ ] Scoring works
- [ ] Firebase logging works
- [ ] Parent dashboard shows vocab attempts

**Test "accept any" questions:**

- [ ] Questions with multiple answers work
- [ ] Different correct answers on different attempts
- [ ] Distractors always wrong
- [ ] Scoring accurate
- [ ] Firebase logs correctly

**Test parent dashboard:**

- [ ] Both attempt types display
- [ ] Filtering works
- [ ] Statistics accurate for both types
- [ ] Chapter breakdown shows both
- [ ] Weak areas identify vocab and questions separately

---

## Questions to Convert - Detailed List

Parent must manually update these questions in the JSON files:

### Chapter 3 (st-francis-ch03.json)

**Question ID:** `ch3_q9`
**Current correctAnswer:**

```
"If he turned back and quit, everyone would judge him as a coward. It would bring shame to his family; he was concerned his father would disinherit him"
```

**Convert to:**

```json
"acceptableAnswers": [
  "Everyone would judge him as a coward",
  "It would bring shame to his family",
  "He was concerned his father would disinherit him"
],
"distractors": [
  "He would lose his way back to Assisi",
  "The other soldiers would attack him",
  "He had no money for the journey home",
  "God would be angry with him",
  "He would disappoint the Pope"
],
"type": "accept-any"
```

### Chapter 4 (st-francis-ch04.json)

**Question ID:** `ch4_q10`
**Current correctAnswer:** (7 acceptable answers separated by semicolons)
**Convert to:**

```json
"acceptableAnswers": [
  "His heart came alive with a burning, consuming love",
  "The sense of separation from God and others began to leave him",
  "He began to feel a oneness with God",
  "The sense of inner shame and sinfulness began to leave him",
  "His awareness of sin caused him to place all his faith and reliance on God",
  "He was filling and reforming his sinful soul with God's grace",
  "He began to experience grace and peace"
],
"distractors": [
  "He became more disciplined in following religious rules",
  "He developed a fear of sin and punishment",
  "He gained knowledge of theology and scripture",
  "He became determined to prove his worthiness to God",
  "He learned to control his emotions through willpower",
  "He started attending more church services regularly"
],
"type": "accept-any"
```

### Chapter 8 (st-francis-ch08.json)

**Question ID:** `ch8_q20`
**Current correctAnswer:**

```
"Frederico Spadalunga; they fought together in the battle against Perugia"
```

**Note:** This one has a semicolon but it's actually two parts of one answer (name + how they met). This might not need "accept-any" treatment - it's fine as-is. Skip this one unless it seems problematic.

### Chapter 10 (st-francis-ch10.json)

**Question ID:** `ch10_q4`
**Current correctAnswer:**

```
"Matthew 19:21 'If you would be perfect, go, sell what you possess and give to the poor...'; Luke 9:3 'Take nothing for your journey'; Luke 9:23 'If any man would come after me, let him deny himself and take up his cross daily and follow me'"
```

**Convert to:**

```json
"acceptableAnswers": [
  "Matthew 19:21 - If you would be perfect, go, sell what you possess and give to the poor",
  "Luke 9:3 - Take nothing for your journey",
  "Luke 9:23 - If any man would come after me, let him deny himself and take up his cross daily and follow me"
],
"distractors": [
  "John 3:16 - For God so loved the world",
  "Matthew 5:3 - Blessed are the poor in spirit",
  "Luke 10:27 - Love the Lord your God with all your heart",
  "Mark 16:15 - Go into all the world and preach the gospel",
  "Matthew 6:33 - Seek first the kingdom of God"
],
"type": "accept-any"
```

### Chapter 14 (st-francis-ch14.json)

**Question ID:** `ch14_q5`
**Current correctAnswer:**

```
"To Compostela: scalloped shell; to Rome: keys; to Holy Land: Jerusalem cross"
```

**Convert to:**

```json
"acceptableAnswers": [
  "To Compostela: scalloped shell",
  "To Rome: keys",
  "To Holy Land: Jerusalem cross"
],
"distractors": [
  "To Compostela: palm branch",
  "To Rome: fish symbol",
  "To Holy Land: star of David",
  "To Compostela: red cross",
  "To Rome: papal seal"
],
"type": "accept-any"
```

### Chapter 15 (st-francis-ch15.json)

**Question ID:** `ch15_q18`
**Current correctAnswer:**

```
"He was different from others; he came with humility and poverty and not the sword, he came as a brother not as an enemy"
```

**Convert to:**

```json
"acceptableAnswers": [
  "He was different from others who came with the sword",
  "He came with humility and poverty",
  "He came as a brother not as an enemy"
],
"distractors": [
  "He was a great scholar of theology",
  "He had miraculous powers from God",
  "He was sent by the Pope to negotiate",
  "He was willing to convert to Islam",
  "He spoke Arabic fluently"
],
"type": "accept-any"
```

### Chapter 17 (st-francis-ch17.json)

**Question ID:** `ch17_q4`
**Current correctAnswer:**

```
"He was losing his eyesight; he was in pain and couldn't walk the whole way when he traveled"
```

**Convert to:**

```json
"acceptableAnswers": [
  "He was losing his eyesight",
  "He was in pain and couldn't walk the whole way when he traveled"
],
"distractors": [
  "He had a persistent cough and fever",
  "He could no longer speak clearly",
  "His hands were crippled with arthritis",
  "He suffered from constant headaches",
  "He had stomach problems and couldn't eat"
],
"type": "accept-any"
```

**Question ID:** `ch17_q8`
**Current correctAnswer:**

```
"They were signs of God's glory; he had learned how to internalize the cross by embracing sufferings and weaknesses"
```

**Convert to:**

```json
"acceptableAnswers": [
  "They were signs of God's glory",
  "He had learned how to internalize the cross by embracing sufferings and weaknesses"
],
"distractors": [
  "They were punishments for his sins",
  "They were tests of his faithfulness",
  "They were attacks from the devil",
  "They were necessary for salvation",
  "They were temporary and would pass"
],
"type": "accept-any"
```

### Chapter 18 (st-francis-ch18.json)

**Question ID:** `ch18_q10`
**Current correctAnswer:**

```
"I have done what is mine; may Christ teach you what is yours"
```

**Note:** This one has a semicolon but it's actually one complete quote. Probably doesn't need "accept-any" treatment. Skip this one.

---

## Summary of Questions to Convert

**Definitely convert (6 questions):**

1. Chapter 3, Question 9 (ch3_q9)
2. Chapter 4, Question 10 (ch4_q10) - MOST IMPORTANT (7 answers)
3. Chapter 10, Question 4 (ch10_q4)
4. Chapter 14, Question 5 (ch14_q5)
5. Chapter 15, Question 18 (ch15_q18)
6. Chapter 17, Question 4 (ch17_q4)
7. Chapter 17, Question 8 (ch17_q8)

**Probably skip (2 questions - semicolons but not multiple acceptable answers):**

- Chapter 8, Question 20 (ch8_q20)
- Chapter 18, Question 10 (ch18_q10)

---

## File Structure After Implementation

```
docs/
├── index.html (updated with vocab mode + accept-any logic + Firebase)
├── dashboard.html (updated to show both attempt types)
└── data/
    ├── st-francis-ch01.json
    ├── st-francis-ch02.json
    ├── st-francis-ch03.json (ch3_q9 updated)
    ├── st-francis-ch04.json (ch4_q10 updated)
    ├── ... (more chapters)
    ├── st-francis-ch10.json (ch10_q4 updated)
    ├── st-francis-ch14.json (ch14_q5 updated)
    ├── st-francis-ch15.json (ch15_q18 updated)
    ├── st-francis-ch17.json (ch17_q4, ch17_q8 updated)
    ├── st-francis-ch18.json
    └── vocab/
        ├── vocab-ch01.json
        ├── vocab-ch02.json
        └── ... (18 vocab files)
```

---

## Estimated Effort

- **Parent's manual JSON updates**: 30-45 minutes (7 questions to convert)
- **Claude Code - Vocabulary mode**: 2 hours (loading, display, Firebase)
- **Claude Code - Accept-any logic**: 1 hour (detection, rotation)
- **Claude Code - Dashboard updates**: 1.5 hours (filtering, stats, display)
- **Testing**: 30 minutes
- **Total**: ~5-6 hours

---

## Firebase Security Note

Make sure Firestore rules allow:

- Writing new attempts (anyone can create)
- Reading attempts (for dashboard)

The rules should already be set from the initial Firebase setup, but verify they handle both `attemptType` values correctly.

---

## Ready to Implement!

Everything is specified:

- ✅ Vocabulary data ready (193 terms)
- ✅ Exact questions to convert listed with examples
- ✅ Firebase structure defined
- ✅ Dashboard requirements specified
- ✅ Step-by-step implementation plan

Parent: Convert the 7 questions listed above
Claude Code: Follow this guide for implementation

Good luck! 🚀
