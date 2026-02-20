# Salable Codex Skill

Codex skill distribution repo for `salable-monetization`.

## Install From GitHub

Use the Codex skill installer:

```bash
~/.codex/skills/.system/skill-installer/scripts/install-skill-from-github.py \
  --repo Salable/salable-codex-skill \
  --path skills/salable-monetization
```

If your default branch is not `main`, add `--ref <branch>`.

## Local Install (without GitHub)

```bash
mkdir -p ~/.codex/skills
cp -R skills/salable-monetization ~/.codex/skills/
```

After install (GitHub or local), restart Codex to pick up new skills.
