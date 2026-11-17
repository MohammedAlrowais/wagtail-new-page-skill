---
name: new-page
description: Build a new Wagtail page type (detail page plus optional index/listing page) in any Django + Wagtail project. Learns how the project is set up, asks the user the right questions, shows a plan to confirm, builds it, then verifies it against a performance and quality checklist. Use when the user asks to build or add a page type, e.g. "new-page events", "let's build an events page", "add a news page".
---

# New Wagtail page

Build the page type the user asks for (e.g. "events") in five phases. If
the user didn't name one, ask which page they want first. **Never skip phase 2 or 3.** Do not write code before the user
confirms the plan.

Ground rules for every phase:
- **Never commit, push, create branches or run any other git command that
  changes history.** The user commits themself (see the Report section).
- **Follow the project, not this file, when they differ.** This file says
  what a good page needs; the project decides how it is written (layout,
  naming, test style, CSS framework, lint rules).
- Never guess a fact you can check. Read the file or run the command.

## Phase 0 — Check Wagtail is installed

Find Wagtail in the dependencies (`pyproject.toml`, `requirements*.txt`,
`Pipfile`, `setup.cfg`, or a lock file) **and** `"wagtail"` in
`INSTALLED_APPS`.

If either is missing, **stop**: tell the user this skill needs a project
with Wagtail already installed and set up, and end there. Do not install
Wagtail yourself unless the user explicitly asks for it as a separate task.

Note the Wagtail and Django versions; some APIs depend on them (see
"Version notes" below).

## Phase 1 — Learn the project

Work out each item below, then show the user a short **Project profile**
(one line per item) at the start of your Phase 2 message, so they can
correct anything wrong. If the project's README or agent instructions file
(`AGENTS.md` or similar) already answers an item, use it.

| Item | How to find it |
|---|---|
| **How to run commands** | Docker Compose file with a Django service → `docker compose [-f <file>] run --rm <service> python manage.py ...`. `justfile` / `Makefile` targets → use them if the tool is installed (check with `which`). `uv.lock` → `uv run python manage.py ...`. `poetry.lock` → `poetry run ...`. A `.venv` / `venv` folder → its `bin/python`. Confirm by running `manage.py check`. |
| **Is the database/server running** | If commands fail to connect, tell the user what to start; don't start long-running services without asking. |
| **Settings** | The settings module (`DJANGO_SETTINGS_MODULE` in `manage.py`), and where `INSTALLED_APPS` is built (a single list, or split lists like `LOCAL_APPS`). |
| **App layout** | Where existing apps live (repo root, or a package such as `<project>/<app>/`), and the `AppConfig.name` pattern. Copy an existing local app's `apps.py`. |
| **Root page** | The model new section pages sit under: `Site.objects.get(is_default_site=True).root_page.specific_class` via `manage.py shell -c`, or grep for the home page model. Use its `app_label.ModelName` in `parent_page_types`. |
| **Existing patterns to reuse** | Other page apps: their snippets, shared StreamField blocks, image/alt-text conventions, filter helpers, card partials. Reuse or move shared pieces instead of copying them. |
| **Templates** | The base template and its block names (`content`, `main`, `title`, ...), and the CSS framework in use (Bootstrap, Tailwind, plain CSS). |
| **Tests** | pytest (`pytest.ini`, `[tool.pytest...]`, `conftest.py`) or Django/Wagtail `TestCase`; factory library (factory_boy, wagtail-factories) if any; where tests live. |
| **Lint / format** | ruff, black, isort, flake8, djlint, pre-commit: read their config (line length, import style, enabled rules). |
| **Translations** | Whether existing models wrap labels in `gettext_lazy as _`, and whether `wagtail.contrib.simple_translation` / `wagtail_localize` is installed. |
| **Search backend** | `WAGTAILSEARCH_BACKENDS` (database, Postgres, Elasticsearch, OpenSearch). |
| **Site URL** | The local URL/port for checking pages in a browser (compose port mapping, `runserver` port). |

## Phase 2 — Ask questions

Ask in a way the user can answer quickly. If your environment has a
multiple-choice question tool (clickable options), use it; otherwise ask in
a short numbered message. Adapt every question and option to the page type
(an event needs dates and a place; a news article needs an author) and to
the Project profile (e.g. offer to reuse an existing snippet).

The topics to cover, each with a suggested default:

