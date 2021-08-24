# c7n-docs-ng

## Setup 

- Install [poetry](https://python-poetry.org/docs/#installation)
- `poetry config virtualenvs.in-project true`
- `poetry install`

To run mkdoc commands, use `poetry run mkdocs`. For example, `poetry run mkdocs serve`

## Commands 

Build:

    poetry run mkdocs build

Live preview:

    poetry run mkdocs serve

Live preview fast mode: 

    poetry run mkdocs serve --dirtyreload

Push to github pages: 

    poetry run mkdocs gh-deploy

## Inspiration 

- [Thanks knative.dev](https://github.com/knative/docs/blob/mkdocs/docs/help/contributor/mkdocs-contributor-guide.md)