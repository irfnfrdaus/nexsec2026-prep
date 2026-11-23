# Forensics — Memory Analysis (Volatility)

If you're handed a memory image (`.raw`, `.mem`, `.vmem`, `.dmp`), Volatility 3 is the standard tool
(see `tools/tool-list.md` for install). Workflow below assumes Volatility 3 syntax
(`vol -f <image> <plugin>`); Volatility 2 uses `vol.py -f <image> --profile=<profile> <plugin>`.

## 1. Identify the OS/profile

```bash
vol -f memory.raw windows.info      # Volatility 3 auto-detects for Windows images
# Volatility 2: vol.py -f memory.raw imageinfo
```

## 2. Process listing — two views matter

```bash
vol -f memory.raw windows.pslist    # standard process list (walks the EPROCESS linked list)
vol -f memory.raw windows.psscan    # scans memory for process structures directly
```

**Compare the two outputs.** A process present in `psscan` but missing from `pslist` is a strong
rootkit/process-hiding indicator (the linked list was tampered with to hide it from normal
enumeration).

```bash
vol -f memory.raw windows.pstree    # parent/child view — spot a shell spawned from an odd parent
```

## 3. Network connections at time of capture

```bash
vol -f memory.raw windows.netscan   # connections/listeners present when the memory was captured
```

## 4. Command-line arguments & environment

```bash
vol -f memory.raw windows.cmdline   # full command line per process — often reveals attacker commands directly
vol -f memory.raw windows.envars    # environment variables per process
```

## 5. Injected code / hollowed processes

```bash
vol -f memory.raw windows.malfind   # flags memory regions with executable+writable permissions and PE headers — classic injection indicator
vol -f memory.raw windows.hollowfind 2>/dev/null   # if available — detects process hollowing specifically
```

## 6. DLLs loaded per process

```bash
vol -f memory.raw windows.dlllist --pid <pid>
```

Look for DLLs loaded from unusual paths (`%TEMP%`, `%APPDATA%`) rather than `System32`.

## 7. Extracting a suspicious process for further analysis

```bash
vol -f memory.raw windows.dumpfiles --pid <pid>   # dump files mapped by a process
vol -f memory.raw windows.pslist --pid <pid> --dump   # dump the process executable itself (Vol3 syntax varies by version — check `vol -h` for the exact flag)
```

## 8. Linux memory images

```bash
vol -f memory.lime linux.pslist
vol -f memory.lime linux.pstree
vol -f memory.lime linux.bash      # recovers bash history straight from memory, even if cleared on disk
vol -f memory.lime linux.netstat
```

`linux.bash` is particularly valuable — it recovers command history from the process's memory space
even when the attacker cleared `.bash_history` on disk.

## Workflow summary

1. `pslist` + `psscan` → spot hidden processes.
2. `pstree` → spot odd parent/child relationships (e.g. `winword.exe` spawning `cmd.exe`).
3. `netscan` → active connections at capture time.
4. `cmdline` → what was actually typed/executed.
5. `malfind` → injected/hollowed code.
6. `dumpfiles`/process dump → pull the actual malicious binary out for a closer look (strings, hash
   lookup if internet/VT access is permitted).

Write down the plugin output and PID/path for every finding — this is your evidentiary trail for the
report.
