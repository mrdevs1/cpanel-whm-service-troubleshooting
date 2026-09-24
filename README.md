# cPanel/WHM Service Troubleshooting

A practical troubleshooting guide for diagnosing and resolving common service problems on **cPanel/WHM servers** running Linux, CloudLinux, LiteSpeed, Apache, PHP-FPM, MySQL/MariaDB, Exim, PowerDNS, AutoSSL, and related services.

The goal of this guide is to provide a structured troubleshooting workflow instead of blindly restarting services.

---

## Overview

A cPanel/WHM server depends on multiple services working together.

Common service problems include:

* Website not loading
* HTTP 500 errors
* PHP errors
* Database connection failures
* Mail delivery problems
* DNS resolution problems
* SSL/AutoSSL failures
* LiteSpeed or Apache failures
* PHP-FPM problems
* Services failing after reboot
* High resource usage causing services to become unstable

Before making changes, identify **which service is failing and why**.

---

## 1. Check Server Load First

Before troubleshooting a service, check the overall server condition:

```bash
uptime
```

```bash
free -h
```

```bash
df -h
```

```bash
df -i
```

Check CPU-consuming processes:

```bash
ps aux --sort=-%cpu | head -20
```

Check memory-consuming processes:

```bash
ps aux --sort=-%mem | head -20
```

A service may appear broken when the actual problem is:

* Full disk
* Memory exhaustion
* High CPU load
* I/O wait
* OOM killer activity
* Inode exhaustion

---

## 2. Check cPanel Service Status

WHM provides a service status interface:

```text
WHM → Service Configuration → Service Manager
```

You can also check individual services from the command line.

For example:

```bash
systemctl status crond
```

```bash
systemctl status named
```

```bash
systemctl status mariadb
```

Depending on the server configuration, service names may differ.

---

## 3. Check Failed Systemd Services

Start with:

```bash
systemctl --failed
```

This gives a quick overview of failed services.

For more information:

```bash
systemctl --failed --no-pager
```

Then inspect the specific service:

```bash
systemctl status SERVICE_NAME
```

Example:

```bash
systemctl status mariadb
```

---

## 4. Check Service Logs

When a service fails, the status output may not contain enough information.

Use:

```bash
journalctl -u SERVICE_NAME
```

For recent logs:

```bash
journalctl -u SERVICE_NAME --since "1 hour ago"
```

For the current boot:

```bash
journalctl -u SERVICE_NAME -b
```

Follow logs in real time:

```bash
journalctl -u SERVICE_NAME -f
```

Replace `SERVICE_NAME` with the actual systemd service.

---

# Apache / LiteSpeed Troubleshooting

## 5. Check LiteSpeed

If using LiteSpeed:

```bash
systemctl status lsws
```

Check the process:

```bash
ps aux | grep -i litespeed
```

Check whether the service is listening:

```bash
ss -lntp | grep -E ':80|:443'
```

If LiteSpeed is not running, inspect the service logs before restarting it.

---

## 6. Check LiteSpeed Configuration

Before restarting after a configuration change, validate the configuration where supported by the installed LiteSpeed version.

Also review:

```text
WHM → Plugins → LiteSpeed Web Server
```

Look for:

* Configuration errors
* PHP handler problems
* Listener problems
* Virtual host issues
* Backend connection problems

---

## 7. Check Apache

If the server uses Apache:

```bash
systemctl status httpd
```

Check the process:

```bash
ps aux | grep httpd
```

Check listening ports:

```bash
ss -lntp | grep -E ':80|:443'
```

Check recent logs:

```bash
journalctl -u httpd --since "1 hour ago"
```

---

## 8. Check Web Server Logs

Common Apache log locations include:

```text
/var/log/httpd/
```

On cPanel servers, domain-specific logs may also be available under account directories and through cPanel/WHM interfaces.

Check recent errors:

```bash
tail -100 /var/log/httpd/error_log
```

Search for common HTTP errors:

```bash
grep -iE 'error|fatal|failed|denied' /var/log/httpd/error_log | tail -50
```

For LiteSpeed, use the LiteSpeed log locations configured on the server.

---

