# HomeWork-LVM
ДЗ. Работа с LVM
На имеющемся образе centos/7 - v. 1804.2

-уменьшить том под / до 8G

-выделить том под /home

-выделить том под /var

-/var - сделать в mirror

-/home - сделать том для снэпшотов

-прописать монтирование в fstab

Попробоватþ с разными опциями и разными файловýми системами ( на выбор)

Работа со снапшотами:
- сгенерить файлы в /home/
- снять снапшот
- удалить часть файлов
- восстановится со снапшота
- залоггировать работу можно с помощью утилиты script

**Начало ДЗ**

```
C:\Vagrant\Centos 7_8_DZ_3>vagrant ssh
Last login: Sat Aug 15 12:06:15 2020
Last login: Sat Aug 15 12:06:15 2020
[vagrant@lvm ~]$ lsblk
NAME                    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                       8:0    0   40G  0 disk
├─sda1                    8:1    0    1M  0 part
├─sda2                    8:2    0    1G  0 part /boot
└─sda3                    8:3    0   39G  0 part
  ├─VolGroup00-LogVol00 253:0    0 37.5G  0 lvm  /
  └─VolGroup00-LogVol01 253:1    0  1.5G  0 lvm  [SWAP]
sdb                       8:16   0   10G  0 disk
sdc                       8:32   0    2G  0 disk
sdd                       8:48   0    1G  0 disk
sde                       8:64   0    1G  0 disk
  ```
  1. разметим диск для будущего использования LVM - создадим PV 
  - создавать первый уровень абстракции - VG
  - создадим Logical Volume
  ```
[vagrant@lvm ~]$ sudo pvcreate /dev/sdb
  Physical volume "/dev/sdb" successfully created.
[vagrant@lvm ~]$ sudo vgcreate otus /dev/sdb
  Volume group "otus" successfully created
[vagrant@lvm ~]$ sudo lvcreate -l+80%FREE -n test otus
  Logical volume "test" created.
```
 Посмотрим информацию о только что созданном Volume Group
 ```
[vagrant@lvm ~]$ sudo vgdisplay otus
  --- Volume group ---
  VG Name               otus
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  2
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                1
  Open LV               0
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <10.00 GiB
  PE Size               4.00 MiB
  Total PE              2559
  Alloc PE / Size       2047 / <8.00 GiB
  Free  PE / Size       512 / 2.00 GiB
  VG UUID               oYJ0bP-pR77-1L8a-NDHV-nRcz-80UK-judjjo
   ```
Детальную информацию о LV получим командой:

```
lvdisplay /dev/otus/test
  ```
 ```
[vagrant@lvm ~]$ sudo lvdisplay /dev/otus/test
  --- Logical volume ---
  LV Path                /dev/otus/test
  LV Name                test
  VG Name                otus
  LV UUID                TdtVXm-3x37-EWZb-t1S4-AHPF-7BOs-w03381
  LV Write Access        read/write
  LV Creation host, time lvm, 2020-08-15 12:38:08 +0000
  LV Status              available
  # open                 0
  LV Size                <8.00 GiB
  Current LE             2047
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:2
   ```
   В сжатом виде информацию можно получить командами vgs и lvs:
 
   ```
   [vagrant@lvm ~]$ sudo vgs
  VG         #PV #LV #SN Attr   VSize   VFree
  VolGroup00   1   2   0 wz--n- <38.97g    0
  otus         1   1   0 wz--n- <10.00g 2.00g
[vagrant@lvm ~]$ sudo lvs
  LV       VG         Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  LogVol00 VolGroup00 -wi-ao---- <37.47g
  LogVol01 VolGroup00 -wi-ao----   1.50g
  test     otus       -wi-a-----  <8.00g
 ```   
   Создадим еще один LV из свободного места. На этот раз создадим не экстентами, а абсолютным значением в мегабайтах:

``` 
 [vagrant@lvm ~]$ sudo lvcreate -L100M -n small otus
  Logical volume "small" created.
[vagrant@lvm ~]$ sudo lvs
  LV       VG         Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  LogVol00 VolGroup00 -wi-ao---- <37.47g
  LogVol01 VolGroup00 -wi-ao----   1.50g
  small    otus       -wi-a----- 100.00m
  test     otus       -wi-a-----  <8.00g
   ``` 
   Создадим на LV файловую систему и смонтируем его
   
   ```
   [vagrant@lvm ~]$ sudo mkfs.ext4 /dev/otus/test
mke2fs 1.42.9 (28-Dec-2013)
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
524288 inodes, 2096128 blocks
104806 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=2147483648
64 block groups
32768 blocks per group, 32768 fragments per group
8192 inodes per group
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632

Allocating group tables: done
Writing inode tables: done
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done
  
   ``` 
   
    
