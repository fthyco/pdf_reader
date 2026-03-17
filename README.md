# PDF to Markdown — Chrome Extension

A Chrome extension that converts PDF documents into clean, structured Markdown — perfect for feeding into AI models, note-taking, or documentation. All processing happens **100% locally** on your machine.

![Chrome Extension](https://img.shields.io/badge/Chrome-Extension-4285F4?logo=googlechrome&logoColor=white)
![Python](https://img.shields.io/badge/Python-3.9+-3776AB?logo=python&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-green)

---

## ✨ Features

- **PDF → Markdown** conversion with full formatting preservation
- **LaTeX math** support — inline and block equations are converted to LaTeX syntax
- **OCR support** — works on scanned PDFs using AI-powered text recognition
- **Page selection** — convert specific pages or the entire document
- **Copy to clipboard** — one-click copy of the generated Markdown
- **100% local** — no data leaves your machine, fully offline after setup
- **Minimal dark theme** UI

---

## 🏗️ Architecture

```
┌──────────────────┐        HTTP POST         ┌─────────────────────┐
│  Chrome Extension │  ──────────────────────► │   Marker Bridge     │
│                    │       (PDF file)         │   (Python Server)   │
│  popup.js          │                          │   localhost:8001    │
│  content.js        │  ◄────────────────────  │                     │
│  markdown.js       │     JSON (Markdown)      │   marker-pdf (AI)   │
└──────────────────┘                           └─────────────────────┘
```

The extension has two parts:

### 1. Chrome Extension (Frontend)

| File | Purpose |
|------|---------|
| `popup.html` / `popup.js` | Extension UI — settings, convert button, output display |
| `content.js` | Injects into the PDF page to grab the PDF binary data |
| `marker-client.js` | Sends the PDF to the local Python server via HTTP |
| `markdown.js` | Post-processes and cleans up the returned Markdown |
| `math-utils.js` | Handles LaTeX math detection and formatting |
| `parser.js` | Text parsing utilities |
| `layout.js` | Layout analysis and formatting |
| `styles.css` | Dark-themed UI styles |

### 2. Marker Bridge (Backend)

`marker_bridge.py` is a lightweight **FastAPI** server that wraps the [`marker-pdf`](https://github.com/VikParuchuri/marker) library:

1. Receives a PDF file via `POST /convert`
2. Saves it to a temp file
3. Runs **marker-pdf** on it (AI-powered OCR + layout detection + text extraction)
4. Splits the output into per-page sections
5. Returns structured Markdown as JSON

---

## 🚀 Setup

### Prerequisites

- **Python 3.9+**
- **Google Chrome** (or Chromium-based browser)
- **pip** (Python package manager)

### 1. Install Python dependencies

```bash
pip install marker-pdf fastapi uvicorn python-multipart
```

### 2. Start the backend server

```bash
python marker_bridge.py
```

You should see:
```
Starting Marker Bridge on http://localhost:8001
Docs available at http://localhost:8001/docs
```

### 3. Load the Chrome Extension

1. Open Chrome and navigate to `chrome://extensions/`
2. Enable **Developer mode** (toggle in top-right corner)
3. Click **"Load unpacked"**
4. Select the project folder (where `manifest.json` is located)
5. The **PDF to Markdown** icon will appear in your toolbar

---

## 📖 Usage

1. Make sure the **Marker Bridge** server is running (`python marker_bridge.py`)
2. Open any PDF in Chrome
3. Click the **PDF to Markdown** extension icon
4. Configure options (page range, OCR toggle)
5. Click **Convert**
6. Copy or view the generated Markdown

---

## 📡 API Reference

The backend exposes two endpoints:

### `GET /health`

Health check — returns `{"status": "ok"}` if the server is running.

### `POST /convert`

Converts a PDF to Markdown.

| Parameter  | Type     | Default | Description |
|------------|----------|---------|-------------|
| `pdf_file` | File     | required | The PDF file to convert |
| `pages`    | string   | `""`    | Comma-separated 1-based page numbers (empty = all) |
| `force_ocr`| boolean  | `true`  | Force OCR on all lines (better for math/scanned docs) |

**Response:**
```json
{
  "success": true,
  "markdown": "# Full markdown output...",
  "pages": {
    "1": "# Page 1 content...",
    "2": "## Page 2 content..."
  },
  "total_pages": 2
}
```

---

## 🛠️ Tech Stack

- **Frontend:** Chrome Extension (Manifest V3), vanilla JavaScript, PDF.js
- **Backend:** Python, FastAPI, Uvicorn
- **AI/ML:** [marker-pdf](https://github.com/VikParuchuri/marker) (OCR, layout detection, LaTeX conversion)
