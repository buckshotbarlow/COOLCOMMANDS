# Harvester HCI Installation Troubleshooting: NVMe, GPT, and `VMFS_volume_member`

This document captures a repeated Harvester HCI installation failure encountered on a server with two Samsung NVMe SSDs. It records the symptoms, evidence collected from the installer emergency shell, disk-cleanup commands, and next diagnostic steps.

> **Status:** The two drives were verified to have no detectable filesystem signatures or partitions after cleanup. A successful reinstall has **not** yet been confirmed. The root cause of the recurring installer error remains **unproven**.

## Intended disk layout

| Device | Hardware shown by installer | Intended role |
|---|---|---|
| `/dev/nvme0n1` | Samsung NVMe SSD, ~1.86 TiB | Harvester operating system |
| `/dev/nvme1n1` | Samsung NVMe SSD, ~1.86 TiB | Longhorn storage |
| `/dev/sda` | SanDisk USB, ~114.6 GiB | Bootable Harvester installation media — **do not wipe** |
| `/dev/loop0` | Installer live filesystem | **Do not wipe** |

Device names can change between boots. **Always confirm the model, capacity, and role before running destructive commands.**

## Installation failure

The Harvester installer repeatedly failed while attempting to mount the OS drive's EFI/boot partition:

```text
Failed mounting device /dev/nvme0n1p1 with label COS_GRUB
mount: /run/cos/efi: unknown filesystem type 'VMFS_volume_member'
Mount failed: exit status 32
Install failed: exit status 1
```

The installer log also referenced `mkfs.ext4`, disk partitioning, and an unsuccessful mount operation. The final mount failure alone does **not** demonstrate that the formatting command failed, nor does it prove a defective NVMe drive.

`VMFS_volume_member` is a signature associated with VMware VMFS storage. Possible explanations include stale VMFS metadata, an installer partitioning/formatting problem, or the kernel/installer observing a different disk state than expected. These remain hypotheses until log and signature evidence establish the sequence.

An earlier `fdisk -l` inspection also showed, on `/dev/nvme0n1`:

```text
GPT PMBR size mismatch ... will be corrected by write.
The backup GPT table is not on the end of the device.
```

Those warnings indicated inconsistent GPT metadata **at that time**. They should not be treated as a verified root cause of the recurring mount error.

## Accessing the installer emergency shell

From the failed installation screen, the installer suggested switching to another TTY using `Ctrl` + `Alt` + `F2` and inspecting `/var/log/console.log`. The screenshot showed the live shell's `rancher` credentials; use the credentials presented by your installer rather than assuming they are universal.

## Read-only inspection: start here

Identify the drives:

```bash
lsblk -o NAME,SIZE,MODEL,FSTYPE,PARTLABEL,PARTTYPE,MOUNTPOINTS
fdisk -l
```

Check filesystem signatures without modifying disks:

```bash
blkid -p /dev/nvme0n1
blkid -p /dev/nvme1n1
wipefs -n /dev/nvme0n1
wipefs -n /dev/nvme1n1
```

If an installer-created partition exists, inspect it too:

```bash
lsblk -f /dev/nvme0n1
blkid -p /dev/nvme0n1p1
wipefs -n /dev/nvme0n1p1
```

If `/dev/nvme0n1p1` does not exist, don't assume the command's failure means hardware failure; list the current partition layout first.

Validate GPT layout (read-only):

```bash
sgdisk -v /dev/nvme0n1
sgdisk -p /dev/nvme0n1
```

Check boot mode and kernel-reported storage errors:

```bash
cat /sys/firmware/efi/fw_platform_size
# Typically 64 for a 64-bit UEFI boot; missing path suggests non-UEFI boot.

dmesg | grep -iE 'nvme|I/O error|buffer|reset|timeout|critical|vfat|fat'
```

Inspect the detailed installer log **before wiping anything**:

```bash
grep -n -E 'mkfs|COS_GRUB|VMFS|nvme0n1p1|Failed mount|error' \
  /var/log/console.log | tail -100
```

Look for the **first failed operation**, not only the final mount error.

## Cleanup performed / considered

> [!CAUTION]
> **DESTRUCTIVE OPERATIONS BELOW.** They can make existing data permanently inaccessible. Only use them on a disk explicitly confirmed to be expendable, unmounted, and not the installer USB. Wiping the OS disk does not require wiping the Longhorn disk; avoid changing `/dev/nvme1n1` unless it must also be reset.

