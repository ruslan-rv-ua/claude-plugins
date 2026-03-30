---
name: adapt
description: Transform a Python script into a self-contained uv script by adding PEP 723 inline metadata (# /// script block) with auto-detected dependencies and requires-python. Handles existing blocks idempotently.
argument-hint: "<path/to/script.py>"
---

Transform the target Python script into a self-contained, uv-executable script by adding or updating the PEP 723 `# /// script` inline metadata block.

## Step 1 — Resolve target file

Use `$ARGUMENTS` as the file path. If empty, use the file open or selected in the current conversation context. If still unclear, ask the user to specify the path.

## Step 2 — Read the script

Read the full content of the target `.py` file.

## Step 3 — Extract import statements

Scan every line for import statements and collect top-level module names:

- `import X` → module is `X`
- `import X as Y` → module is `X`
- `from X.Y.Z import ...` → module is `X` (first segment only)

Skip lines that are inside triple-quoted strings (`"""` / `'''`) or that are themselves comments (`#`).

## Step 4 — Filter out standard-library and built-in modules

Do NOT list these as dependencies. Exclude any module in this set:

`__future__`, `abc`, `argparse`, `ast`, `asyncio`, `base64`, `builtins`, `calendar`, `cmath`, `cmd`, `code`, `codecs`, `collections`, `concurrent`, `configparser`, `contextlib`, `copy`, `csv`, `ctypes`, `dataclasses`, `datetime`, `decimal`, `difflib`, `email`, `enum`, `errno`, `fileinput`, `fnmatch`, `fractions`, `ftplib`, `functools`, `gc`, `getopt`, `getpass`, `glob`, `gzip`, `hashlib`, `heapq`, `hmac`, `html`, `http`, `imaplib`, `importlib`, `inspect`, `io`, `ipaddress`, `itertools`, `json`, `keyword`, `linecache`, `locale`, `logging`, `math`, `mimetypes`, `multiprocessing`, `operator`, `os`, `pathlib`, `pickle`, `platform`, `pprint`, `queue`, `random`, `re`, `shlex`, `shutil`, `signal`, `socket`, `socketserver`, `sqlite3`, `ssl`, `stat`, `statistics`, `string`, `struct`, `subprocess`, `sys`, `tarfile`, `tempfile`, `textwrap`, `threading`, `time`, `timeit`, `tkinter`, `traceback`, `types`, `typing`, `unicodedata`, `unittest`, `urllib`, `uuid`, `warnings`, `weakref`, `xml`, `xmlrpc`, `zipfile`, `zlib`

## Step 5 — Map module names to PyPI package names

Use the table below. For any module not listed, use the import name verbatim as the package name, and append a `# NOTE: verify package name` comment on that line.

| Import name | PyPI package |
|---|---|
| `anthropic` | `anthropic` |
| `bs4` | `beautifulsoup4` |
| `cv2` | `opencv-python` |
| `dateutil` | `python-dateutil` |
| `dotenv` | `python-dotenv` |
| `flask` | `flask` |
| `django` | `django` |
| `httpx` | `httpx` |
| `jwt` | `PyJWT` |
| `matplotlib` | `matplotlib` |
| `numpy` | `numpy` |
| `openai` | `openai` |
| `pandas` | `pandas` |
| `PIL` | `Pillow` |
| `pkg_resources` | `setuptools` |
| `psutil` | `psutil` |
| `psycopg2` | `psycopg2-binary` |
| `pydantic` | `pydantic` |
| `pymongo` | `pymongo` |
| `pytest` | `pytest` |
| `redis` | `redis` |
| `requests` | `requests` |
| `rich` | `rich` |
| `sklearn` | `scikit-learn` |
| `scipy` | `scipy` |
| `sqlalchemy` | `SQLAlchemy` |
| `toml` | `toml` |
| `tomllib` | *(stdlib in Python ≥3.11 — skip entirely)* |
| `tqdm` | `tqdm` |
| `typer` | `typer` |
| `yaml` | `PyYAML` |

## Step 6 — Determine `requires-python`

Inspect the script for the highest version signal found:

| Signal | Minimum version |
|---|---|
| `match` / `case` statement | `>=3.10` |
| `X \| Y` type union in annotations | `>=3.10` |
| `typing.Self` or `typing.LiteralString` | `>=3.11` |
| `tomllib` import (stdlib) | `>=3.11` |
| Walrus operator `:=` | `>=3.8` |
| f-string `=` specifier (`f"{x=}"`) | `>=3.8` |
| Positional-only param `/` in `def` | `>=3.8` |
| No signals detected | `>=3.11` (default) |

Use the highest version constraint found.

## Step 7 — Handle an existing `# /// script` block

Search the script for an existing block matching:

```
# /// script
...
# ///
```

**If found:** Extract its current `dependencies` and `requires-python` values. Merge the detected dependencies with any already listed (union, no duplicates). If the newly determined `requires-python` is higher than the existing one, use the higher version. Replace the existing block in-place.

**If not found:** Insert a new block (see Step 9).

## Step 8 — Handle the shebang

Check line 1:

- Already `#!/usr/bin/env -S uv run` → leave unchanged
- Different shebang (e.g. `#!/usr/bin/env python3`) → replace with `#!/usr/bin/env -S uv run` and note the change in your report
- No shebang → prepend `#!/usr/bin/env -S uv run`

## Step 9 — Compose the final file

The output file must have this structure:

```
#!/usr/bin/env -S uv run
# /// script
# requires-python = ">=X.Y"
# dependencies = [
#   "package-a",
#   "package-b",
# ]
# ///

<rest of original script, byte-for-byte identical>
```

Rules:
- Shebang is always line 1.
- The `# /// script` block immediately follows (one blank line between shebang and block is acceptable but not required).
- `requires-python` comes before `dependencies` inside the block.
- Dependencies are sorted alphabetically (case-insensitive).
- Each dependency line ends with a trailing comma: `"package",`.
- One blank line separates the closing `# ///` from the first line of the original script body.
- Do not alter any other line of the original script.

## Step 10 — Write the file

Write the composed content back to the same file path.

## Step 11 — Report to the user

Output a concise summary:

```
uvified: <filename>

requires-python: >=X.Y
dependencies:
  - package-a
  - package-b

Run with:  uv run <filename>
      or:  ./<filename>   (after chmod +x)
```

If any dependency mapping was uncertain (marked with `# NOTE: verify package name`), list those packages and suggest the user verify them on pypi.org.

If an existing `# /// script` block was found and updated, briefly describe what changed (new deps added, version bumped, etc.).

If a shebang was replaced, mention the original value.
