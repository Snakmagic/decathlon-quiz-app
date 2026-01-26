# Vocabulary Quiz Enhancement - Adjacent Chapter Distractors

## Problem Statement

Currently, vocabulary quizzes pull all distractors from the same chapter being studied. This creates a "process of elimination" problem:

**Example - Chapter 5 has 12 terms:**

- Question 1: Student sees 4 terms out of 12 possible (8 remain unseen)
- Question 2: Student sees 4 terms (4-5 remain unseen)
- Question 10: Student sees 4 terms (only 2 remain unseen)
- Question 11-12: Answers become obvious by elimination

**Result:** The last few questions are too easy because there aren't enough unseen terms left.

## Solution

Pull distractors from **adjacent chapters** in addition to the current chapter. This maintains a large pool of unseen terms throughout the entire quiz, eliminating the process-of-elimination advantage.

---

## Technical Implementation

### Current Logic (Simplified)

```javascript
// CURRENT: Only uses current chapter
function prepareVocabQuestion(chapterNum, vocabItem) {
  const chapterVocab = allVocab[chapterNum];

  // Get 3 distractors from same chapter
  const distractors = chapterVocab
    .filter((v) => v.term !== vocabItem.term)
    .map((v) => v.term)
    .shuffle()
    .slice(0, 3);

  const choices = [vocabItem.term, ...distractors].shuffle();

  return {
    question: vocabItem.definition,
    choices,
    correctAnswer: vocabItem.term,
  };
}
```

### New Logic (With Adjacent Chapters)

```javascript
// NEW: Uses current + adjacent chapters
function prepareVocabQuestion(chapterNum, vocabItem) {
  // Determine which chapters to pull from
  const chaptersToUse = getAdjacentChapters(chapterNum);

  // Get all vocab from these chapters
  const availableVocab = chaptersToUse.flatMap((ch) => allVocab[ch]);

  // Remove the correct answer from the pool
  const distractorPool = availableVocab
    .filter((v) => v.term !== vocabItem.term)
    .map((v) => v.term);

  // Weighted selection: prefer current chapter
  const distractors = selectWeightedDistractors(distractorPool, chapterNum, 3);

  const choices = [vocabItem.term, ...distractors].shuffle();

  return {
    question: vocabItem.definition,
    choices,
    correctAnswer: vocabItem.term,
  };
}

function getAdjacentChapters(chapterNum) {
  // Handle edge cases
  if (chapterNum === 1) {
    return [1, 2, 3]; // Use first 3 chapters
  } else if (chapterNum === 18) {
    return [16, 17, 18]; // Use last 3 chapters
  } else {
    return [chapterNum - 1, chapterNum, chapterNum + 1]; // Normal case
  }
}

function selectWeightedDistractors(pool, currentChapter, count) {
  // Split pool by chapter origin
  const fromCurrent = pool.filter((term) => {
    // Check if term is from current chapter
    return allVocab[currentChapter].some((v) => v.term === term);
  });

  const fromAdjacent = pool.filter((term) => {
    // Check if term is NOT from current chapter
    return !allVocab[currentChapter].some((v) => v.term === term);
  });

  // Select with weighting: 2 from current, 1 from adjacent
  const selected = [];

  // Try to get 2 from current chapter
  if (fromCurrent.length >= 2) {
    selected.push(...shuffle(fromCurrent).slice(0, 2));
    // Get 1 from adjacent
    if (fromAdjacent.length >= 1) {
      selected.push(...shuffle(fromAdjacent).slice(0, 1));
    } else {
      // Fallback: get one more from current
      selected.push(
        ...shuffle(fromCurrent.filter((t) => !selected.includes(t))).slice(
          0,
          1,
        ),
      );
    }
  } else {
    // Not enough in current chapter, take what we can
    selected.push(...shuffle(fromCurrent));
    const needed = count - selected.length;
    selected.push(...shuffle(fromAdjacent).slice(0, needed));
  }

  return selected.slice(0, count);
}
```

---

## Weighting Strategy

**Goal:** Still primarily test current chapter, but prevent process of elimination.

**Distractor composition (3 distractors per question):**

- **2 from current chapter** (67%)
- **1 from adjacent chapters** (33%)

**Why this weighting?**

