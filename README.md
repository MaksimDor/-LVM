# HomeWork-LVM

**ДЗ. Работа с LVM**

На имеющемся образе centos/7 - v. 1804.2

-уменьшить том под / до 8G

-выделить том под /home

-выделить том под /var

-/var - сделать в mirror

-/home - сделать том для снэпшотов

-прописать монтирование в fstab

Попробовать с разными опциями и разными файловýми системами ( на выбор)

Работа со снапшотами:
- сгенерить файлы в /home/
- снять снапшот
- удалить часть файлов
- восстановится со снапшота
- залоггировать работу можно с помощью утилиты script

Уменьшить том под / до 8G
Эту часть можно выполнить разными способами, мы будем
уменьшать / до 8G без исполþзованиā LiveCD.
Перед началом работы поставим пакет xfsdump - он будет необходим для снятия копии тома.

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
  **Установим nano xfsdump**
  ```
  [vagrant@lvm ~]$ sudo yum install nano xfsdump  
  ```
  **посмотрим /etc/fstab**
  ```
 [vagrant@lvm ~]$ sudo nano /etc/fstab  
  ```
 ```
#
# /etc/fstab
# Created by anaconda on Sat May 12 18:50:26 2018
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/VolGroup00-LogVol00 /                       xfs     defaults        0 0
UUID=570897ca-e759-4c81-90cf-389da6eee4cc /boot                   xfs     defaults        0 0
/dev/mapper/VolGroup00-LogVol01 swap                    swap    defaults        0 0
  ```
**Подготовим временный том для / раздела:**

```
[vagrant@lvm ~]$ sudo pvcreate /dev/sdb
[vagrant@lvm ~]$ sudo vgcreate vg_root /dev/sdb
[vagrant@lvm ~]$ sudo lvcreate -n lv_root -l +100%FREE /dev/vg_root
```

**Создадим на нем файловую систему и смонтируем его, чтобы перенести туда данные:**

```
[vagrant@lvm ~]$ sudo mkfs.xfs /dev/vg_root/lv_root
[vagrant@lvm ~]$ sudo mount /dev/vg_root/lv_root /mnt
```

**Скопируем все данные с / раздела в /mnt:**
 ```
[vagrant@lvm ~]$ sudo xfsdump -J - /dev/VolGroup00/LogVol00 | xfsrestore -J - /mnt
 ```
 
**Переконфигурируем grub для того, чтобы при старте перейти в новый /
Сымитируем текущий root -> сделаем в него chroot и обновим grub:**

``` 
[vagrant@lvm ~]$ sudo for i in /proc/ /sys/ /dev/ /run/ /boot/; do mount --bind $i /mnt/$i; done
[vagrant@lvm ~]$ sudo chroot /mnt/
[vagrant@lvm ~]$ sudo grub2-mkconfig -o /boot/grub2/grub.cfg
```

**Обновим образ initrd:**

```
[vagrant@lvm ~]$ sudo cd /boot ; for i in `ls initramfs-*img`; do dracut -v $i `echo $i|sed "s/initramfs-//g; s/.img//g"` --force; done
```

Чтобы при загрузке был смонтирован нужный root нужно в файле /boot/grub2/grub.cfg 
заменить rd.lvm.lv=VolGroup00/LogVol00 на rd.lvm.lv=vg_root/lv_root

**Меняем в файле /etc/fstab монтирование корня и приводим его к виду:**

```
 [vagrant@lvm ~]$ sudo nano /etc/fstab  
  ```
```
#
# /etc/fstab
# Created by anaconda on Sat May 12 18:50:26 2018
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
#/dev/mapper/VolGroup00-LogVol00 /                       xfs     defaults        0 0
/dev/mapper/vg_root-lv_root /                       xfs     defaults        0 0
UUID=570897ca-e759-4c81-90cf-389da6eee4cc /boot                   xfs     defaults        0 0
/dev/mapper/VolGroup00-LogVol01 swap                    swap    defaults        0 0

```
**Перезагружаемся успешно с новым рут томом. 
смотрим командой lsblk**

