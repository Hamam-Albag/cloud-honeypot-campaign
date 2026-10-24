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
- 24 Oct: Noticed Network Out on Azure drop to zero and all containers showing Up 4 hours. Found the cause in the crontab: T-Pot reboots itself every day at 02:45 UTC (13:45 Sydney time). It stops T-Pot, cleans up old Docker containers, images and volumes, then reboots. So there's a gap of a few minutes in the data every day. AWS has the same job at the same UTC time so the gaps line up and the comparison stays fair.

## Check-ins

Copy this block for every check in (every 2 or 3 days).

### Day __ (date)

**Azure**
- Total attacks so far:
- Top 3 honeypots hit:
- Top 3 source countries:
- Top 3 usernames / passwords tried:
- Anything new or odd:

**AWS**
- Total attacks so far:
- Top 3 honeypots hit:
- Top 3 source countries:
- Top 3 usernames / passwords tried:
- Anything new or odd:

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

**Screenshots saved:** ../screenshots/azure/day1-attacks-by-country-port.png, day1-honeypot-attack-counts.png, day1-username-password-tagclouds.png, day1-suricata-alerts-top10.png, day1-suricata-cve-top10.png

**Day 1 notes**
- 32k attacks in under a day on a brand new IP that nobody knew about. It's all automated bots scanning the whole internet, not people picking me out.
- Most of it is bots guessing logins on SSH. The usernames and passwords are the default and lazy ones you'd expect (root, admin, 123456, blank).
- DoublePulsar showed up 1,379 times. That's the NSA backdoor that leaked in 2017 and got used with WannaCry, so bots are still checking for it about 9 years later.
- 1,638 hits came from IPs already on the DShield blocklist, so a lot of this traffic is from known bad sources.
- Some usernames stood out like claude and openclaw. Looks like bots are now trying names linked to AI tools and agents. Worth watching since it ties into AI security.
- Some CVEs are really old (2001 and 2002, RDP and SNMP bugs) and some are from 2025 (CVE-2025-55182 and CVE-2025-29927, both in JavaScript web frameworks). So bots try old and brand new exploits at the same time. Need to check each CVE on NVD before the write up. (Update 23 Oct: the 2001 RDP one is mostly normal RDP connections that a Suricata rule tags with that CVE, see the AWS Day 1 notes.)

### Day 1 (23 Oct 2026, AWS only)

AWS went live on 22 Oct at 10:32 and this check in was about 24 hours later. No downtime this time because swap was set up before going live.

**AWS**
- Total attacks so far: about 28,000
- Top honeypots hit: Honeytrap 8k, Dionaea (SMB, FTP, MSSQL and other Windows services) 8k, Sentrypeer (VoIP) 5k, Cowrie (SSH and Telnet) 5k. After that RDPHoneypot 850, Tanner 126, H0neytr4p 109, Adbhoney (Android debug) 74, ConPot 71, Mailoney (email) 68.
- Top usernames: sa by a long way, then root, admin, user, ubuntu, postgres, oracle. Also azureuser, openclaw, jenkins, kafka, elasticsearch.
- Top passwords: 123456, a blank password, password, 123, root, 12345678, P@ssw0rd
- Suricata alerts (top 3): ET EXPLOIT DoublePulsar Backdoor installation communication (5,847), SURICATA STREAM Packet with broken ack (2,871), ET DROP Dshield Block Listed Source group 1 (2,841)
- Suricata CVEs (top 3): CVE-2001-0540 (686), CVE-2002-0013 (53), CVE-2012-0152 (28)

**Screenshots saved:** ../screenshots/aws/day1-honeypot-attack-counts.png, day1-username-password-tagclouds.png, day1-suricata-alerts-top10.png, day1-suricata-cve-top10.png

