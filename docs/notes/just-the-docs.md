---
title: "Just the Docs Documentation"
nav_order: 50
parent: Notes
nav_exclude: true
---
# What are unique in Just the Docs

## Minimum frontmatter content
Minimum frontmatter is `title` key.  This is assuming you have set the default layout in the `_config.yml` file, like so:
```yml
defaults:
  -
    scope:
      path: "" # an empty string here means all files in the project
    values:
      layout: default
```

It is not required, but *recommended* that child pages of the same parent is organized in one folder. This is really just organization. But you can put them anywhere in the `docs` folder.

## Interpage linking
```markdown
[integration with Obsidian]({% link docs/notes/obsidian-mkdocs.md %})
```

To quote the [Jekyll documentation](https://jekyllrb.com/docs/liquid/tags/#link):
> The path to the post, page, or collection is defined as the path relative to the root directory (where your config file is) to the file, not the path from your existing page to the other page.