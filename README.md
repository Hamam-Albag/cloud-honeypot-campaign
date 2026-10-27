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
- Admin access (SSH and the web dashboard) is locked to my IP only.
- I only watch. I never attack back at any IP that hits the honeypots.
- No captured malware gets downloaded to my own machines. I just look up the file hashes.
- Public IPs, keys and passwords are kept out of this repo.

## What's in here

- `docs/campaign-log.md` is my running log with check ins every few days
- `screenshots/` has dashboard screenshots for each cloud, with the time window written on top
- `data/` will have the exported results once the campaign ends

## Early findings

| | Azure | AWS |
| --- | --- | --- |
| Time from going live to the first hit | 16 seconds | 3 seconds |
| First hit | An empty connection from Driftnet Ltd, an internet scanning company | A bot sending `CONNECT api.ipinfo.io:443` to see if my server is an open proxy |

Nobody knew these IPs existed and they still got found in seconds. Bots scan the whole internet nonstop, so anything you open up gets found straight away.

After the first day on Azure (about 32,000 attacks):

- Most of it was bots guessing SSH logins. The top usernames were root and admin, and the top passwords were 123456 and a blank password.
- Suricata flagged 1,379 DoublePulsar attempts. That's the NSA backdoor that leaked in 2017 and was used with WannaCry, and bots still check for it.
- 1,638 hits came from IPs already on the DShield blocklist.
- Some bots tried usernames like claude and openclaw, which looks like they're going after AI tools now.
- Not every log is an attack. The first entries after going live were my own VMs talking to their cloud's internal services, so part of the job is filtering that out.

### First same window comparison (22 Oct 10:32 to 23 Oct 10:32)

| | Azure | AWS |
| --- | --- | --- |
| Total attacks in 24 hours | about 114,000 | about 28,000 |
| Busiest honeypot | Sentrypeer (VoIP) 51k | Honeytrap and Dionaea 8k each |
| Top username | root | sa (SQL Server admin) |
| RDP connection requests | 1,527 | 686 |
| DoublePulsar alerts | under 674 | 5,847 |

Early pattern from one day: Azure gets a lot more Windows Remote Desktop attacks and AWS gets a lot more SMB and database attacks. I'll check if it holds over the rest of the month.

One thing I learned: a CVE showing up in Suricata doesn't mean that exploit was used. CVE-2001-0540 matched a normal RDP connection alert exactly on both clouds, so it was just one rule tagging normal traffic with an old CVE.

### First 3 days on each cloud

| | Azure | AWS |
| --- | --- | --- |
| Total attacks | about 301,000 | about 82,000 |
| Share that was SSH (Cowrie) | 21% | 9% |
| Share that was SMB and databases (Dionaea) | 5% | 18% |
| DoublePulsar alerts | 6,340 | 12,090 |
| Top source IP | a VPN exit in Germany doing VoIP toll fraud scans (44,560 hits) | one HostPapa server (21,962 hits) |

The pattern from day 2 held. Azure gets way more traffic overall and is SSH heavy, AWS is SMB heavy. Some attackers hit both clouds, the same HostPapa IP is in both top 10 lists.

### Things I investigated

- **Azure's top attacker** was a VPN exit IP with a clean reputation. It sent about one VoIP request a second for over 8 hours, trying different dial out prefixes on the same phone number. That's a toll fraud scan. Suricata didn't alert on any of it, only the honeypot caught it.
- **A network traffic drop to zero** turned out to be T-Pot's built in daily reboot. Found it through uptime and the crontab.
- **AWS looked like it lost data.** I checked boots, memory, disk and containers before finding the real cause, a wrong time range in Kibana.

### Incident: my honeypot was used in a DDoS

On 26 Oct, ConPot (which pretends to be industrial equipment) started getting hundreds of thousands of SNMP requests. The traffic was 100% UDP to port 161 from home and business internet connections. One flow lasted 18.7 hours and my VM sent out about twice the bytes it received, with each reply about 14 times bigger than the request.

That's an SNMP reflection attack. Attackers fake the victim's IP as the sender, my honeypot sends its bigger replies to the victim, and the victim gets flooded. I blocked UDP 161 on both clouds on 27 Oct. Looking back at the timestamps the flood had actually stopped by itself about 35 minutes before the block, so the block is there to stop it happening again. The lesson is that a honeypot can hurt other people if you only watch what comes in and not what goes out.

### After 6 days on Azure

About 754,000 attacks. Other things found: a full range port scan from Kazakhstan that touched almost every port from 0 to 65535 in about 9 hours, and almost every attacking machine fingerprinted as Linux.

More detail is in the [campaign log](docs/campaign-log.md).

## Problems I've hit so far

- On day 1 the Azure VM ran out of memory about an hour after T-Pot started and froze. I found it with Azure Monitor (CPU credits were fine, available memory dropped to almost zero) and fixed it by adding 4 GB of swap. I did the same on AWS before going live.
- On day 3 I found ewsposter (the container that shares data with T-Pot's community threat map) crash looping on AWS. Still looking into it.

## Skills and tools

T-Pot, Cowrie, Dionaea, Suricata, Elasticsearch, Kibana, Azure, AWS EC2, Debian Linux
