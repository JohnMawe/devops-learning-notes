# Dual-Boot Linux and Windows: Installation, Disk Resizing, and Removing an OS

> Practical notes for installing Ubuntu alongside Windows, reclaiming disk space, and returning to a single operating system.
>
> **Scope:** UEFI/GPT systems, Ubuntu Desktop, and a single internal drive. Disk layouts vary; verify your own partitions before making changes.

## Contents

- [1. Understand the goal](#1-understand-the-goal)
- [2. Before changing partitions](#2-before-changing-partitions)
- [3. Install Linux alongside Windows](#3-install-linux-alongside-windows)
- [4. Inspect the disk layout](#4-inspect-the-disk-layout)
- [5. Expand Linux after removing Windows](#5-expand-linux-after-removing-windows)
- [6. Remove Linux and restore Windows](#6-remove-linux-and-restore-windows)
- [7. Keep Windows but give it more space again](#7-keep-windows-but-give-it-more-space-again)
- [8. Troubleshooting and recovery](#8-troubleshooting-and-recovery)
- [9. Command reference](#9-command-reference)
- [10. Lessons from my machine](#10-lessons-from-my-machine)

---

## 1. Understand the goal

Dual boot means Windows and Linux are installed on the same computer, and a boot menu lets you choose which operating system to start.

There are three different tasks that are easy to confuse:

1. **Install Linux alongside Windows:** shrink an existing partition to make unallocated space, then install Linux there.
2. **Remove Windows and expand Linux:** delete the confirmed Windows partitions, then expand Linux into the space that becomes available.
3. **Remove Linux and restore Windows:** remove Linux partitions and expand the Windows partition into suitable unallocated space. Repair the Windows boot setup if necessary.

Deleting a partition and expanding another are separate operations. The partition layout determines whether a partition can be expanded directly or must first be moved.

## 2. Before changing partitions

### Back up first

Back up files from both operating systems to an external drive or another safe location. Include documents, photos, project folders, password-manager recovery information, SSH keys, and other important data.

If Windows device encryption or BitLocker is enabled, make sure you have the recovery key before changing the disk or boot configuration.

### Check the boot mode and disk layout

From Ubuntu, open a terminal and run:

```bash
lsblk -o NAME,SIZE,FSTYPE,LABEL,MOUNTPOINTS
```

This lists disks, partitions, filesystems, labels, and mount points.

```bash
sudo parted -l
```

This shows disk partition tables, partition sizes, and flags.

```bash
df -hT /
```

This shows the filesystem mounted as Ubuntu's root (`/`) and its available space.

```bash
findmnt /boot/efi
```

This identifies the EFI System Partition mounted at `/boot/efi`, if present.

These are inspection commands; they do not intentionally modify partitions.

### Know common partition types

On a UEFI/GPT Windows-and-Ubuntu machine, you may see:

| Partition type | Typical purpose | General guidance |
|---|---|---|
| EFI System Partition (FAT32/vfat) | Stores boot files | Keep it if the operating system you want to keep uses it |
| Microsoft Reserved (MSR) | Windows disk management | Needed for a normal Windows setup; not a user data partition |
| Windows NTFS | Windows system/data | Contains Windows and possibly personal files |
| Windows Recovery | Windows recovery environment | Keep if you intend to retain Windows and its recovery tools |
| Linux ext4 | Ubuntu/Linux filesystem | Contains Linux files; often mounted at `/` |
| Linux swap or swap file | Memory support | Keep/configure as appropriate for the Linux setup |

**Do not identify a partition by size alone.** Use its filesystem, mount point, partition flags, and layout. Some EFI partitions may be shared by both operating systems.

## 3. Install Linux alongside Windows

The exact installer wording differs between Ubuntu releases and disk configurations.

### Preparation

1. Back up important files.
2. In Windows, check whether BitLocker/device encryption is enabled and save the recovery key if needed.
3. If Windows Fast Startup or hibernation is enabled, disable it before sharing NTFS partitions with Linux.
4. Open Windows Disk Management (`diskmgmt.msc`).
5. Right-click the Windows volume you intend to shrink and choose **Shrink Volume**.
6. Create unallocated space for Linux. Do not format this space as NTFS.
7. Create a bootable Ubuntu USB and boot it in the same UEFI mode as Windows.

### Install Ubuntu

1. Boot the Ubuntu USB and choose **Try Ubuntu** if you want to check hardware compatibility first.
2. Start the installer.
3. If the installer offers **Install Ubuntu alongside Windows**, verify the selected disk and proposed allocation before continuing.
4. If using manual partitioning, select only the unallocated space or partitions you have positively identified. Preserve the existing EFI System Partition used for booting.
5. Complete installation, reboot, and test both Ubuntu and Windows.

Manual partitioning is powerful but mistakes can erase data. If the installer choices are unclear, stop and check the disk layout before continuing.

### Why Windows may not shrink as much as expected

Windows may report a smaller maximum shrink amount than the total free space on the drive. Files that cannot easily be moved, such as certain system files or filesystem metadata, can limit the shrink operation. Do not blindly disable security or recovery features; research the specific blocker and maintain a backup before making changes.

## 4. Inspect the disk layout

### Example: list the partitions

```bash
lsblk -o NAME,SIZE,FSTYPE,LABEL,MOUNTPOINTS
sudo parted -l
```

Example output on a hypothetical NVMe drive might include:

```text
nvme0n1       256G
├─nvme0n1p1   100M  vfat   /boot/efi
├─nvme0n1p2    16M
├─nvme0n1p3   214G  ntfs   /media/user/WINDOWS
├─nvme0n1p4   889M  ntfs
└─nvme0n1p5  23.4G  ext4   /
```

This is an illustrative example, not a partition map to copy. Device names and partition numbers differ between computers.

- `nvme0n1` is the whole disk.
- `nvme0n1p5` is the fifth partition on that disk.
- A `/` mount point means that partition currently holds the root filesystem.
- A partition mounted under `/media/...` may be an automatically mounted data volume. Confirm what it contains before deleting it.

Do not assume that `p1` is always EFI or `p3` is always Windows. Verify every partition on your own machine.

## 5. Expand Linux after removing Windows

### Recommended method: use a live USB and GParted

Use this route if you want Linux to occupy the space currently used by Windows.

1. Back up your data and check that you can boot the Ubuntu USB.
2. Boot the USB and select **Try Ubuntu**. Do not boot into the installed Linux system for resizing its active root partition.
3. Open GParted. If it is not included in the live session and internet is available, you can try:

   ```bash
   sudo apt update
   sudo apt install gparted
   sudo gparted
   ```

4. Select the correct physical drive, checking its total capacity and partition layout.
5. Identify Windows partitions by filesystem, flags, and contents—not just by size.
6. Delete only the Windows partitions you have positively identified and no longer need. This may include the Windows NTFS system partition, MSR partition, and Windows recovery partition. A Windows data partition could contain personal files, so check it carefully.
7. Preserve the EFI System Partition if Linux uses it to boot. Do not delete Linux root or other Linux partitions.
8. Inspect the resulting unallocated space. If it is not adjacent to the Linux partition, the Linux partition may need to be moved before it can grow.
9. In GParted, use **Resize/Move** on the Linux partition. Expand it into the intended free space. Read the pending-operation list carefully before applying changes.
10. Apply the operations and wait. Moving a partition can take a long time; keep the computer on AC power and do not interrupt it.
11. Reboot into Linux and verify the result.

After reboot:

```bash
lsblk -o NAME,SIZE,FSTYPE,LABEL,MOUNTPOINTS
df -hT /
```

The size of the Linux partition and the root filesystem should reflect the new allocation.

### Important: moving a partition is riskier than growing it

If free space is **directly after** the Linux partition, growing it is generally straightforward. If free space is **before** the Linux partition, you may need to move the partition's start. That operation is more involved and has more risk if interrupted.

A graphical tool may refuse a resize depending on filesystem state, partition order, or mount status. Do not force it or substitute guessed `parted`, `fdisk`, `resize2fs`, or `mkfs` commands. Stop and reassess the layout.

### Why not just run `resize2fs`?

`resize2fs` can resize an ext filesystem, but it does not by itself rearrange the partition boundaries in the partition table. The partition must first be safely resized to provide the required space. For a root partition that must be moved or expanded, use a suitable live environment and a partitioning tool that understands the complete operation.

## 6. Remove Linux and restore Windows

This is the reverse scenario: keep Windows and reclaim the space previously used by Linux.

### Before starting

- Back up Linux files you want to keep.
- Create or verify Windows recovery/install media.
- If Windows device encryption/BitLocker is enabled, have the recovery key available.
- Confirm that Windows still boots and identify which EFI System Partition its boot files use.
- Do not delete the EFI partition simply because it was created during the original Windows installation; Linux may have added boot files to that same partition.

### General process

1. Boot into Windows and verify that it works.
2. Open **Disk Management** (`Win + X` → Disk Management, or run `diskmgmt.msc`).
3. Identify the Linux ext4 and swap partitions. Windows Disk Management may show Linux filesystems as unknown or without a recognizable filesystem.
4. If Linux partitions are not mounted by Windows, use a trusted partition manager or a live USB tool to remove only the confirmed Linux partitions.
5. Leave the EFI System Partition and any Windows recovery partitions required for Windows intact.
6. In Windows Disk Management, right-click the Windows volume and choose **Extend Volume** if the unallocated space is immediately to its right and the wizard permits it.
7. If the unallocated space is not adjacent to the Windows partition, Disk Management may not be able to extend it. Use a suitable partition manager only after backing up and verifying the layout; alternatively, leave the space unallocated or create a separate Windows data volume.
8. If the computer still displays a Linux/GRUB boot menu, select Windows Boot Manager in the firmware boot menu first. Repair Windows boot files only if needed, using Windows recovery tools and instructions appropriate to the machine.

**Do not format Linux partitions as NTFS until you are certain they contain nothing you need.** Formatting is destructive.

## 7. Keep Windows but give it more space again

You might want to retain both operating systems but change their relative sizes. This can be more complicated than choosing a new size in a slider.

- To give Windows more space, the unallocated space normally needs to be adjacent to the Windows partition.
- To give Linux more space, the unallocated space normally needs to be adjacent to the Linux partition, or the Linux partition may need to be moved.
- Shrinking a partition creates unallocated space; it does not automatically add that space to another partition.
- Moving partitions can take time and should be done from a live environment when the partition being changed is mounted as the active root filesystem.
- Some layouts cannot be safely rearranged using Windows Disk Management alone.

Before resizing, capture a fresh `lsblk` and `sudo parted -l` output and make a plan based on the actual partition order.

## 8. Troubleshooting and recovery

### The laptop boots straight into Windows or Ubuntu

Use the firmware boot menu (on many HP laptops, press `Esc`, then `F9` during startup) to select the desired boot entry. Firmware key sequences vary by model.

If a boot entry is missing, inspect the EFI layout and use the operating system's recovery tools. Do not delete or format the EFI System Partition as a first troubleshooting step.

### Linux does not have the expected free space

Run:

```bash
lsblk -o NAME,SIZE,FSTYPE,LABEL,MOUNTPOINTS
df -hT /
```

`lsblk` shows the partition sizes; `df` shows the mounted filesystem's usable capacity. A partition may have been expanded without the filesystem having been expanded, or the operation may not have completed. Diagnose before attempting any further resize.

### GParted reports a partition is busy or mounted

Boot from a live USB and choose **Try Ubuntu**. Ensure the target Linux partition is not mounted. Do not resize the active root partition from the installed OS.

### The operation takes a long time

Moving a large partition can take a long time. Keep the laptop connected to power and do not force shutdown. Interrupting a move or resize can corrupt the filesystem or cause data loss.

### The disk is GPT and the computer uses UEFI

This is normal for many modern computers. Preserve the EFI System Partition required by the operating system you intend to keep. Avoid mixing legacy/CSM and UEFI boot modes during installation or recovery.

## 9. Command reference

These commands inspect the current layout:

```bash
# List disks and partitions
lsblk -o NAME,SIZE,FSTYPE,LABEL,MOUNTPOINTS

# Show partition tables and flags
sudo parted -l

# Show root filesystem capacity
df -hT /

# Show the mounted EFI partition
findmnt /boot/efi
```

Install GParted in an Ubuntu session when appropriate:

```bash
sudo apt update
sudo apt install gparted
sudo gparted
```

Use the graphical partition editor from a **live USB** to change the installed system's root partition. Review all pending operations before applying them.

These notes intentionally do not provide blind deletion or resize commands such as `parted rm`, `mkfs`, or `resize2fs` with hard-coded device names. The correct commands depend on the exact disk and partition layout; copying a command for another machine can destroy data.

## 10. Lessons from my machine

My example machine is an HP EliteBook 845 G7 with a roughly 256 GB NVMe drive, GPT partitioning, and UEFI boot.

The observed layout was approximately:

| Partition | Size shown | Filesystem / role |
|---|---:|---|
| `/dev/nvme0n1p1` | 100–105 MB | FAT32/vfat EFI System Partition, mounted at `/boot/efi` |
| `/dev/nvme0n1p2` | 16–17 MB | Microsoft Reserved partition |
| `/dev/nvme0n1p3` | 214–230 GB | NTFS Windows partition |
| `/dev/nvme0n1p4` | 889–932 MB | NTFS Windows recovery partition |
| `/dev/nvme0n1p5` | 23.4–25.1 GB | ext4 Ubuntu root partition, mounted at `/` |

Small differences can result from rounded output or differing tools. This table records the observed layout; it is not a universal recipe.

The key lesson: Windows occupied the large middle section of the disk, while Ubuntu's root partition was at the end. After removing Windows, reclaiming the space for Ubuntu required attention to partition order: the free space was before Ubuntu, so the partition had to be moved/expanded from a live environment rather than merely growing a filesystem in place.

### Personal checklist for a future disk change

- [ ] Back up files from both operating systems.
- [ ] Save any BitLocker/device-encryption recovery key.
- [ ] Capture `lsblk` and `sudo parted -l` output.
- [ ] Identify the EFI partition used by the OS I intend to keep.
- [ ] Confirm which partition contains the OS and which contains personal data.
- [ ] Boot from a live USB before moving/resizing the installed Linux root partition.
- [ ] Review the complete pending-operation list in GParted.
- [ ] Keep the laptop on AC power during partition operations.
- [ ] Verify the boot process and filesystem capacity after reboot.

---

## Further reading

- [Ubuntu Desktop installation tutorial](https://ubuntu.com/tutorials/install-ubuntu-desktop)
- [GParted documentation](https://gparted.org/documentation.php)
- [Microsoft: Extend a basic volume](https://learn.microsoft.com/en-us/windows-server/storage/disk-management/extend-a-basic-volume)

> **Final safety rule:** Always work from the partition layout you have just inspected. Never copy partition numbers from a tutorial and assume they match your own computer.
