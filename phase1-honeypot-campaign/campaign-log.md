# Honeypot Campaign #1: Campaign Log

Dual cloud T-Pot campaign, Azure vs AWS, 21 Oct to 21 Nov 2026.
Keep public IPs, passwords and keys OUT of this file if it goes on GitHub.

## Setup

| | Azure | AWS |
| --- | --- | --- |
| Region | Australia East | Asia Pacific (Sydney) ap-southeast-2 |
| VM size | Standard_B2as_v2 (2 vCPU, 8 GB) | m7i-flex.large (2 vCPU, 8 GB) |
| OS | Debian 13 "Trixie" x64 Gen 2 | Debian 13 x64 |
| Disk | 128 GiB Standard SSD | 128 GiB gp3 |
| T-Pot version / type | 24.04.1 / Hive | 24.04.1 / Hive |
| Honeypot ports opened (go live) | 21 Oct 2026, 16:21 AEDT (Azure Activity log, Allow-Honeypots rule) | 22 Oct 2026, 10:32 AEDT (CloudTrail, AuthorizeSecurityGroupIngress) |
| First attack seen | 21 Oct 2026, 16:21:30 AEDT. Honeytrap, port 31019, from Driftnet Ltd (an internet scanning company). Just an empty connection, no data sent. | 22 Oct 2026, 10:32:53 AEDT. Honeytrap, a 63 byte HTTP request: CONNECT api.ipinfo.io:443. A bot checking if my server is an open proxy. |
| Time from go live to first attack | 16 seconds | 3 seconds |
| Campaign end and teardown | | |

## Setup notes and problems hit

- 21 Oct: B2as_v2 only showed up once I picked an x64 image. The ARM sizes needed Security type set to Standard.
- 21 Oct: The new preview Create VM form didn't create the virtual network or public IP, so I switched back to the old form.
- 21 Oct: Lost my first SSH key. Made a new one with Reset password and Add SSH public key.
- 21 Oct: Rebooted before opening port 64295 in the NSG and got locked out. Added the Allow-Mgmt rule from the portal to get back in.
- 21 Oct: About an hour after T-Pot started the Azure VM froze. Azure Monitor showed CPU credits were fine but available memory dropped to almost zero. Restarted it and added 4 GB of swap.
- 22 Oct: Set up AWS and added the same 4 GB swap before going live. First AWS security group had TCP 1 to 64000 twice instead of TCP and UDP, fixed it before going live.
- 22 Oct: AWS credit showed US$120 remaining, so US$20 of the bonus credit was already earned. Azure credit is in the Education hub, not the sponsorships site. Azure shows A$142 credit (about US$100) and A$2.72 used so far in October.
- 22 Oct: The first logs after go live weren't attacks. On Azure it was the VM talking to 168.63.129.16 (Azure's own platform service) and on AWS it was DNS to 172.31.0.2 (the VPC resolver) plus outbound traffic. Have to filter out my own infrastructure's traffic to find the real first attack.

## Check-ins

Copy this block for every check in (every 2 or 3 days).

### Day __ (date)

**Azure**
- Total attacks so far:
- Top 3 honeypots hit:
- Top 3 source countries:
- Top 3 usernames / passwords tried:
- Anything new or odd:
- Credit used (US$):

**AWS**
- Total attacks so far:
- Top 3 honeypots hit:
- Top 3 source countries:
- Top 3 usernames / passwords tried:
- Anything new or odd:
- Credit used (US$):

**Screenshots saved:** (list file names)

### Day 1 (22 Oct 2026, Azure only)

Azure went live on 21 Oct in the afternoon and this check in was about 21 hours later. The VM froze from about 5pm to midnight because it ran out of memory, so these numbers are lower than a full day would be. AWS only went live on 22 Oct so it doesn't have a Day 1 yet.

**Azure**
- Total attacks so far: about 32,000
- Top honeypots hit: Cowrie (SSH and Telnet) 16k, Honeytrap (catches anything on ports nothing else listens on) 12k, Dionaea (SMB, FTP and other Windows services) 1k. After that Sentrypeer (VoIP) 938, Tanner (web) 448, Redishoneypot 423, RDPHoneypot 221, H0neytr4p 196, ConPot (industrial systems) 150, Miniprint (printers) 87.
- Top source countries: Bulgaria, United States, Canada, United Kingdom, The Netherlands. Bulgaria was almost all port 22 (SSH).
- Top usernames: root by a long way, then admin, user, ubuntu, postgres, oracle, test, sa
- Top passwords: 123456, a blank password, 1234, password, 12345678, admin, root
- Suricata alerts (top 3): ET INFO HTTP Request on Unusual Port (1,915), ET DROP Dshield Block Listed Source group 1 (1,638), ET EXPLOIT DoublePulsar Backdoor installation communication (1,379)
- Suricata CVEs (top 3): CVE-2001-0540 (140), CVE-2012-0152 (73), CVE-2002-0013 (46)
- Anything new or odd: see the notes below
- Credit used (US$):

**Screenshots saved:** screenshots/azure/day1-attacks-by-country-port.png, day1-honeypot-attack-counts.png, day1-username-password-tagclouds.png, day1-suricata-alerts-top10.png, day1-suricata-cve-top10.png

**Day 1 notes**
- 32k attacks in under a day on a brand new IP that nobody knew about. It's all automated bots scanning the whole internet, not people picking me out.
- Most of it is bots guessing logins on SSH. The usernames and passwords are the default and lazy ones you'd expect (root, admin, 123456, blank).
- DoublePulsar showed up 1,379 times. That's the NSA backdoor that leaked in 2017 and got used with WannaCry, so bots are still checking for it about 9 years later.
- 1,638 hits came from IPs already on the DShield blocklist, so a lot of this traffic is from known bad sources.
- Some usernames stood out like claude and openclaw. Looks like bots are now trying names linked to AI tools and agents. Worth watching since it ties into AI security.
- Some CVEs are really old (2001 and 2002, RDP and SNMP bugs) and some are from 2025 (CVE-2025-55182 and CVE-2025-29927, both in JavaScript web frameworks). So bots try old and brand new exploits at the same time. Need to check each CVE on NVD before the write up.

## Ideas / questions to look into for the write-up

-
