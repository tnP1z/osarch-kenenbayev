# Lab 6 — Live processes and memory pressure

Pair: none    Driver first half: Kenenbayev Kadirzhan
Machine: ASUS Zephyrus G14 GA403WR (host); guests `lab02-win11` (Windows 11, 8 GB) and `lab01-ubuntu` (Ubuntu 26.04.1 LTS, 4 GB) in VMware
Date: 22.11.2026

## The process I chose, and why

**VMware Tools — `vmtoolsd.exe` on Windows, `vmtoolsd` on Linux.** The lab compares one process across tools on
two different operating systems, so I needed a program that runs on both guests. VMware Tools does: it is the
same program, it starts by itself on both, and on both it runs as a service.

Windows runs **two** copies of it, Linux one:

| PID | Parent | What it is |
|---|---|---|
| Windows **3544** | 900 — `services.exe` | the VMware Tools **service**, started by the system |
| Windows 5488 | 3952 — `explorer.exe` | the user-session agent (clipboard, drag-and-drop, screen resize), started when I logged in |
| Linux **1147** | 1 — `systemd` | the VMware Tools **service** (`open-vm-tools`) |

Ubuntu Server has no desktop session, so it has no second copy. I compare the two services: **3544** on Windows
and **1147** on Linux.

## One process, three tools

| What | Task Manager (Details) | Process Explorer | htop (Linux) |
|---|---|---|---|
| PID | ИД процесса **3544** | PID **3544** | PID **1147** |
| Parent | — not shown | tree view; `Win32_Process`: **900, services.exe** | PPID **1** (systemd) |
| In RAM now | Рабочий набор **24 276 K** | Working Set **24 832 K** | RES **11 084 K** |
| Shared part of it | — | — | SHR **9 460 K** |
| Private, committed | Выделенная память **10 032 K** | Private Bytes **10 816 K** | — |
| Private and in RAM | Память (активный частный рабочий набор) **3 516 K** | — | — |
| Address space | — not shown | Virtual Size **2 151 819 052 K** (≈ 2 TB) | VIRT **111 M** |
| Open objects | Дескрипторы **399** | Handles **400** | — (9 file descriptors in `/proc/1147/fd`) |
| Threads | — | — | 4 (shown as separate rows) |

Text versions of the same figures: `evidence/process-windows.txt` (`Get-CimInstance Win32_Process`) and
`evidence/process-linux.txt` (`ps` and `/proc/1147/status`).

## Where the tools disagree

1. **The same number under three names.** Task Manager's *Выделенная память* (commit size), Process Explorer's
   *Private Bytes* and WMI's *PrivatePageCount* are the same measure — memory that belongs only to this process:
   10 032 K, 10 816 K and 10 788 K.
2. **Task Manager's default memory column is seven times smaller than Process Explorer's.** For the same process
   Task Manager shows 3 516 K and Process Explorer 24 832 K. Both are right: Task Manager's default counts only
   *private pages that are in RAM*, Process Explorer's Working Set counts *everything in RAM*, including shared
   DLL pages. It also shows that of 10.8 MB of private memory only 3.5 MB is in RAM at the moment.
3. **The figures drift between tools.** Working Set was 24 276 K in Task Manager, 24 832 K in Process Explorer and
   24 844 K in WMI — not a disagreement about meaning, but three readings taken a few minutes apart. A working
   set changes constantly.
4. **Handles are not file descriptors.** 400 handles on Windows against 9 file descriptors on Linux for the same
   program. A Windows handle also stands for threads, events, mutexes and registry keys; a Linux descriptor is
   only something opened as a file — a file, socket, pipe or device.
5. **Virtual size: 2 TB against 111 MB.** On Windows the same program has a 2 TB address space, on Linux 111 MB,
   while both have only a few dozen MB in RAM. Virtual size counts address space that is *reserved*, not memory
   that is used; 64-bit Windows reserves very large regions for every process. A big virtual number does not
   mean full RAM.
6. **htop makes one process look like four.** Rows 1206, 1208 and 1209 also say `/usr/bin/vmtoolsd` — they are
   its threads (`Threads: 4` in `/proc/1147/status`), which htop shows as separate rows by default.

## Memory pressure on the Linux guest

Before starting I took a snapshot, `before-lab6`. A second SSH window recorded `free -h` every two seconds while
a Python script in the first window allocated 100 MB per second. Full record: `evidence/memory-pressure.txt`.

| Time (UTC) | Stage | Mem used | buff/cache | available | Swap used |
|---|---|---|---|---|---|
| 12:37:07 | **Before** | 580 Mi | 701 Mi | 2.7 Gi | 0 B |
| 12:39:25 | allocation starts | 660 Mi | 713 Mi | 2.6 Gi | 0 B |
| 12:40:10 | RAM nearly full; **page cache starts shrinking** | 2.7 Gi | 693 Mi | 614 Mi | 0 B |
| 12:40:12 | **swapping starts** | 2.8 Gi | 557 Mi | 515 Mi | 4 Ki |
| 12:40:39 | swapping hard; cache nearly gone | 3.1 Gi | 202 Mi | 125 Mi | 1.0 Gi |
| 12:41:00 | RAM and swap both full | 3.2 Gi | 112 Mi | 41 Mi | 2.1 Gi (0 B free) |
| 12:41:02 | **python3 killed** | 1.7 Gi | 85 Mi | 1.6 Gi | 2.1 Gi |
| 12:42:27 | **After** | 526 Mi | 102 Mi | 2.8 Gi | 18 Mi |

