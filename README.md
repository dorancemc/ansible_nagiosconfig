# Nagios Config

Configure Nagios monitoring objects — hosts, services, contacts, groups, commands, templates — on an existing Nagios installation.

Works against both Nagios Core and Nagios XI. Writes object files only: never installs packages, never writes `nagios.cfg`.

On Nagios XI, objects go to `/usr/local/nagios/etc/static` (read by the engine, not imported into the Core Config Manager, so not visible in the XI web UI).

## Requirements

- Ansible 2.20 or newer
- A running Nagios Core or Nagios XI installation on the target
- `rsync` on the target
- Object directories declared as `cfg_dir` in the live `nagios.cfg` (on XI, `cfg_dir=/usr/local/nagios/etc/static` is enough since `cfg_dir` is recursive). The role checks this and fails with the missing paths.
- Root privileges on target hosts

## Example Playbook

```yaml
- hosts: nagios
  become: true
  roles:
    - role: dorancemc.ansible_nagiosconfig
```

Against Nagios XI, set `nagiosconfig_flavor: xi`.

```bash
ansible-playbook --limit nagios playbook.yml --tags nagiosconfig
```

## Host Definitions

Each monitored host is one YAML file under `nagiosconfig_hosts_path`:

```yaml
---
_host:
  host_name: localhost
  address: 127.0.0.1
  hostgroups: local
_services:
  ping: check_ping!100.0,20%!500.0,60%
  current_load:
    servicegroups: localresources
    check_command: check_local_load!5.0,4.0,3.0!10.0,6.0,4.0
```

A service value is either a check command string or a mapping of service directives.

## Shared Objects and Tenants

Contacts, contactgroups, hostgroups, servicegroups, commands and templates load one file at a time so dicts merge instead of overwriting:

- `nagiosconfig_base_path` — objects shared by every tenant. Empty by default (lookup disabled).
- `_tenant.yaml` — one per directory under `nagiosconfig_hosts_path`, holding that tenant's objects.

### Deploying Some Tenants Only

`nagiosconfig_tenants` limits a run's host/service definitions to specific tenants (directory names under `nagiosconfig_hosts_path`, as a list or comma-separated string) to speed up large deployments:

```bash
ansible-playbook --limit nagios playbook.yml --tags nagiosconfig -e nagiosconfig_tenants=cloud-status
```

When set: an unknown tenant name fails early (listing valid ones); `nagiosconfig_clean_assets` is ignored to preserve other tenants' hosts; shared objects (commands, templates, contacts, groups) still render from every tenant; and `nagios -v` validates the scoped tenants against the others' live definitions. Host files outside any tenant directory are skipped.

## Default Ordering

Object defaults are not sorted alphabetically on purpose: they reproduce the key order of the role this was split from, so re-applying rewrites nothing byte-for-byte. Sorting them rewrites every object file on every server, so treat it as a deliberate change, not cleanup.

## Objects Nagios XI Already Owns

Nagios XI ships its own object library. Some names overlap this role's defaults, and a duplicate template `name` is fatal (`nagios -v` fails). With `nagiosconfig_flavor: xi`, the role skips the names in `nagiosconfig_reserved_objects` (three generic templates, four timeperiods, the `nagiosadmin` contact, the `admins` contactgroup, twenty commands). On `core`, nothing is filtered.

Consequences on XI:
- Objects referencing skipped names resolve to XI's definitions, which differ.
- Custom flags on stock commands (e.g. `check_ping -4`) are lost; give the command your own name if that matters.

Override `nagiosconfig_reserved_objects` in inventory if your XI ships different objects.

## Extra Plugins

`nagiosconfig_extra_plugins` downloads plugins the commands reference but the installation does not ship, into the target's `plugins_path`. It is a dict keyed by destination file name, each entry with a `url` and optional `checksum`.

Nagios runs these on every check, so whoever controls the URL runs code on the monitoring server. Pin a commit in the URL and set the checksum.

## How It Works

The role stages every object file in a temporary directory, validates it with `nagios -v`, and only then syncs it onto the target. An invalid configuration is never applied.

Only object directories are staged, under an `objects/` subdirectory; the copy of `nagios.cfg` sits above it so it is never read as an object file. Only its `cfg_dir` entries pointing at the object path are rewritten, so validation runs against the host's real configuration.

The temporary directory is `nagioscfg-tmp-<random>` under `nagiosconfig_tempdir_base` (`/tmp`). Each run sweeps leftover `nagioscfg-tmp-*` directories first (assumes one run at a time per target). Set `nagiosconfig_tempdir_cleanup: false` to keep a failed tree for inspection.

Applying the change reloads the service on Core and runs `reconfigure_nagios.sh` on XI.

## License

Apache-2.0

## Author Information

Dorance Martinez, DMCi.cloud
