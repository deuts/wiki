---
title: Relearn Notes
parent: Notes
---
*Relearn Hugo theme notes for [mpidocs](https://mpidocs.tuvillo.com) project*

## Child pages

The simplest syntax is:

```html
{% raw %}## Subpages
{{% children %}}{% endraw %}
```

For a more detailed approach, use this instead:

```html
{% raw %}{{% children containerstyle="div" style="h3" description="true" %}}{% endraw %}
```

Documentation available [here](https://mcshelby.github.io/hugo-theme-relearn/shortcodes/children/index.html).