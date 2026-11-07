# Honeypot Campaign: Azure vs AWS

This is my first cybersecurity project as a Bachelor of Cybersecurity student at UTS. I'm running decoy servers (honeypots) on Azure and AWS for a month to see what attacks hit each cloud, then analysing what the attackers do. It's a small version of what a Security Operations Centre (SOC) does every day.

**Status:** Running. Azure went live 21 Oct 2026 and AWS went live 22 Oct 2026. Both run until 21 Nov 2026.

## Setup

I'm running two identical T-Pot honeypots at the same time so the comparison is fair.

| | Azure | AWS |
| --- | --- | --- |
| Region | Australia East (Sydney) | ap-southeast-2 (Sydney) |
| VM | Standard_B2as_v2 (2 vCPU, 8 GB) | m7i-flex.large (2 vCPU, 8 GB) |
| OS | Debian 13 | Debian 13 |
| Honeypot | T-Pot (Hive install) | T-Pot (Hive install) |
| Paid with | Azure for Students credit | AWS Free plan credit |

The main question I want to answer is whether attackers treat Azure and AWS IP ranges differently.

## Safety

- The honeypots only run in the cloud, never on my home network.
- Admin access (SSH and the web dashboard) is locked to my IP only, and SSH only accepts keys.
- Both clouds email me if the honeypot suddenly sends out a lot of traffic, which is how I'd spot it being used in a DDoS.
- I only watch. I never attack back at any IP that hits the honeypots.
- No captured malware gets downloaded to my own machines. I just look up the file hashes.
- Public IPs, keys and passwords are kept out of this repo.

## How I report

Each week runs Thursday 10:32 to Thursday 10:32 (Sydney time), starting from when AWS went live. Both clouds use the exact same window every week, so every comparison is fair. At the end there's one month summary for each cloud.

| Week | Window |
| --- | --- |
| Week 1 | 22 Oct 10:32 to 29 Oct 10:32 |
| Week 2 | 29 Oct 10:32 to 5 Nov 10:32 |
| Week 3 | 5 Nov 10:32 to 12 Nov 10:32 |
| Week 4 | 12 Nov 10:32 to 19 Nov 10:32 |
| Final stretch | 19 Nov 10:32 to 21 Nov |

For the first two weeks I did check ins every few days with different windows on each cloud. Those are still in the campaign log. I switched to weekly reports on 6 Nov because the same window on both clouds makes the comparison cleaner.

## What's in here

- `docs/campaign-log.md` is my running log with every check in, investigation and problem
- `screenshots/` has dashboard screenshots for each cloud, with the time window written on top
- `data/` will have the exported results once the campaign ends

## Week by week

| | Week 1 | Week 2 | Week 3 | Week 4 |
| --- | --- | --- | --- | --- |
| Azure attacks | about 982,000 | about 1.8 million | coming | coming |
| AWS attacks | about 336,000 | about 656,000 | coming | coming |
| Azure vs AWS | 2.9 times more | 2.7 times more | | |
| Azure top honeypot | Sentrypeer (VoIP) | Honeytrap (all other ports) | | |
| AWS top honeypot | Honeytrap | Honeytrap | | |
| Top attacker network | Banatsync Srl (Azure), AO ALMAZ (AWS) | Alsycon B.V. (both) | | |
| Biggest event | SNMP reflection DDoS on Azure | Alsycon service scans on both clouds | | |

### Week 1 (22 to 29 Oct)

**First contact**

| | Azure | AWS |
| --- | --- | --- |
| Time from going live to the first hit | 16 seconds | 3 seconds |
| First hit | An empty connection from Driftnet Ltd, an internet scanning company | A bot sending `CONNECT api.ipinfo.io:443` to see if my server is an open proxy |

Nobody knew these IPs existed and they still got found in seconds. Bots scan the whole internet nonstop, so anything you open up gets found straight away.

**Each cloud has its own personality**

- Azure got 2.9 times more attacks than AWS over the exact same 7 days, and it's SSH heavy. Cowrie (SSH and Telnet) was 23% of Azure's traffic but only 4% of AWS's.
- AWS is SMB and database heavy. Dionaea was 21% of AWS's traffic but only 5% of Azure's, sa (the SQL Server admin account) is its top username, and it had 2.6 times more DoublePulsar alerts.
- The gap isn't the big scanners. Those hit both clouds with almost the same counts. Azure got extra traffic that never touched AWS: a Romanian hosting network (Banatsync, 162k from lots of IPs), the toll fraud VPN, the SNMP reflection and a pair of IPs flooding VoIP.
- DoublePulsar is the NSA backdoor that leaked in 2017 and was used with WannaCry. Bots still check for it.
- Some bots tried usernames like claude and openclaw, which looks like they're going after AI tools now.

**VoIP toll fraud**

Azure's top attacker was a VPN exit in Germany with a clean reputation. It sent about one VoIP request a second for over 8 hours, trying different dial out prefixes on the same phone number. That's a toll fraud scan, it's looking for a phone system that will let it push calls to premium numbers. Suricata didn't alert on any of it, only the honeypot caught it.

**Incident: my honeypot was used in a DDoS**

On 26 Oct, ConPot (which pretends to be industrial equipment) started getting hundreds of thousands of SNMP requests. The traffic was 100% UDP to port 161 from home and business internet connections. One flow lasted 18.7 hours and my VM sent out about twice the bytes it received, with each reply about 14 times bigger than the request.

