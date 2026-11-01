# Ticket 1 — Test machine for a new developer

## 1. What was asked

> From: Aigerim T. (Office Manager)
> Subject: test machine for new dev
>
> Hi, we have someone starting on Monday and he needs a test computer
> set up. Nothing special, just so he can try things without breaking
> anything. Can you write down how to do it so I can do the next one
> myself? Thanks

## 2. What was actually needed

**What the request left out, and what I assumed instead of an answer.**

| Not said | What I would have asked | What I assumed |
|---|---|---|
| Physical PC or virtual machine | “Do you want to buy a computer, or does he have a laptop already?” | Virtual machines on an existing laptop. “Try things without breaking anything” is exactly what snapshots give, and a second physical PC would be bought only to be broken. |
| Which operating system | “What will he develop, and for what?” | Two machines, Ubuntu Server and Windows 11: server-side work usually runs on Linux, and testing clients and office software needs Windows. |
| How much memory and disk the host has | “Which laptop will it run on?” | A laptop like mine: 32 GB of memory, 584 GB free. The document states the minimum (16 GB, 120 GB free) so it can be checked before starting. |
| Internet, and whether the machines see each other | “Does he need to install software? Should his test server and client talk to each other?” | Yes to both — NAT networking (section 3). |
| Who keeps the passwords | “Who should hold the Ubuntu and the Windows encryption passwords?” | The person who builds the machine writes them down; the document says so at the two places they are created. |

**Where the request is wrong.**

- **“Nothing special.”** There are at least five decisions that are easy to get wrong: memory, disk, network
  mode, TPM encryption for Windows 11, and the firewall. A list of clicks without the reasons would work exactly
  once, on exactly one laptop. That is why every step in BUILD.md says *why*, not only *what*.
- **“Write down how to do it so I can do the next one myself.”** Rebuilding from scratch for every new person
  takes about two hours and invites mistakes. The better process is to build one clean machine once and **clone**
  it (VMware: VM → Manage → Clone) for each new starter. I still wrote the full build document, because it was
  asked for and because the template has to be built once by somebody — but the recommendation is the clone.
- **Licences.** The Windows 11 Evaluation stops being usable after **90 days**. For a test machine that is fine;
  if the developer will rely on it longer, the company needs a real Windows licence. VMware Workstation Pro itself
  is free for commercial use since November 2024, so that part costs nothing.

**Who the real reader is.** Not the developer — Aigerim. She has never opened a hypervisor, so BUILD.md starts
before VMware is installed, explains every term it uses, and has a **Check** after each important step so she
knows whether she is still on track without asking anyone.

## 3. What I did

1. Rebuilt both guests from nothing in VMware Workstation Pro 26H1u1 (version 26.0.1), writing BUILD.md as I went and
   taking a screenshot at each step.
2. Calculated the resources from my host's real figures (below).
3. Chose NAT networking, added a firewall rule for ping instead of disabling the firewall, and took a
   `clean-install` snapshot of each machine.
4. Verified internet access and connectivity in both directions (section 4).

**Resources, calculated from the host.**
Host: ASUS ROG Zephyrus G14 GA403WR — AMD Ryzen AI 9 HX 370, **12 cores / 24 threads**, **32 GB** memory
(31.1 GiB visible to Windows), **584.1 GB** free on the VM volume (Lab 0, Lab 3).
*(Lab 1 recorded “12 threads”; Lab 3's `Win32_Processor` shows 12 cores and 24 logical processors, so the correct
figure is 24 threads.)*

| | Memory | vCPU | Disk |
|---|---|---|---|
| Host idle use | ≈ 16.4 GB used (Lab 1) | — | — |
| Ubuntu guest | 4 GB | 2 | 25 GB, dynamic |
| Windows guest | 8 GB | 2 | 64 GB, dynamic |
| **Left for the host with both guests running** | ≈ 32 − 16.4 − 12 = **≈ 3.6 GB** | 24 − 2 − 2 = **20 threads** | 584 − 25 − 64 ≈ **495 GB**, minus snapshots |

