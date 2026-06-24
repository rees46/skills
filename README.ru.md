# REES46 Skills

Скиллы для AI-агентов, которые помогают работать с SDK REES46.

English version: [README.md](README.md).

## Что входит в репозиторий

В репозитории лежат директории скиллов. Каждая директория начинается с файла `SKILL.md` и может содержать дополнительные справочные файлы.

Доступные скиллы:

- `js-sdk/` - интеграция, ревью и отладка REES46 JS SDK v3.

Структура скилла:

```text
js-sdk/
  SKILL.md
  references/
    api.md
    examples.md
    common-mistakes.md
```

Устанавливайте директорию скилла целиком, а не только `SKILL.md`. Агенту нужны файлы из `references/`: описание API, примеры и чеклисты для ревью.

## Рекомендуемый способ: установка через CLI

Самый простой способ установить скиллы - `skills` CLI:

```bash
npx skills add rees46/skills --skill '*' -g
```

Эта команда глобально устанавливает все скиллы и дает CLI самому определить или спросить, для каких агентов их поставить.

Установка для конкретного агента:

```bash
# Codex
npx skills add rees46/skills --skill '*' -g -a codex

# Claude Code
npx skills add rees46/skills --skill '*' -g -a claude-code

# Cursor
npx skills add rees46/skills --skill '*' -g -a cursor

# Windsurf
npx skills add rees46/skills --skill '*' -g -a windsurf
```

Для неинтерактивной установки добавьте `-y`:

```bash
npx skills add rees46/skills --skill '*' -g -a codex -y
```

Посмотреть список скиллов перед установкой:

```bash
npx skills add rees46/skills --list
```

`npx add-skill` может все еще работать как совместимый alias, но старый npm-пакет `add-skill` помечен как deprecated. Для новых установок используйте `npx skills add`.

## Обновление через CLI

Если скиллы были установлены через `npx skills add`, обновите их командой:

```bash
npx skills update -g
```

Для неинтерактивного обновления:

```bash
npx skills update -g -y
```

Обновить только конкретный скилл:

```bash
npx skills update rees46-js-sdk-v3 -g
```

После обновления перезапустите агента, чтобы он перечитал файлы скиллов.

## Ручная установка

Используйте ручную установку, если не можете использовать `npx`, хотите сначала просмотреть файлы или вам нужен полный контроль над тем, куда они будут скопированы.

### 1. Скачайте скиллы

Сначала склонируйте репозиторий:

```bash
git clone https://github.com/rees46/skills.git
cd skills
```

Если вы используете SSH для GitHub:

```bash
git clone git@github.com:rees46/skills.git
cd skills
```

### 2. Установите скиллы для нужного агента

Выберите раздел для агента, которым пользуетесь.

#### Codex

Codex читает пользовательские скиллы из `$CODEX_HOME/skills`. Если `CODEX_HOME` не задан, используйте `~/.codex`.

Установить все REES46 скиллы:

```bash
mkdir -p "${CODEX_HOME:-$HOME/.codex}/skills"
for skill in */SKILL.md; do
  rsync -a --delete "${skill%/SKILL.md}" "${CODEX_HOME:-$HOME/.codex}/skills/"
done
```

Проверить установку:

```bash
find "${CODEX_HOME:-$HOME/.codex}/skills/js-sdk" -maxdepth 2 -type f
```

После установки перезапустите Codex-сессию.

#### Claude Code

Если ваша версия Claude Code поддерживает локальные skills, установите директорию скилла в каталог skills для Claude. Частый вариант локальной установки:

```bash
mkdir -p "$HOME/.claude/skills"
for skill in */SKILL.md; do
  rsync -a --delete "${skill%/SKILL.md}" "$HOME/.claude/skills/"
done
```

После установки перезапустите Claude Code.

Если ваша версия Claude Code не загружает директории `SKILL.md` напрямую, подключите REES46 skill через проектные инструкции. Из директории вашего проекта:

```bash
cat >> CLAUDE.md <<'EOF'

## REES46 JS SDK

When working with REES46 JS SDK v3, read and follow:

- /absolute/path/to/skills/js-sdk/SKILL.md
- /absolute/path/to/skills/js-sdk/references/api.md
- /absolute/path/to/skills/js-sdk/references/examples.md
- /absolute/path/to/skills/js-sdk/references/common-mistakes.md
EOF
```

