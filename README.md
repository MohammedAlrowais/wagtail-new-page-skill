# wagtail-new-page-skill

An AI agent skill that builds a new Wagtail page type (a detail page plus an
optional listing page) in any Django + Wagtail project, then checks the
result against a performance and quality checklist.

A skill is a plain Markdown instruction file ([`new-page/SKILL.md`](new-page/SKILL.md)).
It works with any AI coding agent and any model that can read your files
and run commands in your project.

## What it does

1. Checks that Wagtail is installed. It stops if it isn't.
2. Learns how your project is set up: how to run commands (Docker, uv,
   Poetry, venv), where apps live, your root page, test style, linters,
   templates and CSS framework.
3. Asks you questions (fields, body, structure, listing, categories,
   images, language, app), with a "use defaults" shortcut.
4. Shows a plan and waits for your yes.
5. Builds the models, snippet, StreamField blocks, filters, templates,
   migration and tests.
6. Verifies a 13-point checklist, including a test that the listing's query
   count stays the same with 2 items or 10 (no N+1 queries).
7. Suggests a branch name and commit message. It never runs git itself.

## Install

Get the files:

```bash
git clone https://github.com/MohammedAlrowais/wagtail-new-page-skill.git
```

Then use one of these:

- **Your agent supports skills** (folders with a `SKILL.md`): copy the
  [`new-page`](new-page) folder into its skills folder, either in your
  project or in your home directory. Check your agent's docs for the path.
- **Any other agent:** copy `new-page/SKILL.md` into your project (for
  example `docs/skills/new-page.md`) and point the agent at it.

## Use

With skills support, ask for a page by name, for example:

```
new-page events
```

or just: "let's build a news page".

With any other agent:

```
Follow docs/skills/new-page.md to build an events page.
```

## Requirements

- A Django project with Wagtail installed and set up (Wagtail 5+ recommended).
- An AI coding agent that can read and edit files and run commands (tests,
  migrations) in your project, so it can verify its work.
