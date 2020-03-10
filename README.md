# �������� ������� � LVM
1. ��������� ������������ ��� ��� / �� 8G
2. �������� ��� ��� /var � ��������
3. �������� ��� ��� /home
4. ������� ������� ��� /home


## 1. ���������� ������������� �������

 ������, ��� ���� � �������
```[root@lvm vagrant]# df -h```

> Filesystem                       Size  Used Avail Use% Mounted on
> /dev/mapper/VolGroup00-LogVol00   38G  1.6G   36G   5% /
> devtmpfs                         109M     0  109M   0% /dev
> tmpfs                            118M     0  118M   0% /dev/shm
> tmpfs                            118M  4.6M  114M   4% /run
> tmpfs                            118M     0  118M   0% /sys/fs/cgroup
> /dev/sda2                       1014M   61M  954M   7% /boot
> tmpfs                             24M     0   24M   0% /run/user/0
> tmpfs                             24M     0   24M   0% /run/user/1000
``` [root@lvm vagrant]# lsblk```
> NAME                    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
> sda                       8:0    0   40G  0 disk
> +-sda1                    8:1    0    1M  0 part
> +-sda2                    8:2    0    1G  0 part /boot
> L-sda3                    8:3    0   39G  0 part
>   +-VolGroup00-LogVol00 253:0    0 37.5G  0 lvm  /
>   L-VolGroup00-LogVol01 253:1    0  1.5G  0 lvm  [SWAP]
> sdb                       8:16   0   10G  0 disk
> sdc                       8:32   0    2G  0 disk
> sdd                       8:48   0    1G  0 disk
> sde                       8:64   0    1G  0 disk

 ������ ��� ��� ���������� �������� ������ �����, �.�. ������ / �������� ���������� � ����� ������� ������ � ����������� ��������
������ ���������� ���
``` [root@lvm vagrant]# pvcreate /dev/sdb```
>   Physical volume "/dev/sdb" successfully created.

 ������ ����� VG02
``` [root@lvm vagrant]# vgcreate VG02 /dev/sdb```
>   Volume group "VG02" successfully created

 ��������������� ��������� ���
``` [root@lvm vagrant]# lvcreate -n xtmp -l+100%FREE /dev/VG02```
>   Logical volume "xtmp" created.

 ������ ������� ������� � ��������  
``` [root@lvm vagrant]# mkfs.xfs /dev/VG02/xtmp && mount /dev/VG02/xtmp /mnt```
> meta-data=/dev/VG02/xtmp         isize=512    agcount=4, agsize=655104 blks
>          =                       sectsz=512   attr=2, projid32bit=1
>          =                       crc=1        finobt=0, sparse=0
> data     =                       bsize=4096   blocks=2620416, imaxpct=25
>          =                       sunit=0      swidth=0 blks
> naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
> log      =internal log           bsize=4096   blocks=2560, version=2
>          =                       sectsz=512   sunit=0 blks, lazy-count=1
> realtime =none                   extsz=4096   blocks=0, rtextents=0

 �������� ������ ����������� xfsdump | xsfrestore
