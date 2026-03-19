# Contributing to `hybridops.network`

`hybridops.network` provides network automation roles validated in lab and emulated environments. The scope is practical, testable network automation with vendor-aware behaviour and clear separation of vendor-specific logic.

Design and release context is maintained in the HybridOps.Tech documentation site.

## Contribution scope

Appropriate changes include:

- New roles and tasks for supported platforms (for example Cisco, Arista, VyOS, pfSense).
- Enhancements to existing roles (features, idempotency, expanded platform support).
- Test improvements (role-local smoke tests, example topologies, Molecule where applicable).
- Documentation updates for variables, supported targets, and lab assumptions.

Prefer neutral role design with vendor-specific details isolated and documented.

## Development workflow

### Local setup

Use versions compatible with `requirements.txt`:

```bash
python3 -m venv .venv
. .venv/bin/activate
pip install -r requirements.txt
```

### Change guidelines

- Keep changes focused on a single behaviour per pull request.
- Keep variables explicit; avoid hidden assumptions in defaults.
- Avoid disruptive defaults; use explicit flags for changes that can affect traffic.

### Tests

Role-local smoke tests:

```bash
cd roles/<role_name>
ansible-playbook -i tests/inventory.example.ini tests/smoke.yml
```

Molecule (where defined):

```bash
cd roles/<role_name>
molecule test
```

Platform integration (via the harness):

```bash
# In galaxy-collections-harness
make workspace.clone
make collections.sync
make venv.refresh
make test ROLE=<role_name>
```

### Pull requests

Include:

- Target platform(s) and versions where applicable.
- Summary of changes and expected impact.
- Test evidence (smoke and/or Molecule and/or platform integration), including lab type (physical or emulated).

## Role expectations

Each role should provide:

- `roles/<role_name>/tests/smoke.yml`
- `roles/<role_name>/tests/inventory.example.ini`

Roles should document supported platforms, required connection variables, and any out-of-band dependencies (for example backup servers).

## Security and safety

- Do not commit credentials, keys, or management IPs.
- Consume sensitive values via variables, Vault, or environment lookups.
- Avoid logging sensitive values in task output.
