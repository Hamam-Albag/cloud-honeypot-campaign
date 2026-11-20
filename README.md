# Honeypot Campaign: Azure vs AWS

This is my first cybersecurity project as a Bachelor of Cybersecurity student at UTS. I ran identical decoy servers (honeypots) on Azure and AWS for about a month to see what attacks hit each cloud, then analysed what the attackers did. It's a small version of what a Security Operations Centre (SOC) does every day: watch the alerts, find the one noisy IP behind a spike, check it against threat intel, and decide if it matters.

**Status:** Finished. Azure ran from 21 Oct to 18 Nov 2026 and AWS from 22 Oct to 18 Nov 2026. It was planned to run until 21 Nov, but the Azure student credit ran out on the evening of 18 Nov, so I ended both clouds at the same point.

## Results at a glance

| | Azure | AWS |
| --- | --- | --- |
| Attacks (22 Oct 10:32 to 18 Nov 22:00) | about 4.76 million | about 1.78 million |
| Time to first attack after going live | 16 seconds | 3 seconds |
| What it got hit with most | SSH logins, VoIP scanning and Remote Desktop (RDP) | Random port scanning, VoIP and Windows file sharing (SMB) |
| Top attacker network | Alsycon B.V., a Dutch hosting company (691,618) | Alsycon B.V. again (331,870) |
| Biggest single attacker | 194.50.16.198, Alsycon's scanner (458,589) | 45.141.233.12, an RDP brute force (255,462) |

- **Azure got about 2.7 times more attacks than AWS, and the gap held every single week** (2.9, 2.7, 2.4 and 2.6 times).
- **The big internet scanners treat both clouds the same.** Several IPs hit both with almost identical counts, like a port scanner that sent 62,978 to Azure and 62,979 to AWS, and an RDP brute force that sent 256,406 to Azure and 255,462 to AWS.
- **The gap comes from everything else.** Extra SSH guessing, extra VoIP scanning and attackers that only went after Azure.
- **AWS still got more of some things**, mainly Windows file sharing attacks (port 445, the DoublePulsar backdoor check) and the VoIP toll fraud operation.
- **My Azure honeypot got used in a DDoS.** In week 1 attackers used it to bounce traffic at someone else (an SNMP reflection attack). I caught it, blocked it on both clouds and added outbound traffic alerts.

## Setup

I ran two identical T-Pot honeypots at the same time so the comparison is fair.

| | Azure | AWS |
| --- | --- | --- |
| Region | Australia East (Sydney) | ap-southeast-2 (Sydney) |
| VM | Standard_B2as_v2 (2 vCPU, 8 GB) | m7i-flex.large (2 vCPU, 8 GB) |
| OS | Debian 13 | Debian 13 |
| Honeypot | T-Pot 24.04.1 (Hive install) | T-Pot 24.04.1 (Hive install) |
| Ran | 21 Oct 16:21 to 18 Nov about 10pm | 22 Oct 10:32 to 18 Nov 22:00 (used as the end point) |
| Paid with | Azure for Students credit | AWS Free plan credit |

The main question was whether attackers treat Azure and AWS IP ranges differently.

## Safety

- The honeypots only ran in the cloud, never on my home network.
- Admin access (SSH and the web dashboard) was locked to my IP only, and SSH only accepted keys.
- From 6 Nov both clouds emailed me if the honeypot suddenly sent out a lot of traffic, which is how I'd spot it being used in a DDoS.
- I only watched. I never attacked back at any IP that hit the honeypots.
- No captured malware was downloaded to my own machines.
- Public IPs, keys, passwords and phone numbers are kept out of this repo. Where my own IPs showed up in a screenshot I covered them.

## How I reported

Each week ran Thursday 10:32 to Thursday 10:32 (Sydney time), starting from when AWS went live. Both clouds used the exact same window every week, so every comparison is fair.

| Week | Window |
| --- | --- |
| Week 1 | 22 Oct 10:32 to 29 Oct 10:32 |
| Week 2 | 29 Oct 10:32 to 5 Nov 10:32 |
| Week 3 | 5 Nov 10:32 to 12 Nov 10:32 |
| Week 4 | 12 Nov 10:32 to 18 Nov 22:00 (6.5 days, ended when Azure stopped) |

For the first two weeks I did check ins every few days with different windows on each cloud. Those are still in the campaign log. I switched to weekly reports on 6 Nov because the same window on both clouds makes the comparison cleaner.

## What's in here

- `docs/campaign-log.md` is my running log with every check in, weekly report, investigation and problem
- `screenshots/` has dashboard screenshots for each cloud, with the cloud and time window written on top

## Week by week

