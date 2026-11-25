# Agent Instructions

## Purpose

This repository is a Markdown-first technical knowledge base. MkDocs Material builds the static documentation website. The source repository must not become a custom frontend, backend, database application, CMS, or paid-service integration.

## Repository conventions

- Publishable content belongs under `docs/`.
- Every major category has an `index.md`.
- Articles use lowercase kebab-case filenames.
- Images belong under `docs/assets/images/` or a category-specific asset directory.
- Filenames and headings should use descriptive terminology.
- Keep directory nesting shallow.
- Generated `site/` output, virtual environments, environment files, and secrets must not be committed.

## Markdown authoring

- Use `templates/article-template.md` as the starting point, but omit sections that are not useful for a short reference note.
- Use fenced code blocks with explicit languages.
- Use relative links and verify them after renames or moves.
- Use `pymdownx.superfences` Mermaid blocks for diagrams.
- Use Material admonitions where they improve clarity.
- Do not publish credentials, confidential information, private source code, or unnecessary personal information.

## Adding content

- Add a new technology or subject as a directory with an `index.md`.
- Add articles in the relevant category directory.
- Do not manually edit application code merely to add a page.
- Preserve existing article content unless explicitly asked to change it.
- Avoid duplicate articles without a clear distinction.

## Dependencies and configuration

- Use MkDocs and MkDocs Material.
- Keep dependencies pinned in `requirements.txt`.
- Do not add React, Angular, backend APIs, databases, authentication, CMS tooling, external search, or paid services.
- Preserve compatible, supported MkDocs configuration properties.

## Validation

Before considering a documentation change complete:

1. Run `pip install -r requirements.txt`.
2. Run `mkdocs build --strict`.
3. Resolve broken relative links and missing assets exposed by the build or review.
4. For search-sensitive changes, verify that relevant titles and terms appear in the built search index.

Report every file created or modified and include the validation result.