Замените `/absolute/path/to/skills` на путь к локальному клону этого репозитория.

#### Cursor, Windsurf и другие IDE-агенты

Если агент не поддерживает `SKILL.md` как отдельный формат скиллов, используйте файлы как правила проекта.

1. Склонируйте этот репозиторий в стабильное место на машине.
2. Откройте ваш проект в IDE-агенте.
3. Добавьте project rule со ссылками на файлы REES46 skill.

Пример project rule:

```text
When adding, reviewing, or debugging REES46 JS SDK v3 code, follow these local instructions:

/absolute/path/to/skills/js-sdk/SKILL.md
/absolute/path/to/skills/js-sdk/references/api.md
/absolute/path/to/skills/js-sdk/references/examples.md
/absolute/path/to/skills/js-sdk/references/common-mistakes.md

Use only commands and parameter shapes documented in the skill references.
Do not invent REES46 SDK API calls.
```

Замените `/absolute/path/to/skills` на путь к локальному клону.

## Установка через symlink

Если вы планируете часто обновлять скиллы, можно установить их через symlink вместо копирования файлов.

Codex:

```bash
mkdir -p "${CODEX_HOME:-$HOME/.codex}/skills"
for skill in */SKILL.md; do
  skill_dir="${skill%/SKILL.md}"
  ln -sfn "$PWD/$skill_dir" "${CODEX_HOME:-$HOME/.codex}/skills/$skill_dir"
done
```

Claude Code:

```bash
mkdir -p "$HOME/.claude/skills"
for skill in */SKILL.md; do
  skill_dir="${skill%/SKILL.md}"
  ln -sfn "$PWD/$skill_dir" "$HOME/.claude/skills/$skill_dir"
done
```

После создания symlink перезапустите агента.

## Ручное обновление

Если скиллы установлены копированием:

```bash
cd /path/to/skills
git pull
for skill in */SKILL.md; do
  rsync -a --delete "${skill%/SKILL.md}" "${CODEX_HOME:-$HOME/.codex}/skills/"
done
```

Для Claude Code замените путь назначения на каталог skills для Claude:

```bash
for skill in */SKILL.md; do
  rsync -a --delete "${skill%/SKILL.md}" "$HOME/.claude/skills/"
done
```

Если скиллы установлены через symlink:

```bash
cd /path/to/skills
git pull
```

После обновления перезапустите агента.

## Релизы

Релизы управляются через [release-please](https://github.com/googleapis/release-please-action).

При каждом push в `master` release-please читает Conventional Commits и создает или обновляет release PR. Release PR обновляет `CHANGELOG.md` и `version.txt`. После merge этого release PR создаются GitHub Release и tag.

Правила повышения версии:

- `fix:` или `perf:` создают patch-релиз, например `v1.0.1`;
- `feat:` создает minor-релиз, например `v1.1.0`;
- `feat!:` / `fix!:` или `BREAKING CHANGE:` создают major-релиз, например `v2.0.0`;
- коммиты вроде `docs:`, `chore:`, `ci:`, `refactor:` и `test:` сами по себе не создают релиз, если не настроить другое поведение.

После создания GitHub Release workflow загружает `.tar.gz` и `.zip` assets, куда входят только публичные файлы скиллов и README. Приватный submodule `docs` не checkout'ится и не попадает в архивы релиза.

## Проверка установки

Попросите агента выполнить задачу, которая явно требует REES46 SDK, например:

```text
Проверь интеграцию REES46 JS SDK в этом проекте и найди ошибки инициализации.
```

Ожидаемое поведение:

- агент говорит, что использует REES46 JS SDK skill;
- агент читает `js-sdk/SKILL.md`;
- при необходимости агент использует `references/api.md`, `references/examples.md` и `references/common-mistakes.md`;
- агент не придумывает недокументированные команды SDK.

## Безопасность

Скиллы являются инструкциями для агента. Перед установкой сторонних скиллов всегда читайте `SKILL.md` и все справочные файлы, на которые он ссылается. Не устанавливайте скиллы, которые требуют секреты, меняют права доступа или запускают команды без понятной причины.
