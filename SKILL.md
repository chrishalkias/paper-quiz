---
name: paper-tester
description: Use when the user wants to test or quiz themselves on an academic paper. Invoked with /paper-tester followed by a file path or URL.
---

# Paper Tester

Generate a browser-based quiz for an academic paper, then open it automatically.

## Usage

```
/paper-tester <path-to-pdf-or-url>
```

**Examples:**
```
/paper-tester ~/Downloads/attention-is-all-you-need.pdf
/paper-tester https://arxiv.org/pdf/1706.03762
```

---

## Step 1 — Parse the input

The argument is: `$ARGUMENTS`

- If `$ARGUMENTS` starts with `http`, it is a **URL**.
- Otherwise it is a **local file path**.

---

## Step 2 — Read the paper

- **Local file (PDF or text):** Use the `Read` tool on the path.
- **URL:** Use the `WebFetch` tool on the URL.

Store the full text content for the next step.

---

## Step 3 — Generate the quiz via the `tester` subagent

Spawn the `Agent` tool with **`subagent_type: "paper-tester:tester"`** and pass it a prompt containing only the paper text:

```
<full paper text from Step 2>
```

The `tester` agent knows exactly how to structure the quiz — do not add extra instructions to its prompt. Its only input should be the paper content.

Wait for the agent to return the JSON string. Save it as `QUIZ_JSON`.

---

## Step 4 — Find the quiz template

Use `Glob` to find the quiz template with the pattern:
```
**/paper-tester/quiz-template.html
```

Read the file found. Store the content as `TEMPLATE_HTML`.

---

## Step 5 — Inject quiz data

In `TEMPLATE_HTML`, replace the exact string:
```
"QUIZ_DATA_PLACEHOLDER"
```

with the raw JSON string from `QUIZ_JSON`. The result is a complete, self-contained HTML file. Do not wrap the JSON in extra quotes — the replacement string must be valid JavaScript that evaluates to a JSON object.

**Example of how the injected line should look:**
```js
const QUIZ_DATA = {"paper_title":"...","questions":[...]};
```

---

## Step 6 — Write the output file

Generate a Unix timestamp and write the final HTML to:
```
/tmp/paper-quiz-<timestamp>.html
```

Use the `Write` tool.

---

## Step 7 — Open in browser

Run the appropriate command for the OS:
- **macOS:** `open /tmp/paper-quiz-<timestamp>.html`
- **Linux:** `xdg-open /tmp/paper-quiz-<timestamp>.html`

---

## Step 8 — Confirm to the user

Tell the user:
- The quiz is open in their browser
- The paper title detected
- How many MCQ and open-ended questions were generated
- The path to the HTML file (in case they want to re-open it)
