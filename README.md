🇧🇷 [Leia em português](README.pt-BR.md)

# Live Series Search (Flask + Ajax)

Live search for TV series: every time you type a letter, the page asks the server and updates the list **without reloading**.

A study project based on the CS50 (Harvard) exercise about Ajax with JSON, built and debugged step by step.

## What it does

1. You open the page and see a text field.
2. You type `office`.
3. The list shows: `The Office` and `Office Space`.

If the field is empty, the list disappears.

## Tech stack

| Technology | What it does here |
|---|---|
| **Python + Flask** | web server that receives the request and responds |
| **SQLite (cs50.SQL)** | database with the `shows` table |
| **JavaScript (fetch + JSON)** | asks the server for data and builds the list |
| **HTML** | the text field and the `<ul>` list |

## Project structure

```
Projeto01/
├── app.py              # Flask server (routes "/" and "/search")
├── templates/
│   └── index.html      # page with the input, the list and the JavaScript
├── requirements.txt    # project dependencies
├── .gitignore          # files kept out of GitHub (venv, __pycache__, *.db)
└── README.md
```

The `shows.db` file and the `venv/` folder are not in Git. The database is created by the command in the next section.

## How to run

```bash
# 1. create and activate the virtual environment
python3 -m venv venv
source venv/bin/activate

# 2. install the dependencies
pip install -r requirements.txt

# 3. create a test database with 5 series
sqlite3 shows.db "CREATE TABLE shows (id INTEGER PRIMARY KEY, title TEXT); INSERT INTO shows (title) VALUES ('The Office'), ('Office Space'), ('Breaking Bad'), ('Friends'), ('The Crown');"

# 4. start the server
flask run
```

Open http://127.0.0.1:5000/ in your browser.

Tip: every time you open a new terminal, run `source venv/bin/activate` before `flask run`. The prompt shows `(venv)` when it is active.

## How it works (step by step)

```
you type → JavaScript → fetch → Flask → SQLite → JSON → JavaScript → list on screen
```

**1. You type a letter.** The browser fires the `input` event and JavaScript catches it:

```javascript
input.addEventListener('input', async function() { ... });
```

**2. JavaScript calls the server.** `fetch` requests the `/search` route and sends the typed text in the `q` parameter:

```javascript
let response = await fetch('/search?q=' + input.value);
```

**3. Flask reads `q` and queries the database.** The `?` in the SQL is a parameter, which protects against *SQL injection*:

```python
q = request.args.get("q")
shows = db.execute("SELECT * FROM shows WHERE title LIKE ? LIMIT 50", "%" + q + "%")
```

The `%` on both sides means "the text can be anywhere in the title".

**4. Flask returns JSON**, which is plain data, not a page:

```python
return jsonify(shows)
# [{"id": 1, "title": "The Office"}, {"id": 2, "title": "Office Space"}]
```

**5. JavaScript builds the list.** It converts the JSON, creates one `<li>` per series and puts everything inside the `<ul>`:

```javascript
let shows = await response.json();
// for ... html += '<li>' + title + '</li>';
document.querySelector('ul').innerHTML = html;
```

This is called **Ajax**: the page asks for data in the background and updates only part of itself.

## Concepts used

- **Web route:** `@app.route("/search")` connects a URL to a Python function.
- **URL parameter:** `request.args.get("q")` reads what comes after the `?`.
- **JSON API:** `jsonify(shows)` returns data, not a page.
- **Parameterized SQL:** the `?` in `LIKE ?` prevents SQL injection. Never build SQL by concatenating user text.
- **Ajax / fetch:** JavaScript requests data without reloading the page.
- **async/await:** the browser waits for the server's response without freezing.
- **DOM:** `innerHTML` updates only the list.
- **XSS:** the `replace` tries to prevent HTML injection, but it only replaces the first occurrence. The full fix is to use `textContent` (see "Next steps").

## Errors I ran into (and what each one taught me)

| Error | Cause | Fix |
|---|---|---|
| `SyntaxError: invalid syntax` on the SQL line | the `+` was missing in `"%" + q + "%"` | Python shows the exact line of the error |
| `ModuleNotFoundError: No module named 'cs50'` | Flask was running outside the virtual environment | `source venv/bin/activate` and `pip install` |
| `TemplateNotFound: index.html` | the folder was named `templantes` | rename it to `templates` |
| `Internal Server Error` (500) on search | `shows.db` was empty (0 bytes) | create the `shows` table with `sqlite3` |
| `Address already in use` | another `flask run` was still using port 5000 | `pkill -f "flask run"` or `Ctrl+C` in the other tab |

Main lesson: the browser only shows "500". The **real reason** is in the `flask run` terminal.

## How to see what happens under the hood

1. Open the browser developer tools (`Ctrl + Shift + I`) and go to the **Network** tab.
2. Type `office` letter by letter, **without commas**.
3. Click each `search?q=...` request and check the URL, the `200` status and the JSON response.

Observed result: **one request per key pressed**. That is why a technique called *debounce* exists.

## Next steps

- [ ] Replace `innerHTML` with `textContent` (a real XSS fix)
- [ ] Debounce: wait ~300 ms after the last letter before calling `fetch`
- [ ] Show "no results" when the list comes back empty
- [ ] More columns (year, genre) and `ORDER BY`
- [ ] Handle server errors with `try/catch`
- [ ] Automated test with `pytest`
- [ ] Online deployment

## What I learned

- Python, HTML and JavaScript connect through an API: JavaScript calls a Flask route, which queries the database and answers with JSON.
- A 500 error says nothing by itself. The server terminal shows the cause, and that is how I found the wrongly named folder, the missing `cs50` in the virtual environment and the empty database.
- The browser sends one request to the server for every key typed. I want to learn how to limit that with debounce.
- I made my first Git commit and organized the project with `requirements.txt` and `.gitignore`.