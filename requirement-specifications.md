# Technical Knowledge Base

## Project Specification \& Implementation Requirements

**Document Type:** Software Requirements \& Implementation Specification  
**Project Name:** Tech Knowledge Base  
**Version:** 1.0  
**Status:** Proposed

---

# 1\. Project Overview

## 1.1 Purpose

Create a personal, publicly accessible technical knowledge base containing Markdown-based articles, technical notes, code snippets, tutorials, architecture explanations and reference materials covering various programming languages, frameworks, technologies and software engineering concepts.

The repository will serve as a centralized location for storing and maintaining technical knowledge for personal reference.

The primary content format will be Markdown (`.md`) to ensure:

- Human readability.
- Portability across platforms and tools.
- Version control through Git.
- Easy editing using VS Code or any Markdown editor.
- Compatibility with AI coding assistants and agents.
- Minimal dependency on proprietary content management systems.

## 1.2 Primary Objectives

1. Maintain all technical articles and notes as Markdown files in a GitHub repository.
2. Provide a professionally designed documentation website to browse and read the articles.
3. Support full-text search across the entire knowledge base.
4. Provide intuitive navigation organized by technology and subject.
5. Automatically publish website updates whenever changes are pushed to GitHub.
6. Maintain a simple, low-maintenance architecture without requiring a backend or database.
7. Ensure the repository can be consumed by AI coding assistants as a structured knowledge source.

## 1.3 Non-Goals

The initial implementation must NOT introduce:

- A custom Angular or React application.
- A backend API or server.
- A database.
- User registration or authentication.
- A content management system.
- An online Markdown editor.
- AI-powered search or RAG infrastructure.
- Paid hosting or third-party search services.

These capabilities may be evaluated in future versions if necessary.

---

# 2\. Technology Stack

Use the following technologies.

| Component | Technology | Purpose |
| --- | --- | --- |
| Content format | Markdown | Authoring articles |
| Version control | Git | Change tracking |
| Repository hosting | GitHub | Source repository |
| Documentation generator | MkDocs | Generate static website |
| Theme | Material for MkDocs | Documentation UI |
| Search engine | MkDocs Search / Lunr.js | Client-side full-text search |
| Diagramming | Mermaid | Architecture and flow diagrams |
| Hosting | GitHub Pages | Static website hosting |
| CI/CD | GitHub Actions | Automated deployment |
| Local development | Python 3 | Run MkDocs locally |
| Editor | VS Code | Markdown authoring |

## 2.1 Architectural Principles

- Markdown must remain the single source of truth for documentation content.
- Generated HTML must never be committed to the source repository.
- The website must be completely static.
- No server-side processing should be required for browsing or searching.
- The website must be deployable entirely through GitHub Pages.
- All dependencies must be explicitly declared and version-controlled.
- The solution must not require a local development environment to publish changes.

---

# 3\. High-Level Architecture

The application consists of three logical components.

### 3.1 Content Repository

GitHub stores:

- Markdown articles.
- Documentation images and other assets.
- Website configuration.
- Build and deployment workflows.
- Documentation templates and contribution guidelines.

### 3.2 Static Site Generation

MkDocs Material reads Markdown files and generates:

- HTML pages.
- Navigation menus.
- Search indexes.
- Search functionality.
- Syntax-highlighted code blocks.
- Rendered Mermaid diagrams.
- Responsive documentation layouts.

### 3.3 Hosting and Deployment

GitHub Actions automatically builds and deploys the generated website to GitHub Pages.

Expected workflow:

1. Developer creates or updates a Markdown file.
2. Changes are committed and pushed to GitHub.
3. GitHub Actions starts the deployment workflow.
4. MkDocs builds the documentation website.
5. Build validation is performed.
6. Successfully generated website is deployed to GitHub Pages.
7. Updated documentation becomes publicly accessible.

---

# 4\. Repository Structure

Create the following repository structure.

