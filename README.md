# 🚀 Mega Agent MCP

[![Production Ready](https://img.shields.io/badge/Status-Production_Ready-success.svg)](#)
[![Python 3.11+](https://img.shields.io/badge/Python-3.11%2B-blue.svg)](https://www.python.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![MCP Compatible](https://img.shields.io/badge/MCP-Compatible-purple.svg)](#)
[![Docker Supported](https://img.shields.io/badge/Docker-Supported-2496ED.svg)](#)

**Mega Agent MCP** is a production-ready Model Context Protocol (MCP) server designed to provide Large Language Models with powerful external capabilities including web search, article extraction, GitHub analysis, document parsing, browser automation, secure code execution, and intelligent forum parsing.

The project focuses on providing a fast, lightweight, and self-hostable backend that works with any MCP-compatible client or AI model.

---

## 🤔 Why Mega Agent MCP?

Unlike many MCP servers that implement only one or two tools, Mega Agent MCP combines numerous capabilities into a single optimized server while maintaining **security, performance, and low resource consumption**. 

It eliminates the need to run 5 different MCP servers by providing a unified, async-first AI backend equipped with its own SQLite caching, Docker sandboxing, and rate-limiting.

*(Insert Demo GIF here)*

---

## 🧰 Tool Categories

The server exposes a highly organized set of tools divided by their domain:

```text
🌐 Internet
 ├── web_search
 ├── fetch_page
 ├── search_and_read
 └── browse_url

📚 GitHub
 ├── fetch_github_repo
 └── fetch_github_file

💬 Forums
 └── fetch_thread

📄 Documents
 ├── read_document
 └── render_html_css

💻 Execution
 ├── execute_code
 └── get_current_time
```

---

## ✨ Features

### 🌐 Web Search & Reading
* **Web Search**: Searches the internet through SearXNG with language selection and BM25 ranking.
* **Article Reader**: Extracts clean content using a robust cascade parser (`Trafilatura` → `Readability` → `jusText` → `BeautifulSoup`). Automatically removes ads, sidebars, and scripts.
* **Smart Search + Read**: A Perplexity-style pipeline. Searches → BM25 Ranking → Downloads → Extracts → Merges the top results.
* **JavaScript Browser**: Uses Playwright Chromium for JS-heavy websites (React, Vue, SPAs).

### 📚 GitHub Integration
* **Repository Reader**: Analyzes repository trees, extracts `README.md`, and detects default branches via the GitHub REST API without cloning.
* **File Reader**: Downloads and reads individual source code files or documentation directly from public repositories.

### 💬 Intelligent Forum Parsing
* **Reddit Parser**: Recursively parses nested Reddit discussions through the JSON API. Supports sorting, time filters, and internal BM25 search.
* **4PDA Parser**: Reads entire 4PDA forum topics. Supports `recent`, `full thread`, and `first/last page` modes while skipping duplicated header posts.

### 💻 Secure Code Sandbox
Runs code in **strictly isolated Docker containers** (Python, C, C++, Java, Node.js, PHP).
* **Security Layers**: Network disabled, Read-only filesystem, Non-root user (`nobody`), Capability dropping, Seccomp profile support, `no-new-privileges`, CPU/RAM/PID limits, and `tmpfs` mounts.

### 📄 Document & Visual Processing
* **Document Reader**: Parses PDF, DOCX, PPTX, XLSX, XLS, CSV, and ODS. Features smart PDF text extraction with block ordering.
* **Smart OCR**: Uses Tesseract OCR. If a PDF is a scanned image, it automatically falls back to rendering the page and extracting text visually.
* **HTML Renderer**: Uses Playwright to render raw HTML/CSS and generate screenshots (useful for AI UI generation and previews).

### ⚡ Performance & Caching
* **Asynchronous Architecture**: Fully async HTTP calls, parallel downloads, and database operations.
* **Database Cache**: SQLite cache with WAL mode automatically stores downloaded pages (TTL-based) to drastically reduce network overhead.
* **BM25 Retrieval**: Built-in, RAM-cached semantic-like ranking without requiring heavy GPU embeddings.

---

## 🛡️ Security
Mega Agent MCP includes enterprise-grade security layers:
* **SSRF Protection**: Strict URL validation. Blocks loopback, private networks, multicast, and link-local addresses (IPv4 & IPv6).
* **Input Validation**: Hard limits on request payloads (Max 1 MB input) and file sizes (Max 50 MB documents).
* **Rate Limiting**: Built-in sliding window rate limiting prevents abuse by malicious clients.

---

## 🛠️ Installation

### 1. Clone the repository
```bash
git clone https://github.com/PanPersil/MegaAgent-MCP.git
cd MegaAgent-MCP
```

### 2. Install Dependencies
Make sure you have Python 3.11+ installed. It's recommended to use a virtual environment.
```bash
# Create and activate virtual environment
python -m venv venv
source venv/bin/activate  # On Windows use `venv\Scripts\activate`

# Install python packages
pip install -r requirements.txt

# Install Playwright browser for rendering
playwright install chromium
```

*Note: For OCR and document support, you may need system packages like `tesseract-ocr` and `poppler-utils` depending on your OS.*

### 3. Start SearXNG (Web Search Backend)
We use SearXNG as the private search engine backend. Set it up easily using Docker:
```bash
# Create a dummy settings file if you don't have a custom one
touch settings.yml

# Run SearXNG container
docker run -d   --name searxng   -p 8888:8080   -v $(pwd)/settings.yml:/etc/searxng/settings.yml   searxng/searxng

# Enable auto-restart so it survives reboots
docker update --restart unless-stopped searxng
```

### 4. Run the Server
```bash
python server.py
```
The server will start listening for MCP connections via Streamable-HTTP/WebSocket on `http://0.0.0.0:8100/mcp`.

---

## 🧰 Available Tools Reference

| Tool Name | Category | Description |
|-----------|----------|-------------|
| `web_search` | 🌐 Internet | Standard internet search using SearXNG. |
| `fetch_page` | 🌐 Internet | Reads an article and extracts pure text. |
| `search_and_read` | 🌐 Internet | Searches and automatically reads the top N pages. |
| `browse_url` | 🌐 Internet | JavaScript browser for dynamic SPAs. |
| `fetch_github_repo` | 📚 GitHub | Repository analysis and README extraction. |
| `fetch_github_file` | 📚 GitHub | Reads specific files from a repository. |
| `fetch_thread` | 💬 Forums | Deep parser for Reddit / 4PDA discussions. |
| `execute_code` | 💻 Execution | Secure Docker code execution (Python/C/C++/Java/JS/PHP). |
| `get_current_time` | 💻 Execution | Returns accurate system time. |
| `render_html_css` | 📄 Documents | Renders HTML code to an image screenshot. |
| `read_document` | 📄 Documents | Parses PDF, Word, Excel, PPTX (with OCR). |

---

## 🎯 Intended Use Cases
* **Local AI Assistants**: Connect to `llama.cpp`, Ollama, or LM Studio.
* **Coding Agents**: Automate GitHub exploration and execute code safely.
* **Research Assistants**: Aggregate knowledge, read PDFs, and parse forums.
* **Self-Hosted AI Systems**: A private, no-telemetry backend for your LLMs.

---

## 📈 Project Status
**Current status: Production Ready**

Mega Agent MCP has been designed to be stable, secure, and practical for real-world usage. While the project is considered production-ready and actively maintained, further improvements, optimizations, and community contributions are always welcome!

---

## 📄 License
This project is licensed under the MIT License - see the LICENSE file for details.
