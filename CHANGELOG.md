# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2026-08-20

### Added

- `nagiosconfig_flavor: core|xi` — single role manages monitoring objects for both Nagios Core and Nagios XI.
- Tenant loader (`tasks/objects_load.yml`): discovers `*.yaml` files under `nagiosconfig_base_path` and `_tenant.yaml` files under `nagiosconfig_hosts_path` using per-file `include_vars` with `hash_behaviour: merge`, avoiding the silent last-file-wins collision of `include_vars: dir:`.
- Staging validator (`tasks/objects.yml`): builds config in a tempdir, runs `nagios -v`, and rsyncs to the server only on success. All staging tasks run with `check_mode: false` so `--check` works without a live tempdir.
- `nagiosconfig_reserved_objects` (`defaults/main/objects/reserved.yaml`): list of commands, templates, contacts, and hosts that Nagios XI ships out of the box. All template loops skip reserved names when `nagiosconfig_flavor == xi`, preventing duplicate-definition errors on XI.
- Extra plugins support (`tasks/plugins.yml`, `nagiosconfig_extra_plugins`): downloads additional check scripts to `nagiosconfig_target.plugins_path`, resolved per flavor.
- `nagiosconfig_clean_assets` variable: when `true`, purges object files no longer present in inventory before applying.
- `bootstrap.cfg` awareness: `tasks/objects.yml` references `cfg_file` entries from the live nagios.cfg without copying them to the tempdir, so the pre-flight check sees the same bootstrap objects the engine does.
- XI `static` directory layout: `objects_path` for XI is `/usr/local/nagios/etc/static`; the staging tree is built one level below (`<tmp>/objects/`) so `nagios.cfg` itself is never inside a `cfg_dir`.

### Changed

- Extracted from `ansible_nagioscore` (all object-management tasks, defaults, and templates). Breaking split — `ansible_nagioscore` 4.0.0 is required.
- `host_definition.yml` reads each host YAML with `vars: lookup('ansible.builtin.file') | from_yaml` instead of `include_vars` + `set_fact`, fixing the `hash_behaviour = merge` bug where each host in a loop accumulated services from all previous hosts.
- Default ordering of commands, templates, contacts, timeperiods, and hostgroups preserved from the legacy `ansible_nagioscore` role to produce zero cosmetic diffs on migrated hosts.

### Breaking changes

- Requires `nagiosconfig_flavor` to be set (`core` or `xi`).
- Variable prefix changed from `nagios_*` to `nagiosconfig_*` throughout.
- `nagiosconfig_base_path` and `nagiosconfig_hosts_path` must be set; there is no default path.
