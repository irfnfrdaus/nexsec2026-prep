# Forensics — Timeline Analysis

A timeline is the backbone of a forensic report: it turns scattered artifacts into a narrative the
organizers (or any third party) can follow and verify.

## Building a super-timeline

```bash
# Sleuth Kit body file from a disk image (filesystem timestamps: MACB — Modified, Accessed, Changed, Born)
fls -r -m / evidence.img > bodyfile.txt
mactime -b bodyfile.txt -d > fs_timeline.csv
```

For a richer timeline that also pulls in Event Logs, registry, and other artifacts, `log2timeline`
(part of the Plaso project) is the standard tool if available in your environment:

```bash
log2timeline.py plaso.dump evidence.img
psort.py -o l2tcsv -w full_timeline.csv plaso.dump
```

## What to merge into one timeline

- Filesystem MACB timestamps (file creation/modification/access/metadata-change).
- Event Log entries (Windows) or auth/syslog entries (Linux) — see the OS-specific log-analysis guides.
- Prefetch execution timestamps (Windows) — `PECmd` output.
- Browser history, if relevant to the scenario.
- Registry key last-write timestamps (each registry key has one — useful for pinpointing when a
  persistence mechanism was installed).

## Reading a timeline effectively

1. **Anchor first:** find the earliest point you're confident is "before the incident" (a known-clean
   baseline, or simply the earliest evidence you're given). Everything is relative to this.
2. **Scan for clusters:** real attacker activity tends to cluster in bursts (initial access → quick
   follow-on actions), separated by gaps. A sudden burst of file creation/modification is often more
   informative than any single event.
3. **Work backward from your strongest finding:** if you found a webshell or a malicious scheduled
   task, work the timeline backward from its creation timestamp to find the actual exploitation that
   planted it, and forward from it to find what the attacker did next.
4. **Note gaps and inconsistencies explicitly:** a log that was cleared, a time range with suspiciously
   no data, or a system clock that looks wrong are findings in their own right — say so rather than
   silently working around them.

## Common timestamp pitfalls

- **Timezones.** Confirm what timezone the system/logs are in before building a cross-source timeline
  — mixing UTC and local time silently is a classic source of a wrong narrative.
- **Timestomping.** An attacker can modify a file's MACB timestamps to blend in with surrounding files.
  If a "recently planted" file has an old-looking timestamp but its *content* (compile date inside a
  binary, internal metadata) tells a different story, trust the content over the filesystem timestamp
  and say so in your findings.
- **Copy vs. original timestamps.** Copying evidence can reset Access (and sometimes Modified)
  timestamps if you're not careful — work from forensic tools/read-only mounts, not drag-and-drop
  copies, wherever possible.

## Report structure that follows naturally from a timeline

1. Scope / evidence received.
2. Timeline of events (table: timestamp, source, event, significance).
3. Root cause / entry point.
4. Impact (what the attacker achieved).
5. Conclusion — stated only to the extent the evidence supports it (the guideline is explicit about
   this: "state only what the evidence supports").
