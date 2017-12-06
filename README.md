Role Name
=========
This role can manage bind9 server on Centos7.
I uses package named-chroot instead of just named.
I have designed it to configure 2 external bind9 servers to be masters.
I don't care any dontime and don't use 'rndc reload zone' (maybe it's temporary, maybe not)

Requirements
------------
1-2 Centos7 Servers

Role Variables
--------------
File ansible_vars.yml:
```yaml
---
zones:
  somedomain.org:
    file: "somedomain.org.zone"
...
```

Dependencies
------------
-

Example Playbook
----------------

```yaml
---
- hosts: external_dns
  gather_facts: "true"
  any_errors_fatal: "true"
  serial: "1"
  vars:
    - configs: /root/bind_configs
  vars_files:
    - {{ configs }}/ansible_vars.yml
  roles: tenhishadow.manage_bind
...
```

So, as you can see role requere configuration files to manage BIND.
e.g. you need to need to have files in /root/bind_configs:
```bash
ansible_vars.yml	# dict of zone with name of zone and zonefile
named.j2		# central config which will be /etc/named.conf
somedomain.org.zone	# file of zone mentioned also in ansible_vars.yml
```
Examples:

File ansible_vars.yml:
```yaml
---
zones:
  somedomain.org:
    file: "somedomain.org.zone"
...
```
File named.j2
```yaml
# {{ ansible_managed }}
options {
 listen-on port 53              { {{ ansible_default_ipv4.address }}; };
 listen-on-v6 port 53           { {{ ansible_default_ipv6.address|default('none') }}; };
 directory                      "/var/named";
 dump-file                      "/var/named/data/cache_dump.db";
 statistics-file                "/var/named/data/named_stats.txt";
 memstatistics-file             "/var/named/data/named_mem_stats.txt";
 pid-file                       "/run/named/named.pid";
 session-keyfile                "/run/named/session.key";
 allow-query                    { any; };
 recursion                      no;
 version                        "AIX 5.2";
 auth-nxdomain                  no;
 dnssec-enable                  yes;
 dnssec-validation              auto;
 bindkeys-file                  "/etc/named.iscdlv.key";
 managed-keys-directory         "/var/named/dynamic";
 transfers-out                  100;
};

logging {
 channel  default_debug {
 file     "data/named.run";
 severity dynamic;
 };
};

# START ZONES
{% for key, value in zones.iteritems() %}
zone "{{ key }}" {
 type           master;
 file           "/var/named/data/{{ value.file }}";
 allow-transfer { none; };
};
{% endfor %}
# END ZONES

```

License
-------
GPL v 3.0

Author Information
------------------
Tenhi adm@tenhi.ru
