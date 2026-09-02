# Satellite Configuration Drift POC

## Pattern

1. **configure.yml** — applies yaml-declared config to Satellite (currently: locations)
2. **cleanup.yml** — pulls current state from Satellite, diffs it against the yaml,
   and reports (or removes) anything present in Satellite but not in the yaml

## Usage

Apply desired-state config:
```bash
ansible-playbook configure.yml
```

Check for drift (dry-run, default — reports only, changes nothing):
```bash
ansible-playbook cleanup.yml
```

Actually remove detected drift:
```bash
ansible-playbook cleanup.yml -e cleanup_apply_changes=true
```