``` text
Tech-Knowledge/
│
├── docs/
│   │
│   ├── index.md
│   │
│   ├── dotnet/
│   │   ├── index.md
│   │   ├── csharp/
│   │   │   ├── index.md
│   │   │   ├── async-await.md
│   │   │   ├── dependency-injection.md
│   │   │   └── linq.md
│   │   │
│   │   └── aspnet-core/
│   │       ├── index.md
│   │       └── middleware.md
│   │
│   ├── angular/
│   │   ├── index.md
│   │   ├── signals.md
│   │   └── rxjs.md
│   │
│   ├── architecture/
│   │   ├── index.md
│   │   ├── microservices.md
│   │   ├── event-sourcing.md
│   │   └── cqrs.md
│   │
│   ├── databases/
│   │   ├── index.md
│   │   ├── raven-db.md
│   │   └── redis.md
│   │
│   ├── ai-ml/
│   │   ├── index.md
│   │   ├── rag.md
│   │   └── ai-agents.md
│   │
│   ├── devops/
│   │   └── index.md
│   │
│   ├── design-patterns/
│   │   └── index.md
│   │
│   ├── security/
│   │   └── index.md
│   │
│   └── assets/
│       └── images/
│
├── templates/
│   └── article-template.md
│
├── .github/
│   └── workflows/
│       └── deploy.yml
│
├── mkdocs.yml
├── requirements.txt
├── README.md
├── CONTRIBUTING.md
├── PROJECT_SPEC.md
└── .gitignore
```

## 4.1 Repository Organization Rules

- All publishable documentation must reside under `docs/`.
- Every major technology or subject category must have an `index.md`.
- Articles should be grouped into logical subdirectories.
- Avoid creating deeply nested directory structures unnecessarily.
- Images must be stored under `docs/assets/images/` or an appropriate technology-specific asset directory.
- Images must be referenced using relative paths.
- Filenames must use lowercase kebab-case.
- Avoid spaces and special characters in filenames.
- Do not commit generated website files or Python virtual environments.

## 4.2 Extensibility

The structure must allow new technologies and categories to be introduced simply by adding directories and Markdown files.

Adding a new article must not require changes to the application code.

---

# 5\. Documentation Website Requirements

## 5.1 Theme and Appearance

Use Material for MkDocs.

The website must have:

- Clean, modern documentation-oriented design.
- Responsive layout for desktop, tablet and mobile.
- Light and dark themes.
- Theme toggle.
- Consistent typography and spacing.
- Readable code blocks.
- Collapsible navigation sidebar.
- Article table of contents.
- Breadcrumb-style navigation where supported.

The visual experience should resemble a professional developer documentation portal rather than a traditional blog.

## 5.2 Navigation

The website must provide hierarchical navigation based on the repository's documentation structure.

Example:

``` text
Home
│
├── .NET
│   ├── C#
│   │   ├── Async/Await
│   │   ├── Dependency Injection
│   │   └── LINQ
│   │
│   └── ASP.NET Core
│       └── Middleware
│
├── Angular
│   ├── Signals
│   └── RxJS
│
├── Architecture
│   ├── Microservices
│   ├── Event Sourcing
│   └── CQRS
│
├── Databases
│   ├── RavenDB
│   └── Redis
│
└── AI & Machine Learning
    ├── RAG
    └── AI Agents
```

Navigation should be generated automatically from the documentation directory structure wherever possible.

Use MkDocs' automatic navigation capabilities and section index conventions.

Do not require manually maintaining every individual article in `mkdocs.yml`.

The configuration may explicitly define top-level navigation categories where necessary.

## 5.3 Homepage

The homepage (`docs/index.md`) should serve as the entry point to the knowledge base.

Include:

- Project introduction.
- Brief description of the knowledge base.
- Technology/category navigation.
- Links to major subject areas.
- Recently added or updated articles, if this can be implemented without introducing unnecessary complexity.

Use Material for MkDocs features such as content cards or navigation grids where appropriate.

Avoid maintaining a manually updated article listing.

---

# 6\. Search Requirements

Search is a core requirement.

Use the native MkDocs Material search plugin.

## 6.1 Functional Requirements

The search engine must:

- Search article titles.
- Search article headings.
- Search paragraph content.
- Search relevant code snippets.
- Display matching documents.
- Display relevant matching sections or excerpts.
- Support searching across all documentation categories.
- Provide suggestions while typing.
- Highlight search terms in matching articles.
- Support sharing search queries through URLs where supported.

## 6.2 Technical Requirements

- Use the built-in search plugin.
- Search must execute entirely in the browser.
- Search indexes must be generated during the documentation build.
- No external search service or database is permitted.
- Search must work on GitHub Pages.

## 6.3 Search Validation

Create sample articles containing identifiable technical terms.

