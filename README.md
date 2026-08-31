# Satellite Configuration Drift POC

A small Ansible-based proof of concept for detecting and cleaning up Satellite
configuration drift — i.e. objects that exist in Satellite (created manually
via the web UI, or left over from a previous config) but aren't declared in
the source-of-truth yaml.

## Pattern

1. **configure.yml** — applies yaml-declared config to Satellite (currently: locations)
2. **cleanup.yml** — pulls current state from Satellite, diffs it against the yaml,
   and reports (or removes) anything present in Satellite but not in the yaml

## Structure

```
configure.yml                          # applies yaml config to Satellite
cleanup.yml                            # detects/removes drift
vars/
  lab_connection.yml.example           # copy to lab_connection.yml, fill in real values
  locations.yml                        # desired-state yaml for locations
  mappings/
    locations.yml                      # optional yaml-name -> Satellite-name overrides
playbooks/tasks/
  configure/configure_locations.yml
  get/get_locations.yml
  cleanup/cleanup_locations.yml
```

## Setup

```bash
cp vars/lab_connection.yml.example vars/lab_connection.yml
# edit vars/lab_connection.yml with your real Satellite URL/credentials
ansible-galaxy collection install redhat.satellite   # or: dnf install ansible-collection-redhat-satellite
```

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

## Name mapping

If a name in the yaml doesn't match the actual name in Satellite (e.g. renamed
during a past migration), add an entry to `vars/mappings/locations.yml` so it
isn't wrongly flagged as an orphan:

```yaml
location_name_mapping:
  "yaml-name": "actual-satellite-name"
```

Parent-path mappings must use the full path as written in yaml (e.g.
`"Australia/ACT"`), not per-segment translation.

## Status

- [x] Locations: configure / get / cleanup / dry-run / apply / name mapping
- [ ] Additional resource types (settings, host groups, etc.)
- [ ] Mapping-file-based orphan listing
- [ ] Attribute-level drift detection (beyond existence)

## Safety notes

- `cleanup.yml` defaults to dry-run (`cleanup_apply_changes: false`) — nothing
  is deleted unless explicitly overridden
- `vars/lab_connection.yml` is gitignored — never commit real credentials
- `protected_locations` in `cleanup.yml` excludes Satellite's built-in defaults
  (e.g. "Default Location") from ever being flagged as orphaned
