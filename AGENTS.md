# Repository Guidelines

## Project Structure & Module Organization
This repository is a documentation hub, not an application runtime. Root files such as `README.md`, `SOP_HANDOVER_GUIDE.md`, `GEMINI.md`, and `AGENT.md` define shared process and contributor rules. Project-specific content lives under `projects/`, usually with a `README.md` plus supporting `docs/`. Use `projects/00-template/` when starting a new handover package. Store screenshots and diagrams beside the document set they support, for example `projects/06-cmu-alliance/assets/` or `projects/06-cmu-alliance/docs/procedures/images/`.

## Build, Test, and Development Commands
There is no global build step. Keep changes reviewable with simple repository checks:

- `rg --files .` verifies expected files and paths.
- `rg -n "keyword" projects/` finds cross-project references before duplicating content.
- `git diff --stat` confirms scope before commit.
- `git log --oneline -5` checks recent commit style.

Preview Markdown in your editor and verify links, headings, and image paths before commit.

## Coding Style & Naming Conventions
Write concise Markdown with one `#` title and logical `##`/`###` sections. Prefer short paragraphs and task-oriented bullet lists. Keep filenames descriptive and uppercase for major guides, for example `ARCHITECTURE.md` or `AZURE_SQL_SETUP_GUIDE.md`. Use kebab-case for image filenames such as `azure-vm-network-settings.png`. Match surrounding terminology; this repo primarily uses Traditional Chinese for operational content.

## Testing Guidelines
Testing is manual and document-focused. For each change:

- Verify internal links and relative asset paths.
- Confirm screenshots match the exact UI step described; do not reuse similar Azure overview screens for wizard steps.
- Check that copied procedures still align with current folder structure.

If a change affects a setup guide, note the validation method in the PR description, for example “checked image references and walked through section order.”

## Commit & Pull Request Guidelines
Recent history favors short, imperative subjects such as `docs: add ...`, `docs: refactor ...`, and `feat: add ...`. Use a scoped prefix when possible and keep each commit focused on one document set. Pull requests should include:

- a brief summary of updated documents or folders
- linked Redmine issue or task reference when available
- screenshots only when visual assets or step-by-step guides changed
- notes about any renamed or moved files

## Security & Content Handling
Do not commit secrets, credentials, VPN details, or private keys. Redact sensitive values in screenshots before adding them. After importing downloaded screenshots into the repo, remove the original download copy.
