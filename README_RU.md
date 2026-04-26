# chatgpt-sandbox-env

> 🇬🇧 [Read in English](README.md)

Частичный снимок изолированной Linux-среды исполнения, которую использует ChatGPT (OpenAI) при обработке запросов с функцией code interpreter / computer use. Извлечено из живой сессии контейнера и опубликовано в исследовательских и сравнительных целях.

---

## Что внутри

### Общая информация о системе
| Параметр | Значение |
|----------|----------|
| ОС | Ubuntu 22.04.3 LTS (Jammy Jellyfish) — пакеты базы Debian 12 |
| Ядро | Linux 5.15.0 (инфраструктура Azure) |
| Пользователь | `oai` (домашняя директория: `/home/oai`) |
| Доступное хранилище | `/mnt/data` |
| Среда выполнения | OpenAI CAAS (Container As A Service) |

### `home/oai/` — Домашняя директория пользователя

**Конфиги оболочки** (`.bashrc`, `.profile`):
Стандартная Ubuntu bash-конфигурация с подключением NVM (Node Version Manager) — Node.js доступен через nvm.

**`redirect.html`** — Небольшая HTML-страница с редиректом в домашней директории.

**`.wgetrc`** — Конфигурация wget, вероятно используется для загрузки файлов внутри контейнера.

**`.chromium/`** — Профиль браузера Chromium (Default/Preferences, Local State), подтверждающий наличие headless Chromium для веб-задач.

**`.config/openbox/rc.xml`** — Конфигурация оконного менеджера Openbox, часть десктопного стека (Xvfb + Openbox + XFCE4).

**`.ipython/`** — Профиль IPython со стартовой конфигурацией и `history.sqlite` — база данных истории Jupyter/IPython ядра.

### `home/oai/skills/` — Встроенные наборы инструкций (скиллы)

Markdown и Python файлы, которые указывают модели как надёжно выполнять конкретные задачи. По концепции аналогичны системе skills у Claude.

| Скилл | Файлы | Описание |
|-------|-------|----------|
| `docs` | `skill.md`, `render_docx.py` | Создание и проверка Word-документов через `python-docx` + цикл рендеринга через LibreOffice |
| `pdfs` | `skill.md` | Создание PDF через `reportlab`, проверка через цикл PNG-рендера `pdftoppm` |
| `spreadsheets` | `skill.md`, `spreadsheet.md`, `artifact_tool_spreadsheets_api.md`, `artifact_tool_spreadsheet_formulas.md` + 15 примеров | Полная работа с таблицами через `openpyxl` и проприетарную библиотеку `artifact_tool` |

**Важная деталь:** Скилл spreadsheets явно инструктирует модель *не раскрывать* существование `artifact_tool` пользователям — это проприетарная внутренняя библиотека для рендеринга и пересчёта.

### `home/oai/share/slides/` — Инструментарий для создания слайдов

Полный пайплайн генерации PowerPoint:
- `render_slides.py` — основной рендерер слайдов
- `pptxgenjs_helpers/` — JavaScript-помощники для `pptxgenjs` (SVG, LaTeX, изображения, построители макетов)
- `ensure_raster_image.py` — конвертирует векторные изображения в растровые для вставки
- `create_montage.py` — создаёт монтажи из изображений
- `slides_test.py` — тест-сьют для генерации слайдов

### `openai/cua_chrome/policy_merge.py` — Объединитель политик Chrome

Часть инфраструктуры **CUA (Computer Use Agent)**. Скрипт объединяет несколько JSON-файлов корпоративных политик Chrome в единый `000_policy_merge.json` с резервными копиями. Используется для настройки и блокировки экземпляра Chromium внутри контейнера.

Путь в контейнере: `/openai/project/cua/cua_chrome/cua_chrome/core/policy_merge.py`

### `tmp/jupyter_kernel_connection.json` — Конфигурация Jupyter ядра

Данные живого подключения к IPython ядру на момент снятия снимка:

```json
{
  "shell_port": 49539,
  "iopub_port": 37165,
  "transport": "tcp",
  "signature_scheme": "hmac-sha256",
  "kernel_name": "python3"
}
```

Подтверждает, что среда выполнения Python работает как стандартное Jupyter ядро, общающееся по TCP на localhost.

### `var_log/` — Системные логи

**`apt_history.log`** — Полная история установки apt-пакетов. Ключевые пакеты:
- Инструменты сборки: `gcc`, `g++`, `cmake`, `ninja-build`, `make`
- Медиа: `ffmpeg`, `sox`, `lame`, `flac`, `espeak`
- OCR: `tesseract-ocr` + все языковые пакеты (100+ языков)
- GIS/данные: `gdal`, `proj`, `libgeos`, `libspatialite`, `libnetcdf`
- Клиенты БД: `libpq-dev` (PostgreSQL), `libmariadb-dev`, `unixodbc`
- Среды выполнения: `openjdk-17`, `php8.2`, `ruby-full`
- Рабочий стол: `xvfb`, `openbox`, `xfce4` (для headless браузера и GUI)

**`chrome_supervisord.log`** — Лог запуска supervisor, показывающий все процессы при загрузке контейнера:

| Процесс | Описание |
|---------|----------|
| `caas_package_mirror` | Внутреннее зеркало пакетов |
| `policy_merge` | Объединитель политик Chrome |
| `xvfb` | Виртуальный фреймбуфер (headless дисплей) |
| `openbox` | Оконный менеджер |
| `xfce4` | Окружение рабочего стола |
| `mitmproxy` | Прокси для перехвата сетевого трафика |
| `x11vnc` | VNC-сервер (удалённый рабочий стол) |
| `novnc_proxy` | Веб-клиент noVNC |
| `chromium` | Экземпляр браузера Chromium |
| `libreoffice` | LibreOffice (для конвертации документов) |
| `multikernel_jupyter_tool` | Управление мульти-ядерным Jupyter |
| `notebook_server` | Jupyter notebook сервер |
| `pdf_reader_service` | Сервис чтения PDF |
| `python_tool` | Инструмент выполнения Python |
| `terminal_server` | Сервер доступа к терминалу |
| `rsync_daemon` | Демон синхронизации файлов |
| `container_daemon` | Основной контроллер контейнера |
| `cua_bing_at_home` | Интеграция CUA с Bing |
| `nginx` | Веб-сервер / обратный прокси |

---

## Ключевые отличия от sandbox Claude

| Параметр | ChatGPT (OpenAI) | Claude (Anthropic) |
|----------|-----------------|-------------------|
| ОС | Ubuntu 22.04 | Ubuntu 24.04 |
| Ядро | Linux 5.15 (Azure) | Linux 4.4 (gVisor) |
| Инфраструктура | Azure / CAAS | gVisor контейнер |
| Рабочий стол | Xvfb + Openbox + XFCE4 | Отсутствует |
| Браузер | Chromium (полноценный) | Отсутствует |
| VNC доступ | x11vnc + noVNC | Отсутствует |
| Прокси | mitmproxy | Отсутствует |
| Скиллы | `/home/oai/skills/` | `/mnt/skills/` |
| Запись | `/mnt/data` | `/mnt/user-data/outputs` |
| Java | OpenJDK 17 | Отсутствует |
| PHP | 8.2 | Отсутствует |
| Ruby | 3.1 | Отсутствует |

---

## Лицензия

Этот экспорт предоставляется как есть для образовательных и исследовательских целей.
