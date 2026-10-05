# Rsync SSH Deploy

GitHub Action, который выкладывает каталог (например, собранный статический сайт) на сервер по SSH с помощью `rsync`. Подходит для хостингов, где доступ есть только по SSH: университетский сервер, виртуальная машина, обычный VPS.

Action оформлен как composite: внутри обычные шаги `bash`, поэтому он работает без Docker и запускается быстро.

## Пример использования

```yaml
name: Deploy site
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: pip install -r requirements.txt && mkdocs build --strict

      - name: Deploy over SSH
        id: deploy
        uses: 777werona-afk/rsync-ssh-deploy@v1
        with:
          host: ${{ secrets.DEPLOY_HOST }}
          user: ${{ secrets.DEPLOY_USER }}
          path: /var/www/site
          source: site
          key: ${{ secrets.DEPLOY_KEY }}
          known_hosts: ${{ secrets.DEPLOY_KNOWN_HOSTS }}
          delete: 'true'

      - run: echo "Выложено ${{ steps.deploy.outputs.files_transferred }} файлов в ${{ steps.deploy.outputs.destination }}"
```

## Параметры

| Параметр | Обязательный | По умолчанию | Описание |
|---|---|---|---|
| `host` | да | | имя или адрес сервера |
| `user` | да | | пользователь на сервере |
| `path` | да | | каталог назначения на сервере |
| `source` | нет | `site` | исходный каталог в рабочей папке задания |
| `key` | да | | закрытый SSH-ключ (только через `secrets`) |
| `delete` | нет | `false` | `true` включает `rsync --delete`: на сервере удаляются файлы, которых нет в источнике |
| `port` | нет | `22` | порт SSH |
| `known_hosts` | нет | пусто | строки `known_hosts` сервера; без них отпечаток запрашивается через `ssh-keyscan` (менее безопасно) |
| `dry_run` | нет | `false` | `true` только показывает план без изменений |

## Результаты (outputs)

| Имя | Что содержит |
|---|---|
| `destination` | итоговое назначение в формате `user@host:path` |
| `files_transferred` | сколько файлов передано |
| `files_deleted` | сколько файлов удалено на сервере |

## Защита от ошибок

- Пустой или отсутствующий исходный каталог останавливает выкладку: иначе вместе с `delete: 'true'` сайт на сервере был бы стёрт.
- В режиме `delete` отклоняются опасные пути `/`, `.` и `~`.
- Ключ записывается во временный каталог раннера с правами `600` и удаляется в последнем шаге даже при ошибке.
- Проверка сервера идёт с `StrictHostKeyChecking=yes`.

## Совместимость

| Что | Поддержка |
|---|---|
| Раннеры | `ubuntu-latest` и другие Linux; macOS при наличии `rsync` и `ssh` |
| Windows | не поддерживается (нужен `bash`, `rsync`) |
| Сервер | любой с SSH и установленным `rsync` |
| Синтаксис | GitHub Actions; на платформах, заявляющих совместимость с ним (например, GitVerse), нужно проверять поддержку composite-actions |

## Версионирование

Используется семантическое версионирование: теги `v1.0.0`, `v1.0.1`, `v1.1.0` и так далее. Подвижный мажорный тег `v1` автоматически переставляется на последний релиз `v1.x.y` workflow `Move major tag`. Для стабильности используйте `@v1`, для полной воспроизводимости — точный тег, например `@v1.0.0`.

## Проверка

Workflow `Test action` на каждый push поднимает локальный `sshd` на раннере, выкладывает каталог на `localhost`, проверяет удаление лишних файлов и отказ при пустом источнике.

## Где используется

Action подключён в репозитории [vr-gaze-site](https://github.com/777werona-afk/vr-gaze-site) (отчёт по записям взгляда в VR, сайт на MkDocs).

## Лицензия

[MIT](LICENSE)
