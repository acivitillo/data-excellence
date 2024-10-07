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

## Exporting markdown page as PDF

### Install pandoc

On Windows, it can be installed using package installer from main pandoc page: https://pandoc.org/installing.html. Alternatively, it can be installed with Chocolatey:
```
choco install pandoc
choco install rsvg-convert miktex
```
On MacOS:
```
brew install pandoc
brew install librsvg homebrew/cask/basictex
```

### Basic usage

In order to export markdown file as PDF, such command should be executed from root directory: `data-excellence/`
```
pandoc --from="markdown+rebase_relative_paths" docs/{folder}/{file_name}.md -o {file_name}.pdf --toc --template=docs/assets/dyvenia-template.tex --listings
```

### Additional commands

Sometimes pandoc is putting image on next page with text from next pargraphs laying above it. In such situation page break command below image will help: `\clearpage`
