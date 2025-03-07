---
title: Relearn Notes
parent: Notes
---
*Relearn Hugo theme notes for [mpidocs](https://mpidocs.deuts.org) project*

## Child pages

The simplest syntax is:

```html
{% raw %}## Subpages
{{% children %}}{% endraw %}
```

For a more detailed approach:

```html
{% raw %}
{{% children containerstyle="div" style="h3" description="true" %}}
{% endraw %}
```