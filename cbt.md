# Detailed Analysis & Breakdown of the CBT Code

This code implements a **fully persistent CBT (Computer-Based Test) system** with:

* Dynamic question rendering
* Navigation (Next / Previous)
* Answer saving
* Score calculation
* Timer countdown
* LocalStorage persistence
* Redirect after submission

---

# 1. Initial Data Storage

```
js
localStorage.setItem("cbtQuestions", JSON.stringify(questions));
```

### What This Does

* Saves the original `questions` array into LocalStorage.
* `JSON.stringify()` converts the JavaScript object into a string because:

  > LocalStorage can only store strings.

### Important

This runs immediately when the script loads.
If `questions` already exists in storage, this may overwrite it.

---

# 2. State Variable

```
js
let currentQuestionIndex = 0;
```

### Purpose

Tracks which question is currently being displayed.

This acts like your **application state pointer**.

---

# 3. DOM Selection

```
js
const questionMeta = document.querySelector('.question-meta');
const questionText = document.querySelector('.question-text');
const optionsList = document.getElementById('exam-options');
```

These allow dynamic manipulation of:

* Question number display
* Question text
* Options container

---

# 4. Loading Saved Exam State

```
js
function loadFromStorage() { 
    const savedData = localStorage.getItem('cbtData'); 
    if (savedData) { 
        const parsed = JSON.parse(savedData); 
        questions = parsed.questions; 
        currentQuestionIndex = parsed.currentQuestionIndex; 
        timeInSeconds = parsed.timeInSeconds; 
    } 
}
```

### What Happens Here

1. Retrieve saved exam state.
2. Convert string back to object using `JSON.parse()`.
3. Restore:

   * Questions (including saved answers)
   * Current question position
   * Remaining timer value

### Why This Is Powerful

If user refreshes the page:

* Exam resumes exactly where it stopped.
* Answers remain selected.
* Timer continues.

---

```
js
loadFromStorage();
displayQuestions();
```

Order matters:

1. Load saved state first.
2. Then display the correct question.

---

# 5. displayQuestions()

This is the rendering engine of your CBT system.

```
js
const current = questions[currentQuestionIndex];
```

Fetch current question object.

---

## Updating Question Meta

```
js
questionMeta.innerHTML = `Question ${currentQuestionIndex + 1} of ${questions.length} <span>1.0 Marks</span>`;
```

* `+1` because arrays start at 0.
* Uses template literals.

---

## Updating Question Text

```
js
questionText.textContent = current.question;
```

Safer than `innerHTML`.

---

## Clearing Previous Options

```
js
optionsList.innerHTML = '';
```

Prevents duplication when navigating.

---

## Rendering Options

```
js
const isChecked = (current.userAnswer === i) ? 'checked' : '';
```

### Ternary Operator Explained

If user previously selected this option:

* Restore checked state.

---

```
js
optionsList.innerHTML += `...`
```

Creates radio inputs dynamically.

Each radio:

* Has same name → allows only one selection
* Value = index number
* ID linked to label

---

## Saving Answer Immediately

```
js
radio.addEventListener('change', function () {
    questions[currentQuestionIndex].userAnswer = parseInt(this.value);
});
```

When user selects:

* Save answer into `questions` array.
* `parseInt()` ensures numeric comparison later.

---

# 6. Navigation System

## Next

```
js
if (currentQuestionIndex < questions.length - 1)
```

Prevents overflow beyond last question.

---

## Previous

```
js
if (currentQuestionIndex > 0)
```

Prevents underflow before first question.

---

## Important

Both navigation buttons call:

```
js
saveAnswer();
```

Before moving.

This ensures:

* Answer is not lost during navigation.

---

# 7. saveAnswer()

```
js
const selectedOption = document.querySelector('input[name="quiz-option"]:checked');
```

Finds currently selected radio button.

If exists:

```
js
questions[currentQuestionIndex].userAnswer = parseInt(selectedOption.value);
saveToStorage();
```

Then immediately persists to LocalStorage.

### This makes your system refresh-safe.

---

# 8. submit()

### Step 1: Save Current Answer

```
js
saveAnswer();
```

Ensures last question is captured.

---

### Step 2: Score Calculation

```
js
for (let i = 0; i < questions.length; i++)
```

Loop through all questions.

---

### Count Answered Questions

```
js
if (questions[i].userAnswer !== undefined)
```

Alternative:

```
js
if ('userAnswer' in questions[i])
```

---

### Check Correct Answer

```
js
if (questions[i].userAnswer === questions[i].correctAnswer)
```

Strict equality ensures correct matching.

---

### Percentage Formula

```
js
const percentage = (score / questions.length) * 100;
```

Basic percentage logic.

---

### Clear Storage After Submission

```
js
localStorage.removeItem('cbtData');
```

Why here?

* Exam is finished.
* No need to restore state.
* Prevents auto-resume on refresh.

---

### Redirect User

```
js
window.location.href = 'login.html';
```

After submission:

* User is sent back to login page.

---

# 9. Timer System

```
js
const countdown = setInterval(() => {
```

Runs every 1000 milliseconds.

---

## Convert Seconds to Minutes

```
js
let minutes = Math.floor(timeInSeconds / 60);
```

Removes decimal part.

---

## Get Remaining Seconds

```
js
let seconds = timeInSeconds % 60;
```

Modulo operator:
Returns remainder after division.

Example:
125 % 60 = 5

---

## Add Leading Zero

```js
seconds = seconds < 10 ? '0' + seconds : seconds;
```

If seconds < 10:
Make it "05" instead of "5".

---

## Stop Timer

```js
clearInterval(countdown);
```

Stops the repeating interval.

Without this:
Timer would continue running forever.

---

## Auto Submit When Time Ends

```
js
if (timeInSeconds <= 0)
```

* Stop timer
* Alert user
* Call `submitExam()`

---

## Save Timer Progress

```
js
timeInSeconds--;
saveToStorage();
```

Every second:

* Decrease timer
* Persist state

This ensures timer continues correctly after refresh.

---

# 10. saveToStorage()

```
js
localStorage.setItem('cbtData', JSON.stringify({
    questions,
    currentQuestionIndex,
    timeInSeconds
}));
```

This stores:

* All questions (with answers)
* Current question position
* Remaining time

This is your **application state snapshot**.

---

# Core JavaScript Concepts Used

* DOM Manipulation
* Event Listeners
* Template Literals
* Ternary Operator
* Modulo Operator
* setInterval / clearInterval
* JSON.stringify / JSON.parse
* LocalStorage Persistence
* State Management

---

# Final System Architecture

Your CBT system now supports:

- Dynamic question rendering
- State persistence
- Navigation tracking
- Auto-save answers
- Countdown timer
- Auto submission
- Result calculation
- Page refresh safety
- Redirect after completion

---

# What You've Built

This is no longer a simple quiz script.

This is a:

> Stateful, persistent, timed examination engine.

That is production-level logic for a real CBT platform.