**Day 1 notes (AWS)**
- Similar total to Azure (28k vs 32k) but a very different mix. Azure was mostly Cowrie (SSH) and AWS is spread across Honeytrap, Dionaea, Sentrypeer and Cowrie.
- Top username on AWS is sa, which is the default admin account for Microsoft SQL Server. Azure's was root. So AWS got a lot more database brute forcing.
- DoublePulsar was 5,847 on AWS vs 1,379 on Azure. That lines up with Dionaea (which runs the fake SMB service) being way busier on AWS.
- Bots tried azureuser on my AWS server. That's Azure's default username, so the bots just use one big list of cloud default usernames everywhere.
- CVE-2001-0540 is exactly 686, the same as the three RDP connection alerts. So it's not really people using a 2001 exploit. Some Suricata rules for normal RDP connections have that CVE tagged on them. A CVE showing up in the table doesn't always mean the exploit was used, you have to check what rule fired.
- New CVEs on AWS like CVE-2018-13379 (Fortinet VPN), CVE-2018-10562 (GPON home routers) and CVE-2021-35394 (Realtek router chips). These are the kind botnets like Mirai use to grab routers and IoT devices. Need to confirm each on NVD.
- 544 alerts were ET INFO Reserved Internal IP Traffic, which is my own VPC traffic again, not attacks.
- Careful comparing these to Azure Day 1. Different dates and Azure was down for about 7 hours. For a fair comparison I need Azure's numbers for the exact same window (22 Oct 10:32 to 23 Oct 10:32).

### Same window comparison 1 (22 Oct 10:32 to 23 Oct 10:32 AEDT)

First proper side by side. Both dashboards set to the exact same 24 hours in Kibana. The screenshots have the window written on top of them.

| | Azure | AWS |
| --- | --- | --- |
| Total attacks | about 114,000 | about 28,000 |
| Sentrypeer (VoIP) | 51k | 5k |
| Cowrie (SSH and Telnet) | 32k | 5k |
| Honeytrap (any other port) | 21k | 8k |
| RDPHoneypot | 4k | 850 |
| Dionaea (SMB and databases) | 3k | 8k |
| Top username | root | sa |
| DoublePulsar alerts | not in the top 10 (under 674) | 5,847 |
| RDP connection request alerts | 1,527 | 686 |
| CVE-2001-0540 count | 1,521 | 686 |

**Screenshots:** ../screenshots/azure/day2-*.png and ../screenshots/aws/day1-*.png

**Notes**
- Azure got about 4 times more attacks in the same 24 hours. Almost half of Azure's was Sentrypeer (51k), so I need to check if that's one noisy scanner. Kibana Discover, type : "Sentrypeer", then look at the top src_ip.
- Azure gets way more RDP (Windows Remote Desktop) traffic. RDP connection requests were 1,527 vs 686 and only Azure had ET HUNTING RDP Authentication Bypass Attempt (674) and ET SCAN MS Terminal Server Traffic on Non-standard Port (1,315) in its top 10. My guess is attackers expect Microsoft's cloud to be full of Windows servers.
- AWS gets more SMB and database attacks. Dionaea is about 3% of Azure's traffic but about 29% of AWS's, AWS had way more DoublePulsar, and its top username is sa (SQL Server).
- The CVE-2001-0540 count matches the GPL INFO MS Remote Desktop Request RDP alert exactly on both clouds (1,521 on Azure, 686 on AWS). So that CVE is really just normal RDP connections being tagged by one rule. Same pattern on both clouds so I'm confident about this one.
- On Azure, enable, system and linuxshell showed up as usernames and passwords, plus GPL TELNET Bad Login (752). These look like Mirai style botnets that log into routers over Telnet and then send those commands to break out into a real Linux shell. So some of this is bots hunting for routers and IoT devices.
- Only one day so far. Need a few more windows before calling anything a real difference.

### Day 3 (24 Oct 2026, Azure)

