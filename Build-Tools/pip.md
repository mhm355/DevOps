# pip - Python Package Installer

## Files

### `requirements.txt`
Lists Python dependencies with optional versions.

```txt
flask==2.0.1
requests>=2.25.0
numpy
pandas~=1.3.0
```

**Limitation:** Only lists packages—cannot specify Python version, authors, licensing, or build configs.

### `pyproject.toml`
Modern alternative with all project configuration in a single file.

```toml
[project]
name = "my-project"
version = "1.0.0"
requires-python = ">=3.8"
dependencies = [
    "flask>=2.0",
    "requests"
]

[project.optional-dependencies]
dev = ["pytest", "black"]

[build-system]
requires = ["setuptools>=45"]
build-backend = "setuptools.build_meta"
```

---

## CLI Commands

### Basic Commands
```bash
pip install package_name            # Install latest from PyPI
pip uninstall package_name          # Remove package
pip install package_name --upgrade  # Upgrade package
```

### Project Dependencies
```bash
pip install -r requirements.txt     # Install from requirements file
pip install -r requirements.txt --cache-dir .pip_cache  # With caching
```

### Dependency Management
```bash
pip freeze > requirements.txt       # Generate locked requirements
pip check                           # Validate compatible dependencies
pip list --outdated                 # Show packages with updates
```

### Caching
```bash
pip download --cache-dir .pip_cache -r requirements.txt  # Pre-download packages
```

---

## Virtual Environments

Always use virtual environments to isolate project dependencies:

```bash
# Create virtual environment
python -m venv venv

# Activate (Linux/Mac)
source venv/bin/activate

# Activate (Windows)
venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Deactivate
deactivate
```

---

## Poetry (Alternative)

Modern Python packaging and dependency management.

```bash
poetry init                  # Create new project
poetry add flask            # Add dependency
poetry install              # Install from pyproject.toml
poetry lock                 # Generate lock file
poetry install --no-dev     # Production install
```
