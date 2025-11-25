# BookTures


### Automated Page-Wise Illustration Generator for Books

Booktures converts any uploaded PDF book into a visually enriched reading experience by generating an AI-generated illustration for **every page**, using that page’s text and its surrounding context.
Readers can then view each PDF page alongside its corresponding generated artwork in a clean, modern UI.

---

## Features

### Page-level Text Extraction

* Uses PyMuPDF for fast, reliable PDF parsing.
* Automatically detects and handles scanned pages with optional OCR.
* Extracts paragraphs, removes boilerplate headers/footers, and preserves reading flow.

### Context-Aware Image Generation

* Generates illustrations using Stable Diffusion / SDXL via Hugging Face diffusers.
* Each image uses:

  * the current page’s text
  * summaries of previous pages
  * optional reference images for style or character consistency
* Optional ControlNet for layout control and continuity between pages.

### Complete Web App

* FastAPI backend for processing, job management and image streaming
* React frontend with PDF.js viewer for side-by-side page and illustration display
* Live status tracking for multi-page generation jobs

---

## Architecture Overview

Booktures is built as a modular system:

```
                ┌────────────────────────────┐
                │          Frontend           │
                │  React + PDF.js Viewer      │
                └──────────────┬─────────────┘
                               │
                    HTTP (REST API)
                               │
                ┌──────────────┴──────────────┐
                │            Backend           │
                │        FastAPI Server        │
                ├──────────────┬──────────────┤
                │              │               │
         PDF Extraction   Prompt Builder   Job Manager
         (PyMuPDF/OCR)     + Context AI     (async)
                │              │
                └──────────────┬──────────────┘
                               │
                ┌──────────────┴──────────────┐
                │     Image Generation Engine  │
                │   Stable Diffusion / SDXL    │
                │   ControlNet (optional)      │
                └──────────────┬──────────────┘
                               │
                     Per-page Images Stored
```

---

## Technology Choices

### PDF to Text

* **PyMuPDF (fitz)**: fastest and most accurate for structured PDFs
* Optional OCR via Tesseract or Azure Document Intelligence for scanned pages

### Image Generation

* Hugging Face **diffusers** pipelines
* Supported models:

  * `stabilityai/stable-diffusion-xl-base-1.0`
  * `runwayml/stable-diffusion-v1-5`
  * Custom LoRA or Textual Inversion embeddings

### Context Modeling

* Previous pages summarized using:

  * Sentence-Transformers (MiniLM) for lightweight summarization, or
  * any external LLM for detailed context condensation

### FastAPI Backend

* Handles uploads, job orchestration, per-page processing, and image streaming
* Optionally integrates with Celery + Redis for distributed processing

### React Frontend

* PDF.js to render book pages
* Displays AI-generated illustration beside each page
* Navigation controls and progress indicator

---

## Installation and Setup

### 1. Clone the Repository

```
git clone https://github.com/yourusername/booktures.git
cd booktures
```

### 2. Backend Installation

```
cd backend
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

Install PyTorch with CUDA following the instructions at pytorch.org.

### 3. Run FastAPI

```
uvicorn app:app --host 0.0.0.0 --port 8000
```

### 4. Frontend Installation

```
cd ../frontend
npm install
npm start
```

---

## How Booktures Works

### Step 1: Upload a PDF

User uploads a book via the FastAPI endpoint `/upload`.

### Step 2: Page Extraction

Text is extracted per page. Scanned pages are routed through OCR.

### Step 3: Context Building

The system collects:

* current page text
* summaries of up to 3 previous pages
* global style brief
  to build a unified prompt.

### Step 4: Illustration Generation

For each page:

* a prompt is generated
* Stable Diffusion (or SDXL) generates the image
* images are stored under `/data/{job_id}/page_XXXX.png`

### Step 5: Frontend Display

React UI displays:

* PDF page on left (PDF.js)
* generated artwork on right
* navigation between pages
* automatic refresh as new images are ready

---

## API Reference

### POST `/upload`

Upload a PDF and start a generation job.

```
curl -X POST -F "file=@book.pdf" http://localhost:8000/upload
```

Returns:

```
{"job_id": "uuid", "status": "queued"}
```

### GET `/job/{job_id}`

Returns job state, current page, total pages, and metadata.

### GET `/job/{job_id}/page/{page_number}`

Returns the generated image for that page.

---

## Configuration

### Environment Variables

| Name                | Purpose                         |
| ------------------- | ------------------------------- |
| `MODEL_ID`          | Diffusion model to load         |
| `DEVICE`            | cpu / cuda                      |
| `OCR_ENABLED`       | Enable OCR fallback             |
| `MAX_CONTEXT_PAGES` | Number of previous pages to use |

### Prompt Template Customization

Located in:

```
backend/prompt_templates/default.txt
```

---

## Project Structure

```
booktures/
│
├── backend/
│   ├── app.py
│   ├── workers/
│   ├── core/
│   ├── prompt_templates/
│   ├── data/
│   └── requirements.txt
│
├── frontend/
│   ├── src/
│   ├── public/
│   └── package.json
│
└── README.md
```

---

## Roadmap

* Character-consistent illustrations using LoRA or embeddings
* Style selector and per-page re-generate
* Cloud GPU support (RunPod, Lambda Labs, Modal)
* Sharing pages publicly
* Annotation and commentary layer
* EPUB support

---

## License

 MIT License

---

## Contributing

Pull requests are welcome. Please open an issue to discuss major changes.

---

