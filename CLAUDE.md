# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
uv sync                        # install all dependencies (including dev)
uv run pytest                  # run full test suite
uv run pytest tests/test_notes.py::test_slugify  # run a single test
uv run pytest --cov            # run with coverage (must stay ≥ 80%)
uv run ruff check .            # lint
uv run ruff format .           # format
uv run second_brain --help     # run the CLI
uv run --env-file .env second_brain new "My idea"  # run with dev env vars
uv run mkdocs build            # build static docs
```

Copy `.env.example` → `.env` for local development (sets `LOG_LEVEL=DEBUG` and `SECOND_BRAIN_DIR`). The env file is **not** auto-loaded; pass `--env-file .env` explicitly.

## Architecture

The package lives in `src/second_brain/` and is installed as the `second_brain` console script.

- **`cli.py`** — Click group with three subcommands (`new`, `list`, `show`). Reads `SECOND_BRAIN_DIR` from the environment (fallback: `~/second_brain`). The CLI layer is intentionally thin: it resolves the notes directory, delegates to `notes.py`, and echoes results.
- **`notes.py`** — Pure business logic. `slugify` → `build_note_path` → `create_note` is the call chain. `build_note_path` auto-increments the filename suffix (`-1`, `-2`, …) to avoid overwrites.
- **`app.py`** — `configure_logging()` sets up loguru with a compact pipe-separated format. Called once at CLI entry. Log level controlled by `LOG_LEVEL`; log file by `LOG_FILE` (rotates at 50 KB, keeps 1 file).

## Testing conventions

Tests use `tmp_note_dir` (from `conftest.py`) to redirect `SECOND_BRAIN_DIR` to a temp directory. The `_test_log_file` fixture is `autouse` and redirects `LOG_FILE` for every test. CLI tests use `click.testing.CliRunner` directly — no subprocess.

## Environment variables

| Variable           | Default          | Purpose                        |
|--------------------|------------------|--------------------------------|
| `SECOND_BRAIN_DIR` | `~/second_brain` | Notes storage directory        |
| `LOG_LEVEL`        | `INFO`           | Console log verbosity          |
| `LOG_FILE`         | `app.log`        | Log file path (50 KB rotation) |
