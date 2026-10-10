# Lab 0 — Inventory your own machine
Pair: < none >
Driver first half: < Kenenbayev Kadirzhan>
Machine: < ASUS Zephyrus G14 GA403WR>
Date: <10.10.2026>

## What I did
```
Get-CimInstance Win32_Processor | Select-Object Name, NumberOfCores, 
NumberOfLogicalProcessors
<Name                                   : AMD Ryzen AI 9 HX 370 w/ Radeon 890M           
NumberOfCores                           : 12
NumberOfLogicalProcessors               : 24>
```
```
Get-CimInstance Win32_PhysicalMemory | Select-Object BankLabel, Capacity, 
Speed
<BankLabel    Manufacturer   Capacity Speed ConfiguredClockSpeed
---------    ------------   -------- ----- --------------------
P0 CHANNEL A Samsung      8589934592  8000                 8000
P0 CHANNEL B Samsung      8589934592  8000                 8000
P0 CHANNEL C Samsung      8589934592  8000                 8000
P0 CHANNEL D Samsung      8589934592  8000                 8000

Total GB: 32>
```
```
Get-PhysicalDisk | Select-Object FriendlyName, MediaType, BusType, Size
Get-Volume -DriveLetter C
<FriendlyName : WD PC SN5000S SDEQNSJ-1T00-1002
MediaType    : SSD
BusType      : NVMe
Size         : 1024209543168

DriveLetter FileSystemLabel         Size SizeRemaining
----------- ---------------         ---- -------------
                              1005580288     120553472
            MYASUS             268435456     169652224
            RESTORE          30064766976    2166263808
C           OS              992575033344  627120222208>
```
```
Get-CimInstance Win32_BIOS | Select-Object SMBIOSBIOSVersion, ReleaseDate
(Get-ComputerInfo).BiosFirmwareType
<Manufacturer                            SMBIOSBIOSVersion ReleaseDate       
------------                            ----------------- -----------       
American Megatrends International, LLC. GA403WR.310       21.11.2025 5:00:00>
```
```
systeminfo | Select-String "Virtualization","hypervisor"
<Hyper-V requirements: A low-level shell has been detected. 
The functions required for Hyper-V will not be displayed. 
Name                                            VirtualizationFirmwareEnabled
----                                            -----------------------------
AMD Ryzen AI 9 HX 370 w/ Radeon 890M                                     True>
```

## Result

| What | Value | Where I got it |
|------|-------|----------------|
| CPU model | <AMD Ryzen AI 9 HX 370 w/ Radeon 890M> | `Get-CimInstance Win32_Processor` |
| Cores / threads | <12 / 24> | `Get-CimInstance Win32_Processor` |
| Total RAM | <32 GB> | `Get-CimInstance Win32_PhysicalMemory` |
| RAM modules / speed | <4 модулей / 8000 MT/s> | `Get-CimInstance Win32_PhysicalMemory` |
| Disk model | <WD PC SN5000S SDEQNSJ-1T00-1002> | `Get-PhysicalDisk` |
| Disk type | <NVMe / SSD> | `Get-PhysicalDisk` (BusType, MediaType) |
| Free space (VM volume) | <584,1 GB> | `Get-Volume -DriveLetter C` |
| Firmware type | <UEFI> | `(Get-ComputerInfo).BiosFirmwareType` |
| Firmware version / date | <GA403WR.310 / 21.11.2025 5:00:00> | `Get-CimInstance Win32_BIOS` |
| Virtualization | <enabled> | `systeminfo` |

## What did not work the first time 
- < to create cpu, memory and disk .txt files due to inccorect code line as (-Encoding utf8) and and save data into these txt files>

## Evidence

- [evidence/cpu.txt](evidence/cpu.txt) - CPU Model - AMD Ryzen AI 9 HX 370 w/ Radeon 890M, Cores- 12, Threads - 24
- [evidence/memory.txt](evidence/memory.txt) - RAM size - 32 GB, 4 modules, 8000 MT/s 
- [evidence/disk.txt](evidence/disk.txt) - disk model-WD PC SN5000S SDEQNSJ-1T00-1002, type - UEFI, free space - 584,1