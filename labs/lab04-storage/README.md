# Lab 4 — Partition, mirror, fail, recover

Pair: none    Driver first half: Kenenbayev Kadirzhan
Machine: ASUS Zephyrus G14 GA403WR (host); guests `lab02-win11` (Windows 11) and `lab01-ubuntu` (Ubuntu Server) in VMware Workstation Pro
Date: 06.11.2026

## What I did

1. Added one 2 GB disk to the Windows guest (default controller) and three 2 GB SCSI disks to the Ubuntu guest, with both guests powered off.
2. **Windows:** initialised the new disk as GPT, created one partition with letter E, formatted it NTFS with label LAB4.
3. **Linux:** recorded the layout, partitioned `sdb` and `sdc` with `parted`, built a RAID 1 mirror `md0` with `mdadm`, formatted it ext4 and mounted it at `/mnt/mirror`.
4. Wrote `proof.txt` and a 200 MB `random.bin` onto the mirror and saved their SHA-256 checksums.
5. Failed `sdc1`, saved the degraded state immediately, removed it, unmounted and remounted the mirror and checked the checksums again.
6. Partitioned `sdd`, added it to the array, watched the recovery finish and saved the rebuilt state.

Main commands:

```powershell
Initialize-Disk -Number 1 -PartitionStyle GPT
New-Partition -DiskNumber 1 -UseMaximumSize -DriveLetter E
Format-Volume -DriveLetter E -FileSystem NTFS -NewFileSystemLabel LAB4
```

```bash
sudo parted /dev/sdb --script mklabel gpt mkpart lab4 1MiB 100% set 1 raid on
sudo parted /dev/sdc --script mklabel gpt mkpart lab4 1MiB 100% set 1 raid on
sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb1 /dev/sdc1
sudo mkfs.ext4 -L lab4 /dev/md0
sudo mount /dev/md0 /mnt/mirror
sudo mdadm /dev/md0 --fail /dev/sdc1
sudo mdadm /dev/md0 --remove /dev/sdc1
sudo parted /dev/sdd --script mklabel gpt mkpart lab4 1MiB 100% set 1 raid on
sudo mdadm /dev/md0 --add /dev/sdd1
```

## Windows

**Disk 0 (system disk): GPT, because the guest boots in UEFI mode** (`$env:firmware_type` = `UEFI`).
UEFI firmware boots from a GPT disk with an EFI system partition; a Legacy BIOS boot would have used MBR.
Windows 11 requires UEFI, so its system disk is GPT by necessity, not by choice.

**Disk 1 (new disk): GPT, by choice.** For a 2 GB data disk that nothing boots from, MBR would work just as well.
I chose GPT to match the system disk and because it keeps a backup copy of the partition table at the end of the
disk with a checksum, so one damaged sector does not lose the layout. The choice would actually matter for a disk
over 2 TB (MBR cannot address it), for more than four primary partitions, or for a disk a UEFI machine has to
boot from. One visible side effect: on GPT, Windows created a 16 MB Microsoft Reserved partition automatically
(partition 1 below) before my partition.

```
PS> Get-Disk
Number Friendly Name Serial Number HealthStatus OperationalStatus Total Size Partition Style
------ ------------- ------------- ------------ ----------------- ---------- ---------------
0      VMware Vir... XXXX          Healthy      Online                 64 GB GPT
1      VMware Vir... XXXX          Healthy      Online                  2 GB GPT

PS> Get-Partition -DiskNumber 1
PartitionNumber  DriveLetter Offset       Size Type
---------------  ----------- ------       ---- ----
1                            17408    15.98 MB Reserved
2                E           16777216  1.98 GB Basic
```

## Linux layout — before

The three new disks are the ones that are 2G, have no partitions under them and no mount point.

```
NAME                       SIZE TYPE FSTYPE      MOUNTPOINT
sda                         25G disk
├─sda1                       1M part
├─sda2                       2G part ext4        /boot
└─sda3                      23G part LVM2_member
  └─ubuntu--vg-ubuntu--lv 11.5G lvm  ext4        /
sdb                          2G disk
sdc                          2G disk
sdd                          2G disk
sr0                       1024M rom
```

## Linux layout — after the rebuild

`md0` now sits under `sdb1` and `sdd1`; `sdc1` still carries a RAID label but belongs to no array.

```
NAME                       SIZE TYPE  FSTYPE            MOUNTPOINT
sda                         25G disk
├─sda1                       1M part
├─sda2                       2G part  ext4              /boot
└─sda3                      23G part  LVM2_member
  └─ubuntu--vg-ubuntu--lv 11.5G lvm   ext4              /
sdb                          2G disk
└─sdb1                       2G part  linux_raid_member
  └─md0                      2G raid1 ext4              /mnt/mirror
sdc                          2G disk
└─sdc1                       2G part  linux_raid_member
sdd                          2G disk
└─sdd1                       2G part  linux_raid_member
  └─md0                      2G raid1 ext4              /mnt/mirror
sr0                       1024M rom
```

