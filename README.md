# HomeWork-LVM
ДЗ. Работа с LVM
На имеĀщемсā образе centos/7 - v. 1804.2
уменьшить том под / до 8G

выделить том под /home

выделить том под /var

/var - сделать в mirror

/home - сделать том для снэпшотов

прописать монтирование в fstab
Попробоватþ с разными опциями и разными файловýми системами ( на выбор)

Работа со снапшотами:
- сгенерить файлы в /home/
- снять снапшот
- удалить часть файлов
- восстановится со снапшота
- залоггировать работу можно с помощью утилиты script
**Начало ДЗ**
```
C:\Vagrant\otuslinux\otus-linux>vagrant ssh
Last login: Tue Aug 11 16:55:39 2020 from 10.0.2.2
[vagrant@otuslinux ~]$ lsblk
NAME      MAJ:MIN RM    SIZE RO TYPE   MOUNTPOINT
sda         8:0    0     40G  0 disk
├─sda1      8:1    0     40G  0 part   /
└─sda2      8:2    0 1023.5K  0 part
sdb         8:16   0    250M  0 disk
└─md0       9:0    0    496M  0 raid10
  ├─md0p1 259:0    0     98M  0 md
  ├─md0p2 259:1    0     99M  0 md
  ├─md0p3 259:2    0    100M  0 md
  ├─md0p4 259:3    0     99M  0 md
  └─md0p5 259:4    0     98M  0 md
sdc         8:32   0    250M  0 disk
└─md0       9:0    0    496M  0 raid10
  ├─md0p1 259:0    0     98M  0 md
  ├─md0p2 259:1    0     99M  0 md
  ├─md0p3 259:2    0    100M  0 md
  ├─md0p4 259:3    0     99M  0 md
  └─md0p5 259:4    0     98M  0 md
sdd         8:48   0    250M  0 disk
└─md0       9:0    0    496M  0 raid10
  ├─md0p1 259:0    0     98M  0 md
  ├─md0p2 259:1    0     99M  0 md
  ├─md0p3 259:2    0    100M  0 md
  ├─md0p4 259:3    0     99M  0 md
  └─md0p5 259:4    0     98M  0 md
sde         8:64   0    250M  0 disk
└─md0       9:0    0    496M  0 raid10
  ├─md0p1 259:0    0     98M  0 md
  ├─md0p2 259:1    0     99M  0 md
  ├─md0p3 259:2    0    100M  0 md
  ├─md0p4 259:3    0     99M  0 md
  └─md0p5 259:4    0     98M  0 md
  ```
  
