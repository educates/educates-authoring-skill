# Editor File Clickable Actions

Actions for opening, creating, editing, and managing files through the embedded editor. These require the editor to be enabled in the workshop session.

## editor:open-file

Opens a file in the editor. Optionally positions the cursor on a specific line.

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `file` | string | (required) | File path. Use `~/` or `$HOME/` for home directory |
| `line` | integer | — | Line number to position cursor on (1-indexed) |

Note: When the embedded editor is enabled in a workshop, prefer using this action to display file contents to users rather than using terminal commands like `cat`.

**Example:**

````markdown
```editor:open-file
file: ~/exercises/deployment.yaml
line: 8
```
````

## editor:create-file

Creates a new file with the specified content, or overwrites an existing file entirely. The containing directory must already exist.

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `file` | string | (required) | File path |
| `text` | string | (required) | Content for the file |

**Example:**

````markdown
```editor:create-file
file: ~/exercises/config.yaml
text: |
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: example
  data:
    key: value
```
````

## editor:create-directory

Creates a directory. Use this before `editor:create-file` or `editor:append-lines-to-file` if the target directory does not exist.

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `directory` | string | (required) | Directory path to create |

**Example:**

````markdown
```editor:create-directory
directory: ~/exercises/configs
```
````

## editor:append-lines-to-file

Appends text to the end of a file. If the file does not exist, it will be created.

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `file` | string | (required) | File path |
| `text` | string | (required) | Text to append |

Note: If it is known the file does not already exist, use `editor:create-file` instead for clearer semantic intent.

**Example:**

````markdown
```editor:append-lines-to-file
file: ~/exercises/notes.txt
text: |
  Additional notes added at the end.
```
````

## editor:insert-lines-before-line

Inserts text before a specified line number.

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `file` | string | (required) | File path |
| `line` | integer | (required) | Line number to insert before (1-indexed) |
| `text` | string | (required) | Text to insert |

**Example:**

````markdown
```editor:insert-lines-before-line
file: ~/exercises/app.py
line: 5
text: |
  import logging
  logger = logging.getLogger(__name__)
```
````

## editor:insert-lines-after-line

Inserts text after a specified line number.

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `file` | string | (required) | File path |
| `line` | integer | (required) | Line number to insert after (1-indexed) |
| `text` | string | (required) | Text to insert |

**Example:**

````markdown
```editor:insert-lines-after-line
file: ~/exercises/app.py
line: 3
text: |
  import os
```
````

## editor:append-lines-after-match

Inserts text after the first line containing a matching string.

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `file` | string | (required) | File path |
| `match` | string | (required) | Text to search for in lines |
| `text` | string | (required) | Text to insert after the matching line |

**Example:**

````markdown
```editor:append-lines-after-match
file: ~/exercises/requirements.txt
match: flask
text: |
  flask-cors==4.0.0
```
````

## editor:select-matching-text

Selects (highlights) text in a file based on exact match or regular expression. Useful as a visual aid or as a precursor to `editor:replace-text-selection`.

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `file` | string | (required) | File path |
| `text` | string | (required) | Text or regex pattern to match |
| `isRegex` | boolean | `false` | Treat `text` as a regular expression |
| `group` | integer | — | Regex subgroup to select (when using regex) |
| `before` | integer | — | Extra lines to highlight before match. `0` = full line, `-1` = all lines before |
| `after` | integer | — | Extra lines to highlight after match. `0` = full line, `-1` = all lines after |
| `start` | integer | — | Start line for search range (1-indexed). Negative = offset from end |
| `stop` | integer | — | End line for search range (exclusive). Negative = offset from end |

**Example — exact match:**

````markdown
```editor:select-matching-text
file: ~/exercises/deployment.yaml
text: "image: nginx:1.19"
```
````

**Example — regex with subgroup:**

````markdown
```editor:select-matching-text
file: ~/exercises/deployment.yaml
text: "image: (.*)"
isRegex: true
group: 1
```
````

## editor:select-lines-in-range

Selects a range of lines by line number. The selected text can then be replaced using `editor:replace-text-selection`.

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `file` | string | (required) | File path |
| `start` | integer | (required) | First line to select (1-indexed, inclusive) |
| `stop` | integer | — | Last line to select (inclusive). If omitted, only `start` line is selected |

