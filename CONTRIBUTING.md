# Contributing to `hybridops.network`

`hybridops.network` provides network automation roles tested against **real lab devices** (for example Cisco, Arista, VyOS, pfSense) and emulated environments.

Roles cover:

- Baseline configuration and hardening.
- Routing protocols (OSPF, BGP).
- VLANs and gateway redundancy (HSRP/VRRP).
- NTP, backups, and lightweight compliance checks.

Contributions are welcome as long as they keep the collection practical, testable, and vendor-aware without being vendor-locked.

---

## 1. What to contribute

Examples:

- New roles or tasks for supported platforms (Cisco, Arista, VyOS, pfSense, etc.).
- Enhancements to existing roles (additional features, better idempotency, more vendors supported).
- Improvements to test coverage and example topologies.
- Documentation updates for role parameters and expected lab setups.

Changes should keep the collection as **neutral as possible**, with vendor-specific details clearly isolated and documented.

---

## 2. Development workflow

1. **Fork and branch**

   - Fork the repository.
   - Create a topic branch:
     - `feature/<short-description>` or `fix/<short-description>`.

2. **Local setup**

   Use Python and `ansible-core` versions compatible with `requirements.txt`:

   ```bash
   python3 -m venv .venv
   . .venv/bin/activate
   pip install -r requirements.txt
   ```

3. **Make your change**

   - Target one clear behaviour per change (for example “add NTP support for platform X”).
   - Keep variables explicit rather than hiding assumptions in defaults.
   - Ensure roles can run against a small, documented lab topology.

4. **Run checks and tests**

   From the repo root:

   ```bash
   # Basic hygiene and YAML checks
   pre-commit run --all-files
   ```

   Role-local smoke tests:

   ```bash
   cd ansible_collections/hybridops/network/roles/<role_name>
   ansible-playbook -i tests/inventory.example.ini tests/smoke.yml
   ```

   The example inventory should reference either:

   - Your own lab devices, or
   - A small emulated lab (for example EVE-NG, containerlab).

   Where Molecule scenarios exist, run:

   ```bash
   cd ansible_collections/hybridops/network/roles/<role_name>
   molecule test
   ```

   Ansible linting (from the repo root):

   ```bash
   ansible-lint -c .config/linters/ansible-lint.yml
   ```

5. **Open a pull request**

   In your PR description:

   - Describe the targeted platforms and any assumptions (OS version, feature sets, lab type).
   - Summarise what you changed and why.
   - Include details of how you tested the change (for example “EVE-NG lab with IOS-XE 17.x on two edge routers”).

---

## 3. Expectations for roles

Each public role should:

- Provide `tests/smoke.yml` and `tests/inventory.example.ini`.
- Clearly document:
  - Supported vendors and versions.
  - Required connection variables (transport, credentials, ports).
  - Any dependencies on out-of-band tooling (for example backup servers).

Where possible:

- Keep logic data-driven (for example using per-platform maps) instead of heavy branching.
- Align naming, variable style, and directory layout with existing roles.
- Avoid breaking changes to existing variables without a clear reason and documentation.

---

## 4. Style, safety, and secrets

- Avoid destructive defaults; use explicit flags for changes that can disrupt traffic.
- Never commit real credentials, keys, or management IPs.
- Use variables for anything environment-specific (sites, device names, IP ranges).
- Keep comments short and focused on non-obvious decisions (for example vendor quirks or protocol edge cases).

---

By contributing here, you help build a realistic, lab-tested network automation toolkit that works both inside HybridOps.Studio and in other environments.
