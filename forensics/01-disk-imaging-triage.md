# Forensics — Disk Image Triage (Day 2)

Day 2 is analysis-only: you'll be given evidence (likely a disk image, memory dump, or packet
capture) and asked to reconstruct what happened and justify your conclusions. No live systems.

## First steps with any provided image

1. **Never work on the original file if you can avoid it.** Make a working copy; verify the copy's
   hash matches before analysis, and keep the original read-only.

```bash
sha256sum original.img > original.sha256
cp original.img working_copy.img
sha256sum -c original.sha256   # after copying, verify original is unchanged
```

2. **Identify the image/filesystem type:**

```bash
file evidence.img
fdisk -l evidence.img          # list partitions in a raw/dd image
mmls evidence.img              # Sleuth Kit — shows partition layout clearly
```

3. **Mount read-only for a first look** (if the tooling and permissions allow it):

```bash
# Loopback mount a partition read-only (adjust offset from mmls/fdisk output: offset * sector size)
mkdir /mnt/evidence
mount -o ro,loop,offset=<bytes> evidence.img /mnt/evidence
```

Or use Autopsy / The Sleuth Kit (see `tools/tool-list.md`) to parse the filesystem without mounting at
all — generally safer and preserves more metadata.

## Key artifact locations once you have access to the filesystem

### Linux image
- `/var/log/*` — see `linux/03-log-analysis.md` for what to grep for.
- `/etc/passwd`, `/etc/shadow`, `/etc/sudoers*` — account anomalies.
- `/home/*/.bash_history`, `/root/.bash_history` — command history (may be incomplete/tampered).
- `/tmp`, `/dev/shm`, `/var/tmp` — common locations for dropped tools/payloads.
- Cron/systemd persistence locations — see `linux/02-persistence-hunting.md`.

### Windows image
- `C:\Windows\System32\config\` — registry hives (SAM, SYSTEM, SOFTWARE, SECURITY) — parse offline
  with a tool like `RegRipper` or `Registry Explorer` (see tool list).
- `C:\Windows\System32\winevt\Logs\*.evtx` — Event Logs, parse with `Get-WinEvent -Path` pointed at
  the exported .evtx, or a GUI tool like Event Log Explorer.
- `$MFT` (Master File Table) — file system metadata, timestamps, even for deleted files. Parse with
  `MFTECmd` (Eric Zimmerman's tools) or Autopsy.
- `$UsnJrnl` — change journal, shows file creation/deletion/rename history even after deletion.
- Prefetch files (`C:\Windows\Prefetch\*.pf`) — evidence of what executables actually ran, and how
  many times. Parse with `PECmd`.
- `NTUSER.DAT` per user profile — user-specific registry (recently run, typed URLs, etc.).

## Timeline building

```bash
# Sleuth Kit — build a full filesystem timeline from an image
fls -r -m / evidence.img > bodyfile.txt
mactime -b bodyfile.txt -d > timeline.csv
```

Load `timeline.csv` into a spreadsheet and filter around the window the scenario/evidence suggests the
incident occurred — look for clusters of file creation/modification right before or during suspected
attacker activity.

## Chain of custody

Keep a simple log for every piece of evidence you touch: filename, original hash, your working-copy
hash, what tool you ran, and when. This directly supports the guideline's requirement for "findings a
third party could verify." See `templates/forensic-report-template.md`.
