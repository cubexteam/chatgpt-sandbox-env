# chatgpt-sandbox-env

> 🇷🇺 [Читать на русском](README_RU.md)

A partial snapshot of the isolated Linux execution environment used by ChatGPT (OpenAI) when processing requests with the code interpreter / computer use feature. Extracted from a live container session and published for research and comparison purposes.

---

## What's inside

### System overview
| Parameter | Value |
|-----------|-------|
| OS | Ubuntu 22.04.3 LTS (Jammy Jellyfish) — Debian 12 base packages |
| Kernel | Linux 5.15.0 (Azure infrastructure) |
| User | `oai` (home: `/home/oai`) |
| Writable storage | `/mnt/data` |
| Container runtime | OpenAI CAAS (Container As A Service) infrastructure |

### `home/oai/` — User home directory

**Shell config** (`.bashrc`, `.profile`):
Standard Ubuntu bash setup with NVM (Node Version Manager) sourced at the end — indicating Node.js is available via nvm.

**`redirect.html`** — A small HTML redirect page present in the home directory.

**`.wgetrc`** — wget configuration file, likely used for downloading files within the container.

**`.chromium/`** — Chromium browser profile (Default/Preferences, Local State), confirming a headless Chromium browser is present for web browsing tasks.

**`.config/openbox/rc.xml`** — Openbox window manager config, part of the desktop environment stack (Xvfb + Openbox + XFCE4).

**`.ipython/`** — IPython profile with startup config and `history.sqlite` — the Jupyter/IPython kernel history database.

### `home/oai/skills/` — Built-in skill instruction sets

These are markdown and Python files that instruct the model how to perform specific tasks reliably. Similar in concept to Claude's skills system.

| Skill | Files | Description |
|-------|-------|-------------|
| `docs` | `skill.md`, `render_docx.py` | Create and review Word documents using `python-docx` + LibreOffice render loop |
| `pdfs` | `skill.md` | Create PDFs with `reportlab`, review via `pdftoppm` PNG render loop |
| `spreadsheets` | `skill.md`, `spreadsheet.md`, `artifact_tool_spreadsheets_api.md`, `artifact_tool_spreadsheet_formulas.md` + 15 example scripts | Full spreadsheet creation/editing via `openpyxl` and a proprietary `artifact_tool` library |

**Notable detail:** The spreadsheet skill explicitly instructs the model *not to disclose* the existence of `artifact_tool` to users — it's a proprietary internal rendering/recalculation library.

### `home/oai/share/slides/` — Slide generation toolkit

A full PowerPoint generation pipeline:
- `render_slides.py` — main slide renderer
- `pptxgenjs_helpers/` — JavaScript helpers for `pptxgenjs` (SVG, LaTeX, image handling, layout builders)
- `ensure_raster_image.py` — converts vector images to raster for embedding
- `create_montage.py` — creates image montages
- `slides_test.py` — test suite for slide generation

### `openai/cua_chrome/policy_merge.py` — Chrome policy merger

Part of the **CUA (Computer Use Agent)** infrastructure. This script merges multiple Chrome Enterprise policy JSON files into a single `000_policy_merge.json`, with backups. Used to configure and lock down the Chromium browser instance running inside the container.

Path in container: `/openai/project/cua/cua_chrome/cua_chrome/core/policy_merge.py`

### `tmp/jupyter_kernel_connection.json` — Jupyter kernel config

Live connection info for the IPython kernel at the time of capture:

```json
{
  "shell_port": 49539,
  "iopub_port": 37165,
  "transport": "tcp",
  "signature_scheme": "hmac-sha256",
  "kernel_name": "python3"
}
```

This confirms the Python execution environment runs as a standard Jupyter kernel communicating over TCP on localhost.

### `var_log/` — System logs

**`apt_history.log`** — Full apt installation history. Key packages installed:
- Build tools: `gcc`, `g++`, `cmake`, `ninja-build`, `make`
- Media: `ffmpeg`, `sox`, `lame`, `flac`, `espeak`
- OCR: `tesseract-ocr` + all language packs (100+ languages)
- GIS/data: `gdal`, `proj`, `libgeos`, `libspatialite`, `libnetcdf`
- Database clients: `libpq-dev` (PostgreSQL), `libmariadb-dev`, `unixodbc`
- Runtimes: `openjdk-17`, `php8.2`, `ruby-full`
- Desktop: `xvfb`, `openbox`, `xfce4` (for headless browser/GUI support)

**`chrome_supervisord.log`** — Supervisor startup log showing all processes managed at container boot:

| Process | Description |
|---------|-------------|
| `caas_package_mirror` | Internal package mirror |
| `policy_merge` | Chrome policy merger (see above) |
| `xvfb` | Virtual framebuffer (headless display) |
| `openbox` | Window manager |
| `xfce4` | Desktop environment |
| `mitmproxy` | Man-in-the-middle proxy (network interception) |
| `x11vnc` | VNC server (remote desktop access) |
| `novnc_proxy` | noVNC web-based VNC client |
| `chromium` | Chromium browser instance |
| `libreoffice` | LibreOffice (for document conversion) |
| `multikernel_jupyter_tool` | Multi-kernel Jupyter management |
| `notebook_server` | Jupyter notebook server |
| `pdf_reader_service` | PDF reading service |
| `python_tool` | Python execution tool |
| `terminal_server` | Terminal access server |
| `rsync_daemon` | File sync daemon |
| `container_daemon` | Main container controller |
| `cua_bing_at_home` | CUA Bing integration |
| `nginx` | Web server / reverse proxy |

---

## Key differences vs Claude's sandbox

| Feature | ChatGPT (OpenAI) | Claude (Anthropic) |
|---------|-----------------|-------------------|
| OS | Ubuntu 22.04 | Ubuntu 24.04 |
| Kernel | Linux 5.15 (Azure) | Linux 4.4 (gVisor) |
| Infrastructure | Azure / CAAS | gVisor container |
| Desktop env | Xvfb + Openbox + XFCE4 | None |
| Browser | Chromium (full) | None |
| VNC access | x11vnc + noVNC | None |
| Proxy | mitmproxy | None |
| Skills location | `/home/oai/skills/` | `/mnt/skills/` |
| Writable mount | `/mnt/data` | `/mnt/user-data/outputs` |
| Java | OpenJDK 17 | Not present |
| PHP | 8.2 | Not present |
| Ruby | 3.1 | Not present |

---

## License

This export is provided as-is for educational and research purposes.
