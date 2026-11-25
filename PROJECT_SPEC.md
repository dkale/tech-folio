# Project Specification

This repository implements the project described in `requirement-specifications.md`.

## Architecture

- Markdown is the single source of truth.
- MkDocs Material generates the static website.
- MkDocs search generates a browser-side index.
- Mermaid diagrams render through PyMdown Markdown extensions.
- GitHub Actions builds and deploys the site to GitHub Pages.

## Source layout

- `docs/`: publishable documentation
- `templates/`: article authoring template
- `mkdocs.yml`: site configuration
- `requirements.txt`: pinned Python dependencies
- `.github/workflows/deploy.yml`: GitHub Pages deployment

Generated HTML must remain under `site/` and must not be committed.
