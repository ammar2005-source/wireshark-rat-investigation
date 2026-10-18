# Wireshark Investigation: Finding an Infected Host from a RAT Alert

A hands-on SOC Analyst L1 project. A SIEM alert flagged traffic to a known C2 IP, and I used Wireshark on the packet capture to identify the infected Windows host and write an incident report.

## Scenario
The SIEM raised signature hits for **NetSupport Manager RAT** from `45[.]131[.]214[.]85` over TCP port 443, starting **2026-02-28 19:55 UTC**. The goal was to find the internal host behind the alerts so it could be isolated.

Source: training exercise "Easy as 123" from [malware-traffic-analysis.net](https://www.malware-traffic-analysis.net/training-exercises.html). It is a training exercise, so the names and activity are not a real incident. The PCAP is **not included** in this repo.

## Objective
Answer five questions for the incident report:
1. IP address of the infected client
2. MAC address
3. Host name
4. User account name
5. Full name of the user

## Tools Used
- Wireshark (filters, Statistics, Conversations, Protocol Hierarchy, packet details)
- VirusTotal (IP reputation)
- Git and GitHub

## Findings

| Item | Value | Evidence |
|---|---|---|
| IP address | 10.2.28.88 | SYN packet #2569 to the alerted IP on port 443 |
| MAC address | 00:19:d1:b2:4d:ad | Ethernet II source field, DHCP request |
| Hostname | DESKTOP-TEYQ2NR | DHCP Option 12 and Option 81 (FQDN) |
| User account | brolf | Kerberos AS-REQ / AS-REP with the domain controller |
| Full name | Becka Rolf | SAMR QueryUserInfo response |

**C2 connection:** one persistent TCP connection to port 443, first packet at 2026-02-28 19:55:51 UTC (matching the alert time), about 4 hours 20 minutes long, 550 packets and about 96 kB in total, with very low throughput.

## How I Investigated
1. Filtered the capture by the alerted IP (`ip.addr == 45.131.214.85`) and found the internal client from the SYN packet.
2. Read the MAC address from the Ethernet II source field.
3. Found the hostname in the client's DHCP request (Options 12 and 81).
4. Found the user account in Kerberos traffic with the domain controller.
5. Found the full name in the SAMR `QueryUserInfo` response (`Info21`, `Full Name`).
6. Checked timing and volume with Statistics, then Conversations.
7. Wrote an incident report with evidence, IOCs and recommended actions.

## Repository Structure
- `Reports/incident-report.md`: full incident report
- `Evidence/`: screenshots of the key packets
- `README.md`: this file

## What I Learned
- Reading Wireshark's Protocol Hierarchy and Conversations to understand a capture.
- Identifying an infected host through DHCP, Kerberos and SAMR traffic.
- Why packet direction matters (the SYN sender is the client) and why to read the Source MAC, not the Destination.
- Always reporting times in UTC, so they match SIEM alerts.
- Separating what I verified from what comes from the alert. I did not confirm the malware family myself, because the traffic is TLS encrypted.

## Safety
I only read the capture in Wireshark. I did not extract or run any files from it.

## Author
Ammar Munawwar, Computer Science student preparing for a SOC Analyst L1 role.
