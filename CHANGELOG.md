# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

- `nagiosconfig_tenants`: limits the host and service definitions of a run to some tenants, the directory names under `nagiosconfig_hosts_path`. A name that matches no directory fails the run before anything reaches the target. While it is set `nagiosconfig_clean_assets` is ignored, so the other tenants' live objects are always copied into staging and never deleted; shared objects are still rendered from every tenant.

### Changed

- `_tenant.yaml` files are no longer walked by the host definition loop, which only skipped them.

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
- `sso` as a control key on a contact: excluded from the rendered `define contact` so external roles can flag which contacts are allowed to authenticate without Nagios seeing an unknown directive.
- `_TENANT` on every host definition, taken from the name of the folder the host file lives in under `nagiosconfig_hosts_path`. A host that declares `_TENANT` itself keeps its own value, and a host file sitting directly in the hosts root gets nothing. The macro is `$_HOSTTENANT$`, which lets the perfdata template of `nagios.cfg` carry the tenant into whatever consumes the performance data.
