# Storage
## LVM - Resizing
  Deploy this VM. This will automatically create a VM and set an initial LVM/filesystem structure.
   
  [![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fjonathanbrenes%2Fstorage%2Fmain%2FLabs%2Fepic_storage_lab1.json)
  
  For this exercise, it's recommended to use these commands: 
  ```bash
  lvs -o +devices
  ls -l /dev/disk/azure/scsi1
  ```

  - Extend ```/dev/vgdata01/lvdata01``` to use all available space on the volume group ```vgdata01```.
  - Extend ```/dev/vgdata01/lvdata01```, this time adding ```/dev/disk/azure/scsi1/lun1``` to ```vgdata01```.
    > **Don't resize the filesystem yet!**
  - Reduce ```/dev/vgdata01/lvdata01``` by 2GB of the total space assigned.
  - Check the filesystem ```/opt/data01```
  - Extend ```/dev/vgdata01/lvdata01``` to use all available space on the volume group ```vgdata01```. This time extend the filesystem as well.
  - Reduce ```/dev/vgdata01/lvdata01``` to 3GB.
  - Check the filesystem ```/opt/data01```.
  - Extend ```/dev/vgdata03/lvdata02``` to use all available space on the volume group ```vgdata02``` as well the filesystem.
  - Add ```/dev/disk/azure/scsi1/lun4``` to ```/dev/vgdata02``` and extend all free space on ```/dev/vgdata02/lvdata02```.
    > Got any error? What you need to solve this?

