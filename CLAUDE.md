# CLAUDE.md — BisTracker

This file provides guidance for AI assistants (Claude and others) working in this repository. Keep it up to date as the project evolves.

---

## Project Overview

**BisTracker** is a tracking application hosted at [Gnutmi/BisTracker](http://local_proxy@127.0.0.1:46455/git/Gnutmi/BisTracker).

> This repository is in its initial state. Update this section with a description of BisTracker's purpose, target users, and core functionality once the project is defined.

---

## Repository Structure

```
BisTracker/
├── CLAUDE.md          # This file — AI assistant guidance
└── .git/              # Git metadata
```

> As source files are added, document the layout here. Common patterns to follow:
> - `src/` or `app/` — application source code
> - `tests/` or `__tests__/` — test files
> - `docs/` — documentation
> - `scripts/` — utility/build scripts
> - `config/` — environment and configuration files

---

## Technology Stack

> Document the chosen stack here once decided. Examples:
> - **Language**: Python / TypeScript / Go / Rust
> - **Framework**: FastAPI / Next.js / Gin / Axum
> - **Database**: PostgreSQL / SQLite / MongoDB
> - **ORM/Query layer**: SQLAlchemy / Prisma / GORM
> - **Frontend** (if any): React / Vue / HTMX

---

## Development Setup

### Prerequisites

> List required tools, runtimes, and versions (e.g., Node ≥ 20, Python ≥ 3.11, Docker).

### Install Dependencies

```bash
# Example — replace with actual commands
npm install          # Node.js
pip install -r requirements.txt  # Python
cargo build          # Rust
```

### Environment Variables

> Document all required environment variables. Never commit secrets. Use a `.env.example` file as a template.

```bash
cp .env.example .env
# then edit .env with your local values
```

| Variable | Description | Required |
|---|---|---|
| `DATABASE_URL` | Connection string for the database | Yes |
| `SECRET_KEY` | Application secret key | Yes |

### Running the Application

```bash
# Example — replace with actual commands
npm run dev          # Development server
python main.py       # Python entry point
cargo run            # Rust binary
```

---

## Testing

> Update with the actual test framework and commands once established.

```bash
# Run all tests
npm test
pytest
cargo test

# Run with coverage
npm run test:coverage
pytest --cov=src
cargo tarpaulin
```

### Testing Conventions

- Write tests for all new features and bug fixes.
- Keep unit tests isolated; mock external services.
- Integration tests live in a separate directory from unit tests.
- Aim for meaningful coverage over high coverage numbers.

---

## Build & Deployment

```bash
# Example build commands — replace with actual
npm run build
docker build -t bistracker .
```

> Document CI/CD pipelines, deployment targets, and release process here.

---

## Code Style & Conventions

### General

- Prefer clarity over cleverness.
- Keep functions small and single-purpose.
- Avoid over-engineering: only build what is currently needed.
- Delete dead code rather than commenting it out.

### Naming

- Use descriptive, intention-revealing names.
- Follow the naming conventions of the chosen language (e.g., `snake_case` for Python, `camelCase` for JS/TS, `PascalCase` for types/classes).

### Comments

- Add comments only where the logic is non-obvious.
- Do not add comments that merely restate what the code does.
- Prefer self-documenting code over excessive comments.

### Error Handling

- Handle errors at system boundaries (user input, external APIs, file I/O).
- Do not silently swallow errors.
- Provide actionable error messages.

### Linting & Formatting

> Document the chosen linters and formatters once added (e.g., ESLint + Prettier, Ruff + Black, `rustfmt`). Run them before committing.

---

## Git Workflow

### Branches

- `main` — stable, production-ready code
- `claude/` prefix — branches used by AI assistants
- Feature branches: `feature/<short-description>`
- Bug fix branches: `fix/<short-description>`

### Commit Messages

Use concise, imperative-mood commit messages:

```
Add user authentication module
Fix pagination bug in tracker list view
Refactor database connection pooling
```

- First line: ≤ 72 characters, imperative mood
- Add a blank line then a longer description if needed
- Reference issue numbers where applicable: `Closes #42`

### Pull Requests

- Keep PRs focused on a single concern.
- Include a description of what changed and why.
- Ensure tests pass before requesting review.
- Do not force-push to shared branches.

---

## AI Assistant Guidelines

When working in this repository as an AI assistant:

1. **Read before editing** — always read the relevant files before making changes.
2. **Minimal changes** — only change what is necessary to fulfill the request. Do not refactor surrounding code unless asked.
3. **No unnecessary files** — do not create new files unless they are clearly required.
4. **No over-engineering** — avoid premature abstractions, feature flags, or backwards-compatibility shims.
5. **Security first** — never introduce SQL injection, XSS, command injection, or other OWASP vulnerabilities.
6. **Test your changes** — run the test suite after changes and fix any failures before committing.
7. **Commit clearly** — write descriptive commit messages explaining the *why*, not just the *what*.
8. **Branch discipline** — develop on the designated `claude/` branch; never push to `main` without explicit permission.
9. **Confirm destructive actions** — ask before deleting files, dropping database tables, or force-pushing.
10. **Keep CLAUDE.md current** — update this file whenever significant architectural decisions are made.

---

## Known Issues & TODOs

> Track outstanding issues and decisions here, or link to the issue tracker.

- [ ] Define project scope and technology stack
- [ ] Add initial application skeleton
- [ ] Set up CI/CD pipeline
- [ ] Add linting and formatting configuration
- [ ] Write initial test suite

---

*Last updated: 2026-03-07. This file was auto-generated from an empty repository state and should be updated as the project grows.*