Verify that searching for terms such as:

- Dependency Injection
- Async Await
- Event Sourcing
- RavenDB Indexing
- Angular Signals

returns the expected articles.

---

# 7\. Markdown Authoring Standards

Markdown is the primary content format. Establish consistent authoring conventions.

## 7.1 Article Template

Create `templates/article-template.md` containing the following structure:

```` markdown
# Article Title

> Brief description of the article.

## Overview

Explain the topic and its relevance.

## Key Concepts

Explain the important concepts.

## Detailed Explanation

Provide the detailed technical explanation.

## Examples

Include practical examples.

### Example 1

```csharp
// Code example
````

## Architecture / Diagrams

Include diagrams where useful.

## Best Practices

List recommended practices.

## Common Pitfalls

Document common mistakes and limitations.

## References

- [Official Documentation](<https://example.com>)

## Related Articles

- [Related Article](<../related-article.md>)

````

The template is a starting point. Articles do not need to contain every section.

Avoid forcing unnecessary sections into short reference notes.

## 7.2 Markdown Features

Support the following:

- Standard Markdown.
- Tables.
- Nested lists.
- Blockquotes.
- Task lists.
- Inline code.
- Fenced code blocks.
- Syntax highlighting.
- Images.
- Relative links.
- Mermaid diagrams.
- Admonitions such as notes, warnings and tips.

## 7.3 Code Blocks

Code examples must specify their language wherever possible.

Examples:

```csharp
public class Example
{
    public string Name { get; set; }
}
````

``` typescript
const message: string = "Hello World";
```

``` sql
SELECT * FROM Parties;
```

Syntax highlighting must be enabled for commonly used languages.

## 7.4 Mermaid Diagrams

Enable Mermaid support for diagrams.

Support:

- Flowcharts.
- Sequence diagrams.
- Class diagrams.
- Entity relationship diagrams.
- State diagrams.
- Architecture diagrams.

Example:

``` mermaid
flowchart TD
    A[Client] --> B[API]
    B --> C[Service]
    C --> D[Database]
```

Diagrams must render correctly in the generated website.

---

# 8\. MkDocs Configuration

Create a properly configured `mkdocs.yml`.

Requirements:

1. Set the site name to `Tech Knowledge Base`.
2. Configure the Material theme.
3. Enable light and dark modes.
4. Enable the navigation features.
5. Enable instant navigation where appropriate.
6. Enable search suggestions.
7. Enable search highlighting.
8. Enable search sharing.
9. Enable code copy buttons.
10. Enable Mermaid diagrams.
11. Configure useful Markdown extensions.
12. Configure automatic navigation.
13. Configure the site URL using a placeholder or documented deployment setting.
14. Enable strict build validation.

Use appropriate supported Material for MkDocs configuration options.

Do not add deprecated or unsupported configuration properties.

Recommended Material features include:

``` yaml
theme:
  name: material
  features:
    - navigation.instant
    - navigation.tracking
    - navigation.sections
    - navigation.expand
    - navigation.indexes
    - navigation.top
    - search.suggest
    - search.highlight
    - search.share
    - content.code.copy
```

Configure the palette to support light and dark modes.

Use appropriate Markdown extensions such as:

- `admonition`
- `attr_list`
- `md_in_html`
- `tables`
- `toc`
- `pymdownx.details`
- `pymdownx.superfences`
- `pymdownx.highlight`
- `pymdownx.inlinehilite`
- `pymdownx.tabbed`

Ensure Mermaid integration works with the selected extension configuration.

---

# 9\. Local Development Environment

The repository must support local development.

## 9.1 Prerequisites

- Python 3.
- pip.
- Git.
- VS Code (recommended).

## 9.2 Dependencies

Create `requirements.txt` with explicitly pinned, compatible dependencies.

Include:

- `mkdocs`
- `mkdocs-material`

Include additional packages only when required by the implementation.

Avoid unnecessary dependencies.

## 9.3 Local Execution

Document the following commands in README.md:

Create a virtual environment:

``` bash
python -m venv .venv
```

Activate on Windows PowerShell:

``` powershell
.venv\Scripts\Activate.ps1
```

Activate on macOS/Linux:

``` bash
source .venv/bin/activate
```

Install dependencies:

``` bash
pip install -r requirements.txt
```

Run local development server:

``` bash
mkdocs serve
```