To remove old filesystem signatures and GPT/MBR partition metadata from the **verified OS target**:

```bash
# ONLY after verifying /dev/nvme0n1 is the expendable OS drive.
# Ensure neither the disk nor its partitions is mounted or in use.

# Remove signatures from any existing partitions, if present.
for p in /dev/nvme0n1p[0-9]*; do
    [ -b "$p" ] && wipefs -a "$p"
done

# Clear both primary and backup GPT/MBR partition structures.
sgdisk --zap-all /dev/nvme0n1

# Clear detectable signatures on the whole device.
wipefs -a /dev/nvme0n1
sync
```

The optional `partprobe /dev/nvme0n1` command can request a kernel partition-table reread. If the kernel retains stale partition nodes because the disk is in use, reboot the installer environment and inspect again.

If a **full-device logical discard** is specifically justified, the Linux `blkdiscard /dev/nvme0n1` command may be available on compatible SSDs. This destroys data and is **not required** merely to remove GPT/VMFS signatures; it is not a guaranteed secure erase. It was discussed as an option, **not established as necessary or performed** in this case.

### Verify after cleanup

```bash
lsblk -f /dev/nvme0n1
blkid -p /dev/nvme0n1
wipefs -n /dev/nvme0n1
sgdisk -v /dev/nvme0n1
```

In the final captured emergency-shell output:

- `lsblk -f` displayed only `nvme0n1` and `nvme1n1`, with no partitions or filesystems.
- `blkid -p` returned no signatures for either whole drive.
- `wipefs -n` returned no signatures for either whole drive.
- `sgdisk -v /dev/nvme0n1` reported **“No problems found”** with the disk's sectors free; it also displayed **“Creating new GPT entries in memory.”** This is not a disk write.

**Interpretation:** Basic on-disk partition and filesystem metadata appeared cleared **at the time of inspection**. This does not prove that a subsequent installer attempt will succeed, or that all hardware and installer behavior is healthy.

## Next installation attempt

1. Boot the installer in **UEFI** mode and verify the intended devices by model and capacity.
2. Set `/dev/nvme0n1` as the **Harvester OS** target.
3. Set `/dev/nvme1n1` as the **Longhorn data** target, if requested by the installer and intended by the deployment design.
4. Reattempt installation **without another wipe** unless new evidence calls for it.
5. If it fails, **do not clean up first**: capture `lsblk`, `blkid`, `wipefs -n` and the relevant `/var/log/console.log` lines from that exact failed attempt.

If the same `VMFS_volume_member` mount error returns after a verified clean disk, prioritize understanding partition creation, filesystem formatting, and mount order. In particular, find the commands and exit statuses immediately *preceding* the failed `mount` call. A later empty partition table may reflect cleanup performed after the failure and does not reveal what existed during the failed mount.

## Key takeaways

- The observed installation failure occurred on **`/dev/nvme0n1` (OS)**, not on the dedicated Longhorn disk.
- The error text reported a **VMFS** filesystem signature where Harvester expected to mount a boot-related partition.
- The drives' final inspection showed **no recognizable partitions or signatures**, and `sgdisk -v` found no GPT problem on the OS disk.
- **No verified root cause or successful fix** is established yet.
- **Stop wiping repeatedly** once the disk checks clean; preserve the logs from the next failed installation.

## Useful command reference

| Command | Purpose | Changes disk? |
|---|---|---|
| `lsblk -f` | Show devices, filesystems and partitions | No |
| `fdisk -l` | Display detected partition layout | No |
| `blkid -p DEVICE` | Probe filesystem signatures | No |
| `wipefs -n DEVICE` | List signatures without deleting | No |
| `sgdisk -v DEVICE` | Validate GPT metadata | No |
| `dmesg` | Read kernel messages | No |
| `sgdisk --zap-all DEVICE` | Remove GPT/MBR structures | **Yes** |
| `wipefs -a DEVICE` | Remove filesystem/partition signatures | **Yes** |
| `blkdiscard DEVICE` | Discard device's logical blocks (if supported) | **Yes** |

---

*Incident notes based on installer screenshots and emergency-shell output collected on November 23, 2026. Command availability can differ by Harvester installer version and emergency-shell environment.*