``` [root@lvm vagrant]# xfsdump -J - /dev/VolGroup00/LogVol00 | xfsrestore -J - /mnt```
> xfsrestore: using file dump (drive_simple) strategy
> xfsrestore: version 3.1.7 (dump format 3.0)
> xfsdump: using file dump (drive_simple) strategy
> xfsdump: version 3.1.7 (dump format 3.0)
> xfsdump: level 0 dump of lvm:/
> xfsrestore: searching media for dump
> xfsdump: dump date: Tue Feb 18 15:38:51 2020
> xfsdump: session id: 467b54c4-1665-4dc1-aa93-c4e7800e48e5
> xfsdump: session label: ""
> xfsdump: ino map phase 1: constructing initial dump list
> xfsdump: ino map phase 2: skipping (no pruning necessary)
> xfsdump: ino map phase 3: skipping (only one dump stream)
> xfsdump: ino map construction complete
> xfsdump: estimated dump size: 1695055232 bytes
> xfsdump: creating dump session media file 0 (media 0, file 0)
> xfsdump: dumping ino map
> xfsdump: dumping directories
> xfsrestore: examining media file 0
> xfsrestore: dump description:
> xfsrestore: hostname: lvm
> xfsrestore: mount point: /
> xfsrestore: volume: /dev/mapper/VolGroup00-LogVol00
> xfsrestore: session time: Tue Feb 18 15:38:51 2020
> xfsrestore: level: 0
> xfsrestore: session label: ""
> xfsrestore: media label: ""
> xfsrestore: file system id: b60e9498-0baa-4d9f-90aa-069048217fee
> xfsrestore: session id: 467b54c4-1665-4dc1-aa93-c4e7800e48e5
> xfsrestore: media id: 2a387222-4afe-4e3b-9db8-d14e686b8413
> xfsrestore: searching media for directory dump
> xfsrestore: reading directories
> xfsdump: dumping non-directory files
> xfsrestore: 7280 directories and 46413 entries processed
> xfsrestore: directory post-processing
> xfsrestore: restoring non-directory files
> xfsdump: ending media file
> xfsdump: media file size 1648536920 bytes
> xfsdump: dump size (non-dir files) : 1620809240 bytes
> xfsdump: dump complete: 46 seconds elapsed
> xfsdump: Dump Status: SUCCESS
> xfsrestore: restore complete: 48 seconds elapsed
> xfsrestore: Restore Status: SUCCESS

 �����!
 �������� �� ����� �����
``` [root@lvm vagrant]# ls /mnt```
> bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var

 ������� ����������� ��� ��� chroot
``` [root@lvm vagrant]# for i in /proc/ /sys/ /dev/ /run/ /boot/; do mount --bind $i /mnt/$i; done```
``` [root@lvm vagrant]#```

 ��������� � ����������� ���
``` [root@lvm vagrant]# chroot /mnt/```
``` [root@lvm /]#```

 �������� ���������
``` [root@lvm /]# grub2-mkconfig -o /boot/grub2/grub.cfg```
> Generating grub configuration file ...
> Found linux image: /boot/vmlinuz-3.10.0-862.2.3.el7.x86_64
> Found initrd image: /boot/initramfs-3.10.0-862.2.3.el7.x86_64.img
> done
``` [root@lvm /]#```

 ���������� initramfs ��� ����� ����������� ������
``` [root@lvm /]# cd /boot ; for i in `ls initramfs-*img`; do dracut -v $i `echo $i|sed "s/initramfs-//g; s/.img//g"` --force; done```

