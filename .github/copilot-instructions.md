# Copilot instructions for this repo

## Project overview
- This repository is a GitHub Skills "GitHub Pages" training repo. It is primarily driven by GitHub Actions workflows and step files rather than traditional application code.
- The learner experience is controlled by:
  - `.github/workflows/*.yml` – automation that advances the course steps.
  - `.github/steps/*.md` – step-by-step learner instructions.
  - `.github/steps/-step.txt` – a single-line file storing the current numeric step (0–5 or `X`).
  - `README.md` – shows the current step to the learner; updated automatically by actions.
  - `dependabot.yml` – manages dependency updates (for example, `actions/checkout`).

## Architecture and flow
- Workflows follow a consistent pattern:
  - Job `get_current_step` reads `.github/steps/-step.txt` and exposes `current_step` as an output.
  - A second job (e.g., `on_start`, `on_enable_github_pages`, `on_config_update`, `on_homepage_update`, `on_create_blog_post`, `on_merge`) runs only when:
    - The repository is not the template (`!github.event.repository.is_template`), and
    - `needs.get_current_step.outputs.current_step` matches the expected step number.
  - The main job checks out the repo and uses `skills/action-update-step@v2` to replace one step marker in `README.md` (e.g., from step `1` to step `2`) on the `my-pages` branch.
- Step progression logic:
  - `0-welcome.yml` creates the `my-pages` branch, adds `_config.yml` and `index.md`, commits, pushes, then updates step `0 → 1`.
  - `1-enable-github-pages.yml` listens for a Pages build and updates step `1 → 2`.
  - `2-configure-your-site.yml` triggers on a push to `_config.yml` in `my-pages` and updates step `2 → 3`.
  - `3-customize-your-homepage.yml` triggers on a push to `index.md` in `my-pages` and updates step `3 → 4`.
  - `4-create-a-blog-post.yml` triggers on new/updated markdown files in `_posts/*.md` in `my-pages` and updates step `4 → 5`.
  - `5-merge-your-pull-request.yml` triggers on pushes to `main` and updates step `5 → X`.

## Conventions and patterns
- Always use `actions/checkout@v6` (this repo is on the `dependabot/github_actions/actions/checkout-6` branch).
- All workflows:
  - Run on `ubuntu-latest`.
  - Restrict permissions to the minimum required (`contents: write`, plus `pull-requests: write` only where needed).
  - Use a separate `get_current_step` job and `needs.get_current_step.outputs.current_step` guard instead of inline scripts to control flow.
- Git operations in `0-welcome.yml`:
  - Branch name is hard-coded to `my-pages` and should stay consistent across workflows and actions.
  - Bot identity is set explicitly before committing:
    - `user.name`: `github-actions[bot]`
    - `user.email`: `github-actions[bot]@users.noreply.github.com`

## When editing or adding workflows
- Preserve the step gating pattern:
  - Keep a dedicated `get_current_step` job.
  - Ensure the main job has the `if` condition checking `!github.event.repository.is_template` and the specific step number.
- When using `skills/action-update-step@v2`:
  - Keep `from_step` / `to_step` consistent with the step file name and README instructions.
  - Always pass `branch_name: my-pages` so learner-visible changes happen on the learner branch.
- If you add new steps:
  - Add a new workflow file under `.github/workflows/` following the existing naming and structure.
  - Add a matching markdown file under `.github/steps/` and update any references in existing steps.
  - Update the step progression so that only one workflow advances from a given step value.

## Working with the training content
- `README.md` is learner-facing and is updated automatically; avoid hard-coding step numbers that conflict with automation.
- `.github/steps/*.md` are the canonical source for step text; changes here will be picked up by the course engine, not by direct edits in `README.md`.

## Local development and checks
- There is no traditional build or test suite; the primary "runtime" is GitHub Actions.
- To verify changes:
  - Validate workflow syntax locally with `act` or via GitHub's workflow editor.
  - Use `workflow_dispatch:` to manually trigger workflows when testing in a fork.
- Be cautious when modifying triggers (`on:` blocks); they are tightly coupled to the learner's actions (e.g., editing `_config.yml`, `index.md`, and `_posts/*.md`).