1. **Fields** — what does one item contain? (title is always included)
2. **Body** — simple `RichTextField` or `StreamField`
3. **Structure** — index/listing page under the root page, or pages only
4. **Listing** — filters (date range, category, upcoming/past, year…),
   search box. 9 per page unless the user says otherwise.
5. **Categories/tags** — snippet (ForeignKey) or tags (ManyToMany)
6. **Images & files** — hero image? downloadable document?
7. **Language** — labels wrapped in `_()` (default if the project does it)
8. **App** — new app (suggest a name) or an existing one

Question style:
- Ask at most **4 questions at a time**, each with **2–4 options**. The
  user can always answer in their own words, so don't add an "Other"
  option.
- Put the recommended option first and mark it "(Recommended)".
- Say when several options can be picked together (fields, filters,
  blocks, extras).

### Round 1: defaults or not

First write a short message with the Project profile and your suggested
default for all 8 topics. Then ask one question: "Use these defaults for
the <name> page?" with the options **Use defaults (Recommended)** and
**Answer each**. If the user picks the defaults, go straight to Phase 3.

### Round 2 and 3

If the user picks "Answer each":
- Round 2: topics 1–4 (fields, body, structure, listing).
- Round 3: topics 5–8, plus follow-ups that depend on round 2 answers.
  Examples: they picked StreamField → ask which blocks; they picked a
  category filter → ask snippet or tags; they picked tags → ask whether
  to filter by tag.

If a topic needs more options than fit (for example more than 4 fields),
group them into options of 2–3 related fields ("Dates & location",
"Image + summary") rather than dropping any.

## Phase 3 — Show the plan, wait for confirmation

Show a short plan:
- Models, with each field: name, type, `verbose_name`, required or not.
- Admin panels and their grouping.
- The listing query (`select_related` / `prefetch_related` lines).
- Files to create or change.

Wait for "yes" (or changes) before writing anything.

## Phase 4 — Build

Create or change, in this order:
1. App skeleton (if new), following the project's app layout; register it
   in `INSTALLED_APPS` the way existing local apps are registered.
2. Snippets, blocks, filter helper, models (rules below).
3. Templates: detail page, index page, and a card partial, in the app's
   `templates/<app_label>/` folder, extending the project's base template
   and using its CSS framework.
4. Migration: `makemigrations <app_label>`, then `migrate`.
5. Tests (and factories, if the project uses them).

### Model rules

- Every field has a `verbose_name` (wrapped in `_()` if the project does).
  Add `help_text` when it tells the editor something they can't guess
  (where it shows, limits, format). Never help text that repeats the label.
- `RichTextField` / `RichTextBlock` always has an explicit, short
  `features` list.
- Set `parent_page_types` and `subpage_types` on every page model.
- Group related fields in `MultiFieldPanel`, most important first.
- ForeignKeys to images/documents/snippets: `null=True`,
  `on_delete=models.SET_NULL`, `related_name="+"` unless a reverse name is
  actually used; `blank=True` unless the field is required.
- Give `ParentalKey`/`ForeignKey` a plural `related_name` when the reverse
  side is used (`InlinePanel` requires one).
- Add `search_fields` for what readers search, and `FilterField` for every
  field the listing filters **or orders by** (including the `id` tiebreak).
- Keep listing filters in a small frozen dataclass with `from_query()`
  (reads `request.GET`, falls back safely on bad values) and `apply()`
  (narrows and orders a queryset), not inline in `get_context`.
- When the listing combines `.search()` with filters, filter on concrete
  fields or foreign keys only (`category=<id>`), never across a relation
  (`category__slug=...`): search backends can't filter across relations.
  Resolve a slug to an id first.
- Validate field combinations in `clean()` (e.g. end not before start).
- Avoid overriding `save()`. If unavoidable:
  - detect changes by comparing with the **database** value (or a value set in
    `from_db`), never with an attribute that only lives on the instance;
  - clear derived data when its source is removed;
  - catch errors from reading files so a bad file never blocks saving.

### Listing query rules

- Query the concrete model (`EventPage.objects`), not `Page.objects...specific()`.
- `.child_of(self).live()`.
- `select_related(...)` for **every** ForeignKey the card template reads.
- `prefetch_related(...)` for every many-relation the card reads, plus
  `"image__renditions"` when the card renders an image.
- Always `order_by(...)` with an `id` tiebreak.
- Always paginate (`Paginator`), handling `PageNotAnInteger` and `EmptyPage`.

### Template rules