# PHP / PHP-FPM Troubleshooting

## 9. Check PHP-FPM

For PHP-FPM:

```bash
systemctl --type=service | grep php-fpm
```

Check the relevant PHP-FPM service:

```bash
systemctl status php-fpm
```

The exact service name may vary depending on the installed PHP version and cPanel configuration.

---

## 10. Find PHP Processes

Use:

```bash
ps aux | grep php
```

For CPU usage:

```bash
ps aux --sort=-%cpu | grep php | head -20
```

For memory usage:

```bash
ps aux --sort=-%mem | grep php | head -20
```

Large numbers of PHP workers may indicate:

* High traffic
* Slow PHP requests
* WooCommerce activity
* Plugin problems
* External API delays
* Database problems
* Incorrect PHP-FPM settings

---

## 11. Check PHP Version

Check CLI PHP:

```bash
php -v
```

List installed PHP binaries on systems using cPanel/CloudLinux where appropriate:

```bash
ls -lah /opt/cpanel/ea-php*/root/usr/bin/php
```

Remember that **CLI PHP and website PHP may use different versions**.

A website can therefore fail even when:

```bash
php -v
```

looks correct.

Always verify the PHP version configured for the affected domain.

---

## 12. Common PHP Problems

Common symptoms include:

```text
500 Internal Server Error
Allowed memory size exhausted
Call to undefined function
Class not found
Maximum execution time exceeded
PHP-FPM connection errors
```

Check:

* PHP version
* PHP extensions
* PHP memory limit
* PHP-FPM configuration
* Application logs
* WordPress/plugin compatibility
* File permissions
* Recent configuration changes

Do not increase memory or worker limits blindly.

---

# MySQL / MariaDB Troubleshooting

## 13. Check Database Service

For MariaDB:

```bash
systemctl status mariadb
```

For MySQL installations:

```bash
systemctl status mysqld
```

Check the process:

```bash
ps aux | grep -E 'mysqld|mariadbd'
```

---

## 14. Check Database Logs

Use:

```bash
journalctl -u mariadb --since "1 hour ago"
```

or:

```bash
journalctl -u mysqld --since "1 hour ago"
```

Depending on the installation.

Look for:

* InnoDB errors
* Corrupted tables
* Permission problems
* Disk-space errors
* Connection failures
* Startup failures
* Memory-related errors

---

## 15. Check Active Database Connections

Run:

```bash
mysqladmin processlist
```

Or:

```bash
mysql -e "SHOW FULL PROCESSLIST;"
```

Look for:

* Long-running queries
* Locked queries
* Too many connections
* Repeated queries
* Queries waiting for resources

Do not terminate database processes or delete database files without understanding their purpose.

---

# Exim / Mail Troubleshooting

## 16. Check Exim

Check the service:

```bash
systemctl status exim
```

Check whether Exim is listening:

```bash
ss -lntp | grep :25
```

Depending on the server configuration, submission services may also use ports such as:

```text
465
587
```

---

## 17. Check the Exim Queue

View the queue:

```bash
exim -bp
```

Count queued messages:

```bash
exim -bpc
```

A very large queue should be investigated.

Possible causes include:

* Failed delivery
* Authentication problems
* DNS problems
* Spam activity
* Remote server rejection
* Misconfigured applications

Do not blindly delete the mail queue.

---

## 18. Search Exim Logs

Common locations include:

```text
/var/log/exim_mainlog
/var/log/exim_rejectlog
/var/log/exim_paniclog
```

Check recent messages:

```bash
tail -100 /var/log/exim_mainlog
```

Search for errors:

```bash
grep -iE 'error|failed|rejected|defer' /var/log/exim_mainlog | tail -50
```

---

# DNS / PowerDNS Troubleshooting

## 19. Check DNS Service

For PowerDNS:

```bash
systemctl status pdns
```

Check listening ports:

```bash
ss -lntup | grep :53
```

DNS normally uses:

```text
UDP 53
TCP 53
```

---

## 20. Test DNS Locally

Use:

```bash
dig example.com
```

Test the authoritative nameserver:

```bash
dig @NAMESERVER_IP example.com
```

