
[Snaffler](https://github.com/SnaffCon/Snaffler) can help us perform thorough enumeration i.e. difficult manually.

Three specific file types of interest are `.vhd`, `.vhdx`, and `.vmdk` files. These are `Virtual Hard Disk`, `Virtual Hard Disk v2` (both used by Hyper-V), and `Virtual Machine Disk` (used by VMware). 

```sh
> guestmount -a SQL01-disk1.vmdk -i --ro /mnt/vmdk
  ### Mounting VMDK on Linux

> guestmount --add WEBSRV10.vhdx --ro /mnt/vhdx/ -m /dev/sda1
  ### Mounting VHD/VHDx
```

On windows we can right click and mount or use [Mount-VHD](https://docs.microsoft.com/en-us/powershell/module/hyper-v/mount-vhd?view=windowsserver2019-ps) PowerShell cmdlet for `VHD`/`VHDx`. For a `VMDK` we can right-click and choose `Map Virtual Disk` from the menu or GUI `Disk Management`. We can also use VMWare to map it or add it on our attack vm as an additional virtual hard drive. We can even use `7-Zip` to extract data from a .`vmdk` file. [Guide](https://www.nakivo.com/blog/extract-content-vmdk-files-step-step-guide/).

 If we can locate a backup of a live machine, we can pull down the `SAM`, `SECURITY` and `SYSTEM` registry hives from `C:\Windows\System32\Config` .

---