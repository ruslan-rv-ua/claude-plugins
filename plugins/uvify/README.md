# uvify

Перетворює звичайні Python-скрипти на самодостатні, запускаємі через `uv` скрипти, додаючи блок інлайн-метаданих [PEP 723](https://peps.python.org/pep-0723/) (`# /// script`).

## Скіли

### `/uvify:adapt [path/to/script.py]`

Аналізує Python-скрипт та додає або оновлює блок `# /// script` з:

- **`requires-python`** — автовизначення мінімальної версії Python за синтаксисом скрипта
- **`dependencies`** — список залежностей, виявлених по `import`-заявах (без stdlib)
- **shebang** `#!/usr/bin/env -S uv run` — щоб скрипт можна було запустити напряму

**Аргумент:** шлях до `.py` файла. Якщо не вказано — використовується відкритий/виділений файл у розмові.

## Приклад

**До:**
```python
import requests
import pandas as pd
from bs4 import BeautifulSoup

def fetch():
    resp = requests.get("https://example.com")
    return BeautifulSoup(resp.text, "html.parser")
```

**Після `/uvify:adapt script.py`:**
```python
#!/usr/bin/env -S uv run
# /// script
# requires-python = ">=3.11"
# dependencies = [
#   "beautifulsoup4",
#   "pandas",
#   "requests",
# ]
# ///

import requests
import pandas as pd
from bs4 import BeautifulSoup

def fetch():
    resp = requests.get("https://example.com")
    return BeautifulSoup(resp.text, "html.parser")
```

**Запуск:**
```bash
uv run script.py
# або після chmod +x:
./script.py
```

## Особливості

| Функція | Деталі |
|---|---|
| **Ідемпотентність** | Якщо блок вже є — оновлює, не дублює |
| **Фільтрація stdlib** | `os`, `sys`, `json` та ін. не потрапляють у залежності |
| **Автовизначення версії Python** | `match/case` → `>=3.10`, `tomllib` → `>=3.11`, тощо |
| **Розширений маппінг** | `cv2→opencv-python`, `PIL→Pillow`, `yaml→PyYAML`, `sklearn→scikit-learn` та ін. |
| **Shebang** | Додає або замінює існуючий на `#!/usr/bin/env -S uv run` |

## Встановлення

```
/plugin install uvify@claude-plugins
```

або локально:

```bash
claude --plugin-dir ./plugins/uvify
```
