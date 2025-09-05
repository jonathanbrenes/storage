# NVMe on Linux in Azure
- This document is designed for experienced Linux users who manage or operate Linux-based virtual machines in the Microsoft Azure cloud environment.

## Overview
- This document provides a theoretical foundation for understanding **NVMe (Non-Volatile Memory Express)** storage devices in **Linux virtual machines running on Microsoft Azure**.

   NVMe is a modern storage protocol designed to maximize the performance of SSDs by connecting directly via PCIe. In Azure, NVMe is used to deliver high-throughput, low-latency storage for specific VM families, making it ideal for performance-critical workloads.


## What is NVMe?
- **NVMe (Non-Volatile Memory Express)** is a storage protocol designed for SSDs that connects directly to the CPU via PCIe. Compared to legacy protocols like SATA and SCSI, NVMe offers:

  - Lower latency
  - Higher IOPS and throughput
  - Greater parallelism through multiple queues


> In Linux, NVMe devices typically appear as `/dev/nvme0n1`, `/dev/nvme0n2`, `/dev/nvme1n1` etc.


## NVMe in Azure
- Azure supports NVMe storage in several VM families, including:

  - **Newer VM generations** including **Dav6**, **Eav6**, and **Fav6** typically support **only the NVMe** storage interface.
  - VM families like **Ebsv5**, **Ebdsv5**, **Lsv2**, **Lsv3**, and **Lasv3** introduced **NVMe as an option for temporary disks**, providing high-performance local storage

> For more details, refer to the official documentation:  
> [Azure NVMe Overview](https://learn.microsoft.com/en-us/azure/virtual-machines/nvme-overview)


## Linux Tools for NVMe Management
- Linux provides several tools to manage and monitor NVMe devices:

  | Tool       | Purpose                                      |
  |------------|----------------------------------------------|
  | `lsblk`    | Lists block devices and their mount points   |
  | `nvme`     | CLI for querying NVMe device info and health |
  | `fio`      | Benchmarks I/O performance                   |


## Use Cases for NVMe in Azure
  - NVMe is ideal for workloads requiring high performance and low latency:
    - Databases
    - Big data analytics
    - Machine learning training
    - Web servers and caching layers
    - Temporary scratch space for compute-intensive tasks

## Migration from SCSI to NVMe
  - Azure recommends migrating from SCSI to NVMe for newer VM sizes to improve performance. For Linux VMs, follow the migration guide:  
  [NVMe Migration Guide for Linux](https://learn.microsoft.com/en-us/azure/virtual-machines/nvme-linux)

## References
  - [Azure NVMe Overview](https://learn.microsoft.com/en-us/azure/virtual-machines/nvme-overview)
