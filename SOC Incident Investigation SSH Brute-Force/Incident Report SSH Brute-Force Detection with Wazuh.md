# Incident Report: SSH Brute-Force Detection with Wazuh

Oct 10, 2026 · @Muaz

## Executive summary

A simulated SSH brute-force attack from a Kali Linux machine against an Ubuntu server on \[DATE\] was detected by Wazuh, and a custom detection rule I wrote now flags this pattern automatically. The attack tried \[NUMBER\] passwords in \[DURATION\] using the Hydra tool. Wazuh's built-in rules raised alerts, and my rule 100101 fires when one IP fails 5 SSH logins within 60 seconds. \[The attack did / did not\] gain access. This report covers the evidence, the detection logic and what I would recommend to harden the server.

## Environment and tools

The lab runs on VMware with all machines on a bridged network.

| Machine | Role | Software |
| --- | --- | --- |
| Ubuntu VM | Wazuh manager and SSH target | Wazuh manager, OpenSSH |
| Windows VM | Monitored endpoint | Wazuh agent, Apache web server |
| Kali Linux VM | Attacker | Hydra |

IP addresses: Ubuntu \[IP\], Windows \[IP\], Kali \[IP\].

## Attack timeline

Fill this from the real timestamps in the Wazuh alerts and the Ubuntu auth log.

| Time | Event | Source |
| --- | --- | --- |
| \[HH:MM:SS\] | Hydra attack started from Kali | Kali terminal |
| \[HH:MM:SS\] | First failed SSH login | Ubuntu auth log |
| \[HH:MM:SS\] | Rule 5760 alert (failed password) | Wazuh dashboard |
| \[HH:MM:SS\] | Rule 5715 alert (successful login), if any | Wazuh dashboard |
| \[HH:MM:SS\] | Custom rule 100101 fired | Wazuh dashboard |
| \[HH:MM:SS\] | Attack stopped | Kali terminal |

## Evidence and log analysis

The attacker was \[KALI IP\], targeting the account \[USERNAME\] on the Ubuntu server over SSH port 22. Add a screenshot or log excerpt under each point below.

1. **Raw log:** paste 3-5 failed-login lines from the Ubuntu auth log, with the IP and username visible.
2. **Wazuh alert:** screenshot of the rule 5760 alerts in the dashboard, showing the count over time.
3. **Counts:** failed logins \[NUMBER\], successful logins \[NUMBER\], unique usernames tried \[NUMBER\].
4. **Success check:** state clearly whether a rule 5715 success alert appears from the attacker IP, and what you searched to confirm it.

## Detection rule

Custom rule 100101 fires when the same IP fails 5 SSH logins within 60 seconds, and it is tagged to MITRE ATT&CK T1110 (Brute Force). I added it through the Wazuh rules editor and it builds on the default failed-login rule 5760.

- **Why it is needed:** the default rules alert on each failure, but a correlation rule turns a stream of single alerts into one clear brute-force signal.
- **How I tested it:** I ran Hydra from Kali against the Ubuntu server and confirmed rule 100101 fired.
- **Rule XML:** paste the final rule here in a code block.

## False positive check

A real user mistyping a password once or twice should not trigger rule 100101, because it needs 5 failures from one IP inside 60 seconds. To show this, log in to the Ubuntu server from the Windows VM with 2 wrong passwords and then the right one. Screenshot the Wazuh alerts: you should see rule 5760 for the failures and rule 5715 for the success, but no 100101. Write one line on what that tells an analyst: a few failures followed by a success is normal, while dozens of failures per minute from one IP is an attack.

## Findings and impact

The attack \[succeeded / failed\]. Fill in the points below from your evidence.

- **Result:** \[NUMBER\] failed logins from \[KALI IP\] in \[DURATION\]; successful login from that IP: \[yes / no\].
- **Detection speed:** the first alert fired \[X\] seconds after the attack began.
- **Impact if real:** if the password had been guessed, the attacker would have had shell access to the Ubuntu server, which also runs the Wazuh manager.

## Recommendations

1. **Use SSH keys and turn off password login.** Brute force then has nothing to guess.
2. **Disable direct root login** in the SSH configuration.
3. **Block repeat offenders automatically.** Reuse the Wazuh active response from my earlier lab, triggered by rule 100101, to block the source IP.
4. **Limit exposure.** Restrict SSH to known IP ranges or a VPN.
5. **Keep the rule tuned.** Review the 5-in-60-seconds threshold against real traffic to avoid false positives.

## MITRE ATT&CK mapping

| Technique | ID | How it appears here |
| --- | --- | --- |
| Brute Force | T1110 | Hydra guessing SSH passwords |
| Password Guessing | T1110.001 | Many passwords tried against one account |
| Remote Services: SSH | T1021.004 | Access path if a guess had worked |