That's an SNMP reflection attack. Attackers fake the victim's IP as the sender, my honeypot sends its bigger replies to the victim, and the victim gets flooded. I blocked UDP 161 on both clouds on 27 Oct. Looking back at the timestamps the flood had actually stopped by itself about 35 minutes before the block, so the block is there to stop it happening again. The lesson is that a honeypot can hurt other people if you only watch what comes in and not what goes out.

**The same scanners hit both clouds with the same count**

A port scanner from Kazakhstan sent 62,978 probes to Azure and 62,979 to AWS, 1.5 days apart, with the exact same packet pattern. That's about the number of open TCP ports on each honeypot, so it hit every open port once on each. Three other scanners did the same thing with matching counts, some switching to a neighbouring IP in between. These are whole internet scanning operations that treat every IP the same, not attacks aimed at me.

### Week 2 (29 Oct to 5 Nov)

Azure got about 1.8 million and AWS about 656,000, so 2.7 times more on Azure, close to week 1. Both clouds roughly doubled, mostly because of one Dutch hosting network.

- **RDP brute force on Azure (1 to 2 Nov).** One IP from a US hosting company opened a Remote Desktop session every second for a whole night. RDPHoneypot's logs show each session was a login attempt using NLA (where you send the username and password before seeing the login screen), so that's about 87,000 password guesses in one night.
- **A false alarm I ruled out: SIP reflection on Azure (29 Oct).** Two IPs on different networks hit port 5060 (VoIP) with the exact same packet, one 189 byte request and one 771 byte reply each time. That's the same shape as the SNMP reflection, so I checked if this was a SIP reflection too. Lots of other networks send the exact same probe and get the same reply, and the outbound traffic graph for that day has no bump at all, so it wasn't. It was a SIP scanner sending the same request to extension 1 about once a second for a whole day.
- **One scanning operation behind a huge share of the week.** Two IPs from Alsycon B.V. (a Dutch hosting network) hit both clouds. That's 37% of Azure's week and 49% of AWS's. They didn't just check if ports were open, they checked which service runs on each one, including industrial control systems, Android debug ports and Elasticsearch. Both IPs hit Azure about twice as hard as AWS. Even without them Azure was still about 3.4 times busier.
- **A VNC wave on Azure.** Port 5900 (remote desktop for Linux and Macs) jumped into Azure's top 5 with about 67,000 VNC alerts, all from lots of rented servers at one cheap German VPS provider (Contabo). They connected and tried to log in. AWS barely saw it.
- **The toll fraud VPN switched clouds.** In week 1 it was on Azure. In week 2 a neighbouring IP from the same VPN range did the same thing on AWS (115,000 hits) with a different phone number, and disappeared from Azure's top 10. Its connection came back 3 minutes after AWS's daily reboot, exactly like it did on Azure, so it never stops trying.
- **Real exploit hunting.** Suricata tagged CVE-2018-13379, a Fortinet VPN bug that leaks saved logins and is on the US government's Known Exploited Vulnerabilities list, 22,170 times on Azure and 5,500 on AWS. Azure also got Log4Shell (CVE-2021-44228) and a 2024 PHP bug (CVE-2024-4577). Bots go after old and new bugs at the same time.

### Week 3 (5 to 12 Nov)

Coming.

### Week 4 (12 to 19 Nov)

Coming.

## Things I learned about doing the analysis

- A CVE showing up in Suricata doesn't mean that exploit was used. CVE-2001-0540 matched a normal RDP connection alert exactly, every time, on both clouds. It's just one rule tagging normal traffic with an old CVE.
- Attackers going to specific ports isn't the honeypot steering them. Each honeypot only listens on its own ports, so an IP hitting 5060 picked it because it's looking for phone systems. And a port's list of known vulnerabilities doesn't mean those were used. You need the IDS alerts or honeypot logs for that.
- Spikes come from single IPs, not more attackers. On both clouds the unique source IP line stayed flat while attacks spiked, so the first step is always finding the one noisy IP.
- Always check the Kibana time range and click Update. A wrong range made AWS look like it had lost data, and once made me screenshot the wrong days.
- Check IPs against threat intel before deciding what they are. GreyNoise and AbuseIPDB showed the Alsycon scanner hits everyone (48,000+ reports), while the toll fraud VPN isn't scanning the whole internet, it goes after SIP servers it already knows about.
- Not every log is an attack. The first entries after going live were my own VMs talking to their cloud's internal services, so part of the job is filtering that out.

More detail is in the [campaign log](docs/campaign-log.md).

## Problems I've hit so far

- On day 1 the Azure VM ran out of memory about an hour after T-Pot started and froze. I found it with Azure Monitor (CPU credits were fine, available memory dropped to almost zero) and fixed it by adding 4 GB of swap. I did the same on AWS before going live.
- A network traffic drop to zero turned out to be T-Pot's built in daily reboot. Found it through uptime and the crontab.
- ewsposter (the container that shares data with T-Pot's community threat map) was crash looping on AWS in week 1. By 6 Nov AWS was healthy, but Azure's ewsposter had restarted 52 times. Its logs show it's still sending data, so it crashes now and then and comes back. It doesn't touch my own data so it's low priority.
- On 5 Nov I found T-Pot deletes data after 30 days. That would have deleted my first week right before the month summary, so I changed it to 60 days on both and checked it survived the daily reboot.
- On 6 Nov I set up outbound traffic alerts on both clouds (Azure Monitor and AWS CloudWatch) so I'd get an email if a honeypot gets used in a reflection attack again. The Azure one kept failing with a server error on the Network Out Total metric, so I narrowed it down by testing a different metric and used the older Network Out metric instead.

## Skills and tools

T-Pot, Cowrie, Dionaea, Suricata, Elasticsearch, Kibana, Azure, AWS EC2, Debian Linux
