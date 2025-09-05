# Storage
## NVME
  Deploy this VM. This will automatically create a VM which uses NVME for all disks.
   
  [![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fjonathanbrenes%2Fmentorship%2Frefs%2Fheads%2Fmain%2Fstorage002.json)

  For this exercise, it's recommended to use these commands: 
  ```bash
  lsblk  -f
  nvme list
  ls -l /dev/disk/azure/scsi1
  ls -l /dev/disk/azure
  ```

- List the block device using ```lsblk -f```
- List all nvme controllers using ```nvme list```
- List the files and symlinks on ```/dev/disk/azure```
  > You may found the previous symlinks and files are missing
- For this purpose we are going to install a new udev rule available [here](https://raw.githubusercontent.com/Azure/azure-vm-utils/refs/heads/main/udev/80-azure-disk.rules)
  ```bash
  wget https://raw.githubusercontent.com/Azure/azure-vm-utils/refs/heads/main/udev/80-azure-disk.rules -O /etc/udev/rules.d/80-azure-disk.rules
  ```
- Reboot and check again ```/dev/disk/azure```