# SSH Brute-Force Detection with Wazuh

A home-lab project demonstrating how to detect SSH brute-force attacks using Wazuh SIEM, going beyond default rule alerting to a custom correlation rule.

## Overview

Wazuh ships with default rules that flag individual failed SSH logins, but a single failed login isn't an attack — it's noise. This project builds a custom correlation rule that distinguishes a real brute-force attempt (many failures from the same source in a short window) from normal login mistakes, and verifies it end-to-end with a live attack.

## Lab Environment

- **Wazuh Manager** — Ubuntu VM, monitoring its own system logs (`auth.log`)
- **Target** — the same Ubuntu VM, running OpenSSH server with a test account
- **Attacker** — Kali Linux VM, running Hydra to brute-force the SSH login
- All three VMs on the same internal network (VirtualBox)

## What Was Done

1. Installed and enabled SSH on the Ubuntu target, and created a test user account.
2. Launched a brute-force attack from Kali using Hydra against the SSH service, with a custom password wordlist.
3. Verified Wazuh's default rules caught individual events — failed login attempts and the eventual successful login.
4. Identified a gap: the defaults log *each* event but don't flag the *pattern* of repeated failures as a single, higher-severity finding.
5. Wrote a custom correlation rule that triggers when 5 or more failed SSH logins occur from the same source IP within 60 seconds, mapped to the MITRE ATT&CK technique for brute force (T1110).
6. Re-ran the attack and confirmed the custom rule fired correctly in the Wazuh dashboard, alongside the underlying individual events.

## Key Takeaway

Default SIEM rules are a starting point, not a finished detection strategy. Real brute-force detection depends on correlating multiple low-severity events into one meaningful, higher-severity alert — which is what distinguishes a usable SOC detection from raw log noise.

## Possible Extensions

- Auto-block the attacking IP when the custom rule fires (active response)
- Extend the same detection logic to RDP on the Windows endpoint
- Forward high-severity alerts to Slack/email for real-time notification

---

## Screenshots to Include

0. **Lab topology** (`00-lab-topology.png`) — diagram showing Kali attacking the Ubuntu VM over SSH, with the Wazuh manager on that same VM and alerts flowing to the dashboard.
1. **SSH service confirmation** (`01-ssh-enabled.png`) — terminal output showing SSH is active on the Ubuntu target.
2. **Hydra attack in progress or completed** (`02-hydra-attack.png`) — terminal output showing Hydra running against the target, including the line where it finds the valid password (like your `[22][ssh] host... password: welcome1` result).
3. **Wazuh dashboard — default rule alerts** (`03-default-alerts.png`) — the Threat Hunting/Events view filtered to `rule.id: 5760` and `rule.id: 5715`, showing the raw failed/successful login alerts.
4. **local_rules.xml editor** (`04-custom-rule-xml.png`) — the Wazuh GUI rules editor showing your custom rule 100101 added to the file.
5. **Wazuh dashboard — custom rule alert** (`05-custom-rule-alert.png`) — the Events view filtered to `rule.id: 100101`, showing your correlation rule firing at level 12 with the description "Custom: 5 failed SSH logins from same IP in 60s" (this is the one you already captured).
6. *(Optional)* **Full event timeline** (`06-event-timeline.png`) — the unfiltered events table showing the failed attempts followed by the custom rule alert in sequence, to visually tell the story of the attack.