```
[vagrant@lvm ~]$ sudo lsblk

NAME                    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                       8:0    0   40G  0 disk
├─sda1                    8:1    0    1M  0 part
├─sda2                    8:2    0    1G  0 part /boot
└─sda3                    8:3    0   39G  0 part
  ├─VolGroup00-LogVol01 253:1    0  1.5G  0 lvm  [SWAP]
  └─VolGroup00-LogVol00 253:2    0 37.5G  0 lvm
sdb                       8:16   0   10G  0 disk
└─vg_root-lv_root       253:0    0   10G  0 lvm  /
sdc                       8:32   0    2G  0 disk
sdd                       8:48   0    1G  0 disk
sde                       8:64   0    1G  0 disk
```
**Теперь нам нужно изменить размер старой VG и вернуть на него рут. Для этого удаляем
старый LV размеров в 40G и создаем новый на 8G:**

```
[vagrant@lvm ~]$ sudo lvremove /dev/VolGroup00/LogVol00

[vagrant@lvm ~]$ sudo lvcreate -n VolGroup00/LogVol00 -L 8G /dev/VolGroup00
```

**Проделываем на нем те же операции, что и в первый раз:**
```
[vagrant@lvm ~]$ sudo mkfs.xfs /dev/VolGroup00/LogVol00
[vagrant@lvm ~]$ sudo mount /dev/VolGroup00/LogVol00 /mnt
[vagrant@lvm ~]$ sudo xfsdump -J - /dev/vg_root/lv_root | xfsrestore -J - /mnt
```
**Так же как в первый раз переконфигурируем grub, за исключением правки /etc/grub2/grub.cfg:**

```
[vagrant@lvm ~]$ sudo for i in /proc/ /sys/ /dev/ /run/ /boot/; do mount --bind $i /mnt/$i; done
[vagrant@lvm ~]$ sudo chroot /mnt/
[vagrant@lvm ~]$ sudo grub2-mkconfig -o /boot/grub2/grub.cfg
[vagrant@lvm ~]$ sudo cd /boot ; for i in `ls initramfs-*img`; do dracut -v $i `echo $i|sed "s/initramfs-//g; s/.img//g"` --force; done
```

**Пока не перезагружаемся и не выходим из под chroot - мы можем заодно перенести /var**

**На свободных дисках создаем зеркало:**
```
[vagrant@lvm ~]$ sudo pvcreate /dev/sdc /dev/sdd
[vagrant@lvm ~]$ sudo vgcreate vg_var /dev/sdc /dev/sdd
[vagrant@lvm ~]$ sudo lvcreate -L 950M -m1 -n lv_var vg_var
```

**Создаем на нем ФС и перемещаем туда /var:**

```
[vagrant@lvm ~]$ sudo mkfs.ext4 /dev/vg_var/lv_var
[vagrant@lvm ~]$ sudo mount /dev/vg_var/lv_var /mnt
[vagrant@lvm ~]$ sudo cp -aR /var/* /mnt/ (или rsync -avHPSAX /var/ /mnt/)
```

**На всякий случай сохраняем содержимое старого var (или же можно его просто удалить):**
```
[vagrant@lvm ~]$ sudo mkdir /tmp/oldvar && mv /var/* /tmp/oldvar
```

**Монтируем новый var в каталог /var:**

```
[vagrant@lvm ~]$ sudo umount /mnt
[vagrant@lvm ~]$ sudo mount /dev/vg_var/lv_var /var
```

**Правим fstab для автоматического монтирования /var:**

```
[vagrant@lvm ~]$ sudo echo "`blkid | grep var: | awk '{print $2}'` /var ext4 defaults 0 0" >> /etc/fstab
```

**Исправляем файл /etc/fstab :**

```
 [vagrant@lvm ~]$ sudo nano /etc/fstab  
  ```
 ```
#
# /etc/fstab
# Created by anaconda on Sat May 12 18:50:26 2018
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/VolGroup00-LogVol00 /                       xfs     defaults        0 0
#/dev/mapper/vg_root-lv_root /                       xfs     defaults        0 0
UUID=570897ca-e759-4c81-90cf-389da6eee4cc /boot                   xfs     defaults        0 0
/dev/mapper/VolGroup00-LogVol01 swap                    swap    defaults        0 0
  ```

**После чего можно успешно перезагружаться в новый (уменьшенный root) и удалять временную Volume Group:**