Check the nameserver records:

```bash
dig NS example.com
```

Check an A record:

```bash
dig A example.com
```

Replace the example domain with the affected domain.

---

## 21. Check DNS Zone Problems

When a domain does not resolve, investigate:

* Nameserver delegation
* A records
* AAAA records
* DNS zone configuration
* DNS service status
* Firewall rules
* Glue records
* DNSSEC configuration
* Propagation/cache

Do not assume every DNS problem is caused by DNS propagation.

---

# AutoSSL Troubleshooting

## 22. Check AutoSSL

AutoSSL problems can cause websites to lose or fail to renew SSL certificates.

In WHM:

```text
WHM → SSL/TLS → Manage AutoSSL
```

Check:

* AutoSSL provider
* Domain validation
* DNS records
* HTTP validation
* Port 80 accessibility
* Domain ownership
* Certificate errors

---

## 23. Check HTTP Validation

For HTTP-based validation, make sure the domain can be reached over:

```text
http://example.com/
```

Check from the server:

```bash
curl -I http://example.com/
```

A validation URL may also be affected by:

* Cloudflare
* Redirects
* Firewall rules
* Incorrect DNS
* Web server configuration
* Missing document root

---

# Ports and Connectivity

## 24. Check Listening Ports

Use:

```bash
ss -lntup
```

Common hosting ports include:

```text
22    SSH
25    SMTP
53    DNS
80    HTTP
110   POP3
143   IMAP
443   HTTPS
465   SMTPS
587   SMTP Submission
993   IMAPS
995   POP3S
```

Actual enabled services and ports depend on the server configuration.

---

## 25. Check a Specific Port

For local listening services:

```bash
ss -lntp | grep :443
```

For external connectivity, use appropriate network testing tools.

For example:

```bash
curl -I https://example.com
```

---

# Disk and Filesystem Problems

## 26. Check Disk Space

```bash
df -h
```

Check inode usage:

```bash
df -i
```

Find large directories:

```bash
du -xhd1 /var 2>/dev/null | sort -h
```

For cPanel accounts:

```bash
du -xhd1 /home 2>/dev/null | sort -h
```

A full filesystem can cause multiple services to fail simultaneously.

---

## 27. Check Deleted Files

If disk usage appears inconsistent:

```bash
lsof +L1
```

A deleted file can continue consuming disk space while a process still has it open.

Identify the process before taking action.

---

# Service Failures After Reboot

## 28. Check Failed Services

Run:

```bash
systemctl --failed
```

Then inspect each failed service:

```bash
systemctl status SERVICE_NAME
```

Review the current boot logs:

```bash
journalctl -b
```

Search for errors:

```bash
journalctl -b -p err
```

Common causes include:

* Configuration errors
* Missing files
* Incorrect permissions
* Dependency failures
* Full disk
* Port conflicts
* Invalid certificates
* Database startup problems

---

# Check for Port Conflicts

## 29. Find Which Process Uses a Port

Example for HTTPS:

```bash
ss -lntp | grep :443
```

Example for HTTP:

```bash
ss -lntp | grep :80
```

You can also use:

```bash
lsof -i :443
```

This is useful when a service fails to start because another process is already using its required port.

---

# Security and Resource Checks

## 30. Check Unexpected Processes

Review:

```bash
ps aux
```

Look for unusual:

* Executables
* User accounts
* High CPU processes
* Long-running scripts
* Network connections

On hosting servers, also review:

* CloudLinux account usage
* Imunify360 alerts
* cPHulk activity
* Firewall logs
* Web access logs

Do not assume an unfamiliar process is malicious without investigation.

---

## 31. Check OOM Events

Memory exhaustion can cause Linux to terminate processes.

Run:

```bash
dmesg -T | grep -i -E 'out of memory|oom|killed process'
```

Or:

```bash
journalctl -k | grep -i -E 'out of memory|oom|killed process'
```

If OOM events are present, investigate which processes consumed the memory.

---

# Safe Service Restart Workflow

## 32. Before Restarting a Service

Check:

```bash
systemctl status SERVICE_NAME
```

Review recent logs:

