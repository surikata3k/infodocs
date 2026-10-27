# IT-Hub

A place where I keep IT snippets, tutorials, practice exercises and resources.

Site: <https://surikata3k.github.io/it-hub/> (built with [Zensical](https://zensical.org)).

## Edit locally

```bash
pip install zensical
zensical serve      # preview at http://localhost:8000
```

Pages live in `docs/`, one folder per section. Add a new page to the `nav` list in `zensical.toml`.
Every push to `main` publishes the site through `.github/workflows/docs.yml`.
