# Contributing

Thank you for helping improve Historical Source Toolkit.

## Development setup

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -e ".[dev]"
pytest
```

The standard-library suite is also available without development dependencies:

```bash
PYTHONPATH=src python -m unittest discover -s tests -v
```

## Guidelines

- Keep source-page provenance intact in every transformation.
- Prefer small, explainable cleanup rules to opaque automatic correction.
- Add tests for both Latin-script and CJK text when a rule affects line joining.
- Do not add copyrighted, private, or sensitive source text as a fixture.
- Document any behavior that could alter names, dates, quotations, or identifiers.

Please open an issue before proposing a large dependency or a major change to the JSONL schema.

