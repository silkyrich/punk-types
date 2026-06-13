# Library taxonomy

Two orthogonal classifications — don't conflate them:

## 1. Directory tree — by ORIGIN (where the log came from)
```
<domain>/<vendor-or-product>/<sourcetype>/
  os/linux/sshd            os/linux/sudo        os/macos/bsd-syslog
  web/nginx/nginx-access   network/firewall/cisco-asa
  network/firewall/iptables  network/proxy/squid  network/dns/bind/bind-named
  email/postfix/postfix    db/postgres
```
Domains: os, web, network, email, db, security, apps, ai, host-metrics.
The product/vendor level keeps vendors discoverable; the sourcetype is the leaf.

## 2. CIM models — by MEANING (what the events are about)
A sourcetype feeds one or more Common Information Model datamodels by
producing their canonical fields. This is cross-cutting — a firewall and an
auth log live in different tree branches but both may feed `network_session`.

| model | required fields | example sources |
|---|---|---|
| authentication | action, user | sshd |
| network_traffic | src, dest | cisco-asa, iptables |
| network_session | src | postfix, squid, sshd |
| network_resolution | query | bind-named |
| web | status, method | nginx, squid |
| email | from | postfix |
| change | action, object | sudo |
| dhcp | server_ip | tplink-syslog |

A program "joins" a model simply by extracting its fields (often via
field-ops: `rename src_ip to src`, `$result == "Accepted" { action = "success" }`).
No separate tagging layer — see RFC-004.