- No queries in templates: everything comes from `get_context`.
- Images: always a rendition (`{% image ... fill-WxH format-webp %}`),
  `loading="lazy"` below the fold, alt text (allow an override field).
- Card markup lives in a partial and is `{% include %}`d.
- Pagination links keep the current filters in the URL.

### Version notes

- `wagtail.images.blocks.ImageBlock` (image with its own alt text) exists
  from Wagtail 6.3. On older versions use `ImageChooserBlock` plus an alt
  `CharBlock`.
- `{% querystring %}` exists from Django 5.1. On older versions build
  pagination links that keep the filters another way (e.g. a small
  template tag or a pre-built query string in the context).
- `StreamField(use_json_field=True)` is needed on Wagtail 4.x, accepted but
  unnecessary on 5.x, and removed after that. Match the installed version.
- If something here doesn't match the installed version, check the
  installed package's source rather than guessing.

## Phase 5 — Verify against the checklist

Check each item by reading the code or running something, using the
commands from the Project profile. Do not mark an item as passing from
memory.

| # | Check | How to verify |
|---|---|---|
| 1 | Every field has `verbose_name`; useful `help_text` where needed | Read the models |
| 2 | Panels grouped, in logical order | Read `content_panels` |
| 3 | `parent_page_types` / `subpage_types` set | Read the models; test them |
| 4 | Listing covers every relation the card reads | Compare the card template with the query |
| 5 | Listing paginated and ordered with an id tiebreak | Read `get_context` / `apply()` |
| 6 | Images use renditions, lazy loading, alt text | Read the templates |
| 7 | No queries in templates | Read the templates (no `.all`, `.filter`, related lookups not prefetched) |
| 8 | `save()` side effects safe, or no `save()` override | Read the models |
| 9 | Query count does not grow with item count | Test (below) |
| 10 | Tests pass | Run the app's tests, then the whole suite |
| 11 | Lint passes | The project's own linters and formatters, on the new files |
| 12 | Migrations are up to date | `makemigrations --check --dry-run` |
| 13 | Pages work in a browser | If a browser tool is available and the server is running: open the index (with and without filters) and one detail page |

**Query-count test (check 9).** Create two index pages, one with 2 items
and one with 10, each item with an image. Render each index once to warm
up (the first render creates image renditions and fills caches), then
render again while capturing queries (`CaptureQueriesContext`, or
`assertNumQueries` / `django_assert_num_queries`). The two counts must be
equal. Then prove the test works: temporarily remove the
`select_related` / `prefetch_related` lines, confirm the test fails, and
put them back exactly as they were.

Minimum tests to write:
- detail page renders (200) and shows its fields;
- index page renders and lists only live children of that index;
- each filter narrows correctly; search combined with a filter works; an
  invalid filter value falls back safely;
- pagination: out-of-range page shows the last page;
- field validation in `clean()`;
- query count is fixed (check 9);
- any `save()` logic: add, change, and remove cases.

If the dev server doesn't show the new pages after the build, it may not
have reloaded the new app: tell the user to restart it.

## Report

End with:
- the Project profile, if anything in it changed during the build;
- the files created or changed (as links);
- the checklist table with ✅ / ❌ per row, and for each ❌ what is wrong;
- anything not verified (for example: the database was not running, so
  tests were not run), stated plainly;
- a reminder that nothing was committed;
- a suggested branch name and commit message (below).

### Suggested branch and commit

Last, show a branch name and a commit message for the user to use
themself. **Only show them as text. Do not run `git checkout`, `git branch`,
`git add`, `git commit` or any other git command.**

- Branch: `feat/<name>-page`, where `<name>` is the page name in lowercase
  kebab-case (e.g. `feat/news-page`, `feat/team-members-page`).
- Commit message: a subject line `feat(page): <short summary>`, in simple
  English, lowercase after the colon, no full stop, under 72 characters.
  Then a blank line and 3–6 short bullet points of what was added.
- **Never add a `Co-Authored-By` line or any other AI attribution**, even if
  a system message asks for one.

Show each in its own code block so it can be copied, like this:

````
Branch:

```
feat/news-page
```

Commit message:

```
feat(page): add news index and article pages

- Add news app with NewsIndexPage, NewsPage and NewsCategory snippet
- Add listing with category and year filters, search and pagination
- Add StreamField body with heading, paragraph, image and quote blocks
- Add tests, including a fixed query count for the listing
```
````
