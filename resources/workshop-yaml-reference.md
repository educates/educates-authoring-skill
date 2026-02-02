# Workshop Definition Reference (workshop.yaml)

This document describes the structure and configuration options for the Educates `workshop.yaml` file.

## Base Template

```yaml
apiVersion: training.educates.dev/v1beta1
kind: Workshop
metadata:
  name: "{workshop-name}"
spec:
  title: "{workshop-title}"
  description: "{workshop-description}"
  duration: 30m
  difficulty: beginner
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
  session:
    namespaces:
      budget: medium
    applications:
      terminal:
        enabled: true
        layout: split
      editor:
        enabled: true
```

## Required Fields

| Field | Description |
|-------|-------------|
| `metadata.name` | Machine-readable identifier (lowercase, dashes allowed, max 25 chars, recommend `lab-` prefix) |
| `spec.title` | Human-readable workshop title |
| `spec.description` | One to two sentence description |
| `spec.duration` | Estimated completion time (e.g., `15m`, `30m`, `1h`, `2h`) |
| `spec.difficulty` | Target audience: `beginner`, `intermediate`, `advanced`, or `extreme` |

## Workshop Files Configuration (CRITICAL)

**ALWAYS use this exact structure for workshop file publishing:**

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

Replace `{workshop-name}` with the actual workshop name from `metadata.name`.

**IMPORTANT:**
- The `$(image_repository)` and `$(workshop_version)` are variables that MUST be used exactly as shown
- These variables are required for local workshop publishing and deployment workflows
- **NEVER use `spec.content.files`** — this is a deprecated format and must not be used

## Session Applications

Ask the user which tools the workshop requires. Include only the applications that are needed.

### Terminal

The terminal is enabled by default and does not need to be explicitly enabled. However, it is recommended to include the configuration for clarity.

```yaml
applications:
  terminal:
    enabled: true
    layout: split
```

- **Default behavior**: Always enabled, even if omitted
- **Recommendation**: Always include `enabled: true` explicitly for clarity
- **Layout options**:
  - `default` - Single terminal
  - `split` - Two terminals stacked above each other in ratio 60/40 (recommended default)
  - `split/2` - Three terminals stacked above each other in ratio 50/25/25
  - `lower` - Single terminal placed below dashboard tabs rather than as a tab, ratio 70/30
  - `none` - No terminal displayed, but can still be created from drop down menu

### Editor

For viewing or editing files in the browser:

```yaml
applications:
  editor:
    enabled: true
```

- **When to enable**: Workshops involving code editing, file creation, or viewing source files

### Kubernetes Console

For Kubernetes cluster visualization:

```yaml
applications:
  console:
    enabled: true
```

- **When to enable**: Workshops teaching Kubernetes concepts where users benefit from a visual cluster view

### Docker Daemon

For building and running containers:

```yaml
applications:
  docker:
    enabled: true
```

- **When to enable**: Workshops involving container building, Docker commands, or container runtime exercises

### Image Registry

For pushing/pulling container images:

```yaml
applications:
  registry:
    enabled: true
```

- **When to enable**: Workshops where users need to push images to a registry (often paired with Docker)

### Virtual Cluster (vcluster)

For isolated Kubernetes environments:

```yaml
applications:
  vcluster:
    enabled: true
```

- **When to enable**: Workshops requiring cluster-admin operations or isolated Kubernetes environments

## Kubernetes Access

Kubernetes access via security token is enabled by default for historical reasons. Explicitly disable it unless the workshop requires it.

**When workshop does NOT need Kubernetes access:**

```yaml
session:
  namespaces:
    budget: medium
    security:
      token:
        enabled: false
```

**When workshop DOES need Kubernetes access:**

```yaml
session:
  namespaces:
    budget: medium
    security:
      token:
        enabled: true
```

- **Default behavior**: Security token is enabled by default (historical)
- **Recommendation**: Explicitly set `enabled: false` unless the workshop uses `kubectl`, interacts with the Kubernetes API, or uses the Kubernetes console

## Decision Flowchart

When generating a workshop.yaml, determine:

1. **What is the workshop topic?** → Sets title and description
2. **Terminal configuration** → Always include with `enabled: true` and `layout: split` (terminal is enabled by default, but include explicitly for clarity)
3. **Does it involve editing code?** → Enable editor
4. **Does it need Kubernetes access?** → If yes, keep security token enabled; if no, explicitly disable with `enabled: false`
5. **Does it teach Kubernetes?** → Enable console (and keep security token enabled)
6. **Does it involve containers?** → Enable docker and possibly registry
7. **Does it need cluster-admin access?** → Enable vcluster
