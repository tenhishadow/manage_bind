Role Name
=========
This role can manage bind9 servers on Centos7. It is automation for configuring BIND on managed servers.
"named-chroot" package used instead of just named due to more secure approach.

I have designed this role to configure external bind9 servers to be masters.
Bind will check config and zones befor restart.

Requirements
------------
- Centos 7 or RHEL7
- bind-chroot

Example Playbook
----------------
```yaml
---

- hosts: external_dns
  become: "yes"
  become_user: "root"
  gather_facts: "true"
  any_errors_fatal: "true"
  serial: "1" 			# do it one by one to prevent fail on all BIND-servers
  vars:
    - configs: configs
  vars_files:
    - configs/ansible_vars.yml
  roles:
  - tenhishadow.manage_bind

...
```

Example files for playbook
----------------
```bash
.
├── configs
│   ├── ansible_vars.yml	# variables for playbook incude zones and params
│   ├── example.com.zone	# zone file with the records
│   ├── example1.com.zone	# zone file with the records
│   ├── named.j2		# central config which will be /etc/named.conf 
└── playbook.yml
```

Example content for ansible_vars.yml:
----------------
```yaml
---

zones:
  example.com:
    file: "example.com.zone"
    zone_serial: '2018052901'
    zone_refresh: '3600'
    zone_retry: '7200'
    zone_expire: '3600000'
    zone_minimum: '3600'
  example1.com:
    file: "example1.com.zone"
    zone_serial: '2018081301'
    zone_refresh: '3600'
    zone_retry: '7200'
    zone_expire: '3600000'
    zone_minimum: '3600'

...
```

Example content for named.j2
----------------
```shell
# {{ ansible_managed }}
options {
 listen-on port 53              { any; };
 listen-on-v6 port 53           { any; };
 directory                      "/var/named";
 dump-file                      "/var/named/data/cache_dump.db";
 statistics-file                "/var/named/data/named_stats.txt";
 memstatistics-file             "/var/named/data/named_mem_stats.txt";
 pid-file                       "/run/named/named.pid";
 session-keyfile                "/run/named/session.key";
 allow-query                    { any; };
 recursion                      no;
 version                        "AIX 5.1.0";
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
# END OF ZONES
```
Example content for configs/example.com.zone
----------------
```shell
; {{ ansible_managed }}
$TTL 3600
@                       IN      SOA     ns1.example.com. postmaster.example.com. (
        {{ item.value.zone_serial }}    ; serial YYYYMMDDnn
        {{ item.value.zone_refresh }}   ; refresh
        {{ item.value.zone_retry }}     ; retry
        {{ item.value.zone_expire }}    ; expire
        {{ item.value.zone_minimum }}   ; minimum ttl for zone
)
;### INT ns
                        IN      NS              ns1.example.com.
                        IN      NS              ns2.example.com.
                        IN      NS              ns3.example.com.
ns1                     IN      A               127.0.0.1			; srv1
ns2                     IN      A               127.0.0.1			; srv2
ns2                     IN      AAAA            12:3456:789a			; srv2
ns3                     IN      A               127.0.0.1			; srv3
;### END ns


;### INT site
@                       IN      TXT     "google-site-verification=TEST"
@                       IN      A               127.0.0.1
www                     IN      CNAME           another.domain.com.
;### END site


;### INT mail
@                       IN      MX      0       mx.another.domain.com.
@                       IN      MX      10      mx.another.domain.com.
;### END mail
```

License
-------
GPL v 3.0

Author Information
------------------
Stanislav Cherkasov
Tenhi adm@tenhi.ru
