## System requirements

* Ubuntu 20.04 (focal) or later.
* At least 8GB of RAM.
* At least 2 CPU cores.
* At least 20GB of available storage. For production deployment: at least 60GB of available storage on each host.
* Access to the internet for downloading the required snaps and charms.
* Juju 3.x

## Supported architectures
**This charm should be running on architectures that support AVX**

To check your architecture please execute 
```bash
grep avx /proc/cpuinfo
```

or  
```bash 
grep avx2 /proc/cpuinfo
```