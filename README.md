# Linux Home Lab Hardening & Compliance Audit

A hands-on project hardening an Ubuntu Server VM against CIS Benchmark controls, with before/after evidence and NIST 800-53 control mapping.

## Overview

This project simulates a real-world system hardening and compliance audit workflow: stand up a baseline Linux system, assess it against an industry benchmark, remediate findings, and document the results in a format aligned with federal control frameworks (NIST 800-53).

**Environment:** Ubuntu Server 26.04 LTS (ARM64), UTM on Apple Silicon
**Benchmark used:** CIS Ubuntu Linux Benchmark / Lynis
**Framework mapping:** NIST 800-53 Rev. 5

## Objective

- Establish a security baseline on a fresh Linux install
- Identify and remediate misconfigurations across access control, network exposure, and logging
- Produce audit evidence and a before/after compliance report suitable for a GRC-style review

## What Was Hardened

| Area | Change Made | Control Family (NIST 800-53) |
|---|---|---|
| SSH access | Disabled root login, disabled password auth (key-based only), restricted login to authorized group | AC-3, IA-2 |
| Firewall | Default-deny inbound via `ufw`, explicit allow rules only | SC-7 |
| Services | Disabled unused services (e.g. avahi-daemon, cups) | CM-7 |
| User accounts | Enforced password aging policy, removed stale accounts, tightened sudo access | AC-2, AC-6 |
| File permissions | Removed world-writable files, enforced strict umask | AC-6 |
| Audit logging | Installed and configured `auditd` with rules watching sensitive files (`/etc/passwd`, sudoers, SSH config) | AU-2, AU-12 |

## Results

| Metric | Before | After |
|---|---|---|
| Lynis Hardening Index | 63/100 | 72/100 |
| Suggestions flagged | 45 | (re-scan reflects SSH, firewall, services, users, and auditd fixes applied) |

*(Full Lynis scan output for both baseline and post-hardening runs is included in `/evidence`.)*

## Repository Structure

- `README.md`
- `configs/`
  - `sshd_config.diff`
  - `ufw-rules.txt`
  - `audit.rules`
- `evidence/`
  - `lynis-before.log`
  - `lynis-after.log`
  - `screenshots/`
- `report/`
  - `hardening-audit-report.pdf`

## Methodology

1. Deployed a clean Ubuntu Server VM and snapshotted it as the baseline
2. Ran an initial Lynis scan to establish the pre-hardening score and findings list
3. Applied hardening changes one category at a time (SSH → firewall → services → users → auditd), verifying system stability after each change
4. Re-ran Lynis post-hardening and compared results
5. Mapped each finding and remediation to its corresponding NIST 800-53 control family
6. Documented everything in the full report (`/report/hardening-audit-report.pdf`)

## Key Takeaways

- The auditd rules failed to load silently after I first configured them — `systemctl status` showed the service as failed, but the real cause only surfaced after digging into `/etc/audit/audit.rules` directly and finding a stray line that had accidentally been written into the rules file instead of run as a command. It reinforced that a service reporting "active" doesn't guarantee its configuration actually took effect — you have to verify the actual behavior (in this case, with a live `ausearch` test), not just the service status.
- At scale, I'd automate this entire process with Ansible or a shell script instead of manually editing config files on each host, and I'd build in a config validation step (like `sshd -t` or `augenrules --check`) before ever restarting a service, to catch errors before they cause downtime across a fleet of machines.

## Tools Used

Ubuntu Server, VirtualBox, Lynis, `ufw`, `auditd`, `systemctl`

---

*This project was built as part of my transition into cybersecurity/GRC roles, focused on translating hands-on system administration into control-framework language used in compliance and risk management work.*