The memory is the tight resource: with both guests running, the host keeps only about 3.6 GB in reserve. I did not
go higher because in Lab 2 I promised the guests 40 GB on a 32 GB host and the host stalled for several seconds.
On a 16 GB laptop (the minimum in BUILD.md) both guests should not run at the same time.

**Settings that differ from the wizard's defaults** (defaults read from the wizard's “Ready to Create Virtual
Machine” page before customising).

| Machine | Setting | Wizard default | My value | Why |
|---|---|---|---|---|
| Both | OS installation | Easy Install (if ISO given) | “Install later” | Easy Install skips choices I need, e.g. OpenSSH |
| Ubuntu | Memory | 6144 MB | **4096 MB** | With 6 GB for Ubuntu and 8 GB for Windows the host would keep ≈ 1.6 GB; 4 GB is plenty for Ubuntu Server |
| Ubuntu | Disk | 20 GB (recommended) | **25 GB** | ~5 GB for the OS, the rest for tools; dynamic, so it costs nothing until used |
| Ubuntu | OpenSSH server | not ticked | **ticked** | File copy and remote login; in Lab 1 I had to enable it by hand |
| Windows | Memory | 4096 MB | **8192 MB** | 4 GB is Windows 11's minimum and it is slow there |
| Windows | Encryption | — (must be chosen) | **TPM files only** | Required for the virtual TPM Windows 11 needs; full encryption is slower |
| Windows | Account | Microsoft account | **local account** (Domain join instead) | Does not tie a test machine to a personal account |
| Windows | Firewall | blocks inbound ping | **rule for ICMPv4 echo, local subnet only** | Lets the guests check each other without opening anything else |

**Kept at the default, on purpose:** 2 processors for each guest (enough for both, the host keeps 20 of 24
threads), 64 GB disk for Windows (Windows 11's minimum, dynamic anyway), and NAT networking.

NAT (VMnet8) is the default, but it was chosen, not just left. Bridged was rejected
because the campus network filters by MAC address; Host-only because it has no internet.

**One thing that went wrong.** The first `ping` from Ubuntu to Windows failed while Windows → Ubuntu worked.
The asymmetry pointed at Windows rather than the network: Windows Defender Firewall drops incoming ICMP echo
requests by default. I added a single inbound rule for ICMPv4 type 8 from the local subnet (BUILD.md, step 15)
instead of switching the firewall off, and ping then worked both ways. The same problem now has its own row in
BUILD.md's troubleshooting table, so Aigerim does not have to diagnose it herself.

## 4. Evidence

- [evidence/01-new-vm-wizard.png](evidence/01-new-vm-wizard.png) — the wizard page where “install later” is chosen, avoiding Easy Install
- [evidence/02-disk-settings.png](evidence/02-disk-settings.png) — 25 GB dynamic disk for Ubuntu
- [evidence/03-network-adapter.png](evidence/03-network-adapter.png) — adapter set to NAT
- [evidence/04-guest-uname.txt](evidence/04-guest-uname.txt) — `uname -a` output: the Ubuntu guest runs and which kernel it has
- [evidence/05-ping-between-guests.txt](evidence/05-ping-between-guests.txt) — both guests reach 8.8.8.8 and each other, in both directions
- [evidence/06-snapshot-list.png](evidence/06-snapshot-list.png) — a `clean-install` snapshot exists to return to
- [evidence/build/](evidence/build/) — the remaining step screenshots referenced from BUILD.md

## 5. What I would do differently

I would build the machines **once as a template and clone them**, and write BUILD.md around the clone: three
steps for Aigerim instead of seventeen, and every new starter gets an identical machine. I would also tick
OpenSSH during the Ubuntu installation from the start — in Lab 1 I skipped it and had to enable SSH by hand
afterwards, which is exactly the kind of step a non-technical reader cannot recover from.
