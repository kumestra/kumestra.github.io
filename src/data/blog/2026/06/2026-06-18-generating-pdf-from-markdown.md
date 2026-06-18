---
title: "Generating a PDF from Markdown"
description: "A practical note on turning Markdown into a readable PDF, comparing direct PDF generation with a browser-based workflow."
pubDatetime: 2026-06-18T01:27:56Z
tags: [markdown, pdf, automation, playwright]
---

Turning a short Markdown answer into a PDF sounds simple until the output needs to look decent. In this run, the main problem was not creating *a* PDF. It was creating one with readable Chinese text, natural mixed Chinese/English wrapping, clear code blocks, and clean pagination. 📄

## Starting Point

The source was a Markdown file with three short technical answers. It included:

- Chinese prose mixed with English terms such as `Dockerfile`, `requirements.txt`, `Python`, and `Django`.
- A small shell command block.
- Section headings that needed to stay visually distinct.

The first PDF was generated with a low-level Python approach. It was readable, but the quality was poor: spacing was awkward, inline code stretched strangely, and the page break made the final section feel disconnected.

## Direct PDF Generation

The next approach used a temporary `uv` environment with `reportlab`. The environment was created under `/tmp`, dependencies were installed there, and the resulting PDF was rendered back into images for review.

That workflow looked roughly like this:

```bash
UV_CACHE_DIR=/tmp/... uv venv /tmp/...-venv
UV_CACHE_DIR=/tmp/... uv pip install --python /tmp/...-venv/bin/python reportlab pymupdf
```

`reportlab` generated the PDF, and `PyMuPDF` rendered it into images so the result could be inspected visually.

This worked, but it required manual control over:

- Page margins
- Font choice
- Line wrapping
- Code block drawing
- Pagination

Several iterations were needed. One version wrapped too early. Another used the full line width better, but left punctuation alone on a new line. The final `reportlab` version fixed that by widening the content area and changing the wrapping logic so closing punctuation stayed with the previous line.

## Why Browser Rendering Looked Promising 🌐

The natural next idea was to use a browser. A browser layout engine is usually better at:

- Mixed Chinese/English text
- CSS-based margins and line height
- Code block styling
- Font fallback
- Print-to-PDF pagination

The proposed browser workflow was:

1. Convert Markdown to HTML.
2. Add print-focused CSS.
3. Use headless Chromium to print the HTML to PDF.
4. Render the PDF back to images for review.
5. Delete all temporary environment files.

The environment was kept isolated by setting paths like:

```bash
UV_CACHE_DIR=/tmp/...
PLAYWRIGHT_BROWSERS_PATH=/tmp/...
```

That made cleanup straightforward because the virtual environment, package cache, browser binaries, generated HTML, and review images all lived under known `/tmp` directories.

## What Happened With Playwright

The browser attempt partially worked:

- A temporary `uv` environment was created.
- `playwright` was installed successfully.
- Chromium downloaded successfully into the temporary browser directory.
- The HTML file was generated.

But the Chromium print step hung and had to be interrupted. The existing PDF was not overwritten. Afterward, the temporary virtual environment, package cache, browser binaries, and work directory were deleted and verified as removed.

That result was useful even though the browser route did not complete: it confirmed that environment isolation and cleanup were manageable, while also showing that browser rendering can be heavier and less predictable on a minimal server.

## Practical Takeaways

For a lightweight Markdown-to-PDF job, direct PDF generation can be enough if the document is short and the layout is controlled carefully. It gives predictable cleanup and avoids downloading a full browser.

For the best visual quality, browser rendering is still the better target. CSS and Chromium should produce more natural layout than hand-drawn PDF text, especially for mixed-language documents. The tradeoff is operational: larger downloads, more moving parts, and a higher chance of runtime issues on a minimal server.

The most reliable process is:

- Generate the PDF.
- Convert the PDF to images.
- Review the images, not only the source text.
- Iterate on layout issues.
- Keep all temporary environments and caches under explicit `/tmp` paths.
- Verify cleanup at the end. 🧹

The important lesson was that PDF generation is not just a conversion problem. It is a rendering and review problem.
