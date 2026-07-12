# Gestalt – Copilot Instructions

Gestalt is a personal documentation site built with [MkDocs Material](https://squidfunk.github.io/mkdocs-material/), covering Windows, WSL, macOS, Unraid, and device setup guides.

## Tooling

- **mise** manages the Python version and `uv`; run `mise install` once to bootstrap.
- **uv** manages Python dependencies (no manual `pip` needed).

## Commands

| Task | Command |
|---|---|
| Install dependencies | `mise run install` |
| Local dev server (live reload) | `mise run serve` → http://localhost:8000 |
| Build static site to `site/` | `mise run build` |
| Remove build artifacts | `mise run clean` |

There are no tests or linters in this project.

## Architecture

- `docs/` — all Markdown source files, mirroring the nav structure in `mkdocs.yml`
- `docs/stylesheets/extra.css` — custom CSS overrides for the Material theme
- `mkdocs.yml` — site config: nav, theme features, enabled Markdown extensions
- `site/` — build output (git-ignored)

The nav tree in `mkdocs.yml` is the canonical structure. Any new page must be added there to appear in the site.

## Key Conventions

### Commit and push rules
- Direct pushes to `main` are enforced by a pre-push hook (`.githooks/pre-push`, configured automatically by `mise run install`).
- Only commits that **exclusively modify files under `docs/`** and whose message starts with `docs:` are allowed to push directly to `main`.
- All other changes (config, theme, CI) must go through a pull request.
- **PR titles must use conventional commit format** (e.g., `feat: add macOS SSH guide`, `fix: correct broken link`, `chore: update dependencies`) to stay consistent with the `docs:` prefix convention used for direct pushes.
- To bypass the pre-push hook in emergencies: `git push --no-verify`.

### Markdown extensions in use
The following PyMdown extensions are enabled and should be preferred for formatting:

- **Admonitions** (`!!! note`, `!!! warning`, etc.) and **`???` collapsible blocks** via `pymdownx.details`
- **Code blocks** with syntax highlighting and line numbers via `pymdownx.superfences` / `pymdownx.highlight`
- **Tabbed content** via `pymdownx.tabbed` (use `alternate_style`)
- **Inline highlighting** via `pymdownx.inlinehilite`
- **`attr_list`** for adding HTML attributes to elements

### Deployment
Pushing to `main` triggers `.github/workflows/deploy.yml`, which runs `uv run mkdocs build` and deploys `site/` to GitHub Pages at https://hansmissenheim.github.io/gestalt/.
