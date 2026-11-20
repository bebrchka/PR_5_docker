# Практическая работа № 5 — обновление данных в Docker

Репозиторий: https://github.com/bebrchka/PR_5_docker

Два независимых экземпляра одной статической страницы работают в Nginx:

- `http://localhost:18081` — каталог `bind-data` подключен как bind mount.
- `http://localhost:18082` — HTML и изображения находятся в именованных томах Docker.

## Запуск

Команды выполняются из каталога `PR_5_Docker`:

```powershell
docker compose --profile init run --rm volume-init
docker compose up -d
docker compose ps
```

`volume-init` один раз копирует начальные файлы из `volume-seed` в два именованных тома. При последующем запуске его повторно вызывать не нужно: он заново скопирует исходное состояние.

## Изменение bind mount

Откройте `bind-data/index.html` и замените заголовок или разметку. Страница на `http://localhost:18081` получит новую верстку без пересборки образа и перезапуска контейнера. Чтобы обновить картинку, замените `bind-data/images/poster.svg` файлом `updates/bind-poster-after.svg`; для обновлённой подписи замените `bind-data/content.json` файлом `updates/bind-content-after.json`.

## Изменение named volume

Скопируйте новую верстку и данные картинки с компьютера внутрь работающего контейнера:

```powershell
docker cp .\updates\volume-index-after.html pr5-volume-site:/usr/share/nginx/html/index.html
docker cp .\updates\volume-content-after.json pr5-volume-site:/usr/share/nginx/html/content.json
docker cp .\updates\volume-poster-after.svg pr5-volume-site:/usr/share/nginx/html/images/poster.svg
```

Точка монтирования `/usr/share/nginx/html` у `pr5-volume-site` подключена к named volume. Поэтому `docker cp` записывает файлы в том, и Nginx сразу отдает новую версию.

Второй способ — изменить HTML непосредственно командой внутри контейнера:

```powershell
docker compose exec -T volume-site sh -c "sed -i 's/ОБНОВЛЕНО В NAMED VOLUME/ОБНОВЛЕНО ЧЕРЕЗ КОНСОЛЬ/' /usr/share/nginx/html/index.html"
```

После обновления откройте `http://localhost:18082` и обновите страницу браузера. Тома переживают удаление контейнеров; `docker compose down -v` удаляет тома и данные.

## Bind mount и named volume

Bind mount связывает контейнер с конкретным каталогом хоста: файлы доступны и приложению, и пользователю непосредственно в проекте. Это удобно для разработки и редактирования, но требует существующего пути и зависит от ОС и прав доступа.

Named volume создаётся и хранится под управлением Docker. Его содержимое не привязано к папке исходников, сохраняется после удаления контейнера и удобно для данных приложения. Изменять его можно через контейнер (`docker cp`, `docker compose exec`) или временный контейнер с тем же томом. Для чтения/записи Docker монтирует том в файловую систему контейнера; исходный образ при изменении файлов пересобирать не требуется.

Остановка:

```powershell
docker compose down
```
