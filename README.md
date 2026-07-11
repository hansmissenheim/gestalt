# Gestalt

## Setup

This project uses [mise](https://mise.jdx.dev/) to manage tooling (Python, uv).

```bash
mise install        # install Python and uv
mise run install    # install project dependencies
mise run serve      # start dev server at http://localhost:8000
```

## Tasks

| Command            | Description                          |
|--------------------|--------------------------------------|
| `mise run install` | Install dependencies via uv          |
| `mise run serve`   | Start local dev server (live reload) |
| `mise run build`   | Build static site into `site/`       |
| `mise run clean`   | Remove build artifacts               |