```bash
journalctl -u SERVICE_NAME --since "30 minutes ago"
```

Check resource usage:

```bash
free -h
df -h
uptime
```

Then determine whether a restart is appropriate.

---

## 33. Restart Only When Appropriate

Example:

```bash
systemctl restart SERVICE_NAME
```

Then immediately verify:

```bash
systemctl status SERVICE_NAME
```

Check whether the service is listening:

```bash
ss -lntup
```

Finally, test the affected application.

A restart should be treated as part of the troubleshooting process, not the troubleshooting process itself.

---

# Useful Diagnostic Commands

## System

```bash
uptime
free -h
df -h
df -i
lsblk
```

## Processes

```bash
top
ps aux --sort=-%cpu | head -20
ps aux --sort=-%mem | head -20
pstree -ap
```

## Services

```bash
systemctl --failed
systemctl status SERVICE_NAME
journalctl -u SERVICE_NAME --since "1 hour ago"
```

## Network

```bash
ss -lntup
ss -s
```

## DNS

```bash
dig example.com
dig NS example.com
dig @NAMESERVER_IP example.com
```

## Database

```bash
mysqladmin processlist
mysql -e "SHOW FULL PROCESSLIST;"
```

## Disk

```bash
du -xhd1 / 2>/dev/null | sort -h
lsof +L1
```

## Logs

```bash
journalctl -p warning -b
dmesg -T | tail -100
```

---

# Troubleshooting Workflow

Use this general workflow when a cPanel/WHM service is reported as broken:

```text
1. Identify the affected service
        ↓
2. Check overall server health
        ↓
3. Check service status
        ↓
4. Review service logs
        ↓
5. Check configuration
        ↓
6. Check ports and dependencies
        ↓
7. Check CPU / memory / disk
        ↓
8. Reproduce or test the problem
        ↓
9. Apply the smallest safe fix
        ↓
10. Restart/reload only if necessary
        ↓
11. Verify the service
        ↓
12. Monitor for recurrence
```

---

# Common Symptoms and Areas to Check

| Symptom                      | Areas to Check                           |
| ---------------------------- | ---------------------------------------- |
| Website unavailable          | LiteSpeed/Apache, DNS, firewall, ports   |
| HTTP 500                     | PHP, PHP-FPM, application logs           |
| Database connection error    | MySQL/MariaDB, credentials, connections  |
| Email not sending            | Exim, DNS, queue, SMTP connectivity      |
| Email not receiving          | Exim, DNS/MX, ports, mailbox storage     |
| DNS not resolving            | PowerDNS, zone, delegation, firewall     |
| SSL failed                   | AutoSSL, DNS, HTTP validation            |
| Service won't start          | Logs, configuration, ports, dependencies |
| Server slow                  | CPU, memory, I/O, database, PHP          |
| Disk full                    | `df`, `du`, logs, backups, deleted files |
| Service stopped after reboot | `systemctl --failed`, boot logs          |

---

# Final Checklist

Before closing a cPanel/WHM service issue:

* [ ] Identified the affected service
* [ ] Checked overall server load
* [ ] Checked memory and swap
* [ ] Checked disk space
* [ ] Checked inode usage
* [ ] Checked service status
* [ ] Reviewed service logs
* [ ] Checked configuration
* [ ] Checked listening ports
* [ ] Checked service dependencies
* [ ] Checked related services
* [ ] Tested the affected website/service
* [ ] Applied the smallest appropriate fix
* [ ] Verified the service after the fix
* [ ] Monitored for recurring errors

---

# Key Principle

> **Find the cause before restarting the service.**

A successful restart only confirms that the service can start again. It does not necessarily explain why it stopped.

A proper troubleshooting process should answer:

1. **Which service is failing?**
2. **What caused it to fail?**
3. **Is the problem configuration, resource, dependency, or application related?**
4. **What is the safest fix?**
5. **How can the problem be prevented from returning?**

---

## Disclaimer

This guide is intended for system administration and troubleshooting purposes.

Always review command output before making changes to a production server. Maintain appropriate backups and avoid deleting files, terminating processes, or restarting critical services without understanding their purpose and potential impact.
