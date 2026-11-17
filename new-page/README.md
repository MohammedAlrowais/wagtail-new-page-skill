# new-page: a Claude Code skill for Wagtail

Builds a new Wagtail page type (a detail page plus an optional listing page)
in any Django + Wagtail project, and checks the result against a performance
and quality checklist.

## What it does

1. Checks that Wagtail is installed. It stops if it isn't.
2. Learns how your project is set up: how to run commands (Docker, uv,
   Poetry, venv), where apps live, your root page, test style, linters,
   templates and CSS framework.
3. Asks you questions in pop-ups (fields, body, structure, listing,
   categories, images, language, app), with a "use defaults" shortcut.
4. Shows a plan and waits for your yes.
5. Builds the models, snippet, StreamField blocks, filters, templates,
   migration and tests.
6. Verifies a 13-point checklist, including a test that the listing's query
   count stays the same with 2 items or 10 (no N+1 queries).
7. Suggests a branch name and commit message. It never runs git itself.

## Install

Copy the `new-page` folder (this README and `SKILL.md`) to one of:

| Location | Available in |
|---|---|
| `<your project>/.claude/skills/new-page/` | That project only (and anyone who clones it) |
| `~/.claude/skills/new-page/` | All your projects |

Then start a new Claude Code session (skills load when a session starts).

## Use

```
/new-page events
```

or just ask: "let's build a news page".

## Requirements

- A Django project with Wagtail installed and set up (Wagtail 5+ recommended).
- Claude Code, with permission to run your project's commands (tests,
  migrations) so it can verify its work.
