# my-skills

Personal Claude skills library.

## Structure

```
my-skills/
├── <skill-name>/
│   ├── SKILL.md
│   └── references/   (optional)
└── ...
```

## Install a skill

1. Download the `.skill` file from [Releases](../../releases)
2. Claude.ai → Settings → Skills → Install from file

## Build a skill locally

Requires the [skill-creator scripts](https://github.com/anthropics/claude-skills/tree/main/skill-creator/scripts).

```bash
python -m scripts.package_skill <skill-name> ./dist
```

## Skills

| Skill | Description |
|---|---|
| [guitar-riff-composer](./guitar-riff-composer/) | Composes guitar riffs with tablature, theory context, and tone suggestions |
