# 📄 pdf-read — Low-Token PDF Reading Skill for Claude Code

> Save 60–70% tokens when reading PDFs with Claude Code.  
> Automatically converts PDF to Markdown before AI reads it.

## The Problem

When you ask Claude Code to read a PDF directly, it consumes **5–10× more tokens** than reading the same content as plain text. This is because PDFs store page layout data (coordinates, fonts, lines), not semantic text.

| File (50 pages) | Tokens |
|---|---|
| PDF (direct read) | ~25,000 |
| Markdown (after conversion) | ~6,000 |
| **Savings** | **~76%** |

## The Solution

**pdf-read** is a Claude Code skill that intercepts PDF reading requests and:

1. Checks if a cached `.md` version already exists
2. Converts PDF → Markdown using Python/PyPDF2
3. Reads the Markdown (low token cost)
4. Reads embedded images on-demand only when needed

```
PDF → PyPDF2 → Markdown → Read (cheap)
                  │
                  └── images/ → Read on demand only
```

## Installation

### 1. Install Dependencies

```bash
pip install PyPDF2
```

### 2. Install the Skill

Copy the skill file to your Claude Code skills directory:

```bash
# User-level (available in all projects)
cp skills/pdf-read.md ~/.claude/skills/pdf-read.md

# Or project-level (available in current project only)
cp skills/pdf-read.md your-project/.claude/skills/pdf-read.md
```

### 3. Use It

In Claude Code, just type:

```
/pdf-read path/to/document.pdf
```

Or just ask Claude to read a PDF — the skill triggers automatically.

## How It Works

```
User: "Read this paper.pdf"
         │
    ┌────▼────┐
    │ Step 1  │  Check if paper.md exists & is newer
    └────┬────┘
         │
    ┌────▼────┐
    │ Step 2  │  pandoc (preferred) or PyPDF2 (fallback)
    │         │  paper.pdf → paper.md + paper_media/
    └────┬────┘
         │
    ┌────▼────┐
    │ Step 3  │  Read paper.md (low token!)
    └────┬────┘
         │
    ┌────▼────┐
    │ Step 4  │  Images? Read on-demand only
    └────┬────┘
         │
    ┌────▼────┐
    │ Step 5  │  Answer user's question
    └─────────┘
```

## Requirements

| Component | Required | Notes |
|---|---|---|
| Python 3.7+ | ✅ | |
| PyPDF2 | ✅ | `pip install PyPDF2` |
| pandoc | ⭐ Recommended | Better table/structure preservation |
| pdfplumber | ⭐ Optional | Better extraction for some PDFs |

## Supported PDF Types

| PDF Type | Support | Notes |
|---|---|---|
| Text-based PDF | ✅ Full | Most papers, reports, documents |
| Scanned PDF | ⚠️ Limited | Needs OCR preprocessing (`ocrmypdf`) |
| Chinese PDF | ✅ Full | PyPDF2 handles CJK text well |
| Forms / complex layouts | ⚠️ Partial | May lose some formatting |

## Real-World Test

Tested on a 14-page academic paper (混凝土裂缝缺陷识别):

| Metric | Direct PDF | With pdf-read |
|---|---|---|
| Token consumption | ~18,000 | ~6,000 |
| Content extracted | 14 pages | 14 pages ✅ |
| Images | Inline (expensive) | On-demand (cheap) ✅ |
| Tables | Mixed results | Preserved ✅ |

## Project Structure

```
claude-code-pdf-reader/
├── README.md           ← You are here
├── skills/
│   └── pdf-read.md     ← The skill definition
├── LICENSE             ← MIT
└── .gitignore
```

## Contributing

Found a bug? Have a PDF that doesn't convert well? Open an issue or PR!

## License

MIT © [Your Name]
