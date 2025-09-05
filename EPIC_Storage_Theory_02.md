# Advanced LVM Guide – Mirroring and Backup

- This section expands on the previous LVM guide by introducing **LVM mirroring**, a method for redundancy, and **splitting mirrors** for backup purposes. It is intended for advanced Linux users managing critical data storage.

---

## What is LVM Mirroring?
  - LVM mirroring creates a logical volume with multiple copies of data across different physical volumes (PVs). This ensures data redundancy and resilience against disk failure.
    - **Use Case**: Ideal for systems requiring high availability or backup snapshots.
    - **Mirror vs. RAID1**: LVM mirrors behave similarly to RAID1 but are managed through LVM tools.

---

### Understanding Mirror LVM

- This diagram illustrates the layered structure of **Logical Volume Management (LVM)** in Linux, showcasing how physical storage is abstracted and organized for flexible disk management.
  ![Screenshoot LVM Architecture ](./.attachments/lvm002.png =700x)

  #### LVM Hierarchy
  - **Disks/Partitions**
    `/dev/sdb`: Whole disk used as Physical Volumes (PVs).
    `/dev/sdc1`: A partition used as a PV.


  - **Physical Volumes (PVs)**
    Created from disks or partitions.
    Serve as the building blocks for Volume Groups.


  - **Volume Group (VG)**
    Named `vg_data`, with a Physical Extent (PE) size of 4 MiB.
    Combines multiple PVs into a single storage pool.


  - **Logical Volumes (LVs)**
    Allocated from the VG's extents.
    One type shown:
      - **Mirror LV Extents**: e.g., `/opt/data`.
      



## Creating a Mirrored Logical Volume
  - To create a mirrored LV with two copies:
    ```bash
    lvcreate -L 10G -n lv_mirror -m1 vg_data /dev/sdb /dev/sdc1
    ```

    ### Breakdown:
    - `-L 10G`: Size of the logical volume.
    - `-n lv_mirror`: Name of the logical volume.
    - `-m1`: Specifies one mirror (i.e., two copies total).
    - `/dev/sdb /dev/sdc1`: Physical volumes used for mirroring.

    ### Verify:
    ```bash
    lvs -a -o +devices
    ```

---

## Break a LVM Mirror
  - You can simulate a failure
    ### Step 1: Remove LVM Metadata from the Failed Disk.
    ```bash
    dd if=/dev/zero of=/dev/sdb bs=1M count=1
    ```
    > Overwrites the first 1 MiB of the disk with zeros, erasing the LVM metadata and making the disk unrecognizable as a physical volume.

    ### Step 2: Then check status:
    ```bash
    lvs -a -o +devices
    ```

---

## Recover a Broken LVM Mirror
  - If a device was removed or failed, you can reintroduce it to restore the mirror.
    ### Step 1: Remove Missing PVs from the Volume Group
    Ensure the device is initialized as a physical volume:
    ```bash
    vgreduce --removemissing --mirrorsonly --force /dev/vg_data
    ```

    ### Step 2: Then check status:
    ```bash
    lvs -a -o +devices
    ```
    > Warning messages about missing physical volume are not present any more

    ### Step 3: Add the device back to the mirror
    Adds the (now clean) disk back into the volume group.

    ```bash
    vgextend /dev/vg_data /dev/sdb
    ```

    ### Step 4: Repair the Mirror
    ```bash
    lvconvert --repair /dev/vg_data/lv_mirror
    ```

    ### Step 5: Verify mirror status
    ```bash
    lvs -a -o +devices
    ```

## Splitting a Mirror for Backup
  - Splitting a mirror allows you to detach one copy of the data for backup or migration.

    ### Step 1: Split the mirror
    ```bash
    lvconvert --splitmirror 1 --trackchanges /dev/vg_data/lv_mirror
    ```

    ### Breakdown:
    - `--splitmirror 1`: Detaches one mirror leg.
    - `--trackchanges`: Track changes between the original logical volume so can be merge later.
    - `/dev/vg_data/lv_mirror`: Original mirrored LV.

    ### Step 2: Then check status:
    ```bash
    lvs -a -o +devices
    ```
    > **Notice:** the rimage_1 now is a regular standalone logical volume

    ### Step 3: Mount and backup
    ```bash
    mkdir /mnt/backup
    mount /dev/vg_data/lv_backup_rimage_1 /mnt/backup
    ```

    You can now copy the contents to external storage or archive.
    > **Warning:** The filesystem UUID will be duplicated, depending on the filesystem type the steps can be different

---

## Reintegrating the Split Mirror with `--merge`
  - After completing your backup using the split mirror, you can reintegrate the detached leg (`lv_backup`) back into the original mirrored logical volume (`lv_mirror`) using the `lvconvert --merge` command.
    ### Step 1: Unmount the backup volume
    ```bash
    umount /mnt/backup
    ```

    ### Step 2: Merge the backup LV back into the mirror
    ```bash
    lvconvert --merge /dev/vg_data/lv_backup_rimage_1
    ```

    This command will:
      - Reattach the split mirror leg to `lv_mirror`
      - Convert `lv_backup_rimage_1` after the merge completes

    ### Step 3: Verify the mirror status
    ```bash
    lvs -a -o +devices
    ```

    > **Note**: The merge will only succeed if `lv_backup` has not been modified and `lv_mirror` is still intact.


## Important Notes

  - **Consistency**: Always ensure the mirror is in sync before splitting.
  - **Backup Strategy**: Use split mirrors for point-in-time backups without downtime.
  - **Restoration**: You can later reattach the mirror using `lvconvert --merge`.

---

## Useful Commands Summary
  | Command | Description |
  |--------|-------------|
  | `lvcreate -m1` | Create mirrored LV |
  | `lvconvert --splitmirror` | Split mirror for backup |
  | `lvconvert --merge` | Merge split mirror back |
  | `lvs -a -o +devices` | Show LV and device mapping |
