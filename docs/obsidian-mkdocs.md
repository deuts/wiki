---
title: Obsidian with MkDocs
nav_order: 80
layout: default
---

# Integrating Obsidian with MkDocs

Obsidian is a powerful knowledge base that works on local Markdown files, while MkDocs is a fast, simple static site generator that's perfect for building documentation sites. Integrating these tools can create a seamless workflow from personal notes to published documentation.

## Best Practices for Integration

### 1. Set Up a Consistent File Structure

Create a structure that works well for both tools:

```
project/
├── docs/               # MkDocs content directory
│   ├── index.md
│   └── ...
├── mkdocs.yml          # MkDocs configuration
└── .obsidian/          # Obsidian configuration
```

### 2. Use Compatible Plugins

#### For Obsidian:
- **Templater**: Create templates that follow MkDocs formatting
- **Obsidian Git**: Sync your vault with a Git repository
- **Admonition**: Create notes that are compatible with MkDocs admonitions

#### For MkDocs:
- **Material for MkDocs**: Excellent theme with Obsidian-like features
- **mkdocs-roamlinks-plugin**: Support for Obsidian-style `[[wiki links]]`
- **mkdocs-obsidian-bridge**: Specifically designed for Obsidian integration

### 3. Configure MkDocs for Obsidian Compatibility

Create a `mkdocs.yml` file:

```yaml
site_name: My Documentation
theme:
  name: material
  palette:
    scheme: slate  # Dark mode similar to Obsidian
    primary: indigo
    accent: indigo

plugins:
  - search
  - roamlinks
  - obsidian-bridge
  - tags

markdown_extensions:
  - pymdownx.highlight
  - pymdownx.superfences
  - pymdownx.tasklist:
      custom_checkbox: true
  - pymdownx.tabbed
  - admonition
  - pymdownx.details
  - attr_list
  - footnotes
  - wikilinks
```

### 4. Script for Syncing/Converting

Create a script to handle any necessary conversions between Obsidian and MkDocs:

```python
#!/usr/bin/env python3
import os
import re
import shutil

# Define source and destination directories
obsidian_vault = "/path/to/obsidian/vault"
mkdocs_docs = "/path/to/mkdocs/docs"

# Copy files from Obsidian to MkDocs
for root, _, files in os.walk(obsidian_vault):
    for file in files:
        if file.endswith(".md"):
            # Get relative path
            rel_path = os.path.relpath(os.path.join(root, file), obsidian_vault)
            dest_path = os.path.join(mkdocs_docs, rel_path)
            
            # Create directories if they don't exist
            os.makedirs(os.path.dirname(dest_path), exist_ok=True)
            
            # Copy and convert the file
            with open(os.path.join(root, file), 'r') as src_file:
                content = src_file.read()
                
                # Convert Obsidian-specific syntax to MkDocs compatible syntax
                # Example: Convert [[wiki links]] to [wiki links](wiki-links.md)
                content = re.sub(r'\[\[(.*?)\]\]', r'[\1](\1.md)', content)
                
                with open(dest_path, 'w') as dest_file:
                    dest_file.write(content)
```

### 5. Docker Setup for MkDocs

For easier deployment, you can use Docker with MkDocs:

```yaml
# compose.yml
services:
  mkdocs:
    container_name: mkdocs-obsidian
    image: squidfunk/mkdocs-material
    ports:
      - "8000:8000"
    volumes:
      - .:/docs
    restart: unless-stopped
    command: serve --dev-addr=0.0.0.0:8000
```

## Workflow Tips

1. **Use Obsidian for writing and organizing**: Take advantage of Obsidian's graph view and linking capabilities
2. **Standardize your frontmatter**: Use YAML frontmatter that works in both systems
3. **Keep images in a consistent location**: Store all images in an `/assets` folder
4. **Use Git for version control**: Commit changes from both Obsidian and MkDocs
5. **Automate deployment**: Set up CI/CD to automatically build and deploy your MkDocs site when changes are pushed

## Handling Obsidian-Specific Features

- **Internal links**: Use the roamlinks plugin or convert `[[links]]` to standard Markdown links
- **Callouts/Admonitions**: Use compatible syntax that works in both systems
- **Dataview**: Either avoid using it or create a conversion script for your specific needs
- **Embeds**: Convert to standard Markdown includes or iframes as needed

By following these practices, you can maintain a single source of truth in your Obsidian vault while publishing polished documentation with MkDocs.


