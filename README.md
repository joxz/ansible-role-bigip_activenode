role-bigip_activenode
=========

Ansible role to determine the primary units in Big-IP clusters.
Devices with `failover_state: active` are grouped in ansible inventory group `acive_units` for further use in the playbook. Standalone Big-IPs count as primary units.

This enables to do changes only on the active unit in a HA pair and sync the config to the standby unit later in the playbook.

Requirements
------------

- Requires BIG-IP software version >= 12

- f5-sdk: `pip install f5-sdk`

- Ansible 2.5 or higher

Role Variables
--------------

```
F5_PASSWORD: "passsssssss"
F5_SERVER: "f5.fqdn.net"      # F5 BigIP management IP
F5_USER: "admin"
F5_VALIDATE_CERTS: "no"       # SSL certificate validation
F5_SERVER_PORT                # optional, default: 443
F5_TRANSPORT                  # optional, default: rest
ANSIBLE_NET_SSH_KEYFILE       # optional, SSH keyfile for CLI connection
```

Example Playbook
----------------

Note: `serial: 1` is important to correctly fill the `active_units` group.

`F5_USER, F5_SERVER, F5_PASSWORD` variables are copied to the `active_units` group.

```yaml
---
- name: find active cluster devices
  hosts: all
  connection: local
  gather_facts: no
  serial: 1
  
  tasks:
  - include_role:
      name: ansible-role-bigip_activenode

- name: sys info
  hosts: active_units
  connection: local
  gather_facts: no

  tasks:
  - name: get sys info from bigip
    bigip_device_facts:
      gather_subset:
        - system-info
      provider: "{{provider}}"
    register: sysinfo

  - name: debug sys-info
    debug:
      msg: "uptime: {{sysinfo.system_info.uptime}}"
```

Example Inventory
-----------------

Both YAML and INI inventories work, the only variables needed for each host/group are `F5_USER, F5_SERVER, F5_PASSWORD, F5_VALIDATE_CERTS`

```yaml
all:
  children:
    cluster_bigip:
      hosts:
        bigip01:
          F5_SERVER: "bigip01.fqdn.net"
        bigip02:
          F5_SERVER: "bigip02.fqdn.net"
      vars:
        F5_USER: "admin"
        F5_PASSWORD: "passwd"  
    standalone_bigip:
      hosts:
        bigip03:
          F5_SERVER: "bigip03.fqdn.net"
          F5_USER: "admin"
          F5_PASSWORD: "otherpasswd"
  vars: 
    F5_VALIDATE_CERTS: "no"
    provider:
      password: "{{F5_PASSWORD}}"
      server: "{{F5_SERVER}}"
      user: "{{F5_USER}}"
      validate_certs: "{{F5_VALIDATE_CERTS}}"
```

Example Debug Output
--------------------

Debug Active Node Output:

```
TASK [ansible-role-bigip_activenode : debug facts] ******************************************************************************************************
ok: [bigip01.fqdn.net] => (item={u'state': u'active', u'hostname': u'bigip01.fqdn.net', u'name': u'bigip01'}) => {
    "item": {
        "hostname": "bigip01.fqdn.net",
        "name": "bigip01",
        "state": "active"
    }
}
```

Groups Debug (truncated):

```
...
    "groups": {
        "active_units": [
            "bigip01.fqdn.net",
            "bigip03.fqdn.net"
        ],
        "all": [
            "bigip01.fqdn.net",
            "bigip02.fqdn.net",
            "bigip03.fqdn.net"
        ],
        "cluster_bigip": [
            "bigip01.fqdn.net",
            "bigip02.fqdn.net"
        ],
...
```

License
-------

MIT

Author Information
------------------

Johannes Denninger

[https://github.com/joxz](https://github.com/joxz)
