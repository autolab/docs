# Autolab Documentation

All documentation regarding [Autolab](http://github.com/autolab/Autolab) setup, installation, user guide, and overview

Hosted [here](http://autolab.github.io/docs) on GitHub pages. The built HTML files are stored in the `gh-pages` branch.

Uses the latest version (v1.0.4 as of writing) of the [Mkdocs](http://www.mkdocs.org/) documentation generator.

Once your updated documentation is in `master`, run:

```bash
mkdocs gh-deploy
```

This will build the site using the branch you are currently in (hopefully `master`), place the built HTML files into the `gh-pages` branch, and push to GitHub. GitHub will then automatically deploy the new content in `gh-pages` to the production URL.
