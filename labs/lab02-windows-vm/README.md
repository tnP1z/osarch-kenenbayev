# Lab 2 — A Windows guest, and a network between them

Pair: none                 
Driver first half: Kenenbayev Kadirzhan    
Machine: ASUS Zephyrus G14 GA403WR   
Date: 25.10.2026 


## 1. Virtual Machines

- **Ubuntu VM:** `lab01-ubuntu`
- **Windows VM:** `lab02-win11`
- **Hypervisor:** VMware Workstation Pro
- **Network mode:** NAT (VMnet8)

## 2. Network Mode Choice

I used NAT for both guests. The requirement was for both guests to communicate with each other and access the internet. VMware NAT meets both requirements. I did not choose Bridged because campus networks may block unregistered virtual machine MAC addresses. I did not choose Host-only because it does not provide internet access.

## 3. Network Diagram

```text
                 Internet
                    |
             Host: Windows laptop
                    |
               VMware NAT
                 (VMnet8)
                    |
             Shared virtual network
               /             \
      Ubuntu Server        Windows 11
      lab01-ubuntu         lab02-win11
      IP: [192.168.174.131]      IP: [192.168.174.132]
```

## 4. Connectivity and Firewall

Windows successfully pinged Ubuntu. Initially, Ubuntu could not ping Windows because Windows Defender Firewall blocked incoming ICMP echo requests.

I added the following specific inbound firewall rule in an elevated PowerShell session:

`New-NetFirewallRule -DisplayName "Lab02 - allow ping (ICMPv4 echo)" -Direction Inbound -Protocol ICMPv4 -IcmpType 8 -RemoteAddress LocalSubnet -Action Allow`

After adding the rule, ping worked in both directions. The firewall remained enabled.

## 5. Internet Connectivity

- Windows to `8.8.8.8`: [SUCCESS]
- Ubuntu to `8.8.8.8`: [SUCCESS]

## 6. Memory Overcommitment Experiment

- Host RAM: 32 GB
- Ubuntu memory during experiment: 16 GB
- Windows memory during experiment: 24 GB
- Total promised: 40 GB
- Ratio to host RAM: 1.25×

Before the experiment, host memory showed [15]. During the experiment, host memory showed [31]. The host [First 5 sec were some lags, but after everything was pretty smooth].

## 7. Restored Configuration

After the experiment, I restored Ubuntu to 4 GB and Windows to 8 GB. Both guests booted and network connectivity was checked again.

## 8. Evidence

- `evidence/ipconfig-windows.txt`
- `evidence/ip-addr-linux.txt`
- `evidence/ping-both-ways.txt`
- `evidence/host-memory-before.png`
- `evidence/host-memory-after.png`

## 9. Problems and Observations

While I was trying to run overcommitment experiment 
I could not run "sudo apt install -y stress-ng 
stress-ng --vm 1 --vm-bytes 90% --vm-keep --timeout 300s" commands due to incorrect addresses of repositories inside of Ubuntu.

Observations:
- Available memory before: [15]
- Memory usage after: [31]
- Available memory after: [1]
