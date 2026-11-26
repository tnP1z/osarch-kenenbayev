Allocation
 Ubuntu Server: 2 vCPUs, 2 GB RAM, 25 GB disk (dynamically allocated).
 Adapter 1: Bridged. It connects Ubuntu to the office network and to the internet.
 Adapter 2: Internal Network "labnet" with the address 192.168.50.10. Windows uses this to reach Ubuntu.
 Runs nginx on port 80.

Windows 11: 2 vCPUs, 4 GB RAM, 64 GB disk (dynamically allocated), EFI and TPM 2.0 turned on in the VM settings.
 Adapter 1: NAT. It gives Windows the internet.
 Adapter 2: Internal Network "labnet" with the address 192.168.50.20. Windows reaches Ubuntu through this.

 Arithmetic
I assume the host needs 2 threads, 4 GB RAM and about 30 GB of disk for itself (Windows, a browser, updates).
 CPU: ⟨16⟩ − 2 − 2 = ⟨12⟩ threads left for the host. It needs 2, so this is fine.
 Memory: ⟨16⟩ − 2 − 4 = ⟨10 GB⟩ left. It needs 4 GB, so this is fine.
 Disk: ⟨200⟩ − 25 − 64 = ⟨111 GB⟩ left. It needs 30 GB, so this is fine.
The disks are dynamic, so at first they take much less space (about 25 GB together). I stil counted their full size, so the plan works even when they grow.

 Justification

 Why Bridged for Ubuntu? With Bridged, Ubuntu gets it s own IP in the office network, like any other computer. So other people in the office can open the website. With NAT they couldn't reach it. NAT + port forwarding would also work, but then everyone connectss to my laptop's IP and I have to open port 80 on my laptop. Bridged is easier.

 Why NAT for Windows? With NAT, Windows can use the internet, but nobody from outside can connect to it. Its IP is 10.0.2.15, and that address doesn't exist in the office network. That's what "not visible means. If I used Bridged, Windows would show up in the office network, and we don't want that.

 Why Internal Network for the second adapter? It's a small private network just for the two VMs. My laptop and the office can't see it. So Windows can talk to Ubuntu and still stay hidden. Host-only would work too, but my laptop doesn't need to be in that network. Two normal NAT adapters won't work, because each VM gets its own separate NAT and they can't see each other.

 Why dynamic disks ? A dynamic disk only takes as much space as the VM really uses, so my laptop keeps its free space. A fixed disk would take all 89 GB at once.

 Why these sizes? Ubuntu Server has no GUI, so 2 vCPU and 2 GB are enough for a web server. Windows 11 gets its minimum: 2 cores, 4 GB RAM, 64 GB disk, TPM 2.0. VirtualBox has a virtual TPM, so it doesn't matter if my laptop has one. Nothing is below the minimum.

 Verification plan

Ubuntu gets 192.168.1.50 on the office work

1. Each guest sees the right resources
 On Ubuntu: nproc  2, free -h  about 1.9Gi total, lsblk  sda 25G
 On Windows: Get-CimInstance Win32_ComputerSystem | Select NumberOfLogicalProcessors, TotalPhysicalMemory  2 and about 4 GB. Get-Volume -DriveLetter C  about 64 GB.

2. The office can open the web server
 On Ubuntu: systemctl status nginx active (running)
 On another office computer: curl.exe http://192.168.1.50 a page with "Welcome to nginx!"
 Or: Test-NetConnection 192.168.1.50 -Port 80 TcpTestSucceeded : True

3. Windows can reach Ubuntu
 On Windows: Test-NetConnection 192.168.50.10 -Port 80  TcpTestSucceeded : True

4. Windows is hidden from the office
 On Windows: ipconfig  only 10.0.2.15 and 192.168.50.20. There is no office address (192.168.1.x), so there is nothing for the office to connect to.
 On another office computer: ping 10.0.2.15  Request timed out

5. Both guests have internet
 On Ubuntu: curl -I https://example.com  HTTP/2 200
 On Windows: Test-NetConnection example.com -Port 443  TcpTestSucceeded : True

6. The host is still usable
 On the host, with both VMs running, Task Manager shows about 4 GB or more of free memory and the CPU is not stuck at 100%.