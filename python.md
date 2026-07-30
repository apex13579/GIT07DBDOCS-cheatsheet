### Python

#### HTTP Requests

| Snippet | What It Does |
|---|---|
| `import requests` | Load the requests library |
| `r = requests.get(url, timeout=10)` | GET with a 10s timeout |
| `r = requests.post(url, data={...})` | POST with form data |
| `r.json()` | Parse JSON response body to dict |
| `r.raise_for_status()` | Raise exception on 4xx or 5xx |

#### File & System

| Snippet | What It Does |
|---|---|
| `subprocess.run(["cmd","arg"], check=True)` | Run a shell command, raise error if it fails |
| `os.environ.get("VAR")` | Read environment variables safely |
| `json.loads(text)` | Parse a JSON string to dict |
| `pathlib.Path("f").read_text()` | Read a file in one line |

#### Patterns for Lab Scripts

| Pattern | What It Does |
|---|---|
| `if __name__ == "__main__":` | Entry point guard — only runs when called directly |
| `try: ... except Exception as e:` | Basic error handling — don't let scripts die silently |
| `import argparse` | Parse CLI arguments cleanly |
| `import logging; logging.basicConfig(...)` | Proper log output instead of print statements |
