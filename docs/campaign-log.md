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
- 24 Oct: Noticed Network Out on Azure drop to zero and all containers showing Up 4 hours. Found the cause in the crontab: T-Pot reboots itself every day at 02:45 UTC (13:45 Sydney time). It stops T-Pot, cleans up old Docker containers, images and volumes, then reboots. So there's a gap of a few minutes in the data every day. AWS has the same job but its boot history shows it reboots at about 04:51 UTC (15:51 Sydney), not 02:45 like Azure. So the gaps don't line up exactly, but each one is only a few minutes.
- 4 Nov: SSH timed out on both VMs. My home IP had changed and SSH is locked to my IP, so I updated the source IP on the Azure NSG and AWS security group rules.
- 5 Nov, maintenance check on both VMs:
  - Disk: Azure 31% used (37 GB of 126 GB), AWS 23% (27 GB). Plenty of room for the month.
  - SSH: passwordauthentication no and permitrootlogin without-password on both, so only keys work and root can't log in with a password.
  - ewsposter on AWS had 0 restarts and normal logs. It was only about 30 minutes after AWS's daily reboot though, so need to check again. Checked again on 6 Nov: AWS had 1 restart (fine), but Azure had 52. Azure's logs look normal though (it's sending alerts to the EWS backend and the "already submitted" warnings are just files it already sent), so it crashes now and then and comes straight back. Low priority since it doesn't touch my own data.
  - UDP ports T-Pot exposes on both: 161 (ConPot SNMP, blocked since 27 Oct), 5060 (Sentrypeer SIP), 623 (ConPot IPMI), 69 (Dionaea TFTP) and 5000 (CiscoASA). 623 and 69 are known amplification protocols, so I need to check bytes in vs bytes out on each before deciding to block anything.
  - Data retention: Elasticsearch makes one index per day and T-Pot's ILM policy (tpot) deletes each one after 30 days. My first index (21 Oct UTC) would have been deleted around 4pm on 20 Nov, right before the month summary. Changed the delete age to 60 days on both. Checked again on 6 Nov after the daily reboot and it still says 60d, so it stuck.
  - Found an index called logstash-1970.01.01 with a couple of records in it. That's a broken timestamp (1 Jan 1970 is time zero), harmless.
  - AWS has a logstash-2026.10.21 index even though it went live on 22 Oct. Index names use UTC dates and 22 Oct 10:32 Sydney is still 21 Oct in UTC.
- 6 Nov, outbound traffic alerts so I'd catch another reflection attack while it's happening:
  - Azure: honeypot-outbound-spike emails me if Network Out goes over 8 MB in 5 minutes (normal is a few MB at most, the SNMP flood was 9.5 to 18 MB). Also added honeypot-cpu-maxed (CPU average over 95%, a classic sign of a crypto miner) and Azure has a default VM Availability alert.
  - The Network Out Total metric kept failing with InternalServerError, from the portal and the CLI, even with skipmetricvalidation (Activity ID c106f741-5775-43aa-b9dd-f8bd3903bec2). Microsoft.Insights was registered and a CPU alert worked fine, so it was only that metric. The older Network Out metric worked, and it's arguably better since it only counts traffic going out to the internet.
  - AWS: CloudWatch alarm honeypot-outbound-spike on NetworkOut (Sum over 5 minutes) over 5 MB, emailing me through an SNS topic. AWS's normal peaks over the week were about 3.3 MB. Left all EC2 actions off so an alarm can't stop the instance.
- 6 Nov: Switching from check ins every few days to weekly reports. Each week runs Thursday 10:32 to Thursday 10:32 (from when AWS went live) and both clouds use the exact same window, so the comparisons are fair. The earlier check ins stay as they are. Running totals (days 1 to N) are dropped since they're just the windows added up, and the month summary covers that at the end.
- 7 Nov: Credit check. Azure has spent $29.47 and AWS $20.09 so far, about 17 days in. That's roughly $1.70 a day on Azure and $1.25 a day on AWS, so the full month should come to about $55 on Azure and $40 on AWS. Both are well inside the credit, so there's no risk of the VMs being shut off before 21 Nov.

## Check-ins

Copy this block for every check in. From week 3 these are weekly reports with the same window on both clouds.

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

### Day 3 (25 Oct 2026, AWS)

Kibana range: AWS's first 3 days exactly, 22 Oct 10:32 to 25 Oct 10:33 AEDT. The range is written on top of each screenshot.

**AWS**
- Total attacks in the first 3 days: about 82,000
- Top honeypots hit: Sentrypeer 32k, Honeytrap 21k, Dionaea 15k, Cowrie 7k, RDPHoneypot 3k, Heralding 1k, Tanner 843, ConPot 518, H0neytr4p 431, Ciscoasa 208
- Top countries: United States (almost all 5060), China (mostly 445, SMB), Germany (5060 and 22), The Netherlands (mostly 5060), Indonesia (almost all 445)
- Top usernames: sa, admin, root. Also socks, socks5, proxy, 1080, anonymous, azureuser, openclaw
- Top passwords: a blank password, 123456, password, proxy, socks
- Top attacker networks (AS): HostPapa 21,963, Banatsync Srl 4,789, Google LLC 4,751, China Mobile 3,786, Chinanet 3,635, Telekomunikasi Indonesia 3,145, ONYPHE 3,082, Microsoft Corporation 3,058
- Top source IP: 23.94.174.242 with 21,962
- Suricata alerts (top 3): ET EXPLOIT DoublePulsar Backdoor installation communication (12,090), ET DROP Dshield Block Listed Source group 1 (8,188), SURICATA STREAM Packet with broken ack (6,120)
- Suricata CVEs (top 3): CVE-2001-0540 (2,603), CVE-2022-27255 (574), CVE-2002-0013 (210)

**Screenshots saved:** ../screenshots/aws/day3-honeypot-attack-counts.png, day3-username-password-tagclouds.png, day3-asn-ip-cve-alerts-top10.png, day3-attacks-by-country-port.png, day3-ip-reputation-and-honeypot-share.png

**Day 3 notes (AWS)**
- HostPapa's whole count (21,963) is basically one IP, 23.94.174.242 (21,962). Same thing as the big VPN IP on Azure. One noisy source can make a whole network look bad.
- 23.94.174.242 is also in Azure's top 10 (11,064). So the same attacker is hitting both clouds. 77.239.124.x shows up on both too (77.239.124.44 on Azure, 77.239.124.205 on AWS).
- Three IPs (115.150.176.231, 182.9.1.55, 39.155.8.176) each sent about 3,145. Different countries but the same amount, so probably one botnet splitting the work.
- Lots of proxy hunting. Usernames like socks, socks5, proxy and 1080 (the SOCKS port) plus GPL INFO SOCKS Proxy attempt (1,346). Bots want an open proxy to hide behind, same idea as the CONNECT request on AWS day 1.
- Some usernames are really other protocols, like GET / HTTP/1.1 and OPTIONS rtsp://example.com RTSP/1.0 (RTSP is what IP cameras use). Bots throw web and camera requests at any open port and the honeypot logs whatever comes in as a login.
- One password was a web request header with my own AWS IP in it (a bot sent HTTP to my Telnet port). I covered it in the screenshot before committing.
- Microsoft Corporation (3,058) is in AWS's top attacker networks. So attacks come from inside Azure as well, attackers rent cloud servers like anyone else.
- The CVE-2001-0540 count (2,603) matches the RDP alerts (2,603) again. Third time this pattern held.

### Comparison: first 3 days on each cloud

Azure: 21 Oct 16:21 to 24 Oct 16:21. AWS: 22 Oct 10:32 to 25 Oct 10:33. Not the exact same hours (they overlap for about 54 of the 72) but both are each cloud's first 3 days.

| | Azure | AWS |
| --- | --- | --- |
| Total attacks | about 301,000 | about 82,000 |
| Sentrypeer (VoIP) | 151k (50%) | 32k (39%) |
| Cowrie (SSH and Telnet) | 63k (21%) | 7k (9%) |
| Honeytrap (any other port) | 51k (17%) | 21k (26%) |
| Dionaea (SMB and databases) | 14k (5%) | 15k (18%) |
| RDPHoneypot | 13k (4%) | 3k (4%) |
| Top username | root and sa | sa and admin |
| Top source IP | 185.213.155.228 (44,560, a VPN in Germany) | 23.94.174.242 (21,962, HostPapa) |
| DoublePulsar alerts | 6,340 | 12,090 |
| RDP connection request alerts | 8,424 | 2,603 |
| CVE-2022-27255 (Realtek SIP) | 259 | 574 |

**What it shows so far**
- Azure got about 3.7 times more attacks. A lot of that is one VPN IP doing SIP scanning, but Azure is still ahead without it.
- Azure is SSH heavy. Cowrie is 21% of Azure's traffic but only 9% of AWS's.
- AWS is SMB heavy. Dionaea is 18% of AWS's traffic but only 5% of Azure's, and AWS had about double the DoublePulsar alerts even with way less traffic overall.
- RDP is about the same share on both (4%). Azure just gets more of everything. So my day 2 idea that Azure gets more RDP is really about volume, not share.
- Some attackers hit both clouds (23.94.174.242 and the 77.239.124.x range).

### Days 3 to 6 (27 Oct 2026, Azure)

Two Kibana ranges this time, both written on top of the screenshots:
- Days 3 to 6: 23 Oct 16:32 to 27 Oct 16:33 AEDT
- Days 1 to 6 (everything so far): 20 Oct 16:30 to 27 Oct 16:33 AEDT

**Azure, days 3 to 6**
- Total attacks: about 575,000
- Top honeypots hit: Sentrypeer 207k, Honeytrap 129k, Cowrie 119k, ConPot 42k, Dionaea 39k, RDPHoneypot 29k
- Top source IPs: 185.136.15.17 (62,978), 172.110.223.9 (31,904), 141.153.237.4 (25,178), 185.213.155.228 (20,290), 23.94.174.242 (15,615)
- Top attacker networks (AS): Banatsync Srl 87,855, AO ALMAZ 74,659, Husam A. H. Hijazi 42,002, Google LLC 33,388, Verizon Business 25,302
- Suricata alerts (top 3): GPL SNMP request udp (510,437), GPL SNMP public access udp (510,379), SURICATA STREAM Packet with invalid timestamp (63,005)
- Suricata CVEs (top 3): CVE-2002-0013 (1,020,869), CVE-2001-0540 (24,543), CVE-2005-4050 (1,952)

**Azure, days 1 to 6**
- Total attacks: about 754,000
- Top honeypots hit: Sentrypeer 290k, Honeytrap 165k, Cowrie 163k, Dionaea 45k, ConPot 43k, RDPHoneypot 34k
- Top countries: United States, Bulgaria, Germany, Kazakhstan, Philippines
- P0f (passive OS fingerprinting) says almost every attacking machine runs Linux, very little Windows. Makes sense since bots run on rented Linux servers and hacked Linux IoT devices.

**Screenshots saved:** ../screenshots/azure/days3-6-*.png, ../screenshots/azure/days1-6-*.png, ../screenshots/investigations/*.png

**Notes**
- ConPot jumped from 846 in the first 3 days to 42k in days 3 to 6. That led to investigation 7.
- CVE-2002-0013 (1,020,869) is almost exactly the two SNMP alerts added together (510,437 + 510,379). Fourth time a CVE tag has turned out to sit on a normal rule.
- SURICATA STREAM Packet with invalid timestamp (63,005) almost matches 185.136.15.17's count (62,978). That led to investigation 6.
- 185.213.155.228 (the VPN toll fraud IP) isn't still going. Its histogram only shows 23 Oct to early 24 Oct and its 6 day total (44,560) is the same as its 3 day total. A total doesn't tell you if something is ongoing, the histogram does.

### Days 3 to 6 and days 1 to 6 (28 Oct 2026, AWS)

Two Kibana ranges, written on top of the screenshots:
- Days 3 to 6: 24 Oct 10:20 to 28 Oct 10:35 AEDT
- Days 1 to 6 (everything so far): 22 Oct 10:20 to 28 Oct 10:30 AEDT

**AWS, days 3 to 6**
- Total attacks: about 248,000
- Top honeypots hit: Honeytrap 98k, Sentrypeer 70k, Dionaea 52k, RDPHoneypot 18k, Cowrie 5k, Tanner 2k, ConPot 710, Mailoney 692, H0neytr4p 541, Heralding 372
- That's about 62k a day, more than double AWS's first days (about 28k a day). The port scan alone added about 63k.

**AWS, days 1 to 6**
- Total attacks: about 306,000
- Top honeypots hit: Honeytrap 112k, Sentrypeer 90k, Dionaea 67k, RDPHoneypot 19k, Cowrie 11k, Tanner 2k, Heralding 1k, ConPot 919
- Top source IPs: 185.136.15.17 (62,979), 172.110.223.9 (31,900), 23.94.174.242 (23,592), 138.226.239.44 (13,510), 172.110.223.49 (11,956), 185.136.15.27 (11,470), 213.55.92.81 (11,053)
- Top attacker networks (AS): AO ALMAZ 74,454, Husam A. H. Hijazi 43,900, HostPapa 23,605, Microsoft Corporation 17,426, Digital Network 13,510
- Suricata alerts (top 3): SURICATA STREAM Packet with invalid timestamp (63,007), ET EXPLOIT DoublePulsar Backdoor installation communication (27,237), ET INFO RDP Response To External Host (17,516)
- Suricata CVEs (top 3): CVE-2001-0540 (17,455), CVE-2022-27255 (741), CVE-2002-0013 (433)
- Top usernames: sa way ahead, then admin, root. Also distributor_admin, sqlserver and erp, so lots of database and business software guessing.

**Screenshots saved:** ../screenshots/aws/days3-6-*.png, ../screenshots/aws/days1-6-*.png, ../screenshots/investigations/aws-portscan-*.png

**Notes**
- AWS's ConPot only logged 919 in 6 days and no SNMP alerts made the top 10. So the SNMP reflection only hit Azure, AWS wasn't used.
- AWS is still SMB and database heavy. Dionaea is 67k (22%) and sa is the top username. ET SCAN Suspicious inbound to MSSQL port 1433 (5,679) shows up too.
- P0f labels a big slice of AWS traffic as Solaris 6, an operating system from the 1990s. More likely odd scanner packets matching an old signature than real Solaris machines.
- Several of the same scanners hit both clouds with almost the exact same counts. See investigation 8.

### Days 6 to 12 (4 Nov 2026, Azure)

Missed the 30 Oct and 2 Nov check ins so this one covers 6 days in one go. Kibana range: 27 Oct 16:21 to 2 Nov 16:21 AEDT, written on top of the screenshots.

**Azure, days 6 to 12**
- Total attacks: about 1,000,000. That's about 167k a day, up from about 144k a day in days 3 to 6 (and that included the SNMP flood).
- Top honeypots hit: Cowrie 378k, Sentrypeer 279k, RDPHoneypot 160k, Honeytrap 128k, Dionaea 27k, Tanner 14k, H0neytr4p 10k, ConPot 4k, Mailoney 926, Redishoneypot 785
- Top destination ports: 5060 (SIP), 3389 (RDP), 22 (SSH), 80 (HTTP), 445 (SMB)
- Top countries: United States, the Netherlands, Bulgaria, China, Hong Kong, Bangladesh, Germany, South Korea, United Kingdom, France
- Attacker IP reputation: mostly bad reputation, sip abuse and ssh abuse
- P0f still says almost everything is Linux, with a small slice of Windows
- Top usernames: root way ahead, then admin, Administrator, sa, ubuntu, user and (blank). Also claude and openclaw again, plus minecraft, steam, hadoop, oracle, postgres and jenkins.
- Top passwords: (blank) way ahead, then 123456, admin, Passw0rd, 1qaz@WSX, ubuntu
- I didn't grab the top IP and CVE table this time. The IPs below are the ones I looked into from the histograms and Discover.

**Screenshots saved:** ../screenshots/azure/days6-12-*.png, ../screenshots/investigations/azure-*.png, ../screenshots/investigations/port-*-lookup.png

**Notes**
- Cowrie (SSH and Telnet) went from 119k in days 3 to 6 to 378k and is now the top honeypot. ConPot dropped from 42k to 4k because UDP 161 is blocked now, so the SNMP flood can't come back.
- The attacks histogram sits around 15k per 3 hours most of the time, with two big spikes, one on 29 Oct and a bigger one on 2 Nov. The unique source IPs line stays flat through both spikes, so they came from a handful of IPs, not more attackers.
- The 29 Oct spike is Sentrypeer on port 5060 (investigation 10) and the 2 Nov spike is RDPHoneypot on port 3389 (investigation 9). The Suricata alert categories jump to 60k to 80k per bar on 2 Nov, mostly Misc activity, which matches the RDP flood.
- 345gs5662d34 shows up as both a username and a password. It's a weird string to guess, so it looks like one botnet's hardcoded login. Worth searching later.
- Lots of IPs go to specific ports, mainly 5060. That's not the honeypot steering them. Each honeypot only listens on its own ports (Sentrypeer on 5060, RDPHoneypot on 3389, Cowrie on 22), so an IP hitting 5060 chose to look for phone systems. Honeytrap is the only one that answers on random ports.
- I looked up 3389 and 5060 on a port database and both have long lists of known vulnerabilities. That list is everything ever found in any software that uses the port, not proof these IPs used any of it. To see if an exploit was actually tried I need the Suricata CVE panel or the honeypot logs, not the port.

### Days 6 to 12 (5 Nov 2026, AWS)

Kibana range: 28 Oct 10:32 to 3 Nov 10:32 AEDT, written on top of the screenshots.

**AWS, days 6 to 12**
- Total attacks: about 245,000. That's about 41k a day, down from about 62k a day in days 3 to 6 (which had the big port scans).
- Top honeypots hit: Honeytrap 93k, Sentrypeer 56k, Cowrie 47k, Dionaea 23k, RDPHoneypot 16k, Heralding 3k, H0neytr4p 2k, Tanner 2k, Mailoney 780, ConPot 676
- Top destination ports: 5060 (SIP), 445 (SMB), 3389 (RDP), 22 (SSH), 1433 (MSSQL)
- Top source IPs: 77.83.240.70 (31,169), 1.4.249.123 (21,072), 82.165.34.39 (19,702), 192.210.236.135 (14,595), 185.8.104.235 (12,835), 109.123.227.33 (11,892), 1.2.224.172 (10,835), 194.50.16.198 (9,903)
- Top attacker networks (AS): Alsycon B.V. 44,581, TOT Public Company 31,921, IONOS SE 26,495, UAB Cherry Servers 20,582, HostPapa 14,711, Contabo Asia 11,893
- Top countries: United States, Germany, Thailand, Lithuania, the Netherlands, Australia, France, Ireland, United Kingdom, Seychelles
- Suricata alerts (top 5): SURICATA STREAM RST recv but no session (36,445), ET INFO User-Agent (python-requests) Inbound to Webserver (35,958), SURICATA STREAM FIN recv but no session (35,630), ET EXPLOIT DoublePulsar Backdoor installation communication (32,933), ET INFO RDP Response To External Host (15,441)
- Suricata CVEs (top 4): CVE-2001-0540 (15,369), CVE-2018-13379 (2,696), CVE-2012-0152 (152), CVE-2025-30208 (63)
- P0f says Solaris 6 is the biggest slice again, then Linux

**Screenshots saved:** ../screenshots/aws/days6-12-*.png, ../screenshots/investigations/aws-*.png

**Azure vs AWS, days 6 to 12**
- Azure got about 1,000,000 and AWS about 245,000, so about 4 times more on Azure. Close to the 3.7 times from the first 3 days.
- The windows aren't exactly the same (AWS starts and ends about 18 hours later) but they overlap for over 5 days, so the gap is real.
- Both had 5060, 3389 and 22 in their top ports. But AWS still has SMB (445) and MSSQL (1433) in its top 5, DoublePulsar is still a top alert, and sa is still a top username. Azure's top 5 had 80 instead of 1433 and Cowrie is way bigger there.
- Honeytrap is AWS's top honeypot and Cowrie is Azure's. So Azure gets more SSH guessing and AWS gets more scanning of random ports.
- Alsycon B.V. hit both. 194.50.16.198 did the service scan on Azure (investigation 11) and also sent 9,903 to AWS, and another Alsycon IP was AWS's top attacker (investigation 12).

**Notes**
- CVE-2001-0540 (15,369) is exactly ET INFO RDP connection request (15,369) again. Same as on day 2.
- CVE-2018-13379 (2,696) is a real exploit tag. It's a Fortinet VPN bug where attackers read files like saved VPN logins off the device. So bots are hunting for unpatched Fortinet VPNs. CVE-2025-30208 (63) is a recent one for the Vite web dev server, so bots are already trying new bugs. I should check the payloads before saying these were real attempts.
- The python-requests alert (35,958) means a lot of the web traffic is Python scripts, not real browsers.
- The username and password clouds have HTTP headers in them like GET / HTTP/1.1, Accept: */*, User-Agent: python-requests and Connection: keep-alive. That's bots sending web requests to the Telnet port and Cowrie logging each line as a login attempt. One of the "passwords" had my AWS public IP in a Host header, so I covered it in the screenshot.
- VoIP usernames are showing up too: asterisk, asteriskuser, kamailio, opensips and a2billinguser. Those are all VoIP server software, so the bots that scan 5060 also try to log into the servers behind it.
- 1.4.249.123 and 1.2.224.172 are both TOT (a Thai telecom). That's why Thailand is number 3 this week.
- UAB Cherry Servers (Lithuania) has 20,582 but none of its IPs is in the top 10, so it came from lots of IPs.

## Weekly reports

From here every report uses the exact same window on both clouds. Azure's windows start at 10:30 instead of 10:32, 2 extra minutes out of a week, which doesn't change anything.

| | Week 1 | Week 2 |
| --- | --- | --- |
| Window | 22 Oct 10:32 to 29 Oct 10:32 | 29 Oct 10:32 to 5 Nov 10:32 |
| Azure attacks | about 982,000 | about 1.8 million |
| AWS attacks | about 336,000 | about 656,000 |
| Azure vs AWS | 2.9 times more | 2.7 times more |
| Azure top honeypot | Sentrypeer (393k) | Honeytrap (809k) |
| AWS top honeypot | Honeytrap (123k) | Honeytrap (355k) |
| Azure top network | Banatsync Srl (162,365) | Alsycon B.V. (669,382) |
| AWS top network | AO ALMAZ (75,660) | Alsycon B.V. (319,992) |

Kibana shows Azure week 2 as "2m" but the top 10 honeypots add up to about 1.79 million, so about 1.8 million is closer.

### Week 1 (22 Oct 10:32 to 29 Oct 10:32)

**Azure**
- Total attacks: about 982,000
- Top honeypots: Sentrypeer 393k, Cowrie 229k, Honeytrap 195k, RDPHoneypot 54k, Dionaea 50k, ConPot 45k, Tanner 6k, H0neytr4p 4k, Heralding 3k, Redishoneypot 943
- Top ports: 5060, 3389, 161, 22, 1433
- Top countries: United States, Bulgaria, Germany, Kazakhstan, Bangladesh, Philippines, United Kingdom, France, Hong Kong, the Netherlands
- Top source IPs: 185.136.15.17 (62,978), 185.213.155.228 (44,560), 172.110.223.9 (31,904), 141.153.237.4 (25,178), 91.110.167.131 (22,598), 23.94.174.242 (18,510), 82.165.34.39 (15,686), 8.212.14.105 (13,625)
- Top networks: Banatsync Srl 162,365, AO ALMAZ 74,794, Google 58,447, 31173 Services AB 46,947, Husam A. H. Hijazi 43,924
- Suricata alerts: GPL SNMP request udp (513,720) and GPL SNMP public access udp (513,645) way ahead because of the SNMP reflection, then invalid timestamp (63,031) from the port scan, then RDP
- Top CVEs: CVE-2002-0013 (1,027,453, the SNMP alerts), CVE-2001-0540 (39,665, the RDP alerts), CVE-2005-4050 (1,952), CVE-2006-2369 (1,637), CVE-2022-27255 (741)
- Top usernames: root, sa, admin, Administrator. Passwords: (blank), 123456, 123

**AWS**
- Total attacks: about 336,000
- Top honeypots: Honeytrap 123k, Sentrypeer 98k, Dionaea 72k, RDPHoneypot 20k, Cowrie 14k, Tanner 3k, Heralding 2k, H0neytr4p 1k, ConPot 1k, Mailoney 937
- Top ports: 5060, 445, 1433, 3389, 23
- Top countries: United States, Kazakhstan, Philippines, Germany, United Kingdom, Ethiopia, China, the Netherlands, France, Belgium
- Top source IPs: 185.136.15.17 (62,979), 172.110.223.9 (31,900), 23.94.174.242 (23,592), 138.226.239.44 (15,153), 185.136.15.27 (12,676), 172.110.223.49 (11,956), 82.165.34.39 (11,598), 213.55.92.81 (11,053)
- Top networks: AO ALMAZ 75,660, Husam A. H. Hijazi 43,919, HostPapa 28,040, Microsoft 19,939, IONOS 18,232
- Suricata alerts: invalid timestamp (63,008) from the port scan, DoublePulsar (36,431), DShield block list (18,661), RDP, MSSQL port 1433 (6,455)
- Top CVEs: CVE-2001-0540 (17,860), CVE-2022-27255 (741), CVE-2002-0013 (445)
- Top usernames: sa is the biggest, then root and admin. P0f says Solaris 6 again.

**Comparing week 1**
- Azure got 2.9 times more attacks than AWS over the exact same 7 days.
- Azure is SSH and VoIP heavy. Cowrie is 23% of Azure's traffic and only 4% of AWS's.
- AWS is SMB and database heavy. Dionaea is 21% of AWS's traffic and only 5% of Azure's, sa is AWS's top username and AWS had 2.6 times more DoublePulsar alerts (36,431 vs 13,850).
- The internet wide scanners hit both clouds the same: 185.136.15.17 (62,978 vs 62,979), 172.110.223.9 (31,904 vs 31,900), AO ALMAZ (74,794 vs 75,660) and Husam A. H. Hijazi (43,924 vs 43,919).
- CVE-2022-27255 had exactly 741 on both. It's a Realtek router bug triggered through SIP on port 5060, so one scanner checked both clouds the same way.
- What made Azure bigger was stuff that only hit Azure: Banatsync Srl (162,365, from lots of IPs since none of its IPs are in the top 10), the toll fraud VPN (185.213.155.228), the SNMP reflection (141.153.237.4) and the SIP pair (91.110.167.131 and 8.212.14.105).

**Screenshots:** ../screenshots/azure/week1-*.png and ../screenshots/aws/week1-*.png

### Week 2 (29 Oct 10:32 to 5 Nov 10:32)

**Azure**
- Total attacks: about 1.8 million
- Top honeypots: Honeytrap 809k, Cowrie 428k, Sentrypeer 258k, RDPHoneypot 199k, Dionaea 33k, Heralding 31k, Tanner 17k, H0neytr4p 11k, ConPot 3k, Mailoney 2k
- Top ports: 5060, 3389, 22, 5900, 80
- Top countries: the Netherlands, United States, Bulgaria, China, Germany, Bangladesh, United Kingdom, France, South Korea, Hong Kong
- Top source IPs: 194.50.16.198 (458,403), 77.83.240.70 (208,343), 136.144.199.59 (101,821), 216.250.250.134 (86,861), 115.191.15.226 (48,470), 91.110.167.131 (36,032), 185.8.104.235 (20,075), 8.212.14.105 (18,406)
- Top networks: Alsycon B.V. 669,382, Banatsync Srl 150,114, Signet B.V. 101,821, Majestic Hosting Solutions 88,840, Google 51,019, Beijing Volcano Engine 49,417
- Suricata alerts: python-requests (649,739), RDP (about 148k each), VNC server response (66,729), VNC Server Not Requiring Authentication (30,995), VNC Authentication Failure (30,587)
- Top CVEs: CVE-2001-0540 (145,297), CVE-2006-2369 (31,015), CVE-2018-13379 (22,170), CVE-2024-4577 (1,216), CVE-2021-44228 (110)

**AWS**
- Total attacks: about 656,000
- Top honeypots: Honeytrap 355k, Sentrypeer 185k, Cowrie 49k, Dionaea 32k, RDPHoneypot 18k, Heralding 4k, H0neytr4p 3k, Tanner 3k, Mailoney 2k, Miniprint 2k
- Top ports: 5060, 445, 3389, 22, 1080
- Top countries: the Netherlands, Germany, United States, Thailand, Lithuania, Australia, France, Vietnam, China, Indonesia
- Top source IPs: 194.50.16.198 (218,261), 185.213.155.236 (114,661), 77.83.240.70 (98,378), 1.4.249.123 (21,072), 82.165.34.39 (20,714), 185.8.104.235 (20,407)
- Top networks: Alsycon B.V. 319,992, 31173 Services AB 114,952, IONOS 32,311, TOT 31,921, UAB Cherry Servers 28,315
- Suricata alerts: python-requests (308,730), STREAM RST and FIN with no session (about 36k each), DoublePulsar (25,308), RDP, VNC server response (10,560)
- Top CVEs: CVE-2001-0540 (16,608), CVE-2018-13379 (5,500), CVE-2012-0152 (186), CVE-2025-30208 (63)
- The username cloud has VoIP web panel paths like GET /_asterisk/graph.php and GET /recordings/misc/graph.php, and OPTIONS rtsp://example.com (camera streams). Bots sending web requests to the Telnet port again, this time looking for FreePBX and cameras.

**Comparing week 2**
- Azure got 2.7 times more than AWS, close to week 1.
- Alsycon B.V. was the top network on both. 194.50.16.198 and 77.83.240.70 hit BOTH clouds (458k and 208k on Azure, 218k and 98k on AWS). Together that's about 37% of Azure's week and 49% of AWS's. Both IPs hit Azure about twice as hard as AWS.
- Without Alsycon, Azure would be about 1.13 million and AWS about 336,000, so still about 3.4 times more on Azure.
- Honeytrap became the top honeypot on both because the Alsycon scans hit lots of random ports, which is what Honeytrap catches.
- The toll fraud VPN moved clouds. In week 1 31173 Services AB was on Azure (185.213.155.228, 44,560). In week 2 it was on AWS (185.213.155.236, 114,661) and not in Azure's top 10.
- The SIP pair (91.110.167.131 and 8.212.14.105) only hit Azure, in both weeks.
- New on Azure: a VNC wave. Port 5900 is in the top 5, Heralding (which handles VNC) went from 3k to 31k, and there were about 67k VNC alerts. CVE-2006-2369 (31,015) matches the "VNC Server Not Requiring Authentication" alert (30,995), so it's that rule's CVE tag again, not proof of an exploit. Need to find who it was.
- New exploit tags on Azure: CVE-2024-4577 (1,216), a PHP bug on Windows servers from 2024, and CVE-2021-44228 (110), which is Log4Shell. CVE-2018-13379 (Fortinet VPN) went up to 22,170 on Azure and 5,500 on AWS.
- Port 1080 (SOCKS proxy) is in AWS's top 5, so something was checking for open proxies.
- The password clouds had my own public IP in a Host header again (AWS in both weeks, Azure in week 2), so I covered them.

**Screenshots:** ../screenshots/azure/week2-*.png and ../screenshots/aws/week2-*.png

## Investigations

### 1. Azure Network Out dropped to zero (24 Oct)
- Saw Network Out drop to zero in Azure Monitor and every container showing Up 4 hours.
- uptime showed the VM booted about 4 hours earlier. The crontab had an entry called T-Pot Daily Reboot set for 02:45 UTC every day.
- It stops T-Pot, prunes old Docker containers, images and volumes, then reboots. Attack data lives in ~/tpotce/data so nothing is lost, just a few minutes gap every day.
- Result: not a problem, it's T-Pot's daily refresh.

### 2. Who is 185.213.155.228? (Azure's top attacker)
- Cisco Talos says it's in Frankfurt, owned by 31173 Services AB, category Personal VPN. Reputation Neutral and not on any blocklist.
- So the real attacker is hiding behind a VPN. Blocklists don't list VPN exits because thousands of normal people share them, so reputation tools miss this kind of thing.
- In Kibana all its traffic went to Sentrypeer on port 5060. The called_number field showed the same North American number with different prefixes in front (9, 00, 000). That's a toll fraud scan, it's trying to find which dial out prefix my fake phone system accepts so it can push calls to premium numbers.
- One Suricata flow record lasted 8 hours 27 minutes with 29,953 SIP requests, about one a second, all from the same source port. Every request got a reply so the bot kept going.
- The flow started at 02:49:55 UTC, about 3 minutes after Azure's daily reboot. So it was hitting my IP the whole time and came straight back when the honeypot came back up.
- flow.alerted was false. Suricata didn't raise a single alert for 8 hours of toll fraud scanning. The honeypot is the only reason I saw it. Writing a Suricata rule for this could be my first custom detection in phase 2.

### 3. AWS looked like it was missing data (25 Oct)
- AWS's dashboard showed only 23k for 3 days even though day 1 alone was 28k.
- Checked the VM before blaming the data. uptime, journalctl --list-boots, memory, disk and container status were all fine and there were no out of memory messages.
- The boot history showed AWS reboots every day at about 04:51 UTC (15:51 Sydney), not 02:45 like Azure.
- In the end the Kibana time range was wrong. With the right absolute range it showed 82k. Lesson: check the time picker first on every check in.

### 4. ewsposter keeps restarting on AWS
- docker inspect showed ewsposter had restarted 32 times since that morning's reboot. It's the container that sends my data to T-Pot's community threat map.
- It doesn't store my own data so it didn't cause anything to go missing, but it's broken. Need to check docker logs --tail 20 ewsposter.

### 5. What was 23.94.174.242 doing? (top attacker on AWS, also in Azure's top 10)
- On AWS all its traffic went to Sentrypeer on port 5060, same as the VPN IP on Azure.
- The called_number was 461, a short internal extension number, not an outside phone number. So it was guessing extension numbers to find real phone accounts on my fake phone system.
- That's a different step to the VPN IP. VoIP attacks go: find extensions, crack their passwords, then use them to make fraud calls. This IP was doing step 1 and the VPN IP on Azure was testing dial out prefixes for step 3.

### 6. 185.136.15.17, a full port scan from Kazakhstan (25 to 26 Oct)
- Top source IP for days 3 to 6 with 62,978 hits. Network AO ALMAZ (AS 210328, Kazakhstan). Its neighbour 185.136.15.31 sent another 11,470.
- Almost all of it happened in one burst, about 25 Oct 18:00 to 26 Oct 03:00, 783,539 log entries in about 9 hours.
- I first thought it was hammering port 45636, but every row was a different port (45636, 63496, 7676, 9301, 6906, 57792, 9383). 45636 is just a random high (ephemeral) port, nothing runs there.
- Field statistics on DestPort: the busiest single port was 3389 at only 0.4%, and 98.4% was Other. The distribution covers 0 to 65535 and the average port was about 32,400, right in the middle of the range. That's what random ports across the whole range look like.
- So it's a full range port scan looking for services hidden on unusual ports. Honeytrap answers on every port nothing else uses, so each port it touched counted as an attack.
- Its count matches SURICATA STREAM Packet with invalid timestamp (63,005). Fast scanners like masscan build their own packets to go quicker and that often leaves odd TCP fields, which is why these flows show flow.alerted true.

### 7. My honeypot was being used in an SNMP reflection DDoS (26 to 27 Oct)
**What I noticed**
- ConPot went from 846 to 42k and the top Suricata alerts became GPL SNMP request udp and GPL SNMP public access udp, over 510,000 each.
- SNMP (UDP port 161) is how admins monitor routers, printers and industrial gear. ConPot pretends to be industrial equipment so it answers SNMP.

**Who it was**
- In Lens, the top sources for GPL SNMP request udp were 141.153.237.4 (43%), 46.128.202.246 (8%), 86.128.148.49 (7%), 83.81.66.128 (6%), 89.81.25.16 (5%). It started small on 25 Oct, exploded around 26 Oct 03:00 and kept going.
- These look like home and business internet connections in different countries, not attacker servers. That's what made me suspect spoofing.
- Side lesson: the Discover sidebar said 89.81.25.16 was 38% but Lens said 4.5%. The sidebar only uses a sample of recent records, Lens counts everything.

**Proving it with 141.153.237.4 (Verizon Business, New Jersey)**
- 687,427 records, 100% UDP, and the median destination port was 161. No TCP at all. TCP needs a real handshake so it can't be faked, UDP can.
- One Suricata flow ran from 26 Oct 13:52 to 27 Oct 08:32, about 18.7 hours.
- In: 167,560 packets and 12.7 MB, about 76 bytes each. Out: 25,189 packets and 26.4 MB, about 1,050 bytes each. Each reply was about 14 times bigger than a request, and my VM sent twice as many bytes out as came in.
- Azure Network Out was climbing at the time (9.5 MB then 18 MB in single intervals) on top of its normal baseline.

**What was happening**
- Attackers send small SNMP requests to my honeypot with the source faked as the victim (141.153.237.4 and the others). ConPot sends its much bigger replies to the victim. The attacker does this with lots of exposed devices and the victim gets flooded, while the attacker never shows up in any log. This is a reflection or amplification DDoS.
- So the IPs in my SNMP charts are most likely the victims, not the attackers.

**What I did**
- Blocked UDP 161 on both clouds so the comparison stays fair and everything else keeps collecting.
- AWS, 27 Oct 18:06 AEDT: split the security group's UDP 1 to 64000 rule into UDP 1 to 160 and UDP 162 to 64000, since security groups can't deny.
- Azure, 27 Oct 18:11 AEDT: added NSG rule Deny-SNMP-reflection, UDP 161, Deny, priority 105 (above Allow-Honeypots at 110).

**Result**
- Looking at GPL SNMP request udp from 16:30 to 18:25 on 27 Oct, it was about 50 alerts a minute until about 17:34, then dropped to almost nothing at 17:35 with one stray at 17:52 and none after 18:00.
- Azure Network Out had its big spike and then went back to baseline around the same time.
- So the flood stopped by itself about 35 minutes before I blocked it. The attackers moved on. My block didn't stop this attack but it means it can't happen again since UDP 161 is now closed on both clouds.
- Screenshots: ../screenshots/investigations/snmp-alerts-27oct-1630-to-1825.png and snmp-reflection-azure-network-out-after.png

**Lesson**
- A honeypot can be turned into a weapon against other people if you're not watching it. Running one responsibly means watching outbound traffic too, not just what comes in.
- Suricata flagged the SNMP requests but only as Informational, and nothing alerted on my own VM's outbound replies. A rule for "my VM is suddenly sending lots of outbound UDP" would be a good detection to write in phase 2.

### 8. The same port scanner hit AWS, with the same count (27 Oct)
**What I noticed**
- AWS's dashboard filtered to Honeytrap (the honeypot that answers on every port nothing else uses) had 112k, with a big spike on 27 Oct.
- Attacks by country showed Kazakhstan on top, and the top IP was 185.136.15.17, the same one that port scanned Azure on 25 to 26 Oct.

**What I checked**
- In Discover, 185.136.15.17 had 512,975 log entries between 27 Oct 03:00 and 06:00, with a pause from about 03:15 to 04:10.
- Every row was a different destination port (48953, 24011, 61068, 29351, 37918, 4292, 32982).
- The median destination port in every 5 minute slot was about 30k to 35k, so the ports were spread over the whole range the whole time, not just on average.
- I first wondered if it was another SNMP reflection like Azure, but it wasn't. The traffic was TCP flows on random ports with replies both ways, not UDP to port 161.
- P0f couldn't fingerprint its OS. Fast scanners like masscan build their own packets, so they don't look like any real operating system. Same reason Suricata flagged SURICATA STREAM Packet with invalid timestamp.

**The big finding**
- It sent almost exactly the same number of probes to both clouds. 62,978 to Azure and 62,979 to AWS, with the same packet pattern every time (5 packets in, 3 out, 385 and 207 bytes, 11 second flows).
- My firewalls let TCP in on ports 1 to 64000. Take out the ports the other honeypots use and Honeytrap has roughly 63,000 left. So it hit every open port exactly once on each cloud, about 1.5 days apart.
- Other scanners did the same thing. 172.110.223.9 sent 31,904 to Azure and 31,900 to AWS. 185.136.15.31 (Azure) and 185.136.15.27 (AWS) both sent exactly 11,470. 138.226.239.47 (Azure) and 138.226.239.44 (AWS) sent 13,511 and 13,510. Some switch to a neighbouring IP but keep the same count.
- So these are systematic whole internet scanning operations that treat every IP the same. They weren't targeting me.

**To check**
- AWS's Honeytrap histogram has a smaller second spike on 28 Oct. Need to see who it was.

### 9. 216.250.250.134, an RDP flood on Azure (2 Nov)
- The biggest spike in days 6 to 12. Filtered Discover to this IP on 2 Nov 00:00 to 12:00 and got 350,815 log entries.
- It ran from about midnight to 07:06 at a steady 8,000 entries every 10 minutes, then stopped. Not the daily reboot, that's at 13:45 on Azure.
- Every row was port 3389 (RDP, Windows Remote Desktop). Each flow lasted about 1 second with 8 to 12 packets each way, about 1.7 KB in and 2.5 KB out, and then closed.
- So it was opening a proper RDP session, getting a reply from RDPHoneypot, closing it and going again over and over. That's what RDP brute forcing or credential checking looks like. The flow records don't show the passwords it tried, that would be in RDPHoneypot's own logs.
- Screenshot: ../screenshots/investigations/azure-rdp-216.250.250.134-2nov.png

**Update (7 Nov), RDPHoneypot's own logs**
- Filtered type:RDPHoneypot and this IP from 28 Oct to 7 Nov: 86,848 entries, all on the night of 1 Nov into the morning of 2 Nov, nothing before or after.
- The network is Majestic Hosting Solutions, LLC (AS 396073) in the US.
- Every second it did the same three steps: session.connect, then rdphoneypot.login with auth_method NLA and domain WORKGROUP, then session.closed about a second later.
- NLA (Network Level Authentication) means the client has to send a username and password before it even gets the Windows login screen. So every session was one login attempt. That confirms it was RDP brute forcing, roughly 87,000 attempts in one night. MITRE ATT&CK T1110 Brute Force.
- These logs don't show which usernames or passwords it tried.
- AbuseIPDB (7 Nov): 100% abuse confidence, 257 reports from 78 different people, latest 1 day before. Hosted by Majestic Hosting Solutions (spinservers.com) in Dallas. So it's a known bad IP that hits lots of people, not just me.
- Screenshot: ../screenshots/investigations/lookup-216.250.250.134-abuseipdb.png
- Screenshot: ../screenshots/investigations/azure-rdp-216.250.250.134-rdphoneypot.png (my Azure IP covered)

### 10. Two IPs flooding port 5060 the same way (29 Oct)
**What I saw**
- 91.110.167.131 (CNSERVERS LLC) had 116,682 log entries and 8.212.14.105 (Alibaba US) had 63,756, both on 29 Oct and nothing before or after.
- Both only hit port 5060 (SIP, the protocol VoIP phones use), which is Sentrypeer.
- Every flow from both IPs was exactly the same. 1 packet in (189 bytes) and 1 packet out (771 bytes), flow age 0, and a new flow each time.

**What it could be**
- Two different networks sending the exact same packet at the same time means it's either one operator running the same tool from two servers, or something I've seen before.
- Each reply is about 4 times bigger than the request, one packet in and one out, over UDP. That's the same shape as the SNMP reflection in investigation 7. SIP can be used for reflection too. If the source IPs were faked, these two would be victims and not attackers.
- I haven't proven it either way yet.

**To check**
- Azure Monitor Network Out on 29 Oct. If it's a reflection there should be a bump like the SNMP one.
- What Sentrypeer logged for these IPs. If it's only OPTIONS requests and no REGISTER or INVITE, that points to reflection rather than someone trying to use the phone system.
- If it is reflection, think about rate limiting or blocking UDP 5060 like I did with 161. But Sentrypeer is a big part of the data so I'd lose a lot.
- Screenshots: ../screenshots/investigations/azure-sip-91.110.167.131-29oct.png and azure-sip-8.212.14.105-29oct.png

**Update (7 Nov)**
- All of 91.110.167.131's traffic from 29 Oct 10:30 to 7 Nov: 72,193 entries, all on 29 Oct and nothing after.
- Every row is a Suricata flow record with app_proto_ts failed, meaning Suricata couldn't read the request as proper SIP.
- Looked at all UDP flows on Azure in week 2. The exact same 189 bytes in, 771 bytes out shape shows up from lots of other networks too, like a server in Singapore, ISPs in Bangladesh and Türkiye and Saudi Telecom, all on 5 Nov. So that packet shape is just a common SIP probe plus Sentrypeer's normal 771 byte reply. It's not unique to these two IPs. What made them stand out was the volume, about one a second for a whole day.
- That makes reflection less likely. A 4x reply is small compared to SNMP's 14x, and the same shape is everywhere. Still need Azure Network Out for 29 Oct to rule it out properly, and the Sentrypeer records (type:Sentrypeer) to see the SIP method.
- Screenshots: ../screenshots/investigations/azure-sip-91.110.167.131-all-flows.png and azure-udp-flows-week2.png

**Answer (7 Nov): not a reflection**
- Sentrypeer did log these as real SIP messages, 58,470 of them, all on 29 Oct. Every one had called_number 1, so it kept sending the same SIP request to extension 1, about once a second. I didn't add the sip_method column so I still don't know if it was OPTIONS or INVITE.
- Azure Network Out from 28 Oct to 7 Nov has no bump on 29 Oct at all. It sits at the normal level that day. The SNMP reflection made a clear jump, so if this was a reflection it would show.
- So it was a SIP scanner hammering extension 1, not a DDoS using my honeypot. No need to block UDP 5060.
- The big Network Out peaks are on 2 to 5 Nov, which line up with the RDP brute force, the Alsycon scans and the VNC wave. That's the honeypots replying to a lot of incoming traffic, which is normal.
- Screenshots: ../screenshots/investigations/azure-sentrypeer-91.110.167.131.png and azure-network-out-28oct-7nov.png

### 11. 194.50.16.198, a service scan from the Netherlands (31 Oct)
- 71,715 log entries, all in one burst on the morning of 31 Oct. Network Alsycon B.V. in Amsterdam. Probably a big part of why the Netherlands is the number 2 country this week.
- Unlike the Kazakhstan scanner it wasn't just touching ports. Suricata picked up HTTP and TLS on ports like 2404, 6579, 8083 and 5623, so it was checking what service runs on each port, not just if the port is open.
- Port 2404 is normally used for industrial control systems (IEC 104), so it was looking for those too.
- Screenshot: ../screenshots/investigations/azure-scan-194.50.16.198-31oct.png
- GreyNoise (7 Nov): MALICIOUS, a hosting IP (Alsycon B.V., Amstelveen) tagged as a scanner, not spoofable, actor unknown, last seen 6 Nov. AbuseIPDB: 100% abuse confidence with 48,343 reports from 642 people, the latest 46 seconds before I looked. So it's one of the busiest bad scanners on the internet right now and it hits everyone, not just me.
- Screenshot: ../screenshots/investigations/lookup-194.50.16.198-greynoise-abuseipdb.png

### 12. 77.83.240.70, the same kind of service scan on AWS (3 Nov)
- AWS's top source IP for days 6 to 12 with 31,169. Same network as 194.50.16.198 (Alsycon B.V., AS 49870).
- Filtered Discover to 3 Nov 06:00 to 09:00 and got 149,344 log entries in 3 hours.
- It came in bursts of about 13,000 every 15 minutes with almost nothing in between. That's a scheduled job, not a person.
- Same style as investigation 11. Different ports every row (5555, 9200, 9998, 8009) and Suricata picked up HTTP and TLS, so it was checking what service runs on each port.
- Port 5555 is Android Debug Bridge and Adbhoney logged a session from it. 9200 is Elasticsearch. So it was checking for exposed Android devices and databases too.
- That's the 3 Nov spike at the end of AWS's attacks histogram.
- Alsycon is the top network on AWS (44,581). Correction from the weekly reports: both 194.50.16.198 and 77.83.240.70 hit both clouds, so it's the same two IPs scanning both, not a different IP for each cloud.
- Screenshot: ../screenshots/investigations/aws-scan-77.83.240.70-3nov.png

### 13. The VoIP toll fraud VPN is back, now on AWS (4 to 5 Nov)
- Looked up 185.213.155.236. It's the same network as Azure's toll fraud IP 185.213.155.228 from investigation 2 (31173 Services AB, a VPN in Frankfurt), just a neighbouring IP.
- Same behaviour. All Sentrypeer on 5060, and the called_number is a North American number with different prefixes in front (9009, 1001, 2001). It's testing which dial out prefix works so it can make fraud calls. I covered the numbers in the screenshot.
- The flow_id is the same on every row, so it's one long session from the same source port, like last time.
- Mistake: I set the range to 28 Oct to 3 Nov but didn't click Update, so Kibana was still showing 4 Nov 10:01 to 5 Nov 00:01. The 162,097 entries are from those 14 hours, not days 6 to 12. It's not in the days 6 to 12 top 10 so it most likely started after the window.
- It was still going at midnight on 5 Nov at about 1,700 entries every 10 minutes. It'll probably be AWS's top attacker in the next check in.
- Screenshot: ../screenshots/investigations/aws-sip-185.213.155.236-4nov.png

**Update (7 Nov), with the range fixed**
- 28 Oct 10:32 to 5 Nov 10:32 with Update clicked: 229,369 entries. It started around 9am on 4 Nov and was still going at the end of the window, so it's all in week 2.
- One Suricata flow lasted 49,917 seconds (13.9 hours) with 80,838 packets each way and about 62 to 68 MB each way. That flow started at 15:54 on 4 Nov, 3 minutes after AWS's daily reboot at 15:51. Same as investigation 2 on Azure, where it came back 3 minutes after the reboot. So it never stops, it just reconnects as soon as the honeypot is back.
- Prefixes this time were 7009, 8009, 9009 and 90009. It's a different North American number from the one it tried on Azure in week 1, but the same trick.
- This time Suricata did alert: ET VOIP INVITE Message Flood UDP and GPL VOIP SIP INVITE message flooding, category Attempted Denial of Service. On Azure in week 1 it didn't raise a single alert for 8 hours. Not sure why yet, maybe it was faster this time and crossed the rule's rate limit.
- I also covered the phone numbers in the old week 1 screenshot so neither number is in the repo.
- GreyNoise (7 Nov): NOT OBSERVED, meaning it hasn't been seen mass scanning the whole internet in the last day. AbuseIPDB: 92% abuse confidence, 210 reports from 65 people, latest 14 hours before, 31173 Services AB in Frankfurt.
- So unlike the Alsycon scanners, this one isn't spraying the whole internet. It goes after SIP servers it already knows about, and mine is on its list. Other people have still reported it, so I'm not the only target.
- Screenshot: ../screenshots/investigations/lookup-185.213.155.236-greynoise-abuseipdb.png
- Screenshot: ../screenshots/investigations/aws-tollfraud-185.213.155.236-week2.png

### 14. A VNC wave on Azure from Contabo servers (4 to 5 Nov)
- Filtered dest_port:5900 on Azure for week 2: 364,576 entries, almost all from the evening of 4 Nov to the end of the window, and still climbing at the end.
- Port 5900 is VNC, a remote desktop tool mostly used on Linux and Macs. Heralding (which handles VNC logins) went from 3k in week 1 to 31k in week 2.
- The sources were lots of different servers, all from Contabo GmbH (AS 51167), a cheap German VPS provider, for example 167.86.102.132, 94.250.203.94, 173.249.54.93 and 84.46.252.81. So one operation spread over many rented servers.
- The alerts were GPL INFO VNC server response and VNC Authentication Failure, so it connected, got the VNC handshake and tried to log in. CVE-2006-2369 (a RealVNC login bypass from 2006) matched the "VNC Server Not Requiring Authentication" alert count, so that's the rule's CVE tag again, not a real exploit.
- It barely showed up on AWS (about 10,500 VNC alerts vs 67,000 on Azure).
- Need to check if it carried on into week 3.
- Screenshot: ../screenshots/investigations/azure-vnc-5900-week2.png (my Azure IP covered)

### 15. Fortinet VPN exploit attempts on AWS (CVE-2018-13379)
- 5,500 alerts in week 2, in two short bursts, one on the morning of 3 Nov and one just after midnight on 5 Nov. Dozens of alerts in the same second, so it's an automated tool firing lots of requests at once.
- The alert metadata says the affected product is Fortigate and it's tagged CISA_KEV, which means it's on CISA's Known Exploited Vulnerabilities list. Real attackers use this one, it's not just old noise.
- The MITRE techniques in the alert are T1083 File and Directory Discovery and T1082 System Information Discovery, since the bug lets you read files off the VPN like saved logins.
- Still need the request path and the source IP. In Discover, hover over src_ip and http.url in the field list and click the plus to add them as columns.
- Screenshot: ../screenshots/investigations/aws-cve-2018-13379-week2.png

### Note on AWS monitoring
- EC2's Monitoring tab shows CPU, network and status checks, but not memory. AWS can't see inside the VM's memory unless you install the CloudWatch Agent. Azure shows available memory by default. That's a real difference between the two clouds.

## Ideas / questions to look into for the write-up

- Always compare Azure and AWS over the exact same time window, not their own Day 1s.
- Why does AWS get so much more SMB and database traffic? Maybe AWS IP ranges are on different scan lists.
- Check which Suricata rules carry CVE tags so I don't count normal traffic as exploit attempts.
- Build a Kibana table of the top payload hashes and decode the top 5.
- Check whether the 109.160.32.x IPs all hit the same honeypot.
- Check other UDP services T-Pot exposes for reflection risk (look at top UDP destination ports in Kibana).
- Check who caused AWS's second Honeytrap spike on 28 Oct.
- Check which honeypot 23.94.174.242 hit on Azure (on AWS it was Sentrypeer).
- Find out why ewsposter keeps restarting on AWS.
- Compare top attacker networks (AS) between Azure and AWS.
- Search 345gs5662d34 to see which botnet uses it.
- Check what RDPHoneypot logged for 216.250.250.134 and its full network name.
- Check the payloads behind the CVE-2018-13379 and CVE-2025-30208 alerts on AWS (add src_ip and http.url columns).
- UDP bytes in vs out per port in Lens (623 IPMI and 69 TFTP are the ones to check).
- What caused the AWS Honeytrap spike on 28 Oct.
- Fix the 185.213.155.236 range and see when the AWS toll fraud started.
- Find who caused the VNC wave on Azure in week 2.
- Look up 136.144.199.59 (Signet B.V., 101,821 on Azure week 2) and Banatsync Srl.