| | Week 1 | Week 2 | Week 3 | Week 4 | Total |
| --- | --- | --- | --- | --- | --- |
| Azure attacks | about 982,000 | about 1.8 million | about 700,000 | about 1.27 million | about 4.76 million |
| AWS attacks | about 336,000 | about 656,000 | about 287,000 | about 498,000 | about 1.78 million |
| Azure vs AWS | 2.9 times more | 2.7 times more | 2.4 times more | 2.6 times more | 2.7 times more |
| Azure top honeypot | Sentrypeer (VoIP) | Honeytrap (all other ports) | Cowrie (SSH) | RDPHoneypot | |
| AWS top honeypot | Honeytrap | Honeytrap | Dionaea (SMB) | RDPHoneypot | |
| Top attacker network | Banatsync Srl (Azure), AO ALMAZ (AWS) | Alsycon B.V. (both) | Banatsync Srl (Azure), UAB Cherry Servers (AWS) | MEVSPACE (both) | Alsycon B.V. (both) |
| Biggest event | SNMP reflection DDoS on Azure | Alsycon service scans on both clouds | VNC wave on both clouds | One RDP brute force on both clouds | |

### Week 1 (22 to 29 Oct): first contact, two personalities and a DDoS

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

### Week 2 (29 Oct to 5 Nov): one scanning company and a lot of brute forcing

Azure got about 1.8 million and AWS about 656,000, so 2.7 times more on Azure, close to week 1. Both clouds roughly doubled, mostly because of one Dutch hosting network.

- **RDP brute force on Azure (1 to 2 Nov).** One IP from a US hosting company opened a Remote Desktop session every second for a whole night. RDPHoneypot's logs show each session was a login attempt using NLA (where you send the username and password before seeing the login screen), so that's about 87,000 password guesses in one night.
- **A false alarm I ruled out: SIP reflection on Azure (29 Oct).** Two IPs on different networks hit port 5060 (VoIP) with the exact same packet, one 189 byte request and one 771 byte reply each time. That's the same shape as the SNMP reflection, so I checked if this was a SIP reflection too. Lots of other networks send the exact same probe and get the same reply, and the outbound traffic graph for that day has no bump at all, so it wasn't. It was a SIP scanner sending the same request to extension 1 about once a second for a whole day.
- **One scanning operation behind a huge share of the week.** Two IPs from Alsycon B.V. (a Dutch hosting network) hit both clouds. That's 37% of Azure's week and 49% of AWS's. They didn't just check if ports were open, they checked which service runs on each one, including industrial control systems, Android debug ports and Elasticsearch. Both IPs hit Azure about twice as hard as AWS. Even without them Azure was still about 3.4 times busier.
- **A VNC wave on Azure.** Port 5900 (remote desktop for Linux and Macs) jumped into Azure's top 5 with about 67,000 VNC alerts, all from lots of rented servers at one cheap German VPS provider (Contabo). They connected and tried to log in. AWS barely saw it.
- **The toll fraud VPN switched clouds.** In week 1 it was on Azure. In week 2 a neighbouring IP from the same VPN range did the same thing on AWS (115,000 hits) with a different phone number, and disappeared from Azure's top 10. Its connection came back 3 minutes after AWS's daily reboot, exactly like it did on Azure, so it never stops trying.
- **Real exploit hunting.** Suricata tagged CVE-2018-13379, a Fortinet VPN bug that leaks saved logins and is on the US government's Known Exploited Vulnerabilities list, 22,170 times on Azure and 5,500 on AWS. Azure also got Log4Shell (CVE-2021-44228) and a 2024 PHP bug (CVE-2024-4577). Bots go after old and new bugs at the same time.

### Week 3 (5 to 12 Nov): the quiet week

Azure got about 700,000 and AWS about 287,000, 2.4 times more on Azure. The quietest week on both, with no single huge event.

- **Same personalities.** SSH (Cowrie) was 179k on Azure and only 4k on AWS. Windows file sharing (Dionaea) was 114k on AWS and 28k on Azure.
- **Same IPs, same counts.** 185.8.104.174 was the top IP on both clouds (22,682 vs 22,891).
- **The VNC wave reached AWS.** In week 2 it was almost only Azure. This week it was 144k VNC alerts on Azure and 125k on AWS.
- **Proxy hunting on AWS.** Usernames like socksadmin and proxyadmin plus HTTP CONNECT alerts, so bots were looking for open proxies to route their own traffic through.

### Week 4 (12 to 18 Nov): one RDP brute force on both clouds

Azure got about 1.27 million and AWS about 498,000 in 6.5 days, 2.6 times more on Azure.

- **One IP, two clouds, the same count.** 45.141.233.12 (a Polish hosting company, located in Bulgaria) ran an RDP brute force on both clouds at the same time: 256,406 on Azure and 255,462 on AWS. It made Bulgaria the top country and Remote Desktop the top honeypot on both, and caused the biggest outbound traffic peaks of the month on Azure as the honeypot replied to every session.
- **The toll fraud VPN came back on both clouds at once,** from two neighbouring IPs, one on Azure and one on AWS. Over the month it used 4 different IPs from the same VPN network.
- **Newer and bigger exploit tags.** Azure got the Apache Struts bug behind the 2017 Equifax breach (CVE-2017-5638) and an RDP authentication bypass check.
- **Guessing cloud database and camera logins.** AWS got usernames for cloud database admin accounts (cloudsqlsuperuser, enterprisedb, azure_superuser) and Azure got camera and router ones (hikvision, telecomadmin).

## So do attackers treat Azure and AWS differently?

