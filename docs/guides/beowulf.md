# Debian11 (Bullseye) Beowulf cluster Guide
#### Written by Aaron Beckley

Install necessary services
```
apt install tftpd-hpa isc-dhcp-server debootstrap nfs-kernel-server net-tools
mkdir /pxeroot
  
cd /pxeroot
  
debootstrap bullseye /pxeroot
```
chroot pxeroot

```
chroot /pxeroot
```


Edit fstab inside of chroot
```
#/etc/fstab: static file system information.
# <file system>   <mount point>   <type>   <options>   <dump>    <pass>
/dev/ram0  /       ext2   defaults    0   0
proc       /proc      proc   defaults    0   1
tmpfs      /tmp       tmpfs  defaults    0   1
```

Install packages needed inside chroot
```
apt install linux-image-amd64
```

Leave Chrooot
```
exit
```

edit /etc/dhcpd/dhcpd.conf
```
subnet 192.168.100.0 netmask 255.255.255.0 {
    range 192.168.100.100 192.168.100.200;
    option subnet-mask 255.255.255.0;
    filename "pxelinux.0";
    next-server 192.168.100.1;
    option root-path "192.168.100.1:/pxeroot";
    option broadcast-address 192.168.100.255;
}
```

set static ip on interface which you will be serving ip to nodes (you will want to make this permanent, but for now)
```
ifconfig enp0s8 192.168.100.1
```

run dhcpd
```
dhcpd
```

edit /etc/default/tftpd-hpa
```
#/etc/default/tftpd-hpa
TFTP_USERNAME="tftp"
TFTP_DIRECTORY="/var/lib/tftpboot"
TFTP_ADDRESS=":69"
TFTP_OPTIONS="--secure"
RUN_DAEMON="yes"
OPTIONS="-l -s /var/lib/tftpboot"
```

restart tftpd
```
/etc/init.d/tftpd-hpa restart
```

Create bootable system
```
cd /var/lib/tftpboot
wget http://ftp.debian.org/debian/dists/bullseye/main/installer-amd64/current/images/netboot/pxelinux.0
cp /pxeroot/vmlinuz ./
cp /pxeroot/initrd.img ./
mkdir pxelinux.cfg
```

edit pxelinux.cfg/default
```
DISPLAY boot.txt2
F1 f1.txt
...

DEFAULT linux

LABEL linux
    kernel vmlinuz
    append vga=normal initrd=/initrd.img ramdisk_size=4096 root=/dev/nfs nfsroot=192.168.100.1:/pxeroot ip=dhcp rw --

PROMPT 0
TIIMEOUT 0
```

edit /etc/exports
```
/pxeroot    192.168.100.0/255.255.255.0(rw,sync,no_root_squash,no_subtree_check)
```

restart nfs
```
/etc/init.d/nfs-kernel-server restart
```

Patch glitch with ldlinux.c32 on debian
```
cp /usr/lib/syslinux/modules/bios/ldlinux.c32 /var/lib/tftpboot
```
