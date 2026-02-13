# Educates Authoring Skill

A Claude Code skill for creating and configuring workshops for the [Educates](https://educates.dev/) interactive training platform.

## Features

- Create new workshop projects with proper directory structure
- Generate `workshop.yaml` configuration with appropriate session applications
- Author workshop instruction pages following Educates conventions

## Installation

### Installation using npx skills

You can install this skill directly from the GitHub repository using the `npx skills` command:

```bash
npx skills install https://github.com/educates/educates-authoring-skill
```

> **Note:** This skill is not currently available on [skills.sh](https://skills.sh), but its listing there will be considered in the future.

### Installation from .skill file

#### From GitHub Release

Pre-packaged `.skill` files are available from the [GitHub releases](https://github.com/educates/educates-authoring-skill/releases) page.

To download the latest release:

```bash
curl -fLO https://github.com/educates/educates-authoring-skill/releases/latest/download/educates-authoring.skill
```

To download a specific version (replace `2.0` with the desired version tag):

```bash
curl -fLO https://github.com/educates/educates-authoring-skill/releases/download/2.0/educates-authoring.skill
```

#### From GitHub Source

1. Clone this repository:

   ```bash
   git clone https://github.com/educates/educates-authoring-skill.git
   cd educates-authoring-skill
   ```

2. Create an archive of the skill:

   ```bash
   zip -r educates-authoring.skill . -x ".git/*" -x ".github/*"
   ```

#### Importing into Claude

Once you have the `.skill` file, either downloaded from a GitHub release or built from source, install it using the Claude Code CLI:

```bash
claude skill install educates-authoring.skill
```

Or in the Claude Code desktop app, go to **Settings > Skills** and use "Install from file" to select the `.skill` file.

### Usage

Once installed, if using Claude invoke the skill with:

```
/educates-authoring
```

Or simply ask Claude to help create an Educates workshop and it will use the skill automatically based on context.

If you are using this skill with other AI agents that support the skills format, the method for invoking the skill will depend on the specific tool. Please check the documentation or help resources for the AI agent you are using to determine how to invoke skills.

## Compatibility

This skill is aligned with **Educates version 3.6.0**. Workshops generated using this skill may not work with older versions of Educates.

## Other AI Agents

Although this skill is being developed and tested with Claude, the skills format is standardised to a degree, so it may also work with other AI agents that support the same format. However, since Claude's built-in knowledge can vary compared to other agents, your success with other agents may differ. An agent with less prior knowledge of the Educates platform, or with different strengths in code generation and YAML authoring, may produce results of varying quality.

## Feedback

This skill is continually being improved, and your feedback helps make it better. If you notice areas where the generated workshops could be improved, or where the skill seems to lack knowledge about how Educates environments work and what they provide, please open an issue on the [GitHub issue tracker](https://github.com/educates/educates-authoring-skill/issues).

Examples of useful feedback include:

- Incorrect or suboptimal workshop configuration being generated
- Missing awareness of Educates features or session applications
- Workshop instructions that don't follow Educates best practices
- Suggestions for additional capabilities the skill should support

There's no need to submit a pull request — just describe the issue and we'll use AI itself to determine the best approach for incorporating any corrections or additional knowledge into the skill. Your feedback helps improve the skill's understanding of Educates for everyone.

## Documentation

For more information about Educates, visit the official documentation at https://docs.educates.dev/
