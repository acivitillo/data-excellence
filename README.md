# data-excellence

Opinionated **Data Knowledge Base**.

Link of documentation is [here](https://datakb.com)

## Running the docs locally

### 1. Install Rye

```bash
curl -sSf https://rye.astral.sh/get | bash
```

### 2. Install dependencies

```bash
rye sync
```

### 3. Run the docs

```bash
rye run mkdocs serve
```

The docs will be available at http://127.0.0.1:8000/ (Ctrl+left click to open).
