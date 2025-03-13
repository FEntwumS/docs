# FEntwumS Project Documenation - _An Open-Source FPGA Development Environment_

This documentation should help you get started and contribute to the project.

The docs are written in Markdown and generated via [mkdocs](https://www.mkdocs.org/). 

## Deploy documentation changes

1. Install the [mkdocs python package](https://pypi.org/project/mkdocs/) 
2. Checkout `main` branch
3. run `mkdocs gh-deploy` in root of repository
    MkDocs will build the documentation and generate HTML. It is then commited to the `gh-pages` branch.
4. Github pages automatically serves the content of the `gh-pages` branch via https://fentwums.github.io/docs/
