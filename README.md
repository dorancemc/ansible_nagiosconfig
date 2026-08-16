# Nagios Config

Configure Nagios monitoring objects — hosts, services, contacts, groups, commands and templates — on an existing Nagios installation.

The role works against both Nagios Core and Nagios XI. It writes object files only: it never installs packages and never writes `nagios.cfg`.

On Nagios XI the objects go to `/usr/local/nagios/etc/static`, the directory Nagios documents for manually maintained configuration. They are read by the monitoring engine but are not imported into the Core Config Manager, so they are not visible or editable in the XI web interface.

## Requirements

- Ansible 2.20 or newer
- A Nagios Core or Nagios XI installation already running on the target
- `rsync` on the target
- The object directories declared as `cfg_dir` in the live `nagios.cfg`. On Nagios XI a single `cfg_dir=/usr/local/nagios/etc/static` is enough, because `cfg_dir` is recursive. The role checks this and fails with the missing paths.
- Root privileges on target hosts

## Example Playbook

```ini
[nagios]
nagios.example.com ansible_host=192.168.243.220
```

```yaml
- hosts: nagios
  become: true
  roles:
    - role: dorancemc.ansible_nagiosconfig
```

Against Nagios XI:

```yaml
- hosts: nagios
  become: true
  roles:
    - role: dorancemc.ansible_nagiosconfig
      nagiosconfig_flavor: xi
```

Apply the role:

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

## How It Works

The role stages every object file in a temporary directory built from the live configuration, validates it with `nagios -v`, and only then synchronizes it onto the target. An invalid configuration is never applied.

Applying the change reloads the service on Nagios Core and runs `reconfigure_nagios.sh` on Nagios XI.

## License

Apache-2.0

## Author Information

Dorance Martinez, DMCi.cloud
