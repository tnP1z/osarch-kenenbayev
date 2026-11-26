| Figure | This computer | Command it came from |
|------|-------|----------------|
| CPU model and base clock | <AMD Ryzen AI 9 HX 370 w/ Radeon 890M> | `Get-CimInstance Win32_Processor` |
| Cores / threads | <12 / 24> | `Get-CimInstance Win32_Processor` |
| Total RAM | <32 GB> | `Get-CimInstance Win32_PhysicalMemory` |
| Total Physical Memory | <924 GB> | `systeminfo` |
| RAM modules / speed | <4 модулей / 8000 MT/s> | `Get-CimInstance Win32_PhysicalMemory` |
| Disk model | WD PC SN5000S SDEQNSJ-1T00-1002 | `Get-PhysicalDisk` |
| Disk type | <NVMe / SSD> | `Get-PhysicalDisk` (BusType, MediaType) |
| Free space (VM volume) | <584,1 GB> | `Get-Volume -DriveLetter C` |
| Firmware type | UEFI | `(Get-ComputerInfo).BiosFirmwareType` |
| Firmware version / date | <GA403WR.310 / 21.11.2025 5:00:00> | `Get-CimInstance Win32_BIOS` |
| Virtualization | True | `systeminfo` |
| OS edition, version and build number |  Edition: Майкрософт Windows 11 Домашняя
Verison: 10.0.26300 Н/Д построение 26300
Build number: Multiprocessor Free | `systeminfo` |