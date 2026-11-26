Task 1 a commands:

 `Get-CimInstance Win32_Processor`
 `Get-CimInstance Win32_Processor`
 `systeminfo`
 `Get-CimInstance Win32_BIOS`
 `(Get-ComputerInfo).BiosFirmwareType`
 `systeminfo`
 `Get-CimInstance Win32_PhysicalMemory`
 `Get-PhysicalDisk`
 `Get-PhysicalDisk`
 `Get-Volume -DriveLetter C`
 `(Get-ComputerInfo).BiosFirmwareType - Firmware: UEFI or legacy BIOS; Secure Boot on or off
 `Get-CimInstance Win32_BIOS - Firmware: UEFI or legacy BIOS; Secure Boot on or off
 `systeminfo` - Firmware: UEFI or legacy BIOS; Secure Boot on or off
  systeminfo|

Task 1 b:

Part	Specification
CPU	4 cores / 4 threads, 3.2 GHz (Intel, 2012); VT-x supported, disabled in firmware
Memory	8 GB DDR3 as 2 × 4 GB modules; 4 slots on the board, board supports up to 32 GB
Storage	1 TB HDD, 7200 rpm, SATA, 640 GB free; one free SATA port and power connector
Firmware	Legacy BIOS mode; no TPM
OS	Windows 10 Pro 22H2

My first Ubuntu vm machine specifics:

My host machine has 12 threads, 32 GB of RAM, and 584.1 GB free on the VM volume (measured in Lab 0). Idle, the host already uses about 16,4 GB of RAM. 
- I gave the guest **2 vCPU** of my 12 threads 
- I gave it **4 GB** of my RAM 
- I gave it a **25 GB dynamically allocated disk** 

Windows vm machine: 
- I gave the guest **4 vCPU** of my 12 threads 
- I gave it **8 GB** of my RAM 
- I gave it a **64 GB dynamically allocated disk** 

Both guests?
Answering on question about "**Could this desktop run both of the guests you built in Labs 1–2 at the same time, at the sizes you gave them (take them from your build log)?run both of the guests you built in Labs 1–2 at the same time**": I do not think so, because my own machine have way pore cores and ram memory and by looking at this old office desktop I am pretty sure that both Virtual machines will not run at the same time due to it's weaknesses.

One purchase.
I would definetely purchase RAM memory due to it's lack, because it's necessary for running virtual machines and yes this old office desktop is able to accept more RAM cause it has 4 slots and it's upgradable till 32 GB of RAM. Specifically, I would purchase DDR4 RAM, 2x8 GB so it upgrades till 16 GB of RAM in total. 
About the Operating system, it's good even if it is Win 10 Pro 22H2, the 10th version is stable and this build is also good due to Microsoft security supportion. Nothing to add. 