| Role | Device |
|---|---|
| Mirror half A | sdb1 |
| Failed half | sdc1 |
| Replacement | sdd1 |

## Result: the three states of the array

| State | Saved at | `/proc/mdstat` | `mdadm --detail` State | Events |
|---|---|---|---|---|
| Healthy | 20:49:48 | `sdc1[1] sdb1[0]` `[2/2] [UU]` | clean | 18 |
| Degraded | 20:52:15 | `sdc1[1](F) sdb1[0]` `[2/1] [U_]` | clean, degraded | 20 |
| Rebuilt | 21:13:02 | `sdd1[2] sdb1[0]` `[2/2] [UU]` | clean | 73 |

**The data survived.** The checksums taken before the failure and after it — with the mirror unmounted and
remounted, so the files were read from the one surviving disk and not from RAM — are identical, and
`diff checksum-before.txt checksum-after-failure.txt && echo MATCH` printed `MATCH`:

```
a9bed30ce1a9528da423d0335834db07bc915249677009c687047b95acd02328  /mnt/mirror/proof.txt
d5a8e7070942e6d69d2a8f5d7427feaa9a3e64b3bf751346588f09cdce18c6e8  /mnt/mirror/random.bin
```

## The question

*Your colleague deleted the wrong file from the mirror an hour ago. Does RAID 1 help? Why?*

No. RAID 1 copies every write to both disks at the moment it happens, and a deletion is also a write, so the file
disappeared from both halves of the mirror an hour ago — exactly as `random.bin` stayed identical on both halves in
my experiment. A mirror protects against a disk dying, not against a person or a program changing the data;
what would have helped is a backup or a snapshot taken before the deletion.

## What did not work the first time

1. **I started the Linux part without adding the three disks.** I skipped the step of adding the 2 GB disks in
   VMware and went straight to the commands. `lsblk` showed only my system disk `sda`, the snap `loop` devices and
   the CD drive — there was no `sdb`, `sdc` or `sdd` to build a mirror from. Fix: shut the guest down, added the
   three disks in *Edit virtual machine settings*, booted again, and ran `lsblk` again so that
   `lsblk-before.txt` shows the new disks.

2. **I broke the guest's bootloader by writing to the wrong disk.** Because `sdb` did not exist, I replaced it
   in the `parted` command with `sda1` — a partition of my system disk:
   `sudo parted /dev/sda1 --script mklabel gpt ...`. `sda1` is the 1 MB `bios_grub` partition where GRUB keeps
   the code that starts Ubuntu on a BIOS-booted machine. Writing a new partition table into it overwrote that
   code, and on the next boot the guest hung on the loading screen. My data partitions (`sda2`, `sda3`) were not
   touched. I had no snapshot taken just before the lab, only `clean-install` from Lab 1, which would have lost all
   later work, so I repaired it instead: took a snapshot of the broken state, booted the Ubuntu Server ISO,
   opened a shell from the installer's Help menu, mounted my system and reinstalled GRUB:

   ```bash
   vgchange -ay
   mount /dev/mapper/ubuntu--vg-ubuntu--lv /mnt
   mount /dev/sda2 /mnt/boot
   for d in dev proc sys run; do mount --bind /$d /mnt/$d; done
   chroot /mnt
   grub-install --target=i386-pc /dev/sda
   update-grub
   ```

   After a reboot the guest started normally. What I learned: the brief's warning is literal — before any command
   that writes to a disk, check in `lsblk` that it is 2G, has no children and no mount point; and take a snapshot
   *before* a lab that writes to disks, not after something breaks.

3. **A typo on Windows.** I typed `Get_Disk` instead of `Get-Disk` and got `CommandNotFoundException`. PowerShell
   commands are always *Verb-Noun* with a hyphen.

## Evidence

- [evidence/windows-disk.txt](evidence/windows-disk.txt) — PowerShell transcript: the new disk RAW → GPT, partition E, NTFS, and `firmware_type` = UEFI
- [evidence/lsblk-before.txt](evidence/lsblk-before.txt) — Linux layout before: three empty 2 GB disks `sdb`, `sdc`, `sdd`
- [evidence/array-healthy.txt](evidence/array-healthy.txt) — mirror built and synchronised: `[2/2] [UU]`, State clean, saved at 20:49
- [evidence/checksum-before.txt](evidence/checksum-before.txt) — SHA-256 of both files on the healthy mirror
- [evidence/array-degraded.txt](evidence/array-degraded.txt) — `sdc1` marked faulty: `[2/1] [U_]`, State clean, degraded, saved at 20:52
- [evidence/checksum-after-failure.txt](evidence/checksum-after-failure.txt) — SHA-256 after unmount/remount on one disk: identical to before
- [evidence/array-rebuilt.txt](evidence/array-rebuilt.txt) — mirror rebuilt onto `sdd1`: `[2/2] [UU]`, State clean, saved at 21:13
- [evidence/lsblk-after.txt](evidence/lsblk-after.txt) — layout after: `md0` under `sdb1` and `sdd1`, `sdc1` left outside the array