> Executing: /sbin/dracut -v initramfs-3.10.0-862.2.3.el7.x86_64.img 3.10.0-862.2.3.el7.x86_64 --force
> dracut module 'busybox' will not be installed, because command 'busybox' could not be found!
> dracut module 'crypt' will not be installed, because command 'cryptsetup' could not be found!
> dracut module 'dmraid' will not be installed, because command 'dmraid' could not be found!
> dracut module 'dmsquash-live-ntfs' will not be installed, because command 'ntfs-3g' could not be found!
> dracut module 'multipath' will not be installed, because command 'multipath' could not be found!
> dracut module 'busybox' will not be installed, because command 'busybox' could not be found!
> dracut module 'crypt' will not be installed, because command 'cryptsetup' could not be found!
> dracut module 'dmraid' will not be installed, because command 'dmraid' could not be found!
> dracut module 'dmsquash-live-ntfs' will not be installed, because command 'ntfs-3g' could not be found!
> dracut module 'multipath' will not be installed, because command 'multipath' could not be found!
> *** Including module: bash ***
> *** Including module: nss-softokn ***
> *** Including module: i18n ***
> *** Including module: drm ***
> *** Including module: plymouth ***
> *** Including module: dm ***
> Skipping udev rule: 64-device-mapper.rules
> Skipping udev rule: 60-persistent-storage-dm.rules
> Skipping udev rule: 55-dm.rules
> *** Including module: kernel-modules ***
> Omitting driver floppy
> *** Including module: lvm ***
> Skipping udev rule: 64-device-mapper.rules
> Skipping udev rule: 56-lvm.rules
> Skipping udev rule: 60-persistent-storage-lvm.rules
> *** Including module: qemu ***
> *** Including module: resume ***
> *** Including module: rootfs-block ***
> *** Including module: terminfo ***
> *** Including module: udev-rules ***
> Skipping udev rule: 40-redhat-cpu-hotplug.rules
> Skipping udev rule: 91-permissions.rules
> *** Including module: biosdevname ***
> *** Including module: systemd ***
> *** Including module: usrmount ***
> *** Including module: base ***
> *** Including module: fs-lib ***
> *** Including module: shutdown ***
> *** Including modules done ***
> *** Installing kernel module dependencies and firmware ***
> *** Installing kernel module dependencies and firmware done ***
> *** Resolving executable dependencies ***
> *** Resolving executable dependencies done***
> *** Hardlinking files ***
> *** Hardlinking files done ***
> *** Stripping files ***
> *** Stripping files done ***
> *** Generating early-microcode cpio image contents ***
> *** No early-microcode cpio image needed ***
> *** Store current command line parameters ***
> *** Creating image file ***
> *** Creating image file done ***
> *** Creating initramfs image file '/boot/initramfs-3.10.0-862.2.3.el7.x86_64.img' done ***
``` [root@lvm boot]#```

 ����������� ����� ��������� �������, ������� �������� ���������
``` [root@lvm boot]# sed -i 's/VolGroup00\/LogVol00/VG02\/xtmp/g' /boot/grub2/grub.cfg```
``` [root@lvm boot]#```

 ��������
``` [root@lvm boot]# grep VolGroup00 /boot/grub2/grub.cfg```
>         linux16 /vmlinuz-3.10.0-862.2.3.el7.x86_64 root=/dev/mapper/VG02-xtmp ro no_timer_check console=tty0 console=ttyS0,115200n8 net.ifnames=0 biosdevname=0 elevator=noop crashkernel=auto rd.lvm.lv=VG02/xtmp rd.lvm.lv=VolGroup00/LogVol01 rhgb quiet
> [root@lvm boot]#

 ������ �� chroot � ��������������
``` [root@lvm boot]# exit```
``` [root@lvm vagrant]# reboot```
> PolicyKit daemon disconnected from the bus.
> We are no longer a registered authentication agent.
> Connection to 127.0.0.1 closed by remote host.
> Connection to 127.0.0.1 closed.

 ����� ������������:
``` [root@lvm ~]# lsblk```
> NAME                    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
> sda                       8:0    0   40G  0 disk
> +-sda1                    8:1    0    1M  0 part
> +-sda2                    8:2    0    1G  0 part /boot
> L-sda3                    8:3    0   39G  0 part
>   +-VolGroup00-LogVol01 253:1    0  1.5G  0 lvm  [SWAP]
>   L-VolGroup00-LogVol00 253:2    0 37.5G  0 lvm
> sdb                       8:16   0   10G  0 disk
> L-VG02-xtmp             253:0    0   10G  0 lvm  /
> sdc                       8:32   0    2G  0 disk
> sdd                       8:48   0    1G  0 disk
> sde                       8:64   0    1G  0 disk
``` [root@lvm ~]#```

 ������ ������� ������� ���
``` [root@lvm ~]# lvremove /dev/VolGroup00/LogVol00```
> Do you really want to remove active logical volume VolGroup00/LogVol00? [y/n]: y
>   Logical volume "LogVol00" successfully removed

 ������ �����, �������� �������
