# Quiz

A single-page, dependency-free quiz runner. Point it at a JSON file and it renders an interactive quiz with a question sidebar, live score, explanations, results screen, saved progress, and a dark/light theme toggle.

```
index.html?quiz=<url-to-quiz.json>     (query string)
index.html#quiz=<url-to-quiz.json>     (hash)
index.html#<url-to-quiz.json>          (hash shorthand)
```

Without a quiz URL the page shows a home screen with help and an input box to paste one.

Try it: `index.html?quiz=examples/quiz-app.json`

## URL parameters

| Parameter | Description |
|-----------|-------------|
| `quiz` (aliases `url`, `q`) | URL of the quiz JSON. Absolute, or relative to the page. |
| `shuffle=1` | Shuffle question order and option order. |

Parameters can go in the **query string** (`?quiz=…&shuffle=1`) or in the **hash** (`#quiz=…&shuffle=1`). If both are present, the hash wins. In the hash, if the text doesn't start with one of these keys, the whole hash is used as the quiz URL.

Use the hash on hosts that need the query string for themselves. For example, [gistpreview](https://github.com/gistpreview/gistpreview.github.io) needs `?<gist-id>`:

```
https://gistpreview.github.io/?<gist-id>#quiz=https://gist.githubusercontent.com/<user>/<id>/raw/quiz.json
```

## Quiz file format

```json
{
  "title": "My quiz",
  "description": "Optional intro shown above the questions.",
  "questions": [
    {
      "section": "Basics",
      "text": "Which option is correct?",
      "options": ["First", "Second", "Third", "Fourth"],
      "correct": 1,
      "explanation": "<strong>Second</strong> is correct because…"
    },
    {
      "section": "Basics",
      "text": "Select <em>all</em> prime numbers.",
      "options": ["2", "4", "5", "9"],
      "correct": [0, 2],
      "explanation": "An array in <code>correct</code> makes it multiple-choice."
    }
  ]
}
```

- `text`, `options` (≥ 2) and `correct` are required; everything else is optional.
- `correct` is a **zero-based** option index. An **array** of indexes makes the question multiple-choice: options become checkboxes with a Submit button, and the answer counts only if the exact set is chosen.
- A bare array of questions (no wrapper object) is also accepted.

### HTML in text fields

`text`, `options`, `explanation` and `description` may contain HTML. Because quizzes are loaded from arbitrary URLs, the HTML is **sanitized with an allow-list**: it is parsed in an inert document and rebuilt using only these tags:

`b strong i em u s del ins code pre kbd samp var br p ul ol li a blockquote mark small sub sup span div h3–h6 hr table thead tbody tr th td`

All attributes are removed, except `href` on links, which is kept only for `http:`, `https:` and `mailto:` (links open in a new tab). Dangerous elements (`script`, `style`, `iframe`, `svg`, `form`, …) are dropped with their content; other unknown tags are unwrapped and their text kept. Remember to escape literal `<` as `&lt;` when you mean to show it.

## Hosting

The app is a static `index.html`, so it works on **GitHub Pages**:

1. Repository **Settings → Pages → Build and deployment → Deploy from a branch**.
2. Pick the branch and `/ (root)`, save.
3. Open `https://<user>.github.io/<repo>/#quiz=examples/quiz-app.json`.

Quizzes can live in the same repo (use a relative path) or anywhere that serves them with CORS headers — e.g. `raw.githubusercontent.com`, `gist.githubusercontent.com` (use the gist's **Raw** link), or another GitHub Pages site.

### Local development

`fetch` doesn't work from `file://`, so serve the folder:

```sh
python3 -m http.server 8000
# open http://localhost:8000/#quiz=examples/quiz-app.json
```

## Behaviour

- Single-choice answers lock on click; multiple-choice answers lock on Submit. The correct answer(s) and the explanation are then shown.
- Progress (answers, current question, shuffle order) is saved in `localStorage` per quiz URL and restored on reload. It is discarded automatically if the quiz file changes. **Restart quiz** on the results page clears it.
- Theme follows the OS preference until toggled; the choice is remembered.
- Keyboard: `A`–`Z` / `1`–`9` pick an option, `Enter` submits or advances, `←` / `→` navigate.
