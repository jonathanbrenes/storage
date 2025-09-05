# Storage
## LVM - Mirroring
  Deploy this VM. This will automatically create a VM and set an initial LVM/filesystem structure.
   
  [![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fjonathanbrenes%2Fstorage%2Fmain%2FLabs%2Fepic_storage_lab2.json)
  
## Recovering a broken LVM mirror
  - For this exercise, it's recommended to use these commands: 
    ```bash
    lvs -o +devices
    ls -l /dev/disk/azure/scsi1
    ```

  - To convert an existing logical volume ```/dev/vgdata01/lvdata01``` into a mirrored volume, first add the new disk ```/dev/disk/azure/scsi1/lun1``` to the volume group ```vgdata01``` using vgextend. Then, use lvconvert to add a mirror to the logical volume.

    ```bash
    vgextend /dev/vgdata01 /dev/disk/azure/scsi1/lun1
    lvconvert --mirrors 1 /dev/vgdata01/lvdata01 /dev/disk/azure/scsi1/lun1
    ```
  - Monitor the status of the raid using ```lvs -a -o +devices```. Check for the column ```Cpy%Sync```
  - Once the sync is at 100%, remove LVM Metadata ```/dev/disk/azure/scsi1/lun0```
    ```bash
    dd if=/dev/zero of=/dev/disk/azure/scsi1/lun0 bs=1M count=1
    ```
  - Check the status of the LV using ```lvs```.
    ```bash
    lvs -a -o +devices
    ```
  - Remove Missing PVs from the Volume Group
    ```bash
    vgreduce --removemissing --mirrorsonly --force /dev/vgdata01 # Removes the missing physical volume from the volume group, specifically for mirrored volumes.
    ```

  - Check the status of the LV using ```lvs```.
    ```bash
    lvs -a -o +devices
    ```
    > Warning messages about missing physical volume are not present any more


  - Add back the ```/dev/disk/azure/scsi1/lun0``` to the volume group ```vgdata01```.
    ```bash
    vgextend /dev/vgdata01 /dev/disk/azure/scsi1/lun0
    ```
  - Repair the Mirror
    Re-establishes the mirror for the logical volume, using the newly added disk.
    ```bash
    lvconvert --repair /dev/vgdata01/lvdata01
    ```
  - Verify Mirror Status using ```lvs -a -o +devices```
    ```bash
    lvs -a -o +devices
    ```
    > Checks that the mirror is now restored and both disks are being used.

## Split LVM mirrors
  - This exercise demonstrates how to split a mirrored logical volume in LVM, use the split for backup, and optionally merge it back.

  - Verify the Mirror Status
    Check that your logical volume is mirrored and healthy:

    ```bash
    lvs -a -o +devices
    ```

  - Split the Mirror with Change Tracking
    To split one leg of the mirror and enable future merging, use the `--trackchanges` option:

    ```bash
    lvconvert --splitmirror 1 --trackchanges /dev/vgdata01/lvdata01
    ```

    - `--splitmirror 1`: Detaches one mirror leg.
    - `--trackchanges`: Tracks changes on the original LV for safe merging later.
    - `-n lvbackup`: Names the new split LV as `lvbackup`.


  - Use the Split Volume for Backup
    Mount the split volume and perform your backup:

    ```bash
    mkdir /opt/backup
    # Generate a new UUID for the filesytem as the actual is duplicated
    xfs_admin -U generate /dev/vgdata01/lvdata01_rimage_1
    mount /dev/vgdata01/lvdata01_rimage_1 /opt/backup
    # Perform backup operations here
    umount /opt/backup
    ```

  - Merge the Split Mirror Back
    Once backup is complete and you wish to restore the mirror configuration:

    ```bash
    lvconvert --merge /dev/vgdata01/lvdata01_rimage_1
    ```
    > This command will merge `lvdata01_rimage_1` back into `lvdata01`, restoring the mirror.


  - Verify the Mirror Status
    Check that the mirror is restored and both devices are present:

    ```bash
    lvs -a -o +devices
    ```


## Notes

- If you did **not** use `--trackchanges` during the split, merging will **not** be possible.
- Always unmount the split LV before merging.

