---
name: educates-authoring
description: >
  Comprehensive guide for creating and configuring workshops for the Educates interactive training platform. Includes steps for creating workshops from scratch, configuring workshop definitions and content and writing workshop instructions. Use this skill when creating Educates workshops, configuring workshop settings or writing workshop content and instructions.
---

# Educates Workshop Authoring Skill

This skill provides guidance for creating interactive workshops for the Educates training platform.

## Initial Workshop Creation

When the user asks to create a workshop, follow these steps:

### 1. Gather Workshop Details

Collect the following information from the user or infer from context:

- **Title**: A short, human-readable title for the workshop
- **Description**: A one to two sentence description of what the workshop covers
- **Name**: A machine-readable identifier (lowercase, dashes allowed, max 25 characters, should start with `lab-` prefix)

If the user provides a topic but not explicit values, propose reasonable defaults and confirm before proceeding.

### 2. Determine Workshop Location

Ask the user to choose one of:
- Use the current directory as the workshop root
- Create a new subdirectory using the workshop name

**Deriving the workshop name from the directory:**

If the user chooses to use the current working directory:
1. Check if the directory name satisfies the naming convention (lowercase, dashes allowed, max 25 characters). Note: the `lab-` prefix is recommended but not required when using the directory name.
2. If it does, use the directory name as the workshop name
3. If it does not satisfy the requirements, infer an appropriate name from the workshop topic or title, but confirm with the user before proceeding (unless they already explicitly provided a name)

### 3. Determine Workshop Requirements

The terminal application is always enabled by default. Always include it explicitly with `enabled: true` and `layout: split` for clarity.

Ask the user about additional session applications based on the workshop's technical requirements:

- **Editor**: Will users need to edit or view files in the browser?
- **Kubernetes access**: Will users run kubectl commands or interact with the Kubernetes API?
- **Kubernetes console**: Would a visual cluster view help users?
- **Docker**: Will users build or run containers?
- **Image registry**: Will users push container images?
- **Virtual cluster**: Does the workshop need cluster-admin operations?

Infer sensible defaults from the workshop topic. For example, a Kubernetes workshop likely needs editor, Kubernetes access, and console enabled.

### 4. Create Workshop Structure

Create the following directory structure in the chosen location:

```
<workshop-root>/
├── README.md                    # Workshop title and description
├── resources/
│   └── workshop.yaml            # Educates Workshop definition
└── workshop/
    └── content/                 # Workshop instruction pages (Markdown)
```

### 5. Generate workshop.yaml

Refer to [resources/workshop-yaml-reference.md](resources/workshop-yaml-reference.md) for the complete workshop.yaml structure and options.

Generate the `resources/workshop.yaml` file based on the gathered details.

**CRITICAL: Use the correct publish and workshop.files format.**

The workshop.yaml MUST include these sections with this exact structure (substituting the actual workshop name):

```yaml
spec:
  publish:
    image: "$(image_repository)/{workshop-name}-files:$(workshop_version)"
  workshop:
    files:
    - image:
        url: "$(image_repository)/{workshop-name}-files:$(workshop_version)"
      includePaths:
      - /workshop/**
      - /exercises/**
      - /README.md
```

**IMPORTANT:** Do NOT use `spec.content.files` — this is a deprecated format. Always use `spec.publish` and `spec.workshop.files` as shown above. The `$(image_repository)` and `$(workshop_version)` variables must be used exactly as shown to support local workshop publishing and deployment workflows.

**Additional configuration:**

- Set `metadata.name` to the workshop name
- Set `spec.title` and `spec.description` from gathered details
- Set `spec.duration` to estimated completion time (e.g., `15m`, `30m`, `1h`)
- Set `spec.difficulty` to one of: `beginner`, `intermediate`, `advanced`, `extreme`
- Always include terminal with `enabled: true` and `layout: split`
- Enable only the additional session applications the workshop requires
- Set `security.token.enabled: false` by default (it is enabled by default for historical reasons)
- Only set `security.token.enabled: true` if the workshop needs kubectl or uses the Kubernetes console
- Omit any applications that are not needed (do not include with `enabled: false`)

### 6. Create Workshop Instructions

Workshop instructions are placed in the `workshop/content/` directory as Markdown files rendered by Hugo.

#### Page Structure

Each page requires YAML frontmatter with at least a `title` property:

```markdown
---
title: Page Title
---

This is the introductory paragraph for the page. It appears immediately
after the frontmatter with no heading.

## First Section

Content for the first section...

## Second Section

Content for the second section...
```

**Content guidelines:**

- Use standard Markdown for page content
- Do NOT use a level 1 heading (`#`) — the `title` in frontmatter automatically generates the page header
- Begin immediately with an introductory paragraph after the frontmatter
- Use level 2 headings (`##`) and below for any additional sections

#### File Naming Convention

Use a numeric prefix for ordering pages: `nn-page-name.md`

Recommended structure:
- `00-workshop-overview.md` - Introduction and objectives
- `01-first-topic.md` - First instructional page
- `02-second-topic.md` - Continue incrementing for core content
- `99-workshop-summary.md` - Wrap-up and next steps

Reserve `00-` for the overview and `99-` for the summary. Core instructional pages start at `01-` and increment.

#### Page Bundles

For simple pages, use a single `.md` file. For pages with embedded images local to that page, use a page bundle instead:

```
workshop/content/
├── 00-workshop-overview.md
├── 01-first-topic/
│   ├── index.md
│   └── diagram.png
├── 02-second-topic.md
└── 99-workshop-summary.md
```

A page bundle is a directory containing `index.md` plus any associated assets (images, etc.).

#### Using Images in Workshop Pages

There are two approaches for including images in workshop pages, depending on whether an image is specific to a single page or shared across multiple pages.

**Page-specific images (Page Bundles)**

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

**Shared images (Static Directory)**

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

**Referencing static images:**

Because workshop instructions may be hosted at a URL with a path prefix, you must use a Hugo shortcode to correctly reference images from the static directory. Use the `param` shortcode to insert the `baseurl` data variable:

```markdown
![Logo]({{< param baseurl >}}/logo.png)
```

For images in subdirectories of `workshop/static`:

```markdown
![Shared diagram]({{< param baseurl >}}/images/shared-diagram.png)
```

**Note:** This `{{< param baseurl >}}` syntax is only required for assets in the `workshop/static` directory. Page bundle images use simple relative paths as shown above.

## Skill Version

When asked about the skill version, read the `VERSION.txt` file and report its contents to the user.

## Getting Help

For more information, visit the Educates documentation: https://docs.educates.dev/
