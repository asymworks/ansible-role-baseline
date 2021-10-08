# Ansible Role: Asymworks Linux Baseline

[![Build Status](http://drone-gitea.lan.kraussnet.com/api/badges/kraussnet/ansible-role-common/status.svg)](http://drone-gitea.lan.kraussnet.com/kraussnet/ansible-role-common)

An Ansible Role that installs common files and utilities on Asymworks servers running Debian or Alpine Linux. The default entry point performs the following tasks:

- Updates package lists and installs basic utilties such as `curl` and `vim`
- Disables SSH root login and user password login
- Installs a MOTD which displays the asset name and tag
- Applies tweaks to `sysctl`
- Installs the `restic` backup agent
- Installs and starts the `ntpd` service with a configurable NTP server
- Optionally installs and configures `rsyslog` to send system logs to the central logging server
- Optionally installs and configures `telegraf` to send system metrics to the central metrics server
- Optionally installs and configures `ufw` and opens the SSH port
- Optionally installs and configures `msmtp` to send email messages

## Requirements
None

## Role Variables

Available variables are listed below, along with default values if applicable (see `defaults/main.yml`):

```yaml
asset_name: (no default)
asset_tag: (no default)
```

Asset Name and Tag Number, must be provided as a host variable.

```yaml
etc_timezone: America/Los_Angeles
```

Timezone that will be populated in `/etc/timezone` (Alpine only).

```yaml
sysctl_config: { }
```

Dictionary of options to pass to [sysctl (8)](https://man7.org/linux/man-pages/man8/sysctl.8.html). Stored in `/etc/sysctl.d/80-ansible.conf`.

```yaml
ntp_server: time.nist.gov
```

Preferred NTP time server.

```yaml
rsyslog_enabled: true
syslog_server: logs.lan.kraussnet.com
syslog_port: 10514
```

Remote `syslog` server.

## Role Facts

Facts registered by the role are listed below, with example data:

    alpine_repo_ver: 3.13

The Alpine minor version used by apk (loaded from `/etc/apk/repositories`). Only set on Alpine Linux hosts.

## Dependencies

None

## Additional Entry Points

This role includes two additional entrypoints `pre-tasks.yaml` and `post-tasks.yaml` which are intended to be run in the `pre_tasks:` and `post_tasks:` blocks, respectively, of an Ansible play. These files will include distribution specific files which provide automatic Alpine LBU updates and other functionality.

## Example Playbook

    - hosts: all
      pre_tasks:
        include_role:
          name: kraussnet.common
          tasks_from: pre-tasks.yml
      roles:
        kraussnet.common
      post_tasks:
        include_role:
          name: kraussnet.common
          tasks_from: post-tasks.yml

## License

MIT / BSD

## Author

This role was created in 2019 by Jonathan Krauss for managing home network infrastructure.