Expected local URL:

``` text
http://127.0.0.1:8000/
```

Build the website:

``` bash
mkdocs build --strict
```

The local development server should automatically reload when Markdown files change.

---

# 10\. GitHub Pages Deployment

## 10.1 Deployment Requirements

Use GitHub Actions for automated deployment.

Create:

`.github/workflows/deploy.yml`

The workflow must:

1. Trigger on pushes to the configured default branch.
2. Support manual execution through workflow dispatch.
3. Check out the repository.
4. Set up the required Python version.
5. Install dependencies from `requirements.txt`.
6. Build documentation using strict validation.
7. Deploy the generated website to GitHub Pages.
8. Use the official GitHub Pages deployment actions.
9. Follow least-privilege permissions.
10. Avoid deploying when the build fails.

Pin GitHub Actions to stable versions and follow current GitHub Pages deployment recommendations.

## 10.2 Deployment Configuration

Document the required GitHub repository settings:

- Enable GitHub Pages.
- Select GitHub Actions as the deployment source.
- Configure the appropriate repository permissions for Pages deployment.
- Verify the published URL.

Support the standard GitHub Pages repository URL format:

``` text
https://<username>.github.io/<repository-name>/
```

Configure the correct `site_url` and ensure all assets and navigation links work when deployed under a repository subpath.

Do not assume the website is hosted at the root domain.

---

# 11\. Quality and Validation

The implementation must include build-time validation.

## 11.1 Validation Requirements

- Markdown files must be parsed successfully.
- Invalid MkDocs configuration must fail the build.
- Broken internal links should be detected where practical.
- Missing referenced images should be identified.
- Mermaid diagrams should be configured correctly.
- Website assets must resolve correctly.
- Search index generation must succeed.

Use `mkdocs build --strict` as the baseline validation command.

Evaluate an additional link-checking solution only if it can be integrated without introducing disproportionate complexity.

## 11.2 Error Handling

A failed documentation build must prevent deployment.

The GitHub Actions workflow must expose meaningful build errors so that the issue can be diagnosed.

---

# 12\. Git and Contribution Guidelines

Create `CONTRIBUTING.md`.

Document:

- How to add a new article.
- How to add a new technology category.
- How to add images.
- How to reference other articles.
- Markdown formatting conventions.
- How to preview documentation locally.
- How to validate a build.
- How to publish changes.

Use clear commit message conventions.

Suggested examples:

``` text
docs: add Angular signals article
docs: update RavenDB indexing notes
docs: add event sourcing diagram
config: update Material theme
ci: improve deployment validation
```

---

# 13\. AI Coding Assistant Compatibility

The repository must be structured so that GitHub Copilot, Codex and Claude Code can understand and contribute to it.

## 13.1 AI-Friendly Structure

- Keep documentation categories predictable.
- Use descriptive filenames.
- Avoid duplicate articles covering the same topic without a clear reason.
- Use relative links between related articles.
- Maintain consistent Markdown formatting.
- Keep configuration files straightforward.
- Avoid unnecessary abstractions.

## 13.2 Agent Instructions

Create an `AGENTS.md` file in the repository root.

It must explain:

1. The purpose of the repository.
2. Repository directory conventions.
3. Markdown authoring standards.
4. Article template usage.
5. Naming conventions.
6. How to add categories and articles.
7. How to validate documentation.
8. Rules against introducing unnecessary frameworks or dependencies.

AI agents modifying the repository must:

- Preserve the Markdown-first architecture.
- Avoid introducing a custom frontend or backend.
- Avoid modifying unrelated articles.
- Preserve existing article content unless explicitly requested.
- Verify relative links after moving or renaming articles.
- Run the strict MkDocs build before considering changes complete.
- Report files created, modified and validation results.

---

# 14\. Security and Privacy

The initial website is intended to be publicly accessible.

Therefore:

- Do not store passwords, API keys or credentials in Markdown files.
- Do not publish confidential company information.
- Do not publish proprietary source code without authorization.
- Do not include personal information unnecessarily.
- Do not commit environment files containing secrets.
- Ensure GitHub Actions permissions follow the principle of least privilege.

All published documentation should be considered public.

---

# 15\. Performance and Maintainability

The website should:

- Load quickly on desktop and mobile.
- Avoid unnecessary JavaScript dependencies.
- Avoid external runtime dependencies wherever practical.
- Use optimized images.
- Use static assets.
- Avoid unnecessary plugins.
- Support growth to several hundred or more Markdown articles.

