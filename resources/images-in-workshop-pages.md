# Images in Workshop Pages

This guide covers the two approaches for including images in workshop pages, depending on whether an image is specific to a single page or shared across multiple pages.

## Page-Specific Images (Page Bundles)

For images that are only used on a single page, use a page bundle. Convert the page from a single `.md` file to a directory containing `index.md` plus the image files:

```
workshop/content/
├── 01-first-topic/
│   ├── index.md
│   ├── diagram.png
│   └── screenshot.png
└── 02-second-topic.md
```

Within the page bundle's `index.md`, reference images using standard Markdown syntax with relative paths:

```markdown
![Architecture diagram](diagram.png)
```

## Shared Images (Static Directory)

When you have images that need to be shared across multiple pages, create a `workshop/static` directory and place the shared images there:

```
workshop/
├── content/
│   ├── 01-first-topic.md
│   └── 02-second-topic.md
└── static/
    ├── logo.png
    └── images/
        └── shared-diagram.png
```

**IMPORTANT:** The `workshop/static` directory is only needed when you have shared images or other assets that must be accessible from multiple pages. For images used on a single page, prefer page bundles instead.

### Referencing Static Images

Because workshop instructions may be hosted at a URL with a path prefix, you must use a Hugo shortcode to correctly reference images from the static directory. Use the `param` shortcode to insert the `baseurl` data variable:

```markdown
![Logo]({{< param baseurl >}}/logo.png)
```

For images in subdirectories of `workshop/static`:

```markdown
![Shared diagram]({{< param baseurl >}}/images/shared-diagram.png)
```

**Note:** This `{{< param baseurl >}}` syntax is only required for assets in the `workshop/static` directory. Page bundle images use simple relative paths as shown above.