``` [root@lvm ~]# lvcreate -n VolGroup00/LogVol00 -L 8G /dev/VolGroup00```
> WARNING: xfs signature detected on /dev/VolGroup00/LogVol00 at offset 0. Wipe it? [y/n]: y
>   Wiping xfs signature on /dev/VolGroup00/LogVol00.
>   Logical volume "LogVol00" created.
``` [root@lvm ~]#```

 �� ��� ������ �������
``` [root@lvm ~]# mkfs.xfs /dev/VolGroup00/LogVol00```
> meta-data=/dev/VolGroup00/LogVol00 isize=512    agcount=4, agsize=524288 blks
>          =                       sectsz=512   attr=2, projid32bit=1
>          =                       crc=1        finobt=0, sparse=0
> data     =                       bsize=4096   blocks=2097152, imaxpct=25
>          =                       sunit=0      swidth=0 blks
> naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
> log      =internal log           bsize=4096   blocks=2560, version=2
>          =                       sectsz=512   sunit=0 blks, lazy-count=1
> realtime =none                   extsz=4096   blocks=0, rtextents=0

 �������� ��� �������� ������ � �������� ������
``` [root@lvm ~]# mount /dev/VolGroup00/LogVol00 /mnt```
``` [root@lvm ~]# xfsdump -J - /dev/VG02/xtmp | xfsrestore -J - /mnt```
> xfsrestore: using file dump (drive_simple) strategy
> xfsrestore: version 3.1.7 (dump format 3.0)
> xfsdump: using file dump (drive_simple) strategy
> xfsdump: version 3.1.7 (dump format 3.0)
> xfsdump: level 0 dump of lvm:/
> xfsdump: dump date: Tue Feb 18 15:49:10 2020
> xfsdump: session id: 51889bc0-6e62-47f2-beeb-3450e98d82fa
> xfsdump: session label: ""
> xfsrestore: searching media for dump
> xfsdump: ino map phase 1: constructing initial dump list
> xfsdump: ino map phase 2: skipping (no pruning necessary)
> xfsdump: ino map phase 3: skipping (only one dump stream)
> xfsdump: ino map construction complete
> xfsdump: estimated dump size: 1694311680 bytes
> xfsdump: creating dump session media file 0 (media 0, file 0)
> xfsdump: dumping ino map
> xfsdump: dumping directories
> xfsrestore: examining media file 0
> xfsrestore: dump description:
> xfsrestore: hostname: lvm
> xfsrestore: mount point: /
> xfsrestore: volume: /dev/mapper/VG02-xtmp
> xfsrestore: session time: Tue Feb 18 15:49:10 2020
> xfsrestore: level: 0
> xfsrestore: session label: ""
> xfsrestore: media label: ""
> xfsrestore: file system id: 36f9ba93-017d-439d-b85c-0f547f46cbbe
> xfsrestore: session id: 51889bc0-6e62-47f2-beeb-3450e98d82fa
> xfsrestore: media id: 80f6105e-057e-4ef1-81f5-3ce1de3ff4b7
> xfsrestore: searching media for directory dump
> xfsrestore: reading directories
> xfsdump: dumping non-directory files
> xfsrestore: 7284 directories and 46419 entries processed
> xfsrestore: directory post-processing
> xfsrestore: restoring non-directory files
> xfsdump: ending media file
> xfsdump: media file size 1646946336 bytes
> xfsdump: dump size (non-dir files) : 1619214312 bytes
> xfsdump: dump complete: 47 seconds elapsed
> xfsdump: Dump Status: SUCCESS
> xfsrestore: restore complete: 47 seconds elapsed
> xfsrestore: Restore Status: SUCCESS
> [root@lvm ~]#

 ������� ����������� ��� ��� ��������� ��������, ����� chroot, �������� ���������
