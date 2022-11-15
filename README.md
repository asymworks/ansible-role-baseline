# Ansible Role: Asymworks Linux Baseline

An Ansible Role that installs common files and utilities on Asymworks servers running Debian or Alpine Linux. The default entry point performs the following tasks:

- Updates package lists and installs basic utilties such as `curl` and `vim`
- Disables SSH root login and user password login
- Installs a MOTD which displays the asset name and tag
- Applies tweaks to `sysctl`
- Installs the `restic` backup agent
- Installs and starts the `ntpd` service with a configurable NTP server
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
ntp_bootstrap: false
```

Preferred NTP time server and whether to add hard-coded NIST server IPs to the server list so that Chrony can set the clock at startup even if the main server and DNS is unavailable.

```yaml
ufw_enabled: true
ufw_ipv6: false
ufw_logging: false
ufw_policy: deny
```

Whether the UFW firewall is enabled, and if logging is enabled (via the `LOG_KERN` facility). The `ufw_policy` sets the default policy for the firewall when no specific rules apply.  If `ufw_ipv6` is truthy and the `ip6_tables` kernel module is present, the firewall will be enabled for IPv6.  If either is false, IPv6 will be disabled in UFW.

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
