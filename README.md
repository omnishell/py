# Omni Python Manager (`py`)

# About

**`py`** is a lightweight Python environment, project, library, lockfile, build, and package publishing manager for the [Omni Shell Framework](../omni/README.md). It streamlines Python development by managing `.venv` layouts, `pyproject.toml`, `requirements.txt`, and `py.lock` files, and can build source distributions (`.tar.gz`) and upload them directly to PyPI, TestPyPI, or custom registries via `curl` without requiring external build tools like `build` or `twine`.

## Key Features

- **Single-Command Project Initialization (`py init`)**: Bootstraps `pyproject.toml`, `requirements.txt`, `.env`, and a local `.venv` directory linked to the system Python interpreter, and registers the project in `$OMNIRC`.
- **Isolated Execution (`py exec` / `px`)**: Run scripts or CLI binaries inside the project's `.venv` (`VIRTUAL_ENV`, `PYTHONPATH`, `PATH`) from anywhere without manually activating/deactivating the virtual environment.
- **Library & Lockfile Management (`py lib`)**: Install packages from PyPI or directly from GitHub (`<owner>/<repo>`), track exact installed specs in `py.lock` and `$OMNIRC`, jump into installed package directories (`py lib access`), and sync state between `py.lock` and `$OMNIRC`.
- **Dependency Synchronization (`py dep`)**: Add, remove, and audit installed vs. missing dependencies across `pyproject.toml` and `requirements.txt`.
- **Zero-Dependency Build & Publish (`py proj build` / `push`)**: Bump semantic versions in `pyproject.toml`, build standard `.tar.gz` sdist archives into `dist/`, and publish directly to PyPI/TestPyPI using `curl` multipart uploads with SHA-256 digests.

## Prerequisites

- `python3` and `pip` (`python3 -m pip`)
- `curl`, `tar`, `awk`, `sha256sum` (or `shasum`)

---

# Install

## Via the Official `omni` Provider

```bash
omni providers install omni
omni tools use py --provider omni
```

## From a Local Directory

```bash
omni tools use -p /path/to/omni/py
```

## Directly from GitHub

```bash
omni tools use -u https://github.com/omnishell/py.git
```

---

# Usage

```bash
py <init|provider|proj|lib|dep|exec> [subcommand] [options]
```

## 1. Initialize a Project (`py init`)

Initialize the current directory as an Omni Python project:

```bash
py init [<omni_env>]
```

`py init` performs the following setup:
1. Creates a default `pyproject.toml` if one does not exist.
2. Registers the project in `$OMNIRC` (`PY_PROJS` and `ENV_PY_PROJS`).
3. Creates `.env`, `requirements.txt`, and `.venv/` (configuring `lib/pythonX.Y/site-packages` and `bin/python` / `bin/python3` symlinks).

---

## 2. Running Commands in `.venv` (`py exec` / `px`)

Execute any `.py` script, Python module, or executable inside the project's `.venv` environment (automatically enabled with the `px` alias):

```bash
# Run a Python script
py exec main.py --port 8000
px main.py --port 8000

# Run an entrypoint or interpreter command inside .venv
py run pytest -v
px python3 -c "import sys; print(sys.path)"
```

---

## 3. Project Lifecycle, Versioning, Building & Publishing (`py proj`)

```bash
# List registered Python projects (* indicates active project in $PWD)
py proj ls

# Register an existing directory as a project
py proj set [<name>] [-d|--dir <directory>]

# Unregister a project from $OMNIRC
py proj rm <name>
```

### Semantic Version Management (`py proj version`)

Read or update the `version` field in `pyproject.toml`:

```bash
# Print current project version
py proj version
py proj v

# Increment major, minor, or patch version
py proj version up patch
py proj version up minor
py proj version up major

# Decrement major, minor, or patch version
py proj version down patch

# Set an explicit version string
py proj version 1.2.0
```

### Building Distributions (`py proj build`)

Package the project into `dist/<name>-<version>.tar.gz`:

```bash
py proj build [-e|--env <project_name>]
```

### Publishing to PyPI / TestPyPI (`py proj push` / `publish`)

Upload built distributions in `dist/` to PyPI, TestPyPI, or a custom legacy upload URL:

```bash
# Publish using PYPI_API_TOKEN environment variable or configured pypi provider credentials
export PYPI_API_TOKEN="pypi-AgEIcHlwaS5vcmc..."
py proj push

# Publish to TestPyPI or specify explicit credentials
py proj publish -r|--registry testpypi -u __token__ -p "$TEST_PYPI_TOKEN"

# Publish to a custom package index URL
py proj push --url https://upload.example.com/legacy/ -u <user> -p <pass>
```

---

## 4. Library & Lockfile Management (`py lib`)

Manage installed libraries in `.venv`, tracked in both `py.lock` and `$OMNIRC`:

```bash
# Install a library from PyPI (default) or GitHub into the project's .venv
py lib install <pkg> [--version <ver>] [-a|--as <alias>] [-p|--project <proj>]
py lib i <owner>/<repo> -f|--from github [-a|--as <alias>] [--version <tag_or_branch>]

# List libraries registered for the project
py lib ls [-p|--project <proj>]

# Upgrade an installed library according to its py.lock specification
py lib update <lib_name> [-p|--project <proj>]
py lib up <lib_name>

# Uninstall/remove a library registration from py.lock and $OMNIRC
py lib uninstall <lib_name> [-p|--project <proj>]
py lib u <lib_name>

# Open an interactive subshell directly inside .venv/.../site-packages/<lib>
py lib access <lib_name> [-p|--project <proj>]
py lib . <lib_name>

# Synchronize library definitions between py.lock and $OMNIRC
py lib sync --from lock --to omnirc [-p|--project <proj>]
py lib sync --from omnirc --to lock [-p|--project <proj>]
```

---

## 5. Dependency Management (`py dep`)

Manage declared project dependencies in `pyproject.toml` and `requirements.txt`:

```bash
# Add a dependency to pyproject.toml, requirements.txt, and $OMNIRC
py dep add "requests>=2.31.0" [-p|--project <proj>]

# Remove a dependency from pyproject.toml, requirements.txt, and $OMNIRC
py dep rm requests [-p|--project <proj>]

# List declared dependencies and check whether each is installed in .venv
py dep ls [-a|--all] [-p|--project <proj>] [<requirements_file>]

# List only installed or only missing dependencies
py dep ls -i|--installed
py dep ls -m|--missing
```

---

## 6. Package Providers (`py provider`)

Configure package registries (`pypi`, custom indices) and Git hosts (`github`, custom Git servers) in `$OMNIRC`:

```bash
# List configured providers (defaults include `pypi` and `github`)
py provider ls

# Add or update a registry or git provider
py provider set <name> --type <registry|git> [--url <url>] [--user <user>] [--pass <pass_or_token>]

# Remove a provider
py provider rm <name>
```
