# My Learning Journey

A structured collection of personal notes and resources covering various topics, including Databases, Cloud platforms, Containerization tools, Scripting, and Schedulers.

This site is powered by [MkDocs](https://www.mkdocs.org/) with the `readthedocs` theme.

---

## Getting Started (Local Setup)

To spin up this documentation site locally, follow the steps below depending on your preferred package manager.

### Option 1: Using `uv` (Recommended)

If you have `uv` installed, setting up and running the site is extremely fast:

1. **Clone the repository** (if not already done) and navigate to the directory:
   ```bash
   cd mylearnings
   ```

2. **Sync dependencies and start the local development server**:
   ```bash
   uv run mkdocs serve
   ```
   *This command automatically sets up the virtual environment, installs the required packages from `pyproject.toml`, and boots the server.*

3. **Access the site**:
   Open [http://127.0.0.1:8000](http://127.0.0.1:8000) in your web browser.

---

### Option 2: Using standard Python `venv` & `pip`

If you don't have `uv` installed, you can use the standard python workflow:

1. **Create and activate a virtual environment**:
   ```bash
   python3 -m venv .venv
   source .venv/bin/activate
   ```

2. **Install the dependencies**:
   ```bash
   pip install -r pyproject.toml
   # Or install packages directly:
   pip install mkdocs>=1.6.1 mkdocs-open-in-new-tab>=1.0.8
   ```

3. **Run the development server**:
   ```bash
   mkdocs serve
   ```

4. **Access the site**:
   Open [http://127.0.0.1:8000](http://127.0.0.1:8000) in your web browser.

---

## Build for Production

To generate the static HTML site (e.g., for hosting on GitHub Pages):

```bash
# Using uv
uv run mkdocs build

# Using active virtual environment
mkdocs build
```

This compiles all files in `docs/` and outputs the build to the `site/` directory.
