# hashcott.github.io

Personal site of Nguyễn Đức Hạnh at **<https://hashcott.github.io>**: the projects I'm working on, release announcements, and the occasional post.

Built with GitHub Pages' built-in Jekyll. No theme, no JavaScript; the whole look lives in one CSS file.

## Layout

```
_config.yml         site name, description, author info
_data/projects.yml  projects listed on the home page
_posts/             posts and release updates
_layouts/           default.html (page shell), post.html (posts)
assets/style.css    styles, light and dark
img/                images used in posts
```

## Adding content

**Project:** add an entry to `_data/projects.yml`:

```yaml
- name: project-name
  url: https://github.com/hashcott/project-name
  site: https://hashcott.github.io/project-name/   # optional
  desc: One-sentence description.
```

**Release update:** create `_posts/YYYY-MM-DD-post-name.md`:

```markdown
---
title: Project name 1.2.0
type: update
---

Body...
```

**Post:** same as above without the `type: update` line. You can also add `subtitle:` and `tags: [...]`.

Post URLs look like `/YYYY-MM-DD-post-name/`.

## Running locally

You need Ruby and Bundler:

```sh
bundle install
bundle exec jekyll serve
```

Then open <http://localhost:4000>. Pushing to `master` makes GitHub Pages rebuild the site.
