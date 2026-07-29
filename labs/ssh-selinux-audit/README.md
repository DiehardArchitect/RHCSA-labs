# SSH Log Investigation & SELinux Audit Analysis (RHCSA Lab)

## Overview
This lab focused on investigating SSH authentication events and validating SELinux auditing on a Red Hat Enterprise Linux system.

## Objectives
- Generate failed SSH login attempts
- Investigate SSH daemon logs
- Correlate authentication events
- Verify SELinux enforcement
- Analyze SELinux audit records

## Environment

| Component | Value |
|---|---|
| OS | Red Hat Enterprise Linux |
| Service | OpenSSH (sshd) |
| Logging | systemd-journald, rsyslog |
| Security | SELinux (Enforcing) |
| Audit | auditd |

## Activities Performed

### 1. Generated Authentication Failures
Created multiple failed SSH login attempts using:
- Invalid username (fakeuser)
- Valid username (thearchitect) with an incorrect password

### 2. Investigated SSH Logs
journalctl -u sshd --since "3 hours ago"

Observed:
- Invalid user fakeuser
- Failed password for invalid user fakeuser
- Failed password for thearchitect
- Connection closed by authenticating user

### 3. Cross-Checked System Logs
sudo grep -Ei "Failed|Invalid" /var/log/secure

Verified that /var/log/secure contained the same authentication events recorded by the SSH daemon.

### 4. Verified SELinux Status
getenforce

Output:
Enforcing

### 5. Verified Audit Service
systemctl status auditd

Result:
Active: active (running)

### 6. Checked for SELinux AVC Denials
sudo ausearch -m avc -ts recent

Output:
<no matches>

## Analysis
The failed SSH login attempts were processed by the OpenSSH daemon and PAM authentication framework. Although authentication failed, none of the actions violated an SELinux policy. Because SELinux had nothing to deny, no AVC records were generated.

This demonstrates the distinction between:
- Authentication failures — handled by SSH/PAM
- SELinux access control violations — handled by SELinux policy enforcement

These are related security mechanisms but are logged independently.

## Skills Demonstrated
- Linux log analysis
- OpenSSH troubleshooting
- journalctl
- /var/log/secure investigation
- grep
- SELinux administration
- auditd
- ausearch
- Security event correlation
- Incident investigation

## Commands Reference
journalctl -u sshd --since "3 hours ago"
sudo grep -Ei "Failed|Invalid" /var/log/secure
getenforce
systemctl status auditd
sudo ausearch -m avc -ts recent

## Key Learning
Security investigations often require correlating information from multiple sources. SSH authentication logs, system logs, and SELinux audit records each provide different perspectives. Understanding how these components work together is essential for effective Linux administration and incident response.
