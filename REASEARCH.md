# Research Documentation

I'm only Human, and always forget where I got stuff from, so I write that down.

## Automated Role Documentation

I stumbled on a cool tool called [docsible](https://github.com/docsible/docsible) to fully automated generating Role Documentation.  
The result is pretty useable, so README.md is generated with it.

## GPT Conversion

So using parted, this will flush all partitions on the disk - wich will also not work with the LVM I use on sda2...
I found out using gdisk, this will only convert the partition-tables, not touching the partitions itself.  
Next Problem I encounterd was, that with BIOS Boot, disks with GPT-partition tables will not boot.  
So searching around I found the following Article: https://www.suse.com/de-de/support/kb/doc/?id=000021199
There is the need of an compatibility partition at the End of the Disk.
Luckily my disk-layout is:

```yaml
sda:
  - /boot (xfs)
  - lvm
  - /boot/efi (vfat)
```

So straight forward I shrunk the vfat partition (which was unexpectedly easy) and put a new partition at the end.  
Afterwards I had to install grub2 again on the disk and generating a new Config.  
A Reboot was needed to reload everything correctly and verifiying that the system is still booting up.  
Also getting the right Informations for the next ansible-Steps.  
A simple call with `ansible.builtin.setup` will not give you the right Information.  
Don't ask my why, but a reboot always helps, right?

## EFI Migration

So the migration to EFI, its for such a complicated task, not that hard to do.  
But if it fails, it fails spectacularly.  
If you Use a virtual machine - ansible might have Modules for your hypervisor, so it can be fully automated.  
If you are on Hardware and not having something like iDRAC or so, you'll stuck with manually editing the BIOS/EFI.  
Therefor I just put a `ansible.builtin.pause` at the Poipoint where it has to change.  
Also the system will be shut down before that.

Well I followed mostly this blog-entry: https://www.redhat.com/en/blog/bios-uefi  
I hand already the `/boot/efi` mountpoint, so I jumped over a few steps.  

At last I had a misconfiguration which was a bit tricky to find.  
Using `grub2-mkconfig -o /boot/grub2/grub.cfg` generate a file there, right?  
If you are on EFI, this Command will:  
a) not be correct  
b) This file there, which should be a symlink  
Thats great...  
So How do I generate new Configs?  
Well, you need to remove this file `/boot/grub2/grub.cfg` and others.  
Then reinstall the RPM-Packages. And reexcute the Command without the option.  
And there we go. A correct configured system.

## End Note

Its a bit dangorous process. It is pretty simpel to end with a non-booting system.  
While I developed this I ended up countless times reverting my snapshot.  
Use snapshots, use a newly generated backup, make sure it will be recoverable!  
