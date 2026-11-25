# Contributing

## Add an article

1. Copy `templates/article-template.md` to the appropriate category.
2. Use a lowercase kebab-case filename, such as `cancellation.md`.
3. Start with a clear title and a short description.
4. Keep examples concise and practical.
5. Use relative links to related articles.
6. Preview the article locally and run a strict build.

## Add a category

1. Create a directory under `docs/`.
2. Add an `index.md` describing the category.
3. Use a directory name in lowercase kebab-case.
4. Add articles below the category index.
5. Do not add custom application code or plugins unless the documentation requirement clearly needs it.

## Add images

Place images in `docs/assets/images/` or an appropriate category asset directory. Use descriptive lowercase kebab-case filenames and relative Markdown paths:

```markdown
![Architecture overview](../assets/images/architecture-overview.png)
```

Optimize images before committing them. Do not commit secrets or private information.

## Markdown conventions

- Use ATX headings (`#`, `##`, `###`).
- Specify a language for fenced code blocks.
- Use tables for concise comparisons.
- Use admonitions for important notes:

```markdown
!!! note
    Important context.
```

- Use Mermaid fenced blocks for diagrams.
- Preserve relative links after renaming or moving files.

## Preview and validate

**Windows PowerShell**

```powershell
python -m venv .venv
.venv\Scripts\python.exe -m pip install -r requirements.txt
.venv\Scripts\python.exe -m mkdocs serve
.venv\Scripts\python.exe -m mkdocs build --strict
```

**macOS/Linux**

```bash
python3 -m venv .venv
.venv/bin/python -m pip install -r requirements.txt
.venv/bin/python -m mkdocs serve
.venv/bin/python -m mkdocs build --strict
```

## Commit messages

Use a focused Conventional Commit-style prefix:

- `docs: add Angular signals article`
- `docs: update RavenDB indexing notes`
- `docs: add event sourcing diagram`
- `config: update Material theme`
- `ci: improve deployment validation`