``` [root@lvm ~]# for i in /proc/ /sys/ /dev/ /run/ /boot/; do mount --bind $i /mnt/$i; done```
``` [root@lvm ~]# chroot /mnt/```
``` [root@lvm /]# grub2-mkconfig -o /boot/grub2/grub.cfg```
> Generating grub configuration file ...
> Found linux image: /boot/vmlinuz-3.10.0-862.2.3.el7.x86_64
> Found initrd image: /boot/initramfs-3.10.0-862.2.3.el7.x86_64.img
> done

 ����� ���������� initramfs
``` [root@lvm /]# cd /boot ; for i in `ls initramfs-*img`; do dracut -v $i `echo $i|sed "s/initramfs-//g; s/.img//g"` --force; done```
> Executing: /sbin/dracut -v initramfs-3.10.0-862.2.3.el7.x86_64.img 3.10.0-862.2.3.el7.x86_64 --force
> dracut module 'busybox' will not be installed, because command 'busybox' could not be found!
> dracut module 'crypt' will not be installed, because command 'cryptsetup' could not be found!
> dracut module 'dmraid' will not be installed, because command 'dmraid' could not be found!
> dracut module 'dmsquash-live-ntfs' will not be installed, because command 'ntfs-3g' could not be found!
> dracut module 'multipath' will not be installed, because command 'multipath' could not be found!
> dracut module 'busybox' will not be installed, because command 'busybox' could not be found!
> dracut module 'crypt' will not be installed, because command 'cryptsetup' could not be found!
> dracut module 'dmraid' will not be installed, because command 'dmraid' could not be found!
> dracut module 'dmsquash-live-ntfs' will not be installed, because command 'ntfs-3g' could not be found!
> dracut module 'multipath' will not be installed, because command 'multipath' could not be found!
> *** Including module: bash ***
> *** Including module: nss-softokn ***
> *** Including module: i18n ***
> *** Including module: drm ***
> *** Including module: plymouth ***
> *** Including module: dm ***
> Skipping udev rule: 64-device-mapper.rules
> Skipping udev rule: 60-persistent-storage-dm.rules
> Skipping udev rule: 55-dm.rules
> *** Including module: kernel-modules ***
> Omitting driver floppy
> *** Including module: lvm ***
> Skipping udev rule: 64-device-mapper.rules
> Skipping udev rule: 56-lvm.rules
> Skipping udev rule: 60-persistent-storage-lvm.rules
> *** Including module: qemu ***
> *** Including module: resume ***
> *** Including module: rootfs-block ***
> *** Including module: terminfo ***
> *** Including module: udev-rules ***
> Skipping udev rule: 40-redhat-cpu-hotplug.rules
> Skipping udev rule: 91-permissions.rules
> *** Including module: biosdevname ***
> *** Including module: systemd ***
> *** Including module: usrmount ***
> *** Including module: base ***
> *** Including module: fs-lib ***
> *** Including module: shutdown ***
> *** Including modules done ***
> *** Installing kernel module dependencies and firmware ***
> *** Installing kernel module dependencies and firmware done ***
> *** Resolving executable dependencies ***
> *** Resolving executable dependencies done***
> *** Hardlinking files ***
> *** Hardlinking files done ***
> *** Stripping files ***
> *** Stripping files done ***
> *** Generating early-microcode cpio image contents ***
> *** No early-microcode cpio image needed ***
> *** Store current command line parameters ***
> *** Creating image file ***
> *** Creating image file done ***
> *** Creating initramfs image file '/boot/initramfs-3.10.0-862.2.3.el7.x86_64.img' done ***
``` [root@lvm boot]#```

## 2. �������� ��� ��� /var � ��������
 ������ ���������� ��� � ������ ����� ��� /var