- ✅ Maintains focus on chapter being studied
- ✅ Adjacent terms are contextually similar (same time period in Francis' life)
- ✅ Prevents process of elimination completely
- ✅ Still feels fair - all terms are from the actual book

---

## Edge Cases

### Chapter 1

**Chapters to use:** 1, 2, 3
**Why:** No previous chapter exists, so use next two chapters

### Chapter 18

**Chapters to use:** 16, 17, 18
**Why:** No next chapter exists, so use previous two chapters

### Small Chapters

Some chapters have only 6-8 terms. This is fine:

- Still pull from 3 chapters total (18-24 terms available)
- Plenty of variety even for short quizzes

### "All Chapters" Vocab Mode

If student selects "Vocabulary - All Chapters":

- Pull questions from all 18 chapters
- For each question, still use adjacent chapter logic
- This prevents process of elimination across the entire 193-term quiz

---

## Data Structure (No Changes Needed!)

The current vocabulary JSON files already have everything needed:

```json
{
  "chapterNumber": 5,
  "vocabulary": [
    {
      "id": "ch5_vocab1",
      "term": "leper",
      "definition": "a person affected with leprosy",
      "distractors": [...],  // These are now ignored by the app
      "pageReference": "47"
    }
  ]
}
```

**Important:** The `distractors` field in the JSON files can now be **ignored by the app**. The app will dynamically generate distractors from adjacent chapters instead.

**Note:** Keep the `distractors` field in the JSON for backward compatibility, but the app doesn't need to use them.

---

## Loading Strategy

### Current Loading (Inefficient for New Logic)

```javascript
// Currently loads one chapter at a time
const vocabData = await fetch(`./data/vocab/vocab-ch${chapterNum}.json`);
```

### New Loading (Load All Vocab Upfront)

```javascript
// Load ALL vocabulary files when app starts
async function loadAllVocabulary() {
  const allVocab = {};

  for (let i = 1; i <= 18; i++) {
    const response = await fetch(
      `./data/vocab/vocab-ch${String(i).padStart(2, "0")}.json`,
    );
    const data = await response.json();
    allVocab[i] = data.vocabulary;
  }

  return allVocab;
}

// Call on app initialization
const vocabularyData = await loadAllVocabulary();
```

**Why load all upfront?**

- Need access to adjacent chapters
- Total size: ~193 terms, very lightweight
- One-time load, faster than multiple fetches

---

## User Experience

### Before (Current)

```
Chapter 5 Vocab Quiz (12 terms)

Q1: "a person affected with leprosy"
A) leper ✓    B) repulsed    C) plight    D) podestà

[10 questions later...]

Q11: "exemption from punishment"
A) impunity ✓    B) iniquities    [only 2 unseen terms left!]

Q12: "sin"
A) iniquities ✓   [only 1 term left - too obvious!]
```

### After (With Adjacent Chapters)

```
Chapter 5 Vocab Quiz (12 terms)

Q1: "a person affected with leprosy"
A) leper ✓    B) intuitive (ch3)    C) plight    D) oblate (ch6)

[10 questions later...]

Q11: "exemption from punishment"
A) impunity ✓    B) decorum (ch4)    C) piazza (ch7)    D) solidarity

Q12: "sin"
A) heretics (ch9)    B) iniquities ✓    C) mantle (ch7)    D) disfigured

[No process of elimination possible!]
```

---

## Implementation Checklist

### Code Changes Needed

**File: `/docs/index.html` (or wherever quiz logic lives)**

- [ ] Update vocabulary loading to fetch all 18 chapters upfront
- [ ] Implement `getAdjacentChapters(chapterNum)` function
- [ ] Implement `selectWeightedDistractors()` function
- [ ] Update `prepareVocabQuestion()` to use adjacent chapter pool
- [ ] Test edge cases (Chapter 1, Chapter 18)
- [ ] Test "All Chapters" mode

### Data Changes Needed

- [ ] **NONE!** All JSON files remain unchanged

### Testing

**Test with Chapter 5 (12 terms):**

1. Start vocab quiz for Chapter 5
2. Answer all 12 questions
3. Verify questions 10-12 still have non-obvious choices
4. Verify some distractors are from Chapters 4 and 6 (check term lists)

**Test with Chapter 1 (11 terms):**

1. Start vocab quiz for Chapter 1
2. Verify distractors include terms from Chapters 2 and 3
3. No errors or missing terms

**Test with Chapter 18 (10 terms):**

1. Start vocab quiz for Chapter 18
2. Verify distractors include terms from Chapters 16 and 17
3. No errors or missing terms

**Test "All Chapters" mode:**

1. Start "Vocabulary - All Chapters" quiz
2. Take several questions
3. Verify adjacent chapter logic still applies
4. No process of elimination possible

---

## Performance Impact

**Minimal:**

- Loading all vocab files: ~20-30 KB total (negligible)
- One-time load on app start: adds ~100-200ms
- No ongoing performance impact

---

## Future Enhancements (Optional)

If you want to make it even more sophisticated later:

### Option A: Configurable Weighting

Allow adjustment of current vs. adjacent chapter ratio:

- Current: 2 from current, 1 from adjacent (67/33)
- Harder: 1 from current, 2 from adjacent (33/67)
- Easier: 3 from current, 0 from adjacent (100/0)

### Option B: Thematic Grouping

Group chapters by theme and pull distractors from thematically similar chapters:

- Early life (Ch 1-3)
- Conversion (Ch 4-6)
- Ministry (Ch 7-12)
- Later life (Ch 13-18)

### Option C: "Seen/Unseen" Tracking

Track which terms student has already seen and prefer unseen terms as distractors. This is more complex but maximizes learning.

**For now:** Stick with the simple adjacent chapter approach!

---

## Summary

**What changes:**

- Load all 18 vocab files upfront instead of one at a time
- Pull distractors from current chapter + adjacent chapters
- Weight selection: 2 from current, 1 from adjacent

**What stays the same:**

- JSON file format unchanged
- Firebase logging unchanged
- UI/UX unchanged
- Scoring logic unchanged

**Result:**

- Process of elimination eliminated ✓
- Quiz difficulty more consistent throughout ✓
- Still focuses primarily on chapter being studied ✓
- Contextually appropriate distractors ✓

---

## Estimated Implementation Time

- Code changes: 20-30 minutes
- Testing: 10-15 minutes
- **Total: 30-45 minutes**

---

## Questions?

Let me know if anything is unclear or if you'd like me to adjust the weighting strategy!
