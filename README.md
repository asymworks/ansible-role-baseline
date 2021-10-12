# Ansible Role: Asymworks Linux Baseline

An Ansible Role that installs common files and utilities on Asymworks servers running Debian or Alpine Linux. The default entry point performs the following tasks:

- Updates package lists and installs basic utilties such as `curl` and `vim`
- Disables SSH root login and user password login
- Installs a MOTD which displays the asset name and tag
- Applies tweaks to `sysctl`
- Installs the `restic` backup agent
- Installs and starts the `ntpd` service with a configurable NTP server
- Installs and optionally enables `rsyslog` to send system logs to the central logging server
- Installs and optionally enables `telegraf` to send system metrics to the central metrics server
- Installs and optionally enables `ufw` and opens the SSH port
- Installs and optionally enables `msmtp` to send email messages
- Installs and optionally enables NFSv4 file sharing with ID mapping (`sec=sys` mode)

## Requirements

None

## Role Variables

Available variables are listed below, along with default values if applicable (see `defaults/main.yml`):

```yaml
asset_name: (no default)
asset_tag: (no default)
```

Asset Name and Tag Number, should be provided as a host variable. This is populated into the `/etc/motd` file.

```yaml
timezone: America/Los_Angeles
```

Timezone that will be populated in `/etc/timezone` (Alpine Linux only).

```yaml
sysctl_config: { }
```

Dictionary of options to pass to [sysctl (8)](https://man7.org/linux/man-pages/man8/sysctl.8.html). Stored in `/etc/sysctl.d/80-ansible.conf`.

```yaml
ntp_server: time.nist.gov
```

Preferred NTP time server.

```yaml
ufw_enabled: true
ufw_logging: false
ufw_policy: deny
```

Whether the UFW firewall is enabled, and if logging is enabled (via the `LOG_KERN` facility). The `ufw_policy` sets the default policy for the firewall when no specific rules apply.

```yaml
rsyslog_enabled: true
syslog_server: logs.lan
syslog_port: 10514
```

Whether the remote `syslog` server is enabled and installe.  If enabled, logs are sent in RFC3164 format via UDP to the specified server and port so they can be parsed by e.g. Logstash or Graylog.

```yaml
telegraf_enabled: true
telegraf_global_tags: {}
telegraf_hostname: "{{ ansible_fqdn }}"
telegraf_collection_interval: "10s"
telegraf_collection_jitter: "0s"
telegraf_flush_interval: "10s"
telegraf_flush_jitter: "5s"
```

Whether the Telegraf agent is installed and enabled, global settings, and global tags to include with each metric.

```yaml
telegraf_influx_enabled: false
telegraf_influx_server: http://metrics.lan
telegraf_influx_port: 8086
telegraf_influx_database: telegraf
telegraf_influx_retention_policy: ''
telegraf_influx_username: ''
telegraf_influx_password: ''
```

Remote InfluxDB 1.x configuration, used for the output plugin.  By default, only InfluxDB 2.0 is used.

```yaml
telegraf_influx_v2_enabled: true
telegraf_influx_v2_server: http://metrics.lan
telegraf_influx_v2_port: 8086
telegraf_influx_v2_bucket: metrics
telegraf_influx_v2_organization: kraussnet
telegraf_influx_v2_token: influxdb-v2-api-token
```

Remote InfluxDB 1.x configuration.  The API token must grant write access to the specified organization and bucket.

```yaml
telegraf_conf_path: /etc
telegraf_conf_d_path: >-
  {{
    '/etc/telegraf.conf.d'
      if ansible_facts['os_family']|lower == 'alpine'
      else '/etc/telegraf/telegraf.d'
  }}

```

Override these variables to control where the Telegraf agent dynamic configuration files are stored.  By default the general configuration is stored in `/etc/telegraf.conf` and plugin configurations go in `/etc/telegraf.conf.d` on Alpine or `/etc/telegraf/telegraf.d` on Debian.

```yaml
msmtp_enabled: true
msmtp_accounts:
  - name: default
    host: smtp.gmail.com
    port: 587
    tls: true
    auth: true
    username: username@gmail.com
    password: app-password

msmtp_default_account: default
```

Whether the `msmtp` mail transfer agent is enabled, and the account settings to use for it.  Multiple accounts can be defined here using a YAML list with the options shown.  The `tls` and `auth` options may be omitted, and both default to `false`.  When no account is specified for the `msmtp` command, the account used is set by the `msmtp_default_account` variable.

```yaml
nfsv4_enabled: false
nfsv4_idmap_domain: "{{ ansible_domain }}"
nfsv4_idmap_service: >
  {{
    'rpc.idmapd'
      if ansible_facts['os_family']|lower == 'alpine'
      else 'nfs-idmapd' 
  }}

nfs_automount: true
```

Configures NFSv4 ID mapping and optionally enables automatic mount of NFS volumes at boot (Alpine Linux only).

## Role Facts

Facts registered by the role are listed below, with example data:

```yaml
alpine_sdcard_path: /media/mmcblk0p1
alpine_diskless: true
```

These variable are set when Alpine Linux is installed on a Raspberry Pi in diskless mode (the standard). These variables may be used to trigger the LBU function when configuration files are changed (note that LBU is automatically invoked by the post-tasks in this role for diskless installations).

## Dependencies

None

## Additional Entry Points

This role includes two additional entrypoints `pre-tasks.yaml` and `post-tasks.yaml` which are intended to be run in the `pre_tasks:` and `post_tasks:` blocks, respectively, of an Ansible play. These files will include distribution specific files which provide automatic Alpine LBU updates and other functionality.

## Example Playbook

    - hosts: all
      pre_tasks:
        include_role:
          name: asymworks.baseline
          tasks_from: pre-tasks.yml
      roles:
        asymworks.baseline
      post_tasks:
        include_role:
          name: asymworks.baseline
          tasks_from: post-tasks.yml

## License

MIT / BSD

## Author

This role was created in 2021 by Asymworks, LLC.
