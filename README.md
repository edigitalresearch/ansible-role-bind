# BIND DNS Role for Ansible

This is a role which installs two DNS servers. One primary DNS and one secondary slave with passed records from Ansible variables.

## Organising hosts

This module requires 3 hosts groups: `dns`, `bind-master` and `bind-slave`:

* All DNS servers should belong under the `dns` group.
* Any master DNS servers should belong under `bind-master`
* Any slave DNS servers will refer to primary DNS in the `bind-master` group and they belong under `bind-slave`

### Example Hosts file

```
[dns:children]
bind-master
bind-slave

[bind-master]
ns1.example.com

[bind-slave]
ns2.example.com
```

## Variables

Here is an example variables file

```
---
  bind:
    admin_email: vagrant.edig.co.uk # Admin email for all zones
    nameservers: # A list of nameserver name -> ip pairs
      - name: ns1.example.co.uk.
        ip: 172.16.10.14
      - name: ns2.example.co.uk.
        ip: 172.16.10.15
    zones: # A dictionary of zones to create
      example.co.uk:
        records: # A dictionary of record types
          a:
            web: 172.16.10.13
      10.16.172.in-addr.arpa: # A reverse zone for servers
        records:
            # PTR records - the key is the last octet of your server IP. For example ip is 172.16.10.15 the key would be 15
            ptr:
              13: web.example.co.uk.
              14: ns1.example.co.uk.
              15: ns2.example.co.uk.

```

The role will create the forward zone `example.co.uk` with one A record `web.example.co.uk` pointing to 172.16.10.13 in this case. It will also create the reverse zones so both servers can answer reverse lookups.

## Handlers

This role provides three handlers

* `reload-bind`
* `start-bind`
* `restart-bind`

## Example Task with role

```
- name: DNS | BIND Setup
  hosts: dns
  roles:
    - edr.bind
```
