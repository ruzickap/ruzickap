# AI Agent Guidelines

## Project Overview

This is a **GitHub Profile README repository** (`ruzickap/ruzickap`).
It contains no application source code -- only Markdown, YAML (GitHub
Actions workflows), JSON/JSON5 configs, and TOML configs. CI validation
is handled by MegaLinter (documentation flavor).

## Build / Lint / Test Commands

There is no build step or test suite. All validation runs via CI.
To lint locally, use these individual tools:

```bash
# Markdown linting
rumdl .

# Shell linting (standalone scripts or extracted code blocks)
shellcheck --exclude=SC2317 file.sh

# Shell formatting
shfmt --case-indent --indent 2 --space-redirects file.sh

# JSON linting (comments allowed)
jsonlint --comments file.json

# Link checking
lychee --config lychee.toml .

# GitHub Actions workflow validation
actionlint

# Terraform (if .tf files are added)
tflint
checkov --quiet -d .
trivy fs --severity HIGH,CRITICAL --ignore-unfixed .

# TypeScript/JavaScript formatting (if applicable)
prettier --html-whitespace-sensitivity=ignore --write file
```

There are no individual test commands; MegaLinter runs all linters
together in CI on push to non-main branches.

## Code Style Guidelines

### Markdown

- Lint with `rumdl` (config: `.rumdl.toml`); `CHANGELOG.md` excluded
- Proper heading hierarchy -- never skip levels
- Wrap lines at **80 characters**
- Use fenced code blocks with language identifiers (`bash`, `json`,
  `yaml`, etc.)
- Prefer code fences over inline code for multi-line examples
- Use semantic HTML only when necessary
- Shell code blocks (tagged `bash`, `shell`, or `sh`) are extracted
  and validated by `shellcheck` and `shfmt` during CI

### Shell Scripts and Code Blocks

- Must pass `shellcheck` (SC2317 excluded)
- Format with `shfmt --case-indent --indent 2 --space-redirects`
- Use **2 spaces** for indentation, never tabs
- Variables in **UPPER_CASE**: `${MY_VARIABLE}`
- Quote variable expansions: `"${VARIABLE}"`
- Use `set -euxo pipefail` in multi-line scripts
- Indent `case` statements

### YAML (GitHub Actions Workflows)

- Start files with `---` document separator
- Include a block comment header describing the workflow purpose
- Set `permissions: read-all` at the top level; scope overrides
  per job as needed
- **Pin all actions to full SHA** with a version comment:
  `uses: actions/checkout@<sha> # v6.0.2`
- Set `timeout-minutes` on every job
- Use `# keep-sorted` comments for alphabetically ordered blocks
- Environment variables in UPPER_CASE

### JSON / JSON5

- Must pass `jsonlint --comments`
- Comments are allowed (JSON5 style `//` and `/** */`)
- Use `# keep-sorted` comments for alphabetical ordering

### General Formatting

- **2 spaces** for indentation everywhere (YAML, JSON, Markdown, shell)
- No tabs
- Consistent formatting maintained by automated linters

## Naming Conventions

- **Shell variables**: UPPER_CASE (`${GITHUB_TOKEN}`)
- **Branch names**: `<type>/<description>` using Conventional Branch
  format (e.g., `feat/add-widget`, `fix/broken-link`,
  `chore/update-deps`)
- **Files**: lowercase with hyphens (e.g., `blog-post-workflow.yml`)

## Error Handling

- Shell scripts: always use `set -euxo pipefail` or equivalent
  shell option (`bash -euxo pipefail {0}` in workflow `defaults`)
- Quote all variable expansions to prevent word splitting

## Security

CI runs multiple security scanners (`checkov`, `devskim`, `kics`,
`trivy`). When modifying workflows or IaC:

- Never expose secrets in logs
- Use `${{ secrets.* }}` for sensitive values
- Pin GitHub Actions to full SHA, not tags
- Prefer `permissions: read-all`; grant write only where needed

## Version Control

### Commit Messages

Conventional commit format validated by `commit-check` action:

- **Format**: `<type>: <description>` (lowercase, imperative mood)
- **Types**: `feat`, `fix`, `docs`, `chore`, `refactor`, `test`,
  `style`, `perf`, `ci`, `build`, `revert`
- **Subject**: max 72 characters, no trailing period
- **Body**: optional; wrap at 72 characters; explain what and why
- **References**: use `Fixes`, `Closes`, or `Resolves` for issues

```text
feat: add automated dependency updates

- Implement Dependabot configuration
- Configure weekly security updates

Resolves: #123
```

### Branching

Follow [Conventional Branch](https://conventional-branch.github.io/):

- `feat/` or `feature/` -- new features
- `fix/` or `bugfix/` -- bug fixes
- `hotfix/` -- urgent production fixes
- `release/` -- release prep (e.g., `release/v1.2.0`)
- `chore/` -- maintenance tasks

Use lowercase, hyphens, no consecutive/leading/trailing hyphens.
Include issue numbers when applicable: `feat/issue-42-new-widget`.

### Pull Requests

- Always create as **draft** initially
- Title must follow conventional commit format (enforced by
  `semantic-pull-request` action)
- Include a clear description and link related issues

## CI Pipelines

| Workflow                | Trigger             | Purpose                          |
|-------------------------|---------------------|----------------------------------|
| `mega-linter`           | Push (non-main)     | Linting and security checks      |
| `commit-check`          | PR to main          | Commit message validation        |
| `semantic-pull-request` | PR events           | PR title validation              |
| `release-please`        | Push to main        | Automated releases and changelog |
| `renovate`              | Push to main/weekly | Dependency updates               |
| `codeql`                | Push to main/weekly | Static security analysis         |
| `scorecards`            | Push to main/weekly | OSSF security scoring            |
