# Django project template

A [Copier](https://copier.readthedocs.io) template for new Django projects with uv, PostgreSQL, Redis, Celery, Docker Compose, ruff, ty, prek, pytest, Django Silk and Django Extensions.

Only the `template/` folder is copied into new projects. This README and `copier.yml` stay here.

## Requirements (on your Mac)

- [uv](https://docs.astral.sh/uv/)
- [prek](https://github.com/j178/prek), for the Git hooks
- Docker Desktop
- Git

## Create a new project

```bash
uvx copier copy --trust gh:osamahasanone/django_template ~/Desktop/my_project
```

Copier asks for:

- **Project name**: lowercase with underscores, e.g. `my_project`. Also used for the PostgreSQL database, user and password in `.env`.
- **Description**: goes into `pyproject.toml`.
- **Python version**: press Enter to keep `3.14`. The same value is written to `pyproject.toml`, `.python-version` and the `Dockerfile`, so they always match.

`--trust` is needed because the template runs commands after copying the files. Those commands:

1. Run `git init`.
2. Install all packages with `uv add`, always the latest versions.
3. Generate a unique `DJANGO_SECRET_KEY` and add it to `.env`.
4. Install the prek Git hooks.
5. Run ruff to fix and format the code.

Then start the project:

```bash
cd ~/Desktop/my_project
docker compose up -d --build
docker compose exec web uv run python manage.py migrate
docker compose exec web uv run python manage.py createsuperuser
```

Open http://localhost:8000/admin and log in, and http://localhost:8000/silk/ to see profiled requests and SQL queries.

## What's in a new project

| File | Purpose |
|---|---|
| `pyproject.toml` | Project metadata, ruff and pytest settings |
| `.env` / `.env.example` | Real local values (never committed) / the list of variables to set (committed) |
| `config/settings.py` | Reads `.env`, uses PostgreSQL, configures Celery, enables Silk when `DEBUG` is on |
| `config/urls.py` | Admin, plus `/silk/` when `DEBUG` is on |
| `config/celery.py` | Creates the Celery app |
| `Dockerfile`, `.dockerignore` | Image for `web` and `celery` |
| `compose.yaml` | `web` (Django), `celery` (worker), `db` (PostgreSQL), `redis` (broker), with healthchecks |
| `prek.toml` | Checks that run on every commit |
| `.vscode/tasks.json` | Common commands via `Cmd + Shift + P` → Tasks: Run Task |
| `.vscode/settings.json` | Lets Pytest Runner run tests inside the `web` container |
| `.copier-answers.yml` | Your answers, used by `copier update` |

## Change the template

1. Edit files in `template/`. Files ending in `.jinja` can use `{{ project_name }}`, `{{ description }}` and `{{ python_version }}`. Copier removes `.jinja` from their names.
2. Commit and tag a new version:
   ```bash
   git commit -am "Describe the change"
   git tag v1.1.0
   git push --follow-tags
   ```
3. Pull the change into an existing project (run inside that project, with its changes committed):
   ```bash
   uvx copier update --trust
   ```

## Optional tools

Run all tests:

```bash
docker compose exec web uv run pytest -v
```

Pytest Runner (VS Code extension): in *Preferences: Open Keyboard Shortcuts*, set `pytest-runner.run-test-docker` to `Cmd+Q Cmd+1` and `pytest-runner.run-module-test-docker` to `Cmd+Q Cmd+2`.

Django Silk (request and SQL profiler) is a dev dependency and only runs when `DJANGO_DEBUG=True`, so it never runs in production. Browse it at http://localhost:8000/silk/.

Django Extensions adds management commands like `shell_plus`, `runserver_plus` and `graph_models`. `ipython` is installed too, for a better `shell_plus` REPL:

```bash
docker compose exec web uv run python manage.py shell_plus
```

DBeaver (database viewer):

```bash
brew install --cask dbeaver-community
```

Connect with host `localhost`, port `5432`, and the `POSTGRES_*` values from `.env`.
