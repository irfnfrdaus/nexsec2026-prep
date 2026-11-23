# Network / PCAP Analysis

If you're given a packet capture, or want to check live traffic on an assigned box, this is the
workflow for spotting C2 traffic, exfiltration, and exploitation attempts.

## First look with tshark (command-line Wireshark — faster for triage than the GUI)

```bash
tshark -r capture.pcap -q -z io,phs              # protocol hierarchy — what's actually in this capture
tshark -r capture.pcap -q -z conv,tcp             # TCP conversation summary — who talked to whom, how much data
tshark -r capture.pcap -q -z conv,ip              # same at the IP layer
```

The conversation summary is often the fastest way to spot anomalies: a connection with an unusually
large byte count (possible exfiltration), or a connection to an external IP that doesn't fit the
expected traffic pattern.

## Follow a specific conversation

```bash
tshark -r capture.pcap -Y "ip.addr==10.0.0.5 && ip.addr==203.0.113.9"
tshark -r capture.pcap -Y "tcp.stream==4" -T fields -e frame.time -e ip.src -e ip.dst -e data.text
```

In the Wireshark GUI: right-click a packet → **Follow → TCP Stream** to read the full conversation as
text — this is usually the fastest way to read what a suspicious connection actually contains.

## Spotting C2 / beaconing behavior

- Look for **regular, periodic connections** to the same external host — a classic beacon pattern
  (e.g. a connection every 60 seconds regardless of user activity).
- Check for **unusual ports** (uncommon high ports, or common ports used for the wrong protocol — e.g.
  HTTP traffic that isn't actually HTTP on port 80/443).
- Check **DNS queries** for suspicious patterns: very long subdomains (possible DNS tunneling/exfil),
  queries to domains with no legitimate business reason, high query volume to a single domain.

```bash
tshark -r capture.pcap -Y "dns" -T fields -e dns.qry.name | sort | uniq -c | sort -rn | head -30
```

## Spotting exfiltration

```bash
tshark -r capture.pcap -q -z conv,tcp | sort -k 7 -rn | head -10   # sort conversations by bytes transferred
```

Large outbound transfers to an external IP, especially over an unusual protocol or at an unusual time,
are the main signal. Extract the actual file if HTTP/FTP was used:

```bash
tshark -r capture.pcap --export-objects http,extracted_files/
tshark -r capture.pcap --export-objects smb,extracted_files/
```

## Spotting exploitation attempts (web/service traffic)

```bash
tshark -r capture.pcap -Y "http.request" -T fields -e ip.src -e http.request.method -e http.request.uri |
  grep -Ei "union|select|\.\./|<script>|cmd="
```

## Wireshark display filter cheat sheet

| Goal | Filter |
|---|---|
| Traffic to/from one host | `ip.addr == 10.0.0.5` |
| One TCP conversation | `tcp.stream == 4` |
| HTTP requests only | `http.request` |
| DNS queries only | `dns` |
| Failed TCP connections (resets) | `tcp.flags.reset == 1` |
| Large packets (possible data transfer) | `frame.len > 1000` |
| Plaintext credentials in traffic | `http.authbasic or ftp.request.command == "PASS" or pop.request.command == "PASS"` |

## Workflow summary

1. Protocol hierarchy → what's in the capture at all.
2. Conversation summary (sorted by bytes) → spot the biggest/most unusual transfers.
3. Follow suspicious streams as text → read what actually happened.
4. Check DNS for tunneling/beaconing patterns.
5. Export objects (files transferred) if relevant to recover exfiltrated data or dropped tools.

Note every suspicious IP, port, and timestamp in your findings log — this maps directly into the
timeline you'll build for the final report (see `forensics/03-timeline-analysis.md`).
