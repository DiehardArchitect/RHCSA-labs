# RHCSA Lab: SSH Troubleshooting and Configuration Override Investigation

## Objective

Troubleshoot an SSH connectivity issue between two RHEL servers and identify the root cause preventing successful authentication.

---

## Environment

| Hostname | IP Address     |
| -------- | -------------- |
| servera  | 192.168.100.10 |
| serverb  | 192.168.100.11 |

---

## Initial Problem

The servers could successfully communicate using ICMP:

```bash
ping serverb
ping servera
```

Both tests succeeded.

However, SSH connections from `servera` to `serverb` failed.

---

## Initial SSH Attempt

```bash
ssh student@serverb
```

Result:

```text
ssh: connect to host serverb port 22: Connection refused
```

Using the IP address produced the same result:

```bash
ssh student@192.168.100.11
```

Result:

```text
ssh: connect to host 192.168.100.11 port 22: Connection refused
```

---

## Investigation Step 1: Verify SSH Service

On `serverb`:

```bash
systemctl status sshd
```

Result:

```text
Active: active (running)
```

The SSH service was operational.

---

## Investigation Step 2: Verify Listening Ports

On `serverb`:

```bash
ss -tlnp | grep sshd
```

Result:

```text
LISTEN 0 128 0.0.0.0:2022
LISTEN 0 128 [::]:2022
```

### Finding

SSH was not listening on the default port 22. Instead, it was listening on port 2022.

---

## Investigation Step 3: Connect Using Correct Port

From `servera`:

```bash
ssh -p 2022 student@192.168.100.11
```

Result:

```text
Permission denied (publickey,gssapi-keyex,gssapi-with-mic)
```

### Finding

Connection was reaching the SSH service, but password authentication was being rejected.

---

## Investigation Step 4: Verify Effective SSH Configuration

On `serverb`:

```bash
sshd -T | grep passwordauthentication
sshd -T | grep pubkeyauthentication
```

Result:

```text
passwordauthentication no
pubkeyauthentication yes
```

### Finding

SSH was configured to allow key-based authentication only. Password authentication was disabled.

---

## Investigation Step 5: Check Main SSH Configuration

```bash
grep -E 'PasswordAuthentication|PubkeyAuthentication' /etc/ssh/sshd_config
```

Result:

```text
PasswordAuthentication yes
```

### Finding

The main config file showed PasswordAuthentication yes, but the effective config showed no. A drop-in file was overriding it.

---

## Investigation Step 6: Search All SSH Configuration Files

```bash
grep -R "PasswordAuthentication" /etc/ssh/
```

Result:

```text
/etc/ssh/sshd_config.d/99-custom.conf:PasswordAuthentication no
/etc/ssh/sshd_config:PasswordAuthentication yes
```

### Root Cause Identified

`/etc/ssh/sshd_config.d/99-custom.conf` was overriding the main configuration.

---

## Resolution

```bash
vi /etc/ssh/sshd_config.d/99-custom.conf
# Change: PasswordAuthentication no
# To:     PasswordAuthentication yes

systemctl restart sshd
sshd -T | grep passwordauthentication
```

---

## Verification

```bash
ssh -p 2022 student@192.168.100.11
```

Successful password prompt confirms resolution.

---

## Key Concepts

### Configuration Precedence

Drop-in files win because they load after the main file. A file named 99-custom.conf has the highest override weight by convention.

### Always Verify Effective Config

```bash
sshd -T
```

This shows what sshd is actually using after all files are merged. Never trust the main config file alone.

### Full Troubleshooting Workflow
Drop-in files win because they load after the main file. A file named 99-custom.conf has the highest override weight by convention.

### Always Verify Effective Config

```bash
sshd -T
```

This shows what sshd is actually using after all files are merged. Never trust the main config file alone.

### Full Troubleshooting Workflow
### Production Security Note

Re-enabling PasswordAuthentication yes resolves the lab but is a security regression in production. The correct production path is key-based auth only:

```bash
ssh-copy-id -p 2022 student@192.168.100.11
```

---

## Commands Reference

| Command | Purpose |
|---|---|
| `systemctl status sshd` | Check service state |
| `ss -tlnp \| grep sshd` | Check listening port |
| `sshd -T` | Show effective runtime config |
| `grep -R "keyword" /etc/ssh/` | Find config across all files |
| `firewall-cmd --list-all` | Check firewall rules |
| `semanage port -l \| grep ssh` | Check SELinux port policy |
| `systemctl restart sshd` | Apply config changes |

---

## RHCSA Skills Demonstrated

- SSH administration and port management
- SSH authentication troubleshooting
- Service management with systemd
- Socket inspection with ss
- Configuration precedence and drop-in overrides
- SELinux port policy awareness
- Firewall rule verification
- Root cause analysis methodology
