# CLAUDE.md

Guidance for working in this repository.

## What this project is

This repo distills a large collection of **Atari Jaguar developer documentation**
(originally Atari Corp. PDFs, mostly from 1993–1995, with community-restored
"Version 10" editions by Stephen Moss, 2021) into a set of **cross-linked Markdown
files** that serve as a clean developer reference.

The deliverable is the Markdown documentation under [`docs/`](docs/), navigable
from [`docs/index.md`](docs/index.md). The PDFs are the *sources*; the Markdown is the
*product*.

## GitHub Pages

The site is published with GitHub Pages from the **`/docs` folder on `main`**.
Key pieces: `docs/_config.yml` (Jekyll config — theme `minima`,
`baseurl: /jag-docs`, default `layout: default`, and the default
`jekyll-relative-links` / `jekyll-optional-front-matter` plugins) and
`docs/index.md` as the site index (do **not** rename it back to `README.md` — the
in-page "Home" links and Jekyll's index both target `index.md`). The repo-root
`README.md` is the GitHub landing page and links out to the published site at
`https://asterick.github.io/jag-docs/`. The `_extract/` scratch folder is
git-ignored and excluded from the build.

## Scope and priorities

These subsystems are all in scope and carry **equal weight** — none is more
important than another. Document each thoroughly:

- **Atari Jaguar core system** — Tom (object/video processor, GPU, Blitter, CRY
  colour), Jerry (DSP, audio synth, serial I/O), memory map, video timing,
  controllers. Sources: `TechRef_V10`, `SoftRef_V10`, `TechOver_V10`,
  and the older `TechRef V8`, `Midsummer Tech Ref V6` for cross-checking.
- **CD-ROM subsystem** — BIOS API, data/session format, programming guidelines,
  hardware. Main source: `sources/CD-ROM.pdf` (**scanned**, see OCR note below),
  plus `sources/Schematics/Jaguar CD ROM.pdf` and `sources/Other Documents/Jag CD testpro4.pdf`.

> **Assume a toolchain-agnostic dev environment.** Document the hardware and the
> CD BIOS at the platform level only. Content tied to the official Atari dev kit
> is **invalid** for users on other toolchains and is excluded: the CD-ROM
> emulator, the Authoring Tool, the Track Creator, mastering-software workflows,
> and the Atari debugger workflow (`CDBIOS??.DB` soft-loads, `rdbjag`/`wdb`,
> `load`/`read`/`aread` scripts). Keep only what any toolchain needs — the BIOS
> calls, the on-disc format, and programming guidance.

**Excluded — do not document or redistribute** (third-party/copyright IP, or
toolchain-tied rather than hardware): the toolchain (`Madmac`, `ALN`, `Tools`,
`Debugger`), the libraries (`librarys` — 3D/JPEG/networking/music), the licensed
`Cinepak` codec and `QSound` audio module, and the toolchain-dependent
`GStarted`/`Getting Started` setup guide. These are catalogued under "Excluded"
in [the source index](docs/reference/source-documents.md) but never transcribed.

Candidate hardware/programming topics still open (toolchain-independent):
`Workshop` (worked examples), `Appendix` (hardware appendices), `VModem` (Voice
Modem subsystem), and `Samples`.

> **Samples caveat:** sample programs may be included, but **exclude any sample
> built around a copyright library**. Samples depending only on standard /
> freely-redistributable code (e.g. C `stdlib`) are fine.

## Source PDFs — text vs. scanned

Some PDFs have a real text layer; others are **scanned images** with no text and
require OCR. Determined via PyMuPDF:

| Type | Extraction | Notable docs |
|------|-----------|--------------|
| Text layer | `pdftotext` / PyMuPDF `get_text()` | TechRef_V10, SoftRef_V10, TechOver_V10, TechRef V8, Midsummer V6, Jag CD testpro4, most Schematics |
| **Scanned (image)** | Render page → PNG → transcribe via the **Read tool** (visual) | **CD-ROM.pdf**, librarys, Debugger, Cinepak, Madmac, Tools, Workshop, VModem, ALN, Appendix, Samples, the "Atari Original" set, "Flair II 1992" TechRef |

There is **no OCR binary** (`tesseract`) installed. The working OCR method is to
render scanned pages to PNG with PyMuPDF and read them with the Read tool, which
presents images visually for transcription. Scans here are clean and legible.

## Tooling

- Python 3.14 at `/c/Python314/python` (Git-Bash path). `PyMuPDF` (`import fitz`)
  is installed (user site). `pdftotext.exe` is on PATH.
- Extraction workspace: `_extract/text/` (per-doc text dumps, page-marked) and
  `_extract/img/` (rendered PNGs). This is scratch — regenerate as needed; not
  part of the deliverable.

Re-extract text from a PDF:
```bash
/c/Python314/python -c "import fitz; d=fitz.open('sources/TechRef_V10.pdf'); \
open('_extract/text/TechRef_V10.txt','w',encoding='utf-8').write( \
''.join(f'\n===== PAGE {i} =====\n'+p.get_text() for i,p in enumerate(d,1)))"
```

Render a scanned page to PNG for transcription:
```bash
/c/Python314/python -c "import fitz; d=fitz.open('sources/CD-ROM.pdf'); \
d[0].get_pixmap(dpi=150).save('_extract/img/CDROM_p01.png')"
```

## Diagrams (vector art)

Diagrams are **SVG** files in [`docs/assets/`](docs/assets/), embedded from a
page with a normal Markdown image (alt text plus a relative path such as
`../assets/<name>.svg`). Prefer this over ASCII art. Conventions:

- **Inline presentation attributes only** (`fill="…"`, `stroke="…"`) — *not* CSS
  `<style>`/class selectors. Some SVG renderers (incl. the PyMuPDF preview used
  here) ignore `<style>` and render everything black.
- Each SVG opens with a light rounded background rect (`fill="#fcfcfe"
  stroke="#d0d7de"`) so it reads on light *and* dark page themes.
- Palette in use: Tom `#dbeafe`, Jerry `#ffedd5`, CPU/neutral `#e5e7eb`/`#f1f5f9`,
  CD `#dcfce7`, Object Processor `#ede9fe`, ROM `#fde68a`, DRAM `#bfdbfe`,
  accent `#c7d2fe`; ink `#0f172a`, sub-text `#475569`, edges `#334155`.
- Always set a descriptive `role="img" aria-label="…"` and a meaningful image
  `alt`. Avoid `<tspan>` inline runs (PyMuPDF mis-renders them).
- Tabular data dressed as preformatted text becomes a **Markdown table**, not an
  SVG.

Regenerate the parametric SVGs (bit strips, memory maps) from the generator
snippets in the project history; preview any SVG by rendering to PNG:
```bash
/c/Python314/python -c "import fitz; fitz.open('docs/assets/foo.svg')[0].get_pixmap(dpi=110).save('_extract/img/preview.png')"
```

## Documentation conventions

- One subsystem per directory under `docs/`; one focused topic per file.
- Every file starts with an `H1` title and a one-line summary, then a
  **"Source:"** line citing the originating PDF(s) and page range.
- Cross-link liberally with relative Markdown links so the set is navigable.
- Preserve hardware facts exactly: register addresses, bit fields, equate names
  (e.g. `B_CMD`, `VMODE`, `$F00000`). When a source has an obvious OCR/typo
  artifact, fix it silently but never invent values — if a value is illegible,
  mark it `(illegible)` rather than guessing.
- Keep register/equate names in the casing the source uses; addresses in the
  `$XXXXXX` hex style the manuals use.
- Tables for register lists and bit fields; fenced code for assembly/struct
  layouts.

## Don'ts

- Don't transcribe the lower-priority toolchain docs unless asked.
- Don't invent register values, timings, or API signatures. Cite the source.
- Don't treat `_extract/` as deliverable content.
