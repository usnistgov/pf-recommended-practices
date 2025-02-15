# Phase Field Method Recommended Practices

A set of recommended practices that help to ensure the best outcomes from your
use of the phase-field method.

## Usage

### Building the book

If you'd like to develop and/or build the Phase Field Method Recommended
Practices book, you should:

1. Clone this repository
2. Run `pip install -r requirements.txt` (it is recommended you do this within
   a virtual environment)
3. (Optional) Edit the books source files located in the
   `pf-recommended-practices/` directory
4. Run `jupyter-book clean pf-recommended-practices/` to remove any existing
   builds
5. Run `jupyter-book build pf-recommended-practices/`

A fully-rendered HTML version of the book will be built in
`pf-recommended-practices/_build/html/`. Render using `python -m http.server`
in the `pf-recommended-practices/_build/html/` directory.

### Hosting the book

Please see the [Jupyter Book documentation][jb-docs] to discover options for
deploying a book online using services such as GitHub, GitLab, or Netlify.

For GitHub and GitLab deployment specifically, the
[cookiecutter-jupyter-book][eb-cook] repository includes templates for, and
information about, optional continuous integration (CI) workflow files to help
easily and automatically deploy books online with GitHub or GitLab. For
example, if you chose `github` for the `include_ci` cookiecutter option, your
book template was created with a GitHub actions workflow file that, once pushed
to GitHub, automatically renders and pushes your book to the `gh-pages` branch
of your repo and hosts it on GitHub Pages when a push or pull request is made
to the main branch.

## Contributors

We welcome and recognize all contributions. You can see a list of current
contributors in the [contributors tab][gh-cont].

## Credits

This project is created using the excellent open source [Jupyter Book][jb]
project and the [executablebooks/cookiecutter-jupyter-book][eb-cook] template.

<!-- links -->
[gh-cont]: https://github.com/guyer/pf-recommended-practices/graphs/contributors

[eb-cook]: https://github.com/executablebooks/cookiecutter-jupyter-book
[jb]:      https://jupyterbook.org/
[jb-docs]: https://jupyterbook.org/publish/web.html