**Example:**

````markdown
```editor:select-lines-in-range
file: ~/exercises/app.py
start: 5
stop: 10
```
````

## editor:replace-text-selection

Replaces the currently selected text in a file (selected via `editor:select-matching-text` or `editor:select-lines-in-range`).

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `file` | string | (required) | File path |
| `text` | string | (required) | Replacement text |

**Example:**

````markdown
```editor:replace-text-selection
file: ~/exercises/deployment.yaml
text: nginx:latest
```
````

## editor:replace-matching-text

Find and replace text in a single step, without needing to first select and then replace separately.

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `file` | string | (required) | File path |
| `match` | string | (required) | Text or regex to find |
| `replacement` | string | (required) | Replacement text |
| `isRegex` | boolean | `false` | Treat `match` as regex |
| `group` | integer | — | Regex subgroup to replace |
| `start` | integer | — | Start line for search range |
| `stop` | integer | — | End line for search range (exclusive) |
| `count` | integer | `1` | Number of matches to replace. `-1` = all matches |

**Example — simple replacement:**

````markdown
```editor:replace-matching-text
file: ~/exercises/deployment.yaml
match: "nginx:1.19"
replacement: "nginx:1.25"
```
````

**Example — regex replace all occurrences:**

````markdown
```editor:replace-matching-text
file: ~/exercises/deployment.yaml
match: "replicas: [0-9]+"
replacement: "replicas: 3"
isRegex: true
count: -1
```
````

## editor:replace-lines-in-range

Replaces a range of lines with new content.

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `file` | string | (required) | File path |
| `start` | integer | (required) | First line to replace (1-indexed, inclusive) |
| `stop` | integer | (required) | Last line to replace (inclusive) |
| `text` | string | (required) | Replacement content |

**Example:**

````markdown
```editor:replace-lines-in-range
file: ~/exercises/app.py
start: 5
stop: 10
text: |
  def new_function():
      return "replaced"
```
````

## editor:delete-lines-in-range

Deletes a range of lines from a file.

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `file` | string | (required) | File path |
| `start` | integer | (required) | First line to delete (1-indexed, inclusive) |
| `stop` | integer | — | Last line to delete (inclusive). If omitted, only `start` line is deleted |

**Example:**

````markdown
```editor:delete-lines-in-range
file: ~/exercises/app.py
start: 5
stop: 10
```
````

## editor:delete-matching-lines

Deletes lines matching a string or regex, optionally including surrounding lines.

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `file` | string | (required) | File path |
| `match` | string | (required) | Text or regex to match |
| `isRegex` | boolean | `false` | Treat `match` as regex |
| `before` | integer | `0` | Lines before match to also delete. `-1` = all lines before |
| `after` | integer | `0` | Lines after match to also delete. `-1` = all lines after |

**Example:**

````markdown
```editor:delete-matching-lines
file: ~/exercises/app.py
match: "# TODO: remove this"
after: 1
```
````

## editor:copy-file

Copies a file to a new location.

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `src` | string | (required) | Source file path |
| `dest` | string | (required) | Destination file path |
| `open` | boolean | `true` | Open the destination file in editor after copying |

**Example:**

````markdown
```editor:copy-file
src: ~/exercises/template.yaml
dest: ~/exercises/deployment.yaml
```
````

## editor:rename-file

Renames or moves a file.

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `src` | string | (required) | Current file path |
| `dest` | string | (required) | New file path |
| `open` | boolean | `true` | Open file in editor after renaming |

**Example:**

````markdown
```editor:rename-file
src: ~/exercises/old-name.yaml
dest: ~/exercises/new-name.yaml
```
````

## editor:close-file

Closes a file tab in the editor. No-op if the file is not open.

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `file` | string | (required) | File path |

**Example:**

````markdown
```editor:close-file
file: ~/exercises/deployment.yaml
```
````

## editor:delete-file

Deletes a file from the filesystem. If open in the editor, it will be closed first.

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `file` | string | (required) | File path |

**Example:**

````markdown
```editor:delete-file
file: ~/exercises/old-config.yaml
```
````
