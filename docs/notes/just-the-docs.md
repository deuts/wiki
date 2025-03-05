---
title: "Just the Docs"
nav_order: 50
parent: Notes
---
# What are unique in Just the Docs
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