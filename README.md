# Tech Knowledge Base

A Markdown-first technical knowledge base published as a static documentation website with MkDocs Material.

## Prerequisites

- Python 3
- pip
- Git

## Local development

Create and activate a virtual environment:

**Windows PowerShell**

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
```

**macOS/Linux**

```bash
python3 -m venv .venv
source .venv/bin/activate
```

Install dependencies and start the development server. These commands use the virtual environment's Python directly, so the `mkdocs` executable does not need to be on `PATH`.

**Windows PowerShell**

```powershell
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe -m mkdocs serve
```

**macOS/Linux**

```bash
python -m pip install -r requirements.txt
python -m mkdocs serve
```

The local website is available at `http://127.0.0.1:8000/`.

Build the complete website with strict validation:

**Windows PowerShell**

```powershell
.\.venv\Scripts\python.exe -m mkdocs build --strict
```

**macOS/Linux**

```bash
python -m mkdocs build --strict
```

The generated site is written to `site/`, which is intentionally ignored by Git.

## Adding content

Add Markdown files under `docs/`. Add an `index.md` for every major category. Use lowercase kebab-case filenames and relative links between articles.

See [CONTRIBUTING.md](CONTRIBUTING.md) for authoring conventions.

## GitHub Pages deployment

1. Push changes to the `main` branch.
2. Open **Settings > Pages** in the GitHub repository.
3. Set **Source** to **GitHub Actions**.
4. Allow the repository workflow to request Pages deployment.
5. Open the `deploy.yml` workflow run or push a new commit to deploy.

The published URL follows `https://<username>.github.io/<repository-name>/`. This repository is configured for `https://dkale.github.io/tech-folio/`. Update `site_url` in `mkdocs.yml` if the repository is moved or hosted under a different URL.
