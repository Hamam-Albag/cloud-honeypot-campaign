# Homelab: Honeypots, IDS/IPS and a SIEM

This is my first cybersecurity project as a Bachelor of Cybersecurity student at UTS. I'm building a small version of what a Security Operations Centre (SOC) runs. Attackers hit decoy servers in the cloud, I collect what they do, then I analyse it and write detections.

**Status:** Phase 1 running. Azure went live 21 Oct 2026 and AWS went live 22 Oct 2026. Both run until 21 Nov 2026.

## Phases

| Phase | What | Status |
| --- | --- | --- |
| 1 | Honeypot campaign on Azure and AWS, compare what attacks each cloud gets | Running |
| 2 | Home lab on my PC in VMware with OPNsense + Suricata (IDS/IPS) and Wazuh (SIEM) | Planned |
| 3 | Long running honeypot feeding logs into Wazuh | Planned |
| 4 | Using machine learning and LLMs on the attack data | Planned |

## Phase 1 setup

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

- `phase1-honeypot-campaign/campaign-log.md` is my running log with check ins every few days
- `phase1-honeypot-campaign/screenshots/` has dashboard screenshots (IPs blurred)
- `phase1-honeypot-campaign/data/` will have the exported results once the campaign ends

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

More detail is in the [campaign log](phase1-honeypot-campaign/campaign-log.md).

## Problems I've hit so far

- On day 1 the Azure VM ran out of memory about an hour after T-Pot started and froze. I found it with Azure Monitor (CPU credits were fine, available memory dropped to almost zero) and fixed it by adding 4 GB of swap. I did the same on AWS before going live.

## Tools

T-Pot, Cowrie, Dionaea, Suricata, Elasticsearch, Kibana, Azure, AWS EC2, Debian Linux
