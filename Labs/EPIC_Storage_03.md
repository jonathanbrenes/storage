# Storage
## NVME
- Deploy this VM. This will automatically create a VM which uses NVME for all disks.
   
  [![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fjonathanbrenes%2Fstorage%2Frefs%2Fheads%2Fmain%2FLabs%2Fepic_storage_lab3.json)

  For this exercise, it's recommended to use the following commands:
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

## Converting Virtual Machines Running Linux from SCSI to NVMe
- For this exercise, we are following the instructions from [Converting Virtual Machines Running Linux from SCSI to NVMe](https://learn.microsoft.com/en-us/azure/virtual-machines/nvme-linux). se Azure CloudShell PowerShell, as PowerShell is required for the conversion script.
- Deploy this VM. This will automatically create a VM which uses SCSI for all disks.
    
  [![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fjonathanbrenes%2Fstorage%2Frefs%2Fheads%2Fmain%2FLabs%2Fepic_storage_lab4.json)

  For this exercise, it's recommended to use the following commands: 
  ```bash
  lsblk  -f
  nvme list
  ls -lR /dev/disk/azure
  ```

  Expected Outcome:
    - `lsblk` should show SCSI devices like `sda`, `sdb`, etc. After migration to NVMe, devices should appear as `nvme0n1`, `nvme1n1`, etc.
    - `nvme list` confirms controller and namespace details.
    - `/dev/disk/azure` may lack expected symlinks due to missing udev rules. Install the udev rule as previously recommended.


- Check controller type using PowerShell
   ```powershell
   $vm = Get-AzVM -name <your-vm-name>
   $vm.StorageProfile.DiskControllerType
   ```
   > Replace `<your-vm-name>` with the correct VM name.

- Download the script using PowerShell command
   ```powershell
   Invoke-WebRequest -Uri "https://raw.githubusercontent.com/Azure/SAP-on-Azure-Scripts-and-Utilities/refs/heads/main/Azure-NVMe-Utils/Azure-NVMe-Conversion.ps1" -OutFile ".\Azure-NVMe-Conversion.ps1"
   ```
   > **Warning:** At the time of writing, this step has a known issue in the public documentation https://learn.microsoft.com/en-us/azure/virtual-machines/nvme-linux#222-download-the-script

- Execute the script
  ```powershell
  .\Azure-NVMe-Conversion.ps1 -ResourceGroupName <your-RG> -VMName <your-vm-name> -NewControllerType NVMe -VMSize Standard_E2bs_v5 -StartVM -FixOperatingSystemSettings
  ```
  > Ensure you use the correct resource group and VM name.  
  
  > Verify that the original VM size supports NVMe. If not, choose a compatible size. In this example, `Standard_E2bs_v5` is used because it supports both SCSI and NVMe.
  
- Post-Conversion Checks
   - Confirm NVMe devices with `lsblk` and `nvme list`.
   - Validate `/etc/fstab` entries.
   - Reboot and verify boot integrity.