Kibana range: Azure's first 3 days exactly, 21 Oct 16:21 to 24 Oct 16:21 AEDT. The range is written on top of each screenshot. AWS will get the same 3 day window from its go live (22 Oct 10:32 to 25 Oct 10:32). Elasticsearch was slow to load at first but the VM was healthy.

**Azure**
- Total attacks in the first 3 days: about 301,000 (roughly 100k a day)
- Top honeypots hit: Sentrypeer (VoIP) 151k, Cowrie 63k, Honeytrap 51k, Dionaea 14k, RDPHoneypot 13k, Tanner 4k, Heralding 1k, H0neytr4p 1k, ConPot 846, Redishoneypot 640
- Top countries: United States, Germany, Bangladesh (all almost only port 5060, SIP), Bulgaria (almost only port 22, SSH), China (mostly 5060)
- Top usernames: root, sa, admin, user, enable, system, manager. Still seeing azureuser, openclaw and claude.
- Top passwords: a blank password, 123456, linuxshell, password, 1234, 12345678
- Top attacker networks (AS): 31173 Services AB 46,152, Banatsync Srl 41,240, Google LLC 24,135, CyberForest 13,185 and 10,224, Alsycon 11,302, HostPapa 11,078
- Top source IP: 185.213.155.228 with 44,560 (about 15% of everything)
- Suricata alerts (top 3): ET INFO RDP Response To External Host (11,512), ET INFO RDP connection request (8,424), GPL INFO MS Remote Desktop Request RDP (8,410)
- Suricata CVEs (top 3): CVE-2001-0540 (8,412), CVE-2006-2369 (1,110), CVE-2012-0152 (332)

**Screenshots saved:** ../screenshots/azure/day3-honeypot-attack-counts.png, day3-honeypot-attacks-bar.png, day3-attacks-by-country-port.png, day3-username-password-tagclouds.png, day3-asn-ip-cve-alerts-top10.png

**Day 3 notes**
- Half of everything is Sentrypeer on port 5060 (SIP, the protocol VoIP phones use). Bots scan for phone systems so they can make calls through them for free or for scams.
- One IP, 185.213.155.228, sent 44,560 hits. Its whole network (AS 39351) only has 46,152, so it's basically all that one IP. Need to check in Discover which honeypot it hit, my guess is Sentrypeer.
- Five IPs next to each other (109.160.32.107, .108, .114, .130, .131) each sent almost exactly 6,000 hits. Same network and same amount, so it's probably one operator or botnet spreading the work over several IPs.
- The 8,400ish for CVE-2001-0540 matches the RDP connection alerts again (8,412 vs 8,410 vs 8,424). That's normal RDP connections being tagged with an old CVE, not a 2001 exploit being used.
- ET INFO RDP Response To External Host (11,512) is my RDPHoneypot replying to attackers, not attacks on their own.
- Most attacks come from hosting companies and cloud providers (Google LLC, HostPapa, CyberForest) and scanning companies like Modat. Country tells you where the server is, not who the attacker is.
- New CVEs: CVE-2022-27255 (259), a Realtek router bug in how routers handle SIP. So some of the 5060 traffic is bots trying to hack routers through VoIP. CVE-2006-2369 (1,110) is a RealVNC login bypass and GPL INFO VNC server response (2,744) showed up too, so bots are hunting VNC as well as RDP. Need to confirm both on NVD.
- New honeypot in the top 10: Heralding, which catches logins for things like FTP, POP3, IMAP and VNC.

## Ideas / questions to look into for the write-up

- Always compare Azure and AWS over the exact same time window, not their own Day 1s.
- Why does AWS get so much more SMB and database traffic? Maybe AWS IP ranges are on different scan lists.
- Check which Suricata rules carry CVE tags so I don't count normal traffic as exploit attempts.
- Build a Kibana table of the top payload hashes and decode the top 5.
- Check which honeypot 185.213.155.228 hit, and whether the 109.160.32.x IPs all hit the same thing.
- Compare top attacker networks (AS) between Azure and AWS.