Search performance should remain acceptable as the knowledge base grows.

Do not introduce a backend or external search infrastructure solely to optimize for hypothetical future scale.

---

# 16\. Implementation Phases

Implement the project in the following phases.

## Phase 1: Repository Foundation

- Create the directory structure.
- Add README.md.
- Add PROJECT\_SPEC.md.
- Add AGENTS.md.
- Add CONTRIBUTING.md.
- Add .gitignore.
- Add article template.

## Phase 2: Documentation Engine

- Configure MkDocs.
- Install Material for MkDocs.
- Configure theme.
- Configure navigation.
- Configure search.
- Configure Markdown extensions.
- Configure Mermaid.
- Add homepage.

## Phase 3: Sample Documentation

Create a small set of representative sample articles to validate the implementation.

Include:

- C# Async/Await.
- Dependency Injection.
- Angular Signals.
- Microservices.
- Event Sourcing.
- RAG and AI Agents.

These articles should be concise examples demonstrating the supported Markdown features, not extensive technical tutorials.

## Phase 4: Local Development

- Configure Python dependencies.
- Verify local environment setup.
- Validate `mkdocs serve`.
- Validate `mkdocs build --strict`.

## Phase 5: CI/CD

- Create GitHub Actions workflow.
- Configure GitHub Pages deployment.
- Document repository settings.
- Validate automated deployment.

## Phase 6: Final Verification

Verify:

- Homepage loads.
- Navigation works.
- All sample articles render.
- Search returns expected results.
- Search highlighting works.
- Code syntax highlighting works.
- Mermaid diagrams render.
- Light/dark theme switching works.
- Relative links work.
- Mobile layout works.
- GitHub Pages deployment works.

---

# 17\. Acceptance Criteria

The project is considered complete when all the following conditions are satisfied.

| ID | Acceptance Criteria |
| --- | --- |
| AC-01 | Repository structure is created as specified. |
| AC-02 | Markdown is the primary content format. |
| AC-03 | MkDocs Material generates the website successfully. |
| AC-04 | Website has hierarchical navigation. |
| AC-05 | Navigation supports adding new articles without manually maintaining every page entry. |
| AC-06 | Search works across documentation content. |
| AC-07 | Search is client-side and requires no backend. |
| AC-08 | Code blocks support syntax highlighting. |
| AC-09 | Mermaid diagrams render correctly. |
| AC-10 | Light and dark themes are available. |
| AC-11 | Website works on mobile and desktop. |
| AC-12 | Local development works using `mkdocs serve`. |
| AC-13 | Strict documentation build completes successfully. |
| AC-14 | GitHub Actions automatically deploys changes. |
| AC-15 | GitHub Pages hosts the website successfully. |
| AC-16 | Internal links and images resolve correctly. |
| AC-17 | AI agent instructions are available in AGENTS.md. |
| AC-18 | No backend, database or paid service is required. |

---

# 18\. Instructions to the Implementing AI Agent

You are responsible for implementing this project based on the specification.

Follow these instructions:

1. Inspect the existing repository before making changes.
2. If the repository is empty, initialize the complete structure.
3. If files already exist, preserve relevant content and avoid overwriting existing work.
4. Follow the technology stack and architectural constraints defined in this document.
5. Implement all configuration and workflow files.
6. Create the sample articles specified in Phase 3.
7. Ensure all configuration uses currently supported MkDocs Material features.
8. Use compatible, explicitly pinned dependencies.
9. Validate the project using the documented commands.
10. Fix any configuration, rendering or build errors before completing the implementation.

Do not stop after producing a proposed solution or a list of instructions. Actually create and configure the required files in the repository.

## Final Deliverables

At completion, provide:

- Complete repository directory structure.
- All configuration files.
- Markdown article template.
- Sample documentation articles.
- MkDocs Material website configuration.
- GitHub Actions deployment workflow.
- AI agent instructions.
- Local development instructions.
- GitHub Pages deployment instructions.
- Build and validation results.

Provide a concise implementation summary identifying any remaining manual steps, particularly GitHub Pages configuration.

**Important:** Do not introduce unnecessary functionality beyond this specification. Prioritize simplicity, portability, maintainability and ease of adding new Markdown articles.