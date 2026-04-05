# CLI And Registry

Use this reference only when the task involves discovering, installing, publishing, or updating AO skills.

## Search skills

```bash
ao skill search --query "review"
ao skill search --source user
ao skill search --registry community
```

## Install a skill

```bash
ao skill install --name code-review --registry community
ao skill install --name code-review --version "^1.0" --allow-prerelease
```

## List skills

```bash
ao skill list
ao skill list --source project
```

## Show or update a skill

```bash
ao skill show --name code-review
ao skill update --name code-review --version "^2.0"
```

## Publish a skill

```bash
ao skill publish --name code-review --version "1.0.0" --source my-org --registry community
```

## Registry commands

```bash
ao skill registry add --id community --url https://github.com/ao-skills/registry
ao skill registry list
ao skill registry remove --id community
```
