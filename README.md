# EduVault - Auto Upload Questions Demo (Demo)

A standalone, single-file demo of a question bank + form-fill portal for educational platforms. Built to replicate (and improve on) the manual data-entry workflow of uploading questions from Word documents into an online platform.

This is a **demo build** — no backend, no database, no login system. Everything runs in the browser using vanilla HTML/CSS/JavaScript and the [mammoth.js](https://github.com/mwilliamson/mammoth.js) library (loaded from CDN) to parse `.docx` files client-side. No data ever leaves the browser, which sidesteps the privacy concerns that prompted this build in the first place.

---

## Files

| File | Purpose |
|------|---------|
| `tuition_portal.html` | The entire app — open in any modern browser |
| `Sec1_LinearEq.docx` | Sample worksheet showing the expected document format |

---

## Running it

Just open `tuition_portal.html` in a browser. No server, no install, no build step.

Tested on Chrome, Safari, and Firefox.

---

## What it does

### Two interfaces

**Teacher view**
- Upload `.docx` files; questions are extracted automatically
- Edit each parsed question in-form before importing
- View full question bank with mark schemes and explanations
- Search + filter by grade, subject, tutor, type, date, and difficulty
- Copy questions to clipboard, remove individual questions

**Student view**
- Same search and filter sidebar
- Browse questions, see prompts and MCQ options
- Type free-text answers or select MCQ choices
- Save drafts and submit answers
- No access to model answers, mark schemes, or explanations

Switch between the two from the landing page or via "Switch Role" in the header.

---

## Document format

The parser recognises the format used by the sample doc (`Sec1_LinearEq.docx`). The structure matters for parsing, the expected format is outlined below.

### Cover page (auto-skipped)
Everything before the first `Question 1` (or `Q1` / `1.` / `1)`) is treated as the cover page. The parser extracts:
- **Grade / Level** — e.g. "Secondary 1", "Sec 1", "Primary 5", "P5", "JC2"
- **Subject** — e.g. "Mathematics", "Science", "Chinese"
- **Tutor** — from "Tutor: …" or "Ms/Mr/Mrs/Dr [Name]"

These auto-fill the metadata form. You can override anything before importing.

The full skipped cover text is shown in a collapsible block so you can verify nothing important was lost.

### Question structure
Each question follows this pattern:

```
Question 1  [EASY]
Solve each of the following linear equations.

a)  x + 9 = 15
        A)  x = 4                    B)  x = 6
        C)  x = 7                    D)  x = 24
Answer: B)  x = 6
Working:
Subtract 9 from both sides:
    x + 9 - 9  =  15 - 9
    x  =  6

b)  3x = 27
        A)  x = 3                    B)  x = 8
        ...
```

This becomes one question record with:
- **Main question text**: `"Solve each of the following linear equations."`
- **Difficulty**: `Easy` (extracted from `[EASY]`, then stripped)
- **Parts**: array of sub-questions, each with its own prompt, options, answer, and explanation

### Recognised patterns

| Element | Accepted formats |
|---------|------------------|
| Main question | `Question 1`, `Q1`, `Q. 1`, `1.`, `1)` |
| Sub-part | `a)`, `b)`, `(a)`, `1.1`, `1.2`, `(i)`, `(ii)` |
| MCQ option | `A)`, `B)`, `(A)` (uppercase only — lowercase = sub-part) |
| Answer | `Answer:`, `Ans:`, `A:` |
| Working / explanation | `Working:`, `Solution:`, `Explanation:`, `Sol:` |
| Difficulty | `[Easy]`, `[Medium]`, `[Hard]`, `(Hard)`, `Difficulty: Medium` |

Multiple MCQ options on one line (e.g. `A) x = 4    B) x = 6`) are split correctly.

Footer lines (page numbers, repeating header text) are filtered out automatically.

---

## Editing parsed questions

After parsing, you'll see a Q1 / Q2 / Q3 navigator. Click any question to edit:

- **Main Question** — the intro text or standalone question
- **Type** — MCQ / Short Answer / Essay / Fill in the Blank (auto-inferred but editable)
- **Difficulty** — Easy / Medium / Hard / None
- **Parts** — for each sub-question:
  - Label (a, b, c…)
  - Prompt
  - Options (one per line, leave blank for non-MCQ)
  - Answer
  - Explanation
  - Add / remove parts as needed

Edits autosave as you type. Click **Import All** to add everything to the question bank.

---

## Data model

Each question in the bank looks like:

```js
{
  id: 100,
  mainText: "Solve each of the following linear equations.",
  parts: [
    {
      label: "a)",
      prompt: "x + 9 = 15",
      options: ["x = 4", "x = 6", "x = 7", "x = 24"],
      answer: "B) x = 6",
      explanation: "Subtract 9 from both sides..."
    },
    // ...
  ],
  subject: "Mathematics",
  grade: "S1",
  tutor: "Ms Tan",
  type: "MCQ",
  difficulty: "Easy",
  date: "2026",
  uploadDate: "7 May 2026",
  imported: true
}
```

---

## Known limitations

This is a v1 demo, so:

- **No persistence** — refresh the page and your imported questions are gone. No localStorage either (artifact restriction). Drop in real backend storage when ready.
- **No authentication** — the role toggle is purely a view switch, not a real auth layer.
- **No real submission** — student "Submit" just shows a toast; nothing is actually sent or saved.
- **Parser assumes consistent formatting** — heavily inconsistent or unconventional docs may need manual cleanup in the editor before import.
- **Tables in docs are flattened** — mammoth extracts table contents as plain text, so they may appear as separate lines. The cover-page admin table is part of the skipped section, so this rarely matters in practice.
- **Images, diagrams, and formulas as images aren't captured** — only text content. LaTeX or Unicode math characters work fine.
- **Sample data is hardcoded** — the 3 sample questions on first load are placeholders to show the UI. Replace or delete in the `ALL_QUESTIONS` array near the top of the script.

---

## Customising

A few common tweaks:

- **Add subjects / grades / tutors**: edit the `<option>` lists in both the sidebar filters and the upload modal metadata fields.
- **Change colours**: all theme colours are CSS variables at the top of the `<style>` block (`--accent`, `--accent2`, `--teacher`, `--student`, etc.).
- **Swap fonts**: replace the Google Fonts `<link>` in the `<head>`.
- **Disable cover-page detection**: in `splitCoverAndBody()`, return `{ coverText: '', bodyText: raw }` immediately.

---

## Tech stack

- Vanilla HTML / CSS / JS — no framework
- [mammoth.js](https://github.com/mwilliamson/mammoth.js) for `.docx` text extraction
- Google Fonts (Syne + Karla) for typography

No build step, no package.json, no dependencies to install.

---

## Notes on the sample doc

`Sec1_LinearEq.docx` is a fictional Secondary 1 maths worksheet on Linear Equations in One Variable. It contains:

- Cover page with title, level, topic, difficulty count table, student info fields, tutor sign-off
- Instructions section
- 5 main questions (Q1–Q5), each with 2–3 sub-parts
- Each sub-part has 4 MCQ options, an answer line, and a working/explanation block
- Difficulty labelled per question: 2 Easy, 2 Medium, 1 Hard

Use it to verify the parser is working as expected, or as a template for creating your own worksheets.
