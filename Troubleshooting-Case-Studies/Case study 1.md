# Case Study 1: Rebuilding a UEFI Boot Partition

## Issue

My Windows installation was not fully self-contained. The operating system was installed on one drive, while Windows Boot Manager was being loaded from a different physical drive.

This can cause problems if the drive containing the boot files is removed, fails, or is selected incorrectly in BIOS. I wanted the Windows drive to have its own EFI System Partition and boot files so it could boot independently in UEFI mode.

<img width="749" height="756" alt="ee96bd63-3957-4c23-8ff0-ec76dfbb727a" src="https://github.com/user-attachments/assets/2b9d1bb4-21f4-44e9-b4bc-d5912a2889ab" />

## Symptoms

* Windows Boot Manager was associated with a different drive from the main Windows installation.
* The system's boot configuration was more complicated than it needed to be.
* I needed to confirm which disk contained the Windows installation, which partitions were already present, and where the boot files should be rebuilt.

## Initial Checks

Before making any changes, I checked the disk layout using DiskPart.

I confirmed that:

* The Windows installation was accessible at `C:\Windows`.
* The disks were using GPT partition tables, which is required for UEFI booting.
* There were existing FAT32 system partitions and recovery partitions, so I needed to be careful not to modify the wrong one.
* The target Windows disk was Disk 2.

I used commands including:

```CMD
diskpart
list disk
select disk 2
list volume
```

## Troubleshooting Steps

1. Reviewed the disk and volume layout to identify the Windows partition and existing system partitions.

2. Created a dedicated EFI System Partition on the same disk as the Windows installation.

3. Formatted the new partition as FAT32 and labelled it `SYSTEM`.

4. Confirmed that it had the correct EFI partition type where i crossreferenced the GUID of the system partition online.

```text
c12a7328-f81f-11d2-ba4b-00a0c93ec93b
```

5. Assigned the EFI partition a temporary drive letter so the Windows boot files could be copied to it.

6. Rebuilt the UEFI boot files using `bcdboot`.

## Fix

The repair involved creating a 200 MB EFI System Partition, formatting it correctly, then rebuilding the Windows boot files.

```text
diskpart
select disk 2
create partition efi size=200
format quick fs=fat32 label="System"
assign letter=Z
exit
```

I then rebuilt the boot files:

```text
bcdboot C:\Windows /s Z: /f UEFI
```

This copied the required Windows boot files from the Windows installation to the new EFI System Partition.

## Result

A new FAT32 EFI System Partition was created on the same disk as the Windows installation.

The partition was labelled `SYSTEM`, configured as an EFI partition, and used to store rebuilt UEFI boot files. This made the Windows installation less dependent on another physical drive for its boot configuration.

Afterwards, the system was configured to boot in UEFI mode, with CSM inactive and Secure Boot enabled.

## What I Learned

* The Windows installation and Windows Boot Manager can be located on separate drives, even if Windows appears to work normally.
* UEFI systems use an EFI System Partition, normally formatted as FAT32, to store boot files.
* GPT disk layouts, the EFI partition type, and firmware settings all matter when repairing a boot issue.
* `bcdboot` can rebuild boot files from an existing Windows installation.
* DiskPart is powerful, so identifying the correct disk and partition before making changes is important.
