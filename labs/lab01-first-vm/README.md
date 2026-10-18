# Lab 1 — Your first virtual machine

Pair: none                 
Driver first half: Kenenbayev Kadirzhan    
Machine: ASUS Zephyrus G14 GA403WR   
Date: 18.10.2026   

## 1. Resource Justification
My host machine has 12 threads, 32 GB of RAM, and 584.1 GB free on the VM volume (measured in Lab 0). Idle, the host already uses about 16,4 GB of RAM. 
- I gave the guest **2 vCPU** of my 12 threads because it provides a good balance for server tasks without starving the host.
- I gave it **4 GB** of my RAM because Ubuntu Server runs comfortably on it, and I am keeping 6 GB reserved for the Windows guest in Lab 2.
- I gave it a **25 GB dynamically allocated disk** because it leaves plenty of free space on my volume while giving the OS room to grow.

## 2. What Was Broken and How It Was Confirmed
- **Chosen Option:** [Option A - deleted cat /etc/hostname]
- **Break Command:** `[sudo rm /etc/hostname]`
- **Confirmation Command (Broken):** `[cat /etc/hostname -> No such file or directory]`
- **Rollback:** Reverted to the `clean-install` snapshot.
- **Confirmation Command (Healthy):** `[cat /etc/hostname -> prints lab01]`

## 3. What Did Not Work / Troubleshooting
- [The installation went smoothly, but I had a trouble about SSH because it was not active(running), so I had to write down this command `< sudo systemctl enable --now ssh >` and everything rolled back successfully]

## Evidence

- [evidence/](evidence/) — one line per file on what it proves
