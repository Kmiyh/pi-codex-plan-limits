# Публикация пакета и автопубликация через GitHub

Этот репозиторий настроен по той же схеме, что и соседний `repo-pi-skills-menu`:

- публикация по git tag `v*`
- публикация в **npm**
- публикация в **GitHub Packages**

Workflow лежит в:

```text
.github/workflows/publish.yml
```

## Что нужно настроить один раз

### 1. npm token

В GitHub repo добавь secret:

- `NPM_TOKEN`

### 2. GitHub Packages

Используется встроенный:

- `${{ secrets.GITHUB_TOKEN }}`

Дополнительный secret не нужен.

## Базовый release flow

```bash
npm run typecheck
npm pack --dry-run
npm version patch
git push --follow-tags
```

Для новой фичи вместо `patch` используй `minor`, для breaking change — `major`.