The four stages from the lecture happened in order. First the kernel shrank the page cache (713 Mi → 85 Mi) to
make room. Then it started moving pages to swap, and swap filled from 0 to 2.1 GiB in 48 seconds. When there was
nothing left in RAM or swap, the OOM killer killed the process and `python3` printed `Killed`. Within two seconds
the memory was back. 18 Mi stayed in swap afterwards: pages of other processes that were moved out and have not
been needed since, so nothing has brought them back.

Responsiveness: I noticed how my tab lagged for couple seconds and information stopped displaying, after all it came back to normal behaviour.

### The kernel's record

`evidence/kernel-log-extract.txt`, from `journalctl -k`:

```
Nov 22 12:41:02 lab01 kernel: python3 invoked oom-killer: gfp_mask=0x140dca(...), order=0, oom_score_adj=0
Nov 22 12:41:02 lab01 kernel: oom-kill:constraint=CONSTRAINT_NONE,...,global_oom,...,task=python3,pid=2692,uid=1000
Nov 22 12:41:02 lab01 kernel: Out of memory: Killed process 2692 (python3) total-vm:5292328kB, anon-rss:2994420kB, ...
```

The time matches the jump in `memory-pressure.txt` to the second. The killed process had 5.0 GiB of address space
and 2.9 GiB of its own data in RAM — far more than any other process, so it had the highest `oom_score`.
`global_oom` means the whole machine ran out, not a limit set on one group. `journalctl -u systemd-oomd` has no
entries: Ubuntu Server does not run systemd-oomd, so the kernel's own OOM killer acted.

### The Windows side

On the Windows guest I ran the same kind of allocation in PowerShell until it failed, then read the System log.
`evidence/windows-event-2004.txt`:

```
TimeCreated  : 11/22/2026 6:52:22 PM
Id           : 2004
ProviderName : Microsoft-Windows-Resource-Exhaustion-Detector
Message      : Windows successfully diagnosed a low virtual memory condition. The following programs consumed the
               most virtual memory: powershell.exe (14000) consumed 13364527104 bytes, MsMpEng.exe (3668) consumed
               329601024 bytes, and explorer.exe (3952) consumed 134168576 bytes.
```

The two systems handle the same situation differently. **Linux kills** a process to free memory and logs whom it
killed. **Windows refuses** further allocations — PowerShell's allocation failed — and logs a diagnosis naming
the biggest consumers. PowerShell had committed 12.4 GiB on a guest with 8 GB of RAM: the page file absorbed the
difference until the commit limit was reached.

## What did not work the first time

1. **The PID variable came out empty.** After `PID=$(pgrep -x vmtoolsd)`, `echo $PID` printed nothing.
   `pgrep -a vmtools` showed the process was running as `1147 /usr/bin/vmtoolsd`, so the problem was my command,
   not the process. I set `PID=1147` directly.
2. **The first Windows event was the wrong one.** Filtering the System log only by `Id=2004` returned an event from
   `Microsoft-Windows-Setup` — *Successfully logged OS information* — which has the same ID but nothing to do with
   memory. Event IDs are only unique per provider. The low-memory event did not exist yet, because this guest had
   never run out of memory. I triggered it with a PowerShell allocation loop, then filtered by
   `ProviderName='Microsoft-Windows-Resource-Exhaustion-Detector'`.
3. **Process Explorer's columns were hard to find.** *Select Columns* spreads them over tabs: Private Bytes,
   Working Set Size and Virtual Size are on *Process Memory*, Handles on *Process Performance*.

## Evidence

- [evidence/task-manager.png](evidence/task-manager.png) — Task Manager, Details: both `vmtoolsd.exe` with PID, working set, private working set, commit size and handles
- [evidence/process-explorer.png](evidence/process-explorer.png) — Process Explorer: the same two processes with Private Bytes, Working Set, Handles and Virtual Size
- [evidence/htop.png](evidence/htop.png) — htop on Linux: `vmtoolsd` with PPID, VIRT, RES, SHR, and its threads as separate rows
- [evidence/process-windows.txt](evidence/process-windows.txt) — `Win32_Process` for both Windows copies: parent PIDs and memory as text
- [evidence/process-linux.txt](evidence/process-linux.txt) — `ps` and `/proc/1147/status`: PPID 1, VmSize, VmRSS, 4 threads, 9 file descriptors
- [evidence/memory-pressure.txt](evidence/memory-pressure.txt) — `free -h` before, every 2 seconds during, and after the allocation
- [evidence/kernel-log-extract.txt](evidence/kernel-log-extract.txt) — the kernel's OOM-killer record for python3 at 12:41:02, and the empty systemd-oomd journal
- [evidence/windows-event-2004.txt](evidence/windows-event-2004.txt) — Windows event 2004 from Resource-Exhaustion-Detector: low virtual memory, PowerShell the top consumer