## Leveraging MkDocs' Native Markdown Linking with Obsidian

A key advantage of MkDocs over Hugo: MkDocs handles Markdown file references naturally, creating a workflow that aligns perfectly with Obsidian's approach to linking. This creates a more seamless integration between your knowledge management in Obsidian and your published documentation.

### The MkDocs Linking Advantage

#### Natural Markdown References

In MkDocs, you can simply link to another Markdown file using standard Markdown syntax:

```markdown
[Link to another page](another-page.md)
```

When MkDocs builds your site, it automatically converts these references to the correct HTML links:

```html
<a href="another-page.html">Link to another page</a>
```

This works regardless of your directory structure, making it compatible with how you naturally organize content in Obsidian.

#### Obsidian-to-MkDocs Workflow Benefits

1. **Write once, publish directly**: No need to modify links when moving from Obsidian to MkDocs
2. **Maintain natural organization**: Your folder structure works in both environments
3. **Preview consistency**: Links that work in Obsidian preview will generally work in MkDocs
4. **No shortcodes required**: Unlike Hugo, you don't need to learn and implement special shortcodes

### Optimizing the Obsidian-MkDocs Link Experience

#### Using Wiki-Style Links

If you prefer Obsidian's `[[wiki-style]]` links, you can use the `mkdocs-roamlinks-plugin` to maintain compatibility:

```yaml
# In mkdocs.yml
plugins:
  - roamlinks
```

This allows you to write `[[Page Name]]` in Obsidian and have it correctly converted to `[Page Name](page-name.md)` in MkDocs.

#### Relative Path Handling

MkDocs handles relative paths intelligently. For example, from a file in `docs/section1/page.md`, you can link to:

- Same directory: `[Another page](another-page.md)`
- Parent directory: `[Home page](../index.md)`
- Different section: `[Other section](../section2/page.md)`

This matches how you'd naturally navigate in Obsidian.

#### Section References

MkDocs also supports linking to specific sections within documents:

```markdown
[Link to a specific section](another-page.md#section-heading)
```

This works with Obsidian's ability to link to headings using `[[page#heading]]`.

### Implementation Tips

1. **Consistent file naming**: Use kebab-case for filenames in both Obsidian and MkDocs for consistent URL generation
   ```
   my-great-note.md → my-great-note.html
   ```

2. **Index files**: Create `index.md` files in each folder to serve as landing pages, which MkDocs will render as `folder/index.html`

3. **Path configuration**: In your `mkdocs.yml`, ensure proper path handling:
   ```yaml
   use_directory_urls: true  # This creates clean URLs without .html extension
   ```

4. **Link validation**: Use MkDocs' built-in link validation to catch broken links:
   ```bash
   mkdocs build --strict
   ```

5. **Handling attachments**: For images and other attachments, maintain a consistent path structure:
   ```markdown
   ![Image](../assets/images/my-image.png)
   ```

### Example Directory Structure

```
project/
├── docs/
│   ├── index.md                  # Home page
│   ├── assets/                   # Shared resources
│   │   └── images/               # Images referenced from multiple pages
│   ├── section1/
│   │   ├── index.md              # Section landing page
│   │   ├── topic1.md             # Can link to ../section2/topic2.md
│   │   └── assets/               # Section-specific resources
│   └── section2/
│       ├── index.md
│       └── topic2.md             # Can link back to ../section1/topic1.md
├── mkdocs.yml
└── .obsidian/
```

By leveraging MkDocs' natural handling of Markdown references, you can maintain a single source of truth between your Obsidian knowledge base and your published documentation, avoiding the shortcode complexity required by Hugo and creating a more intuitive authoring experience.

## How `mkdocs-roamlinks-plugin` Handles Wiki Links with Duplicate Filenames

The `mkdocs-roamlinks-plugin` has specific behavior when dealing with wiki links that could refer to multiple files with the same name in different folders. This is an important consideration when migrating from Obsidian, where wiki links typically don't specify paths.

### Default Behavior with Duplicate Filenames

By default, when you use a wiki link like `[[filename]]` and there are multiple Markdown files named `filename.md` in different directories, the plugin follows these rules:

1. **First match priority**: The plugin typically resolves to the first matching file it encounters during processing
2. **No disambiguation by default**: Without configuration, there's no built-in mechanism to specify which file you want when duplicates exist

