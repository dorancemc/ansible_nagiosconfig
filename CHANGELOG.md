# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2026-08-20

First release. Extracted from `ansible_nagioscore`, which from 4.0.0 installs Nagios but no longer manages monitoring objects.

### Added

- `nagiosconfig_flavor: core|xi` — one role for Nagios Core and Nagios XI. Paths and identities come from `vars/<flavor>[-<os>].yaml`.
- Monitoring objects — hosts, services, contacts, groups, commands, templates and timeperiods — as dicts keyed by object name, merged over the role defaults.
- Tenant loader (`tasks/objects_load.yml`): reads `nagiosconfig_base_path` and `_tenant.yaml` one file at a time so the dicts merge instead of overwriting each other.
- Staging validator (`tasks/objects.yml`): builds the objects in a temporary directory, runs `nagios -v`, and rsyncs to the target only on exit code 0. Staging runs with `check_mode: false`, so `--check` validates for real.
- Cleanup of the staging trees left behind by an interrupted run: every run removes the `nagioscfg-tmp-*` directories under `nagiosconfig_tempdir_base` before creating its own. Assumes one run at a time per target.
- `nagiosconfig_reserved_objects`: the commands, templates, contacts and hosts Nagios XI already ships, skipped on `xi` so they are not defined twice.
- `nagiosconfig_extra_plugins` (`tasks/plugins.yml`): downloads extra check scripts to the target's plugins path, with checksum.
- `nagiosconfig_mode: once|always|never` and `nagiosconfig_clean_assets` to decide which objects the role owns and which it deletes.
