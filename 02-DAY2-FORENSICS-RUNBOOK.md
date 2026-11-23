# Day 2 Forensics Runbook — "I don't know where to start" → follow this

Day 2 is analysis-only, no live systems. Same idea as the Day 1 runbook: a linear path through the
forensics guides so you're never staring at evidence with no idea what to do next.

## 0. Before you touch anything (2 min)

- [ ] Confirm exactly what evidence you've been given (disk image? memory image? pcap? logs only?)
      and what question you're actually being asked to answer.
- [ ] Copy `templates/forensic-report-template.md` for this evidence item.
- [ ] Hash the original evidence file and make a working copy — see
      `forensics/04-chain-of-custody.md`. **Never analyze the original directly.**

## 1. Identify what you're working with (5 min)

```bash
file evidence.img      # or evidence.mem, evidence.pcap, etc.
```

- Disk image → go to step 2.
- Memory image → go to step 3.
- Packet capture → go to step 4.
- Logs only (no image) → go to step 5, apply the same log-analysis guides from Day 1
  (`linux/03-log-analysis.md`, `windows/03-event-log-analysis.md`) directly against the provided files.

## 2. Disk image workflow

Follow `forensics/01-disk-imaging-triage.md` in full:
1. Identify filesystem/partitions (`mmls`, `file`).
2. Mount read-only or open in Autopsy.
3. Pull the OS-appropriate artifacts listed in that guide (passwd/shadow, registry hives, Event Logs,
   Prefetch, $MFT, etc.).
4. Build a filesystem timeline (`fls` + `mactime`, or Autopsy's built-in timeline view).
→ Then go to step 5 (timeline) below.

## 3. Memory image workflow

Follow `forensics/02-memory-forensics.md` in full, in this order:
1. `pslist` + `psscan` (compare for hidden processes)
2. `pstree` (odd parent/child relationships)
3. `netscan` (connections at capture time)
4. `cmdline` (what was actually run)
5. `malfind` (injected code)
6. Dump and examine anything suspicious found above.
→ Then go to step 5 (timeline) below.

## 4. Packet capture workflow

Follow `forensics/05-network-pcap-analysis.md` in full, in this order:
1. Protocol hierarchy + conversation summary (what's here, what's big).
2. Follow suspicious streams as text.
3. Check DNS for tunneling/beaconing.
4. Export any transferred files/objects.
→ Then go to step 5 (timeline) below.

## 5. Build the timeline (the step that ties everything together)

Follow `forensics/03-timeline-analysis.md`:
- Anchor on a known reference point.
- Merge every timestamped artifact you pulled in steps 2–4 into one chronological view.
- Find the earliest anomalous event — that's your starting point for the narrative.
- Note any gaps (cleared logs, timestomped files) explicitly rather than filling them in.

## 6. Write the conclusion

Use `templates/forensic-report-template.md` sections 4–6 specifically:
- State the root cause / entry point.
- State the impact.
- **For every claim, cite the specific evidence it comes from** — this is what `04-chain-of-custody.md`
  calls "findings a third party could verify," and it's explicitly part of what's graded.
- Flag inference vs. direct evidence explicitly wherever you're not 100% certain.

## 7. Before you submit

Run through `templates/report-submission-checklist.md`.

## When you're truly stuck

1. Re-check you've actually identified the right evidence type (step 1) — misreading a memory image as
   a disk image, for example, wastes a lot of time.
2. Re-run the automated tooling (Autopsy's ingest modules, Volatility's plugin list) — sometimes the
   answer is in output you already generated but didn't scroll to.
3. Ask the team — someone with a different piece of evidence may hold the missing piece of context.
4. Write down what the evidence does and doesn't show, even if your conclusion is partial — the
   guideline explicitly values honestly-scoped findings over a confident guess.
