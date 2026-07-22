# Mega Agent MCP

[![Production Ready](https://img.shields.io/badge/Status-Production_Ready-success.svg)](#)
[![Python 3.11+](https://img.shields.io/badge/Python-3.11%2B-blue.svg)](https://www.python.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![MCP Compatible](https://img.shields.io/badge/MCP-Compatible-purple.svg)](#)
[![Docker Supported](https://img.shields.io/badge/Docker-Supported-2496ED.svg)](#)

**Mega Agent MCP** is a production-ready Model Context Protocol (MCP) server that gives Large Language Models a wide set of external capabilities: web search, article extraction, GitHub analysis, document parsing, browser automation, secure code execution, and forum parsing.

The project focuses on being a fast, lightweight, and self-hostable backend that works with any MCP-compatible client or AI model.

---

## Why Mega Agent MCP

Most MCP servers implement one or two tools. Mega Agent MCP combines many capabilities into a single optimized server while keeping security, performance, and resource usage under control.

It removes the need to run five different MCP servers by providing one unified, async-first backend with its own SQLite caching, Docker sandboxing, and rate limiting.

*(Demo GIF placeholder)*

---

## Architecture

```
LLM
 │
 ▼
MCP Client
 │
 ▼
Mega Agent MCP
 ├── Web Search
 ├── GitHub
 ├── Docker Sandbox
 ├── Browser
 ├── OCR
 ├── Documents
 └── SQLite Cache
```

---

## Tool Categories

```
Internet
 ├── web_search
 ├── fetch_page
 ├── search_and_read
 └── browse_url

GitHub
 ├── fetch_github_repo
 └── fetch_github_file

Forums
 └── fetch_thread

Documents
 ├── read_document
 └── render_html_css

Execution
 ├── execute_code
 └── get_current_time
```

---

## Features

**Web Search & Reading**
- Web Search — searches the internet through SearXNG with language selection and BM25 ranking.
- Article Reader — extracts clean content via a cascade parser (`Trafilatura` → `Readability` → `jusText` → `BeautifulSoup`). Automatically strips ads, sidebars, and scripts.
- Smart Search + Read — a Perplexity-style pipeline: search → BM25 ranking → download → extract → merge top results.
- JavaScript Browser — uses Playwright Chromium for JS-heavy sites (React, Vue, SPAs).

**GitHub Integration**
- Repository Reader — analyzes repo trees, extracts `README.md`, and detects default branches via the GitHub REST API without cloning.
- File Reader — downloads and reads individual source files or docs directly from public repositories.

**Forum Parsing**
- Reddit Parser — recursively parses nested Reddit discussions via the JSON API, with sorting, time filters, and internal BM25 search.
- 4PDA Parser — reads full 4PDA forum topics, supporting `recent`, `full thread`, and `first/last page` modes while skipping duplicate header posts.

**Secure Code Sandbox**
Runs code in strictly isolated Docker containers (Python, C, C++, Java, Node.js, PHP).
- Security layers: network disabled, read-only filesystem, non-root user (`nobody`), capability dropping, seccomp profile support, `no-new-privileges`, CPU/RAM/PID limits, `tmpfs` mounts.

**Document & Visual Processing**
- Document Reader — parses PDF, DOCX, PPTX, XLSX, XLS, CSV, and ODS, with smart PDF text extraction and block ordering.
- Smart OCR — uses Tesseract OCR; if a PDF is a scanned image, falls back to rendering the page and extracting text visually.
- HTML Renderer — uses Playwright to render raw HTML/CSS and generate screenshots (useful for AI UI generation and previews).

---

## Performance

- Fully asynchronous architecture — async HTTP calls, parallel downloads, async DB operations
- SQLite cache with WAL mode, TTL-based, to cut network overhead
- Persistent Playwright browser sessions
- RAM-cached BM25 retrieval — lightweight, semantic-like ranking without heavy GPU embeddings
- Minimal memory footprint

---

## Security

Mega Agent MCP includes several security layers by default:
- **SSRF protection** — strict URL validation; blocks loopback, private networks, multicast, and link-local addresses (IPv4 and IPv6)
- **Input validation** — hard limits on request payloads (max 1 MB input) and file sizes (max 50 MB documents)
- **Rate limiting** — sliding-window rate limiting to prevent abuse

---

## Installation

### 1. Clone the repository
```bash
cd ~
git clone https://github.com/PanPersil/MegaAgent-MCP.git
cd MegaAgent-MCP
```

### 2. Install dependencies
Requires Python 3.11+. A virtual environment is recommended.
```bash
# Create and activate virtual environment
python -m venv venv
source venv/bin/activate  # On Windows use `venv\Scripts\activate`

# Install python packages
pip install -r requirements.txt

# Install Playwright browser for rendering
playwright install chromium
```


### 3. System dependencies
Not everything is installable via pip. Mega Agent MCP also requires:
- Docker
- Chromium (installed automatically via `playwright install chromium`, *Linux note*: if you encounter an `error while loading shared libraries`, run `sudo playwright install-deps chromium`)
- Tesseract OCR
- SearXNG

On Debian/Ubuntu:
```bash
sudo apt install tesseract-ocr tesseract-ocr-rus tesseract-ocr-eng
```

### 4. Start SearXNG (web search backend)
```bash
# Create directory to SearXNG service
mkdir ~/searxng && cd ~/searxng

# Copy settings.yml from the cloned repository
cp ~/MegaAgent-MCP/settings.yml ~/searxng/settings.yml

# Run SearXNG container
docker run -d --name searxng -p 8888:8080 -v $(pwd)/settings.yml:/etc/searxng/settings.yml searxng/searxng

# Enable auto-restart so it survives reboots
docker update --restart unless-stopped searxng
```

### 5. Run the server
```bash
python server.py
```
The server listens for MCP connections via Streamable-HTTP/WebSocket on `http://0.0.0.0:8100/mcp`.

---

## Available Tools Reference

| Tool Name | Category | Description |
|---|---|---|
| `web_search` | Internet | Standard internet search using SearXNG. |
| `fetch_page` | Internet | Reads an article and extracts pure text. |
| `search_and_read` | Internet | Searches and automatically reads the top N pages. |
| `browse_url` | Internet | JavaScript browser for dynamic SPAs. |
| `fetch_github_repo` | GitHub | Repository analysis and README extraction. |
| `fetch_github_file` | GitHub | Reads specific files from a repository. |
| `fetch_thread` | Forums | Deep parser for Reddit / 4PDA discussions. |
| `execute_code` | Execution | Secure Docker code execution (Python/C/C++/Java/JS/PHP). |
| `get_current_time` | Execution | Returns accurate system time. |
| `render_html_css` | Documents | Renders HTML code to an image screenshot. |
| `read_document` | Documents | Parses PDF, Word, Excel, PPTX (with OCR). |

---

## Comparison

| Feature | Mega Agent MCP | Typical MCP Server |
|---|---|---|
| Web Search | Yes | Partial |
| GitHub Integration | Yes | No |
| OCR | Yes | No |
| Docker Sandbox | Yes | No |
| Forum Parsing | Yes | No |
| Browser Rendering | Yes | Partial |

---

## Roadmap

- [x] Web search
- [x] GitHub support
- [x] Docker sandbox
- [x] OCR
- [x] Browser automation

---


## Intended Use Cases

- **Local AI Assistants** — connect to `llama.cpp`, Ollama, or LM Studio.
- **Coding Agents** — automate GitHub exploration and execute code safely.
- **Research Assistants** — aggregate knowledge, read PDFs, and parse forums.
- **Self-Hosted AI Systems** — a private, no-telemetry backend for your LLMs.

---

## Project Status

**Current Status: Production Ready**

Mega Agent MCP is considered production-ready for self-hosted and local AI deployments. While actively maintained and stable, minor bugs or edge cases may still exist. Contributions, issue reports, and feature suggestions are always welcome.

---

## License

This project is licensed under the MIT License — see the LICENSE file for details.
