
# Grub enable boot menu

Yesterday, I started debugging a Kubernetes cluster that had issues with kube-proxy after an upgrade. It was throwing some errors related to iptables, and the pod was restarting infinitely (`CrashLoopBackOff`). I concluded that I should downgrade the kernel, and this is on some local VMs running in Proxmox

Fortunately Proxmox has a nice feature in the web interface for accessing the VM consoles. Unfortunately, many modern Linux distributions no longer display that boot menu where you can choose to boot with an older kernel. Well, you can press some key at the exact second when it boots and that menu appears, a silly thing that saves a few seconds at boot. So I decided to reactivate the boot menu so that I can choose to boot with an older kernel.

To remind myself how to do this, I searched the internet and found dozens of pages mentioning the following process:

1. edit the file `/etc/default/grub`
2. change from `GRUB_TIMEOUT=0` to `GRUB_TIMEOUT=10` or other number, that's seconds
3. few pages mention adding `GRUB_TIMEOUT_STYLE=menu`, obviously, this is also necessary because the default is `GRUB_TIMEOUT_STYLE=hidden`!
3. then run `update-grub`, then `reboot` and follow the console booting

Aaand... *NOTHING*! It booted exactly the same as before, without the boot menu! Wtf?

In case you are curious, the output of the `update-grub` command was:

```
$ update-grub
Sourcing file `/etc/default/grub'
Sourcing file `/etc/default/grub.d/50-cloudimg-settings.cfg'
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-6.8.0-57-generic
Found initrd image: /boot/initrd.img-6.8.0-57-generic
Found linux image: /boot/vmlinuz-6.8.0-52-generic
Found initrd image: /boot/initrd.img-6.8.0-52-generic
Warning: os-prober will not be executed to detect other bootable partitions.
Systems on them will not be added to the GRUB boot configuration.
Check GRUB_DISABLE_OS_PROBER documentation entry.
Adding boot menu entry for UEFI Firmware Settings ...
done
```

Did you notice why *NOTHING* at boot?"

-----

Because the file `/etc/default/grub.d/50-cloudimg-settings.cfg` is included (`Sourcing file...`) afterwards, 
 it overrides the settings defined in `/etc/default/grub`! 
So this one has to be modified and obviously, then everything works perfectly.

What surprised me is that absolutely none of the articles online mentions `/etc/default/grub.d/`. The moral:
**skill #1 for devops/sysadmin: always read and understand what is displayed on the screen**.

And booting with an older kernel solved the initial problem with kube-proxy CrashLoopBackOff.