```[root@lvm boot]# pvcreate /dev/sdc /dev/sdd```
>   Physical volume "/dev/sdc" successfully created.
>   Physical volume "/dev/sdd" successfully created.
```[root@lvm boot]# vgcreate vg_var /dev/sdc /dev/sdd```
>   Volume group "vg_var" successfully created
  
 ������ ��� ���. ����� "-m1" ��������� ���������� ������
```[root@lvm boot]# lvcreate -L 950M -m1 -n lv_var vg_var```
>   Rounding up size to full physical extent 952.00 MiB
>   Logical volume "lv_var" created.
 �������� ������� �� ����� ����
```[root@lvm boot]# mkfs.ext4 /dev/vg_var/lv_var```
> mke2fs 1.42.9 (28-Dec-2013)
> Filesystem label=
> OS type: Linux
> Block size=4096 (log=2)
> Fragment size=4096 (log=2)
> Stride=0 blocks, Stripe width=0 blocks
> 60928 inodes, 243712 blocks
> 12185 blocks (5.00%) reserved for the super user
> First data block=0
> Maximum filesystem blocks=249561088
> 8 block groups
> 32768 blocks per group, 32768 fragments per group
> 7616 inodes per group
> Superblock backups stored on blocks:
>         32768, 98304, 163840, 229376
> 
> Allocating group tables: done
> Writing inode tables: done
> Creating journal (4096 blocks): done
> Writing superblocks and filesystem accounting information: done
 �������� ��� �������� ������
```[root@lvm boot]# mount /dev/vg_var/lv_var /mnt```

 �������� ������. --quiet ��� ���������� ������ � ��������
```[root@lvm boot]# rsync -avHPSAX --quiet /var/ /mnt/```
 ```[root@lvm boot]#```
 
 ��������
```[root@lvm boot]# ls /mnt```

> adm  cache  db  empty  games  gopher  kerberos  lib  local  lock  log  mail  nis  opt  preserve  run  spool  tmp  yp

 ��������� ����� ������ � �������� �����. �� ������ ��������.
```[root@lvm boot]# mkdir /tmp/oldvar && mv /var/* /tmp/oldvar```

```[root@lvm boot]#```

 ����������� � ���������� � ����������� � ������� �����
```[root@lvm boot]# umount /mnt && mount /dev/vg_var/lv_var /var```

```[root@lvm boot]#```

 �������� ������ � fstab
```echo "`blkid | grep var: | awk '{print $2}'` /var ext4 defaults 0 0" >> /etc/fstab```

 ������ �� chroot, ��������������
```[root@lvm boot]# exit```

```[root@lvm ~]# reboot```

> Connection to 127.0.0.1 closed by remote host.
> Connection to 127.0.0.1 closed.

 ������ ��������� ������...
```[root@lvm ~]# lvremove /dev/VG02/xtmp```

> Do you really want to remove active logical volume VG02/xtmp? [y/n]: y
>   Logical volume "xtmp" successfully removed

 ...������ �����...  
```[root@lvm ~]# vgremove /dev/VG02```

>   Volume group "VG02" successfully removed

 ...� ���������� ���
```[root@lvm ~]# pvremove /dev/sdb```

>   Labels on physical volume "/dev/sdb" successfully wiped.
```[root@lvm ~]#```

## 3. ��������� ���� ��� /home
```[root@lvm ~]# lvcreate -n LV_Home -L 2G /dev/VolGroup00```

>   Logical volume "LV_Home" created.

 ������ �������� �������  
```[root@lvm ~]# mkfs.xfs /dev/VolGroup00/LV_Home```

> meta-data=/dev/VolGroup00/LV_Home isize=512    agcount=4, agsize=131072 blks
>          =                       sectsz=512   attr=2, projid32bit=1
>          =                       crc=1        finobt=0, sparse=0
> data     =                       bsize=4096   blocks=524288, imaxpct=25
>          =                       sunit=0      swidth=0 blks
> naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
> log      =internal log           bsize=4096   blocks=2560, version=2
>          =                       sectsz=512   sunit=0 blks, lazy-count=1
> realtime =none                   extsz=4096   blocks=0, rtextents=0

