# Claude Plugins — Власний маркетплейс плагінів

Цей репозиторій є одночасно:
1. **Маркетплейсом плагінів** для Claude Code — каталог плагінів, які можна підключити та встановити
2. **Середовищем розробки** нових плагінів за допомогою Claude Code

---

## Зміст

- [Структура репозиторію](#структура-репозиторію)
- [Підключення маркетплейсу](#підключення-маркетплейсу)
- [Встановлення плагінів](#встановлення-плагінів)
- [Розробка нових плагінів](#розробка-нових-плагінів)
- [Структура плагіна](#структура-плагіна)
- [Рекомендовані плагіни для розробки](#рекомендовані-плагіни-для-розробки)

---

## Структура репозиторію

```
claude-plugins/
├── .claude-plugin/
│   └── marketplace.json        # Каталог маркетплейсу
├── .claude/
│   └── settings.json           # Налаштування проєкту (підключення маркетплейсу)
├── plugins/                    # Всі плагіни зберігаються тут
│   └── plugin-creator/         # Плагін для розробки нових плагінів
│       ├── .claude-plugin/
│       │   └── plugin.json
│       └── skills/
│           └── new-plugin/
│               └── SKILL.md
├── drafts/                     # Чернетки (ігноруються git)
│   └── .gitkeep
├── .gitignore
└── README.md
```

---

## Підключення маркетплейсу

### Варіант 1 — через GitHub (рекомендовано)

Після того як репозиторій запушено на GitHub (`your-username/claude-plugins`):

```
/plugin marketplace add your-username/claude-plugins
```

> Замініть `your-username` на ваш GitHub username.

### Варіант 2 — локальний шлях

Якщо репозиторій є локально:

```
/plugin marketplace add ./path/to/claude-plugins
```

або через абсолютний шлях:

```
/plugin marketplace add C:/dev/claude-plugins
```

### Варіант 3 — через settings.json (для команди)

Додайте до `.claude/settings.json` вашого проєкту:

```json
{
  "extraKnownMarketplaces": {
    "claude-plugins": {
      "source": {
        "source": "github",
        "repo": "your-username/claude-plugins"
      }
    }
  }
}
```

Коли інший розробник відкриє проєкт, Claude Code автоматично запропонує підключити маркетплейс.

---

## Встановлення плагінів

### Переглянути доступні плагіни

```
/plugin
```

Відкриється вкладка Discover де видно всі підключені маркетплейси та їх плагіни.

### Встановити конкретний плагін

```
/plugin install plugin-creator@claude-plugins
```

Формат: `назва-плагіна@назва-маркетплейсу`

### Локальне тестування плагіна (без встановлення)

```bash
claude --plugin-dir ./plugins/plugin-creator
```

Для кількох плагінів одночасно:

```bash
claude --plugin-dir ./plugins/plugin-creator --plugin-dir ./plugins/інший-плагін
```

### Перезавантажити плагіни під час сесії

```
/reload-plugins
```

---

## Розробка нових плагінів

### Процес розробки

1. **Покладіть матеріали до папки `drafts/`**

   Додайте файли з вимогами, промптами, описом функціональності майбутнього плагіна. Формат довільний — `.md`, `.txt`, або будь-який текстовий формат.

   Приклад `drafts/requirements.md`:
   ```markdown
   # Плагін: git-helper

   ## Опис
   Автоматизує Git операції: staging, commit з гарним повідомленням, push, PR.

   ## Скіли
   - /git-helper:commit — аналізує зміни, генерує commit message, комітить
   - /git-helper:pr — створює PR з описом на основі commit history
   ```

2. **Запустіть скіл `new-plugin`**

   ```
   /plugin-creator:new-plugin
   ```

   Claude прочитає всі файли з `drafts/`, проаналізує вимоги та:
   - Створить структуру плагіна в `plugins/<назва>/`
   - Заповнить `plugin.json` та `SKILL.md`
   - Додасть плагін до `marketplace.json`

3. **Протестуйте плагін локально**

   ```bash
   claude --plugin-dir ./plugins/<назва-плагіна>
   ```

4. **Очистіть папку `drafts/`**

   Видаліть або перемістіть файли з `drafts/` перед розробкою наступного плагіна.

5. **Закомітьте та запушіть**

   ```bash
   git add plugins/<назва-плагіна>
   git add .claude-plugin/marketplace.json
   git commit -m "feat: add <назва-плагіна> plugin"
   git push
   ```

   Після пуша плагін стане доступним у маркетплейсі.

---

## Структура плагіна

Кожен плагін — це папка в `plugins/` з такою структурою:

```
plugins/my-plugin/
├── .claude-plugin/
│   └── plugin.json         # Обов'язковий маніфест плагіна
├── skills/                 # Скіли (slash-команди)
│   └── my-skill/
│       └── SKILL.md        # Інструкції для Claude
├── hooks/
│   └── hooks.json          # Хуки (опційно)
├── .mcp.json               # MCP сервери (опційно)
└── README.md               # Документація (опційно)
```

### plugin.json

```json
{
  "name": "my-plugin",
  "description": "Опис що робить плагін",
  "version": "1.0.0",
  "author": {
    "name": "Ваше ім'я",
    "email": "you@example.com"
  },
  "repository": "https://github.com/your-username/claude-plugins",
  "license": "MIT"
}
```

> Поле `name` стає префіксом для всіх скілів: `my-plugin` → `/my-plugin:my-skill`

### SKILL.md

```markdown
---
name: my-skill
description: Короткий опис що робить скіл і коли використовувати. Починайте з ключових слів (до 250 символів).
allowed-tools: Read, Write, Bash, Glob, Grep
---

Детальні інструкції для Claude що потрібно зробити при виклику цього скілу.

Аргументи доступні через $ARGUMENTS або $0, $1, $2...
```

**Корисні параметри SKILL.md:**

| Параметр | Опис |
|---|---|
| `name` | Назва скілу (lowercase, тільки дефіси) |
| `description` | Опис для автоматичного визначення контексту |
| `allowed-tools` | Інструменти без запиту дозволу |
| `disable-model-invocation: true` | Тільки ручний виклик `/skill` |
| `context: fork` | Запуск в ізольованому субагенті |
| `model` | Конкретна модель Claude |
| `effort` | `low` / `medium` / `high` / `max` |

### Реєстрація в marketplace.json

Кожен новий плагін додається до `.claude-plugin/marketplace.json`:

```json
{
  "plugins": [
    {
      "name": "my-plugin",
      "source": "./plugins/my-plugin",
      "description": "Опис плагіна",
      "version": "1.0.0",
      "category": "development",
      "tags": ["тег1", "тег2"],
      "author": { "name": "claude-plugins" }
    }
  ]
}
```

---

## Рекомендовані плагіни для розробки

Встановіть ці плагіни з офіційного маркетплейсу для покращення процесу розробки:

```
/plugin install plugin-dev@claude-plugins-official
/plugin install commit-commands@claude-plugins-official
/plugin install pr-review-toolkit@claude-plugins-official
/plugin install code-review@claude-plugins-official
```

| Плагін | Що робить |
|---|---|
| `plugin-dev` | Інструменти для розробки та відлагодження плагінів |
| `commit-commands` | Git staging, генерація commit messages, push, PR |
| `pr-review-toolkit` | Агенти для рев'ю pull request-ів |
| `code-review` | Рев'ю коду та рекомендації |

### LSP плагіни (для підказок та діагностики в реальному часі)

Якщо розробляєте плагіни з TypeScript/JavaScript:
```
/plugin install typescript-lsp@claude-plugins-official
```

---

## Категорії плагінів

При розробці нових плагінів використовуйте ці категорії в `marketplace.json`:

- `development` — інструменти розробки
- `git` — Git операції
- `testing` — тестування
- `deployment` — деплоймент
- `documentation` — документація
- `productivity` — продуктивність
- `integration` — інтеграції з сервісами

---

## Валідація

Перевірте коректність структури маркетплейсу або плагіна:

```
/plugin validate .
```

або для конкретного плагіна:

```
/plugin validate ./plugins/my-plugin
```

---

## Оновлення плагінів

Після пуша нових змін до GitHub, плагіни оновляться автоматично при наступному використанні. Примусове оновлення:

```
/plugin update
```
