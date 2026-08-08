# dns320-reface

**English** | [Русский](#русский)

Custom lightweight web UI for the D-Link DNS-320 NAS, replacing the stock interface.

## Why

The DNS-320 (2011–2012) is a solid little file storage box, but its stock web interface is outdated. The hardware is too weak for any modern stack — so instead of fighting it, this project embraces it.

## Hardware constraints

| Component | Spec |
|-----------|------|
| CPU | Marvell Kirkwood 88F6281, ARMv5, 800 MHz |
| RAM | 128 MB |
| Web server (stock) | lighttpd + PHP 5.2/5.3 |
| Data disks | `/mnt/HD/HD_a2/` (disk 1), `/mnt/HD/HD_b2/` (disk 2) |

Any modern backend (Node.js, Go, Python 3) would strangle this box. Compiling for ARMv5 is a dead end today.

## Architecture

**Static frontend + thin PHP API, no compilation at all.**

- **Backend:** a single `api.php` in the NAS web server folder, written in plain PHP 5 syntax. It only reads directories (`scandir()`), serves and accepts files, and returns JSON. Interpreted — edit and it just runs.
- **Frontend:** pure HTML/JS/CSS built as a static single-page app. All rendering happens in the client browser; the NAS CPU only hands out JSON and file bytes.
- **Development loop:** the frontend is developed on a PC (live server), talking over the network to the real `api.php` on the NAS. No cross-compilation, no SSH deploy cycle. When ready, the static files are copied next to `api.php` — that's the whole hosting.

## Data safety during development

- All backend work is jailed to a sandbox folder: `$root = "/mnt/HD/HD_a2/test_space/";`
- No `unlink()` / `rmdir()` in the code until the UI is complete; read-only first, write/delete added last and tested on throwaway files.

## Roadmap

1. **Read** — connect to the NAS, list folders as JSON, render a folder tree.
2. **Streaming** — serve files with partial content support (HTTP 206) so media can be seeked in the browser.
3. **Write** — upload (multipart/form-data), create folders, delete.
4. **Optimization** — indexer script that periodically scans the disks into a local database for instant search.

## Alternative path: Alt-F firmware

[Alt-F](https://sites.google.com/site/altfirmware/) is a free open-source firmware for D-Link DNS-320/321/323/325. It replaces the stock system with a Debian-like Linux (proper SSH, `opkg` package manager, fresher PHP/web servers) while keeping the data on the disks intact. It can be tried risk-free in RAM-only mode (`alt-f.bin` loaded via the stock firmware-update page; a power cycle reverts to stock). This project targets the stock firmware first; Alt-F remains the fallback if the stock web server proves too restrictive.

---

# Русский

[English](#dns320-reface) | **Русский**

Свой лёгкий веб-интерфейс для NAS D-Link DNS-320 взамен штатного.

## Зачем

DNS-320 (2011–2012) — нормальное файловое хранилище, но штатный веб-интерфейс устарел. Железо не тянет современный стек — поэтому проект не борется с ним, а отталкивается от него.

## Ограничения железа

| Компонент | Характеристика |
|-----------|----------------|
| Процессор | Marvell Kirkwood 88F6281, ARMv5, 800 МГц |
| ОЗУ | 128 МБ |
| Веб-сервер (сток) | lighttpd + PHP 5.2/5.3 |
| Диски с данными | `/mnt/HD/HD_a2/` (первый), `/mnt/HD/HD_b2/` (второй) |

Любой современный бэкенд (Node.js, Go, Python 3) задушит эту коробку. Компиляция под ARMv5 сегодня — тупик.

## Архитектура

**Статический фронтенд + тонкий PHP-API, вообще без компиляции.**

- **Бэкенд:** один файл `api.php` в веб-папке NAS, на старом синтаксисе PHP 5. Умеет только читать папки (`scandir()`), отдавать и принимать файлы, отвечать JSON'ом. Интерпретируемый — написал строку, она работает.
- **Фронтенд:** чистый HTML/JS/CSS, собранный в статический single-page app. Вся отрисовка — в браузере клиента; процессор NAS лишь отдаёт JSON и байты файлов.
- **Процесс разработки:** фронтенд пишется на ПК (live server) и по сети ходит к реальному `api.php` на NAS. Никакой кросс-компиляции и деплой-цикла по SSH. Когда готово — статика копируется рядом с `api.php`, это и есть весь хостинг.

## Безопасность данных при разработке

- Бэкенд жёстко залочен на песочницу: `$root = "/mnt/HD/HD_a2/test_space/";`
- Ни `unlink()`, ни `rmdir()` в коде, пока интерфейс не готов; сначала только чтение, запись и удаление добавляются последними и проверяются на мусорных файлах.

## План работ

1. **Чтение** — подключиться к NAS, отдавать список папок JSON'ом, отрисовать дерево папок.
2. **Стриминг** — отдача файлов с partial content (HTTP 206), чтобы медиа перематывалось в браузере.
3. **Запись** — загрузка файлов (multipart/form-data), создание папок, удаление.
4. **Оптимизация** — скрипт-индексатор, периодически сканирующий диски в локальную базу для мгновенного поиска.

## Запасной путь: прошивка Alt-F

[Alt-F](https://sites.google.com/site/altfirmware/) — бесплатная open-source прошивка для D-Link DNS-320/321/323/325. Заменяет сток на Debian-подобный Linux (нормальный SSH, пакетный менеджер `opkg`, более свежие PHP и веб-серверы), сохраняя данные на дисках. Пробуется без риска в бездисковом режиме (файл `alt-f.bin` подсовывается через штатную страницу обновления прошивки; после выключения питания NAS возвращается на сток). Проект сначала целится в штатную прошивку; Alt-F — запасной вариант, если стоковый веб-сервер окажется слишком зажатым.
