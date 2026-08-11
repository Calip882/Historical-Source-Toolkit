# Historical Source Toolkit

[![Python 3.11+](https://img.shields.io/badge/python-3.11%2B-blue.svg)](https://www.python.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Status: Alpha](https://img.shields.io/badge/status-alpha-orange.svg)](#project-status)

**Historical Source Toolkit (`hst`) is a small, reproducible preprocessing pipeline for OCR-based historical sources.** It imports OCR text, applies conservative cleanup rules, retains the mapping from every cleaned passage to its source page, exports JSONL or Markdown, and provides transparent keyword search.

It is designed for historians and digital-humanities researchers who need a dependable step between OCR and later analysis—not another OCR model, hosted service, or opaque AI system.

> **中文简介：** `hst` 是一个面向历史研究的轻量 OCR 文本预处理工具。它可以导入单个 TXT 或逐页 TXT，完成 Unicode、空白、断行及基础页眉页脚清理，并始终保留“来源页码 → 清洗文本”的映射。项目不调用云 API，也不包含 OCR 模型或向量数据库。

## Why page mapping matters

OCR text is useful only when a researcher can return to the scan. Every JSONL record therefore preserves:

- `source_page`: the page number inferred from the page filename, or its order in a form-feed-separated file;
- `source_file`: the original TXT filename;
- `raw_text`: the imported OCR text, unchanged;
- `cleaned_text`: the result of the selected cleanup rules.

The canonical format is one JSON object per source page:

```json
{"source_page": 12, "source_file": "scan_0012.txt", "raw_text": "...", "cleaned_text": "..."}
```

This format is intentionally simple. It can be inspected with ordinary text tools and used later by databases, notebooks, or language-model workflows without losing citation provenance.

## Features

- Import a UTF-8 `.txt` file or a directory containing one `.txt` file per page.
- Split a single text file into pages on form-feed (`\f`) characters.
- Apply Unicode NFKC normalization and normalize horizontal whitespace.
- Join common OCR line wraps, including English hyphenation and continuous CJK text.
- Remove a fixed number of header/footer lines, or detect identical repeated margin lines.
- Export page-mapped JSONL and readable Markdown.
- Search cleaned text with a literal keyword or regular expression and report source pages.
- Run locally on Python 3.11+ with no runtime dependencies and no network access.

## Installation

Clone or download this repository, then install it in editable mode:

```bash
cd historical-source-toolkit
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -e .
hst --version
```

For development and `pytest`:

```bash
python -m pip install -e ".[dev]"
```

You can also run the CLI without installation while working in the repository:

```bash
PYTHONPATH=src python -m historical_source_toolkit --help
```

## Quick start

The bundled sample is fictional and contains no restricted or copyrighted source text.

### 1. Import and clean per-page OCR files

```bash
hst clean examples/sample_ocr/pages examples/output/sample.jsonl \
  --header-lines 1 \
  --footer-lines 1
```

The filenames `page_001.txt` and `page_002.txt` become source pages 1 and 2. If filenames have no digits, page numbers follow natural filename order.

To detect identical headers or footers appearing near the margins of most pages:

```bash
hst clean examples/sample_ocr/pages examples/output/sample.jsonl \
  --detect-repeated-margins
```

The detector inspects the first and last two non-empty lines by default and removes exact normalized lines found in at least 60% of pages (and at least two pages). Change the inspection depth with `--margin-window`.

### 2. Export Markdown

```bash
hst export examples/output/sample.jsonl examples/output/sample.md \
  --title "Sample historical source"
```

Markdown page headings retain source-page labels, while HTML comments retain source filenames.

To normalize or copy a corpus back to JSONL:

```bash
hst export examples/output/sample.jsonl examples/output/copy.jsonl
```

### 3. Search while retaining page references

```bash
hst search examples/output/sample.jsonl "地方社会"
hst search examples/output/sample.jsonl "railway|station" --regex
hst search examples/output/sample.jsonl "OCR" --json
```

Search is case-insensitive by default. Add `--case-sensitive`, `--context 100`, or `--limit 10` as needed. The command returns exit status `1` when there are no matches, which makes it usable in scripts.

### Single multi-page TXT input

A single file may contain form-feed characters between pages:

```bash
hst clean examples/sample_ocr/multipage.txt examples/output/multipage.jsonl \
  --header-lines 1 \
  --footer-lines 1
```

Without form feeds, a single TXT file is treated as one source page.

## Cleanup behavior

The v0.1.0 rules are deliberately conservative and explainable:

1. Convert line endings to `\n`.
2. Normalize each line with Unicode NFKC.
3. Collapse tabs, full-width spaces, non-breaking spaces, and repeated spaces.
4. Remove configured fixed lines or repeated margin text.
5. Preserve blank-line paragraph boundaries.
6. Join a Latin word split with a hyphen before a lowercase continuation (`commit-` + `tee`).
7. Join adjacent lines when a CJK character continues into another CJK character.
8. Join ordinary wrapped Latin lines with one space.
9. Keep likely headings and lines ending in sentence punctuation separate.

OCR cleanup is never a substitute for checking the scan. Names, dates, archival identifiers, quotations, and numeric tables should still be verified against page images.

## Command reference

```text
hst clean INPUT OUTPUT.jsonl [--header-lines N] [--footer-lines N]
          [--detect-repeated-margins] [--margin-window N]

hst export INPUT.jsonl OUTPUT [--format markdown|jsonl] [--title TITLE]

hst search INPUT.jsonl QUERY [--case-sensitive] [--regex]
          [--context N] [--limit N] [--json]
```

Run `hst COMMAND --help` for complete option descriptions.

## Project layout

```text
historical-source-toolkit/
├── examples/
│   ├── output/
│   └── sample_ocr/
├── src/historical_source_toolkit/
│   ├── cleaning.py
│   ├── cli.py
│   ├── exporters.py
│   ├── ingest.py
│   ├── models.py
│   ├── search.py
│   └── storage.py
├── tests/
├── CHANGELOG.md
├── CONTRIBUTING.md
├── LICENSE
├── README.md
└── pyproject.toml
```

## Tests

The test suite uses only the Python standard library:

```bash
PYTHONPATH=src python -m unittest discover -s tests -v
```

If the optional development dependency is installed:

```bash
pytest
```

## Scope and limitations

Version 0.1.0 does **not**:

- run OCR on images or PDFs;
- correct names, dates, or domain-specific OCR errors automatically;
- infer printed page numbers from page images;
- store embeddings or run semantic search;
- call cloud APIs or language models;
- modify the source TXT files.

The repeated-margin detector uses exact normalized line matching. Headers containing a changing page number may need fixed-line removal or manual preprocessing.

## Project status

This is an alpha release intended as a clear, testable foundation. Backward compatibility is not guaranteed before v1.0.0. Good next steps include configurable cleanup profiles, explicit printed-page labels, and ALTO/hOCR import—while preserving the same page-first provenance model.

## Contributing

Bug reports, small reproducible fixtures, and conservative cleanup rules are welcome. See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

MIT. See [LICENSE](LICENSE).

