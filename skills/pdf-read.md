---
name: pdf-read
description: Low-token PDF reading — auto-convert to Markdown first, read images on demand
---

# pdf-read — Low Token PDF Reading

## Trigger

This skill activates when the user asks to read, analyze, or summarize a PDF file, or when `/pdf-read <file>` is invoked.

---

## Workflow

### Step 1: Check Cache

For the target PDF `<name>.pdf`, check if `<name>.md` exists in the same directory:

- **If `.md` exists AND is newer than `.pdf`** → skip to Step 4, read the cached `.md`
- **Otherwise** → proceed to Step 2

```bash
test -f "<md_path>" && test "<md_path>" -nt "<pdf_path>" && echo "CACHE_HIT" || echo "NEED_CONVERT"
```

### Step 2: Convert PDF → Markdown

**Preferred — pandoc (best structure preservation: tables, headings, lists):**

```bash
pandoc "<pdf_path>" -o "<md_path>" --extract-media="<media_dir>" --wrap=none
```

- `<pdf_path>` — full path to the original PDF
- `<md_path>` — output path: `<original_name>.md`
- `<media_dir>` — image extraction directory: `<original_name>_media/`

**Fallback — if pandoc unavailable or conversion fails (exit code ≠ 0):**

Use Python with PyPDF2:

```bash
python -c "
import os
from PyPDF2 import PdfReader

reader = PdfReader('<pdf_path>')
with open('<md_path>', 'w', encoding='utf-8') as f:
    for i, page in enumerate(reader.pages, 1):
        text = page.extract_text()
        if text:
            f.write(f'## Page {i}\n\n')
            f.write(text + '\n\n')

print(f'Pages: {len(reader.pages)}')
print(f'Output: {os.path.getsize(\"<md_path>\")} bytes')
"
```

If PyPDF2 is also unavailable, try pdfplumber as a second fallback:

```bash
python -c "
import pdfplumber

with pdfplumber.open('<pdf_path>') as pdf:
    with open('<md_path>', 'w', encoding='utf-8') as f:
        for i, page in enumerate(pdf.pages, 1):
            text = page.extract_text()
            if text:
                f.write(f'## Page {i}\n\n')
                f.write(text + '\n\n')
"
```

### Step 3: Verify Conversion

- Check that `<md_path>` was generated and is non-empty
- If the `.md` file is empty or near-empty, the PDF is likely a **scanned/image-based PDF**
- In this case, inform the user: "This appears to be a scanned PDF. OCR preprocessing is needed (e.g., `ocrmypdf`)."
- Do NOT delete the generated files — they serve as cache for next time

### Step 4: Read Markdown

Use the Read tool to read `<md_path>`.

**Important:** Do NOT read the original PDF file — that is the entire purpose of this skill.

### Step 5: Read Images On-Demand

If the Markdown contains image references like `![...](<media_dir>/imageXXX.png)`:

- **Do NOT read all images at once**
- **Wait for the user to ask about image content**, OR
- **Proactively read relevant images** if the user's question clearly requires visual information (e.g., "what does this chart show?")
- Use the Read tool to read individual image files

### Step 6: Answer

Answer the user's question based on the Markdown text + any images read on-demand.

---

## Notes

1. **Never Read PDF directly** — the sole reason this skill exists
2. **Don't skip long `.md` files** — their token cost is still far lower than PDF
3. **Don't delete `.md` and `media/`** after reading — they serve as cache for future reads
4. **Scanned PDFs** — if conversion produces empty output, suggest OCR (`ocrmypdf`)
5. **Chinese/ CJK PDFs** — PyPDF2 handles CJK text well; pandoc may produce garbled output for CJK