```
[vagrant@lvm ~]$ sudo lvremove /dev/vg_root/lv_root
[vagrant@lvm ~]$ sudo vgremove /dev/vg_root
[vagrant@lvm ~]$ sudo pvremove /dev/sdb
```

**Выделяем том под /home по тому же принципу что делали для /var:**

```
[vagrant@lvm ~]$ sudo lvcreate -n LogVol_Home -L 2G /dev/VolGroup00
[vagrant@lvm ~]$ sudo mkfs.xfs /dev/VolGroup00/LogVol_Home
[vagrant@lvm ~]$ sudo mount /dev/VolGroup00/LogVol_Home /mnt/
[vagrant@lvm ~]$ sudo cp -aR /home/* /mnt/ 
[vagrant@lvm ~]$ sudo rm -rf /home/*
[vagrant@lvm ~]$ sudo umount /mnt
[vagrant@lvm ~]$ sudo mount /dev/VolGroup00/LogVol_Home /home/
```
**Правим fstab длā автоматического монтирования /home:**

```
[vagrant@lvm ~]$ sudo echo "`blkid | grep Home | awk '{print $2}'` /home xfs defaults 0 0" >> /etc/fstab
```

**Для работы со снапшотами, сгенерируем файлы в /home/:**

```
[vagrant@lvm ~]$ sudo touch /home/file{1..20}
```

**Снять снапшот:**

```
[vagrant@lvm ~]$ sudo lvcreate -L 100MB -s -n home_snap /dev/VolGroup00/LogVol_Home
```

**Удалить часть файлов:**

```
[vagrant@lvm ~]$ sudo rm -f /home/file{11..20}
```

**Процесс восстановления со снапшота:**


```
[vagrant@lvm ~]$ sudo umount /home
[vagrant@lvm ~]$ sudo lvconvert --merge /dev/VolGroup00/home_snap 
[vagrant@lvm ~]$ sudo mount /home
```
**Выполним команду sudo nano /etc/fstab**
  ```
 [vagrant@lvm ~]$ sudo nano /etc/fstab  
 ```
```
#
# /etc/fstab
# Created by anaconda on Sat May 12 18:50:26 2018
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/VolGroup00-LogVol00 /                       xfs     defaults        0 0
#/dev/mapper/vg_root-lv_root /                       xfs     defaults        0 0
UUID=570897ca-e759-4c81-90cf-389da6eee4cc /boot                   xfs     defaults        0 0
/dev/mapper/VolGroup00-LogVol01 swap                    swap    defaults        0 0
UUID="b4d7f6cd-2a83-4c4c-973e-49cbb8bcfe4e" /var ext4 defaults 0 0
UUID="a8a537c8-a800-4501-8e60-093280b96233" /home xfs defaults 0 0
```

**Выполним команду sudo lsblk**

```
[vagrant@lvm ~]$ sudo lsblk
 ```
  ```
	NAME                       MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
	sda                          8:0    0   40G  0 disk
	├─sda1                       8:1    0    1M  0 part
	├─sda2                       8:2    0    1G  0 part /boot
	└─sda3                       8:3    0   39G  0 part
	  ├─VolGroup00-LogVol00    253:0    0    8G  0 lvm  /
	  ├─VolGroup00-LogVol01    253:1    0  1.5G  0 lvm  [SWAP]
	  └─VolGroup00-LogVol_Home 253:7    0    2G  0 lvm  /home
	sdb                          8:16   0   10G  0 disk
	sdc                          8:32   0    2G  0 disk
	├─vg_var-lv_var_rmeta_0    253:2    0    4M  0 lvm
	│ └─vg_var-lv_var          253:6    0  952M  0 lvm  /var
	└─vg_var-lv_var_rimage_0   253:3    0  952M  0 lvm
	  └─vg_var-lv_var          253:6    0  952M  0 lvm  /var
	sdd                          8:48   0    1G  0 disk
	├─vg_var-lv_var_rmeta_1    253:4    0    4M  0 lvm
	│ └─vg_var-lv_var          253:6    0  952M  0 lvm  /var
	└─vg_var-lv_var_rimage_1   253:5    0  952M  0 lvm
	  └─vg_var-lv_var          253:6    0  952M  0 lvm  /var
	sde                          8:64   0    1G  0 disk

```