This can lead to unpredictable linking behavior if you have files with the same name in different directories.

### Solutions for Handling Duplicate Filenames

#### 1. Use Qualified Wiki Links

The most reliable approach is to include path information in your wiki links:

```markdown
[[folder/filename]]
```

This explicitly tells the plugin which file you're referring to, avoiding ambiguity.

#### 2. Configure Roamlinks Plugin

You can configure the plugin in your `mkdocs.yml` to handle duplicates more predictably:

```yaml
plugins:
  - roamlinks:
      # Enable warnings about duplicate files
      warn_on_duplicate_file: true
      # Prioritize files in the same directory
      prioritize_local_files: true
```

With `prioritize_local_files` enabled, links will prefer files in the same directory as the source file.

#### 3. Use File IDs or Unique Naming

Adopt a naming convention that ensures uniqueness:

- Add a unique prefix to each file: `project1-config.md` vs `project2-config.md`
- Use a hierarchical naming scheme: `project1.config.md` vs `project2.config.md`

#### 4. Leverage Obsidian's Full Path Feature

In newer versions of Obsidian, you can configure it to show the full path when linking:

1. Go to Obsidian Settings
2. Navigate to "Files & Links"
3. Enable "Show full path of files in quick switcher"

This helps you create more precise wiki links that include path information.

### Example Scenarios

#### Scenario 1: Same Filename in Different Directories

Directory structure:
```
docs/
├── projects/
│   └── setup.md
└── tutorials/
    └── setup.md
```

If you write `[[setup]]` in a document, the plugin might link to either file unpredictably.

**Solution**: Use qualified paths in your wiki links:
```markdown
[[projects/setup]] for project setup
[[tutorials/setup]] for tutorial setup
```

#### Scenario 2: Nested Directories with Same Filename

Directory structure:
```
docs/
├── client/
│   └── api/
│       └── config.md
└── server/
    └── api/
        └── config.md
```

**Solution**: Use the full path in your wiki links:
```markdown
[[client/api/config]] for client configuration
[[server/api/config]] for server configuration
```

### Custom Script Solution

For a more automated approach, you can create a pre-processing script that adds context to ambiguous wiki links:

```python
#!/usr/bin/env python3
import os
import re
import glob

# Find all markdown files
md_files = glob.glob("docs/**/*.md", recursive=True)

# Create a dictionary of filenames and their paths
filename_map = {}
for file_path in md_files:
    filename = os.path.basename(file_path)
    if filename in filename_map:
        filename_map[filename].append(file_path)
    else:
        filename_map[filename] = [file_path]

# Find duplicates
duplicates = {k: v for k, v in filename_map.items() if len(v) > 1}

# Process each markdown file
for file_path in md_files:
    with open(file_path, 'r') as f:
        content = f.read()
    
    # Find all wiki links
    wiki_links = re.findall(r'\[\[(.*?)\]\]', content)
    
    # Check each wiki link
    modified = False
    for link in wiki_links:
        # If the link is to a file with duplicates
        if link + '.md' in duplicates:
            # Determine the correct path based on proximity or other rules
            current_dir = os.path.dirname(file_path)
            best_match = None
            
            # Simple rule: prefer files in the same directory
            for dup_path in duplicates[link + '.md']:
                if os.path.dirname(dup_path) == current_dir:
                    best_match = dup_path
                    break
            
            if best_match:
                # Get relative path
                rel_path = os.path.relpath(best_match, os.path.dirname(file_path))
                rel_path = os.path.splitext(rel_path)[0]  # Remove .md extension
                
                # Replace the wiki link with a qualified one
                content = content.replace(f'[[{link}]]', f'[[{rel_path}]]')
                modified = True
    
    # Write back if modified
    if modified:
        with open(file_path, 'w') as f:
            f.write(content)
```

### Best Practices

1. **Be consistent with naming**: Develop a naming convention that minimizes duplicate filenames
2. **Use descriptive filenames**: Instead of generic names like "config.md", use more specific names
3. **Organize by domain**: Group related files together to reduce the need for cross-directory references
4. **Test your links**: Use MkDocs' strict mode to validate links before publishing
5. **Consider a pre-processing step**: Run a script to validate and fix ambiguous links before building

By understanding how `mkdocs-roamlinks-plugin` handles duplicate filenames and implementing appropriate strategies, you can maintain a reliable linking system that works well in both Obsidian and MkDocs.