Yes, but not in the way I expected.

The biggest operations, the internet wide scanners and brute forcers, don't care which cloud an IP belongs to. They hit both with almost the same numbers, sometimes within one or two packets of each other. If I'd only looked at those, I'd have said there's no difference.

The difference is in everything else. Over the same 4 weeks Azure got about 2.7 times more attacks, every week, and it leaned towards SSH and VoIP. AWS got less overall but more Windows file sharing and SQL Server attacks, and it was the main target of the VoIP toll fraud operation. My best guess is that attackers keep separate target lists for different jobs and the cloud IP ranges land on different lists, but with one honeypot per cloud I can't prove that. Running a few honeypots per cloud would be the way to test it.

## MITRE ATT&CK mapping

| What I saw | Technique |
| --- | --- |
| Internet wide port and service scans (Alsycon, the Kazakhstan scanner) | T1595 Active Scanning, T1046 Network Service Discovery |
| SSH, RDP and VNC password guessing | T1110 Brute Force |
| Fortinet VPN, Log4Shell, PHP and Apache Struts exploit attempts | T1190 Exploit Public-Facing Application |
| My honeypot being used to bounce SNMP traffic at someone | T1498.002 Reflection Amplification |
| VoIP toll fraud using phone systems to make paid calls | T1496 Resource Hijacking |

## Things I learned about doing the analysis

- A CVE showing up in Suricata doesn't mean that exploit was used. CVE-2001-0540 matched a normal RDP connection alert exactly, every time, on both clouds. It's just one rule tagging normal traffic with an old CVE.
- Attackers going to specific ports isn't the honeypot steering them. Each honeypot only listens on its own ports, so an IP hitting 5060 picked it because it's looking for phone systems. And a port's list of known vulnerabilities doesn't mean those were used. You need the IDS alerts or honeypot logs for that.
- Spikes come from single IPs, not more attackers. On both clouds the unique source IP line stayed flat while attacks spiked, so the first step is always finding the one noisy IP.
- Always check the Kibana time range and click Update. A wrong range made AWS look like it had lost data, and once made me screenshot the wrong days.
- Check IPs against threat intel before deciding what they are. GreyNoise and AbuseIPDB showed the Alsycon scanner hits everyone (48,000+ reports), while the toll fraud VPN isn't scanning the whole internet, it goes after SIP servers it already knows about.
- Not every log is an attack. The first entries after going live were my own VMs talking to their cloud's internal services, so part of the job is filtering that out.
- Big round numbers in Kibana can hide a lot. It shows 1.27 million as "1m", so I added up the honeypot counts for the real total.

## Problems I hit

- On day 1 the Azure VM ran out of memory about an hour after T-Pot started and froze. I found it with Azure Monitor (CPU credits were fine, available memory dropped to almost zero) and fixed it by adding 4 GB of swap. I did the same on AWS before going live.
- A network traffic drop to zero turned out to be T-Pot's built in daily reboot. Found it through uptime and the crontab.
- ewsposter (the container that shares data with T-Pot's community threat map) was crash looping on AWS in week 1. By 6 Nov AWS was healthy, but Azure's ewsposter had restarted 52 times. Its logs showed it was still sending data, so it crashed now and then and came straight back. It never touched my own data so I left it.
- On 5 Nov I found T-Pot deletes data after 30 days. That would have deleted my first week right before the month summary, so I changed it to 60 days on both and checked it survived the daily reboot.
- On 6 Nov I set up outbound traffic alerts on both clouds (Azure Monitor and AWS CloudWatch) so I'd get an email if a honeypot gets used in a reflection attack again. The Azure one kept failing with a server error on the Network Out Total metric, so I narrowed it down by testing a different metric and used the older Network Out metric instead.
- On 18 Nov the Azure for Students credit ran out about 3 days early and Microsoft disabled the subscription, which stopped the VM. My credit estimate on 7 Nov said it would last, so I got that wrong. To get the last two weeks of Azure data I upgraded to pay as you go for one session, locked every inbound rule to my IP first, took the screenshots and then deleted everything.

## What I'd do differently

- **Set up outbound traffic alerts on day 1.** I only added them after the SNMP reflection. They should be part of the setup.
- **Set a budget alert on day 1 too,** and check the credit every week. My estimate was wrong and the campaign ended early.
- **Use the same window on both clouds from the start.** I wasted time early on comparing different time ranges.
- **Check the data retention setting at the start.** T-Pot was going to delete my first week before the month ended and I almost missed it.
- **Export the raw data before shutting down.** I only kept screenshots, so I can't go back and dig into anything new now.
- **Run more than one honeypot per cloud.** With one each I can't tell if a difference is about the cloud or just that one IP.

## Skills and tools

T-Pot, Cowrie, Dionaea, Suricata, Elasticsearch, Kibana, Azure (VMs, NSGs, Azure Monitor alerts, Azure CLI), AWS (EC2, security groups, CloudWatch, SNS), Debian Linux, SSH, GreyNoise, AbuseIPDB, MITRE ATT&CK

More detail on every check in, weekly report and investigation is in the [campaign log](docs/campaign-log.md).
