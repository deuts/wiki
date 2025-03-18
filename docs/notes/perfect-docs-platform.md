---
title: "Perfect Docs Platform"
layout: default
nav_order: 40
parent: Notes 
---

# The search for the perfect documentation platform

## Docmost
- Experienced delays while writing
	- Delays when writing titles
	- Delays to reflect the updated page when navigating back to that page
- Can add members, assign them to groups, and limit access to spaces within the whole instance
- Cannot nest bullet points within tasks and vice versa
  - You can only nest tasks within tasks, and bullet points within bullet points but not combine

## Otterwiki
- Has permission settings but not as granular as Docmost
- Has git integration
- Changelog/history included
- With customizable menus
- Python under the hood
- Can paste images from clipboard directly into the editor and saves in the file system
- What I hate:
  - File naming convention:
  	- Should have the same filename.md and filename directory for subpages
  	- Auto-capitalize names
  		- Even if you have all caps - for example FAQ will be converted to Faq
  	- URL includes `%20` for spaces

## Bookstack
- PHP and MariaDB under the hood
- Limited customization options
  - There's no template gallery to speak of
  - If we're talking about PHP and MySQL/MariaDB anyway, why not just use WordPress instead?
- 3 levels of page hierarchy: `Book >> Chapter >> Page` only
- With robust API
- With version (revision) history
- Exported markdown is not strict markdown. It could sometimes contain HTML tags
- I just don't like the `Book` method of organizing my documents/pages. I really need a navigation sidebar, and that navigation is not limited to 3 levels deep only.
- I think I'm finally going for this one:
  - Advanced access control for books, chapters and pages
  - I need a way to let my users edit the documents
    - This is an ever evolving set of documents that I can't just maintain all by myself
  - They are books after all, so expect a somewhat flat format

## Hugo
- More geared towards blogs rather than documentation
- Renders HTML
- Finally leaning for this because:
  - I'm quite familiar already with the templating system
  - Just 1 executable file for the program, no more plugins to install
  - I may be integrating some dynamic data into my documentation site using python in the future
- Have been playing with the Relearn theme, but I feel like it just won't cut it.

## Outline
- Just couldn't make it work seamlessly
  - Experienced several error while navigating the page
  - Export didn't work
  - Just so unreliable in my opinion

## SiYuan
- Too cluttered for my taste
- Looks to aim to be a Notion alternative, with all its database feature
- The documentation leaves a lot to be desired. They appear to have been written in Chinese and used Google Translate to translate into English

## MkDocs
- Uses python out of the box
- With better [integration with Obsidian]({% link docs/notes/obsidian-mkdocs.md %})
  - Useful for interpage linking of markdown files
  	- Carries over to the rendered HTML page
- Material for MkDocs is so much for what I need
- Renders HTML which can easily be shared in the future
- Can have pdf export if necessary
- Lot of plugins, lot of extensability, but unlike Hugo, the latter being just 1 executable file.
  - The use of [Awesome Nav for MkDocs](https://lukasgeiter.github.io/mkdocs-awesome-nav/) plugin could streamline the navigation creation process for MkDocs, particularly for Material for MkDocs
- Just looks good actually. The obsidian integration is 1 factor I'm contemplating using this for `mpidocs`
- What I hate about MkDocs:
  - It can't parse 2-space indentation for nested (sub) lists. You need to make it 4 spaces to properly render the sub-list
    - In this case, we'll use tabs instead of 2 spaces for nested lists, so it'll be compatible for both Obsidian and MkDocs, and any other system we might use the markdown files.

## Jekyll
- I really liked the simplicity of `Just the Docs` theme
- Might be useful for my other wiki/documentation project, at least I'm learning something new


## Mintlify
- Hosted documentation platform built on Next.js and leverages React for its front-end
- It is not self-hosted, operates as a SaaS (Software-as-a-Service), meaning you need to use their infrastructure rather than hosting it on your own server.
- Uses MDX (Markdown + JSX) for documentation. This allows you to write standard Markdown while embedding React components directly within your docs.
- Even on the hobby plan (free version), you can:
  - Have custom domain
  - Github integration
    - You can do edits or collaborate on Github and changes are pushed to your site
    - It can serve as backup as well if you primarily make edits in the Mintlify platform
- See my working documentation site at: https://deutsnet.mintlify.app