```[root@lvm ~]#```

 �������� �� ��������� ����� ��� �������� ������
```[root@lvm ~]# mount /dev/VolGroup00/LV_Home /mnt/```

> [root@lvm ~]#

 �������� ������, ������ �� � �������� �����, ����������� ����� �� ������� ����� � ����������� ����� �����, �������� ������ � fstab
```[root@lvm ~]# rsync -avHPSAX --quiet /home/ /mnt/```

```[root@lvm ~]# rm -rf /home/*```

```[root@lvm ~]# umount /mnt && mount /dev/VolGroup00/LV_Home /home/```

```[root@lvm ~]# echo "`blkid | grep Home | awk '{print $2}'` /home xfs defaults 0 0" >> /etc/fstab```

```[root@lvm ~]#```

## 4. �������� ��������
 ������ ����� ��� ����� ������ ��������
```[root@lvm ~]# touch /home/file{1..20}```

```[root@lvm ~]#```

 ������ ��� �������
```[root@lvm ~]# lvcreate -L 100MB -s -n home_snap /dev/VolGroup00/LV_Home```

>   Rounding up size to full physical extent 128.00 MiB
>   Logical volume "home_snap" created.
```[root@lvm ~]#```

 ������ ����� �������� �����
```[root@lvm ~]# rm -f /home/file{2..19}```

> [root@lvm ~]#

 ���������� �������� ����������
```[root@lvm ~]# umount /home```

 ��������� �������������� ������
```[root@lvm ~]# lvconvert --merge /dev/VolGroup00/home_snap```

```[root@otuslinux ~]# mount /home  Merging of volume VolGroup00/home_snap started.```

>   VolGroup00/LV_Home: Merged: 100.00%

 �������� �������
```[root@lvm ~]# mount /home```

```[root@lvm ~]#```

 �������� ��������� ��������������
```[root@lvm ~]# ls -l /home```
> total 0
> -rw-r--r--. 1 root    root     0 Feb 18 16:11 file1
> -rw-r--r--. 1 root    root     0 Feb 18 16:11 file10
> -rw-r--r--. 1 root    root     0 Feb 18 16:11 file11
> -rw-r--r--. 1 root    root     0 Feb 18 16:11 file12
> -rw-r--r--. 1 root    root     0 Feb 18 16:11 file13
> -rw-r--r--. 1 root    root     0 Feb 18 16:11 file14
> -rw-r--r--. 1 root    root     0 Feb 18 16:11 file15
> -rw-r--r--. 1 root    root     0 Feb 18 16:11 file16
> -rw-r--r--. 1 root    root     0 Feb 18 16:11 file17
> -rw-r--r--. 1 root    root     0 Feb 18 16:11 file18
> -rw-r--r--. 1 root    root     0 Feb 18 16:11 file19
> -rw-r--r--. 1 root    root     0 Feb 18 16:11 file2
> -rw-r--r--. 1 root    root     0 Feb 18 16:11 file20
> -rw-r--r--. 1 root    root     0 Feb 18 16:11 file3
> -rw-r--r--. 1 root    root     0 Feb 18 16:11 file4
> -rw-r--r--. 1 root    root     0 Feb 18 16:11 file5
> -rw-r--r--. 1 root    root     0 Feb 18 16:11 file6
> -rw-r--r--. 1 root    root     0 Feb 18 16:11 file7
> -rw-r--r--. 1 root    root     0 Feb 18 16:11 file8
> -rw-r--r--. 1 root    root     0 Feb 18 16:11 file9
> drwx------. 3 vagrant vagrant 74 May 12  2018 vagrant

## �����
1. �������� �������� ��� 
2. ������ ������ /var � ��������
3. ������� ��� ��� /home
4. ������ ������� ��� /home � �������������� ��� ������