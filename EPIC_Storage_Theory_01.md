# Advanced LVM Guide - Manage of linear and stripped logical volumes

- This reference guide is intended for advance Linux users working with Logical Volume Manager (LVM). It covers the creation and resizing of Volume Groups (VGs) and Logical Volumes (LVs), including alternate examples for `lvextend`.

---


## Overview of LVM

- LVM allows flexible disk management by abstracting physical storage into logical volumes. It enables resizing, snapshots, and dynamic volume creation without downtime.

### Key Components

- **Physical Volume (PV)**: The raw disk or partition initialized for LVM.
- **Volume Group (VG)**: A pool of storage created from one or more PVs.
- **Logical Volume (LV)**: A virtual partition created from a VG.

---

### Understanding LVM Architecture in Linux

- This diagram illustrates the layered structure of **Logical Volume Management (LVM)** in Linux, showcasing how physical storage is abstracted and organized for flexible disk management.
  ![Screenshoot LVM Architecture ](./.attachments/lvm001.png =700x)

  #### LVM Hierarchy
  - **Disks/Partitions**
    `/dev/sdb` and `/dev/sdd`: Whole disks used as Physical Volumes (PVs).
    `/dev/sdc1`: A partition used as a PV.


  - **Physical Volumes (PVs)**
    Created from disks or partitions.
    Serve as the building blocks for Volume Groups.


  - **Volume Group (VG)**
    Named `vg_data`, with a Physical Extent (PE) size of 4 MiB.
    Combines multiple PVs into a single storage pool.



  - **Logical Volumes (LVs)**
    Allocated from the VG's extents.
    Two types shown:
      - **Linear LV Extents**: e.g., `/opt/app`, `/opt/backup`.
      - **Striped LV Extents** (`A` and `B`): e.g., `/opt/logs`.


## Creating Physical Volumes (PVs)
  - Before creating a Volume Group, initialize one or more physical devices as PVs:
    ```bash
    pvcreate /dev/sdb
    pvcreate /dev/sdc
    ```
  - Verify PVs:
    ```bash
    pvs
    ```

---


### Creating a Volume Group (VG)
  - Combine PVs into a Volume Group:
    ```bash
    vgcreate vg_data /dev/sdb /dev/sdc
    ```
  - Check VG details:
    ```bash
    vgs
    ```

---

### Creating a Logical Volume (LV)
  - Create an LV from the VG:
    ```bash
    lvcreate -L 110G -n lv_storage vg_data
    ````
    - `-L 110G`: Specifies size (110 GB)
    - `-n lv_storage`: Names the LV

  - Verify LV:
    ```bash
    lvs
    ```

  - Format and mount:
    ```bash
    mkfs.ext4 /dev/vg_data/lv_storage
    mkdir /opt/storage
    mount /dev/vg_data/lv_storage /opt/storage
    ```

---

## Resizing Logical Volumes

### Extending an LV
  - Extend LV by **specific size**:
    ```bash
    lvextend -L +5G /dev/vg_data/lv_storage
    ```
  - Extend LV using **all free space** in VG:
    ```bash
    lvextend -l +100%FREE /dev/vg_data/lv_storage
    ```
  - Extend LV by *specific size in MB*:
    ```bash
    lvextend -L +500M /dev/vg_data/lv_storage
    ```

  - After extending, resize the filesystem:
    ```bash
    resize2fs /dev/vg_data/lv_storage
    ```
    > XFS filesystem resize command ``xfs_growfs`` will received the mount point of the filesystem and not the block device
---

### Reducing an LV (Advanced)
  - Shrinking a filesystem:
    ```bash
    umount /opt/storage
    e2fsck -f /dev/vg_data/lv_storage 
    resize2fs /dev/vg_data/lv_storage 8G
    lvreduce -L 8G /dev/vg_data/lv_storage
    mount /dev/vg_data/lv_storage /opt/storage
    ```

    > **Warnings:**
    > - Reducing LVs can cause data loss if not done carefully. Always unmount and back up before reducing.
    > - XFS doesn't support shrinking

---

## Useful Commands Summary

- `pvs`, `vgs`, `lvs`: Display PV, VG, LV info
- `lvextend` : Extend LV size
- `lvreduce` : Reduce LV size
- `resize2fs`: Resize filesystem after LV changes

---

### Alternate Examples for `lvextend`
- Extend by **absolute size**:   `lvextend -L 20G /dev/vg_data/lv_storage`
- Extend by **relative size**:   `lvextend -L +2G /dev/vg_data/lv_storage`
- Extend using **all free space**:  `lvextend -l +100%FREE /dev/vg_data/lv_storage`

---

## LVM striping
  - LVM striping is used to distribute data across multiple Physical Volumes (PVs) for higher performance. Striping requires a minimum of two PVs and spreads writes across these devices.

    To create a striped LV, use:
    ```bash
    lvcreate -L 10G -n lv_striped vg_data -i 2 -I 64 /dev/sdb /dev/sdc
    ````
  - Breakdown
      ```lvcreate```: Command to create a logical volume.
      ```-L 10G```: Sets the size of the logical volume to 10 gigabytes.
      ```-n lv_striped```: Names the logical volume ```lv_striped```.
      ```vg_data```: The volume group from which the logical volume is created.
      ```-i 2```: Specifies 2 stripes, meaning data will be split across two physical volumes.
      ```-I 64```: Sets the stripe size to 64 KB, which it is the default size
      ```/dev/sdb /dev/sdc```: Physical volumes used for striping.

    > **Warning:**
    > When increasing the size of a **striped logical volume**, you must add the same number of **additional physical volumes (PVs)** as originally used.  
    > In this example, since the volume is striped across **2 PVs**, expanding it would require adding **2 more PVs**.  
    >  
    > Alternatively, you can increase the size of the **existing PVs** to accommodate the expansion without adding new devices.
