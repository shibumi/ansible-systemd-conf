Ansible Role: systemd-conf
==============================

Ansible role to configure systemd-networkd profiles, systemd config
files, systemd timers and systemd service files. This Project is a fork
of ansible-systemd-networkd of Anthony Ruhier. Thanks to Anthony for the
good work with the template

Role Variables
--------------

```yaml
---

systemd_conf: {}
systemd_conf_link: {}
systemd_conf_netdev: {}
systemd_conf_network: {}
systemd_conf_path: {}
systemd_conf_service: {}
systemd_conf_timer: {}
```

Dependencies
------------

Systemd

Example Playbook
-------------------------

1) Configure a network profile

```yaml
systemd_conf_network:
  eth0:
    - Match:
      - Name: "eth0"
    - Network:
      - DHCP: "no"
      - IPv6AcceptRouterAdvertisements: "no"
      - DNS: 8.8.8.8
      - DNS: 8.8.4.4
      - Domains: "your.tld"

      - Address: "192.0.2.176/24"
      - Gateway: "192.0.2.1"

      - Address: "2001:db8::302/64"
      - Address: "fc00:0:0:103::302/64"
      - Gateway: "2001:db8::1"
```

It will create a `eth0.network` profile in `/etc/systemd/network/`, and enable
`systemd-networkd` and `systemd-resolved`.

Every key under `systemd_conf_*` corresponds to the profile name to create
(`.network` in `systemd_conf_network`, `.link` in systemd_conf_link,
etc…). Then every key under the profile name is a section documented in
systemd-networkd, which contains the couples of `option: value` for your
profile. Each couple is then converted to the format `option=value`.

2) Configure a bonding interface

```yaml
systemd_conf_netdev:
  bond0:
    - NetDev:
      - Name: "bond0"
      - Kind: "bond"
    - Bond:
      - Mode: "802.3ad"
      - LACPTransmitRate: "fast"

systemd_conf_network:
  bond0:
    - Match:
      - Name: "eth*"
    - Network:
      - DHCP: "yes"
      - Bond: "bond0"
```

What will create an LACP bond interface `bond0`, containing all interfaces
starting by `eth`.

3) Configure a general config file like `/etc/systemd/resolved.conf`
```yaml
systemd_conf:
  resolved:
    - Resolved:
      - DNS: "8.8.8.8,8.8.8.4.4"
      - FallbackDNS: "1.1.1.1"
      - Domains: "myDomain"
      - DNSSEC: "yes"
```

4) Configure a systemd service
```yaml
systemd_conf_service:
    iwd:
    - Unit:
      - Description: "Wireless Service"
    - Service:
      - ExecStart: "/usr/lib/iwd/iwd"
      - LimitNPROC: "1"
    - Install:
      - WantedBy: "multi-user.target"
```

5) Configure a systemd timer
```yaml
systemd_conf_timer:
    knock:
    - Unit:
      - Description: "Run knock.service on boot"
    - Timer:
      - OnActiveSec: "0"
      - OnUnitActiveSec: "11h"
      - Unit: "knock.service"
    - Install:
      - WantedBy: "timers.target"
```

6) Configure a systemd path
```yaml
systemd_conf_path:
    passwd-mon:
    - Unit:
      - Description: "Monitor the /etc/passwd file for changes"
    - Path:
      - PathModified: "/etc/passwd"
      - Unit: "passwd-mon.service"
    - Install:
      - WantedBy: "multi-user.target"
```

License
-------

Tool under the BSD license. Do not hesitate to report bugs, ask me some
questions or do some pull request if you want to!
