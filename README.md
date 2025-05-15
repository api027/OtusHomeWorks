Домашнее задание: работа с mdadm


Задание
• Добавить в виртуальную машину несколько дисков
• Собрать RAID-0/1/5/10 на выбор
• Сломать и починить RAID
• Создать GPT таблицу, пять разделов и смонтировать их в системе.




----Выполнение практической работы:

1). 

-----Определим, какие диски у нас имеются?
sudo fdisk -l | grep sd

Disk /dev/sda: 127 GiB, 136365211648 bytes, 266338304 sectors
/dev/sda1     2048   2203647   2201600  1.1G EFI System
/dev/sda2  2203648   6397951   4194304    2G Linux filesystem
/dev/sda3  6397952 266336255 259938304  124G Linux filesystem
Disk /dev/sdc: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk /dev/sde: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk /dev/sdd: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk /dev/sdf: 10 GiB, 10737418240 bytes, 20971520 sectors

-----Или смотрим блочные устройства:
lsblk

NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
loop0                       7:0    0 63.8M  1 loop /snap/core20/2571
loop1                       7:1    0 63.3M  1 loop /snap/core20/1828
loop2                       7:2    0 91.9M  1 loop /snap/lxd/24061
loop3                       7:3    0 91.9M  1 loop /snap/lxd/32662
loop4                       7:4    0 50.9M  1 loop /snap/snapd/24505
loop5                       7:5    0 49.9M  1 loop /snap/snapd/18357
sda                         8:0    0  127G  0 disk
+-sda1                      8:1    0  1.1G  0 part /boot/efi
+-sda2                      8:2    0    2G  0 part /boot
L-sda3                      8:3    0  124G  0 part
  L-ubuntu--vg-ubuntu--lv 252:0    0   62G  0 lvm  /
sdb                         8:16   0   10G  0 disk
sdc                         8:32   0   10G  0 disk
sdd                         8:48   0   10G  0 disk
sde                         8:64   0   10G  0 disk
sdf                         8:80   0   10G  0 disk


-----Или так list hardware: 
lshw:
/0/2/0.0.0    /dev/sda   disk       136GB Virtual Disk
/0/2/0.0.3    /dev/sdb   disk       10GB Virtual Disk
/0/2/0.0.4    /dev/sdc   disk       10GB Virtual Disk
/0/2/0.0.5    /dev/sdd   disk       10GB Virtual Disk
/0/2/0.0.1    /dev/sde   disk       10GB Virtual Disk
/0/2/0.0.2    /dev/sdf   disk       10GB Virtual Disk


----Диски для работы по 10GB: sd{b..f}

2.)
----Удаляем метаданные с дисков(обнуляем суперблоки):
api027@otus1:~$ sudo mdadm --zero-superblock --force /dev/sd{b,c,d,e,f}
mdadm: Unrecognised md component device - /dev/sdb
mdadm: Unrecognised md component device - /dev/sdc
mdadm: Unrecognised md component device - /dev/sdd
mdadm: Unrecognised md component device - /dev/sde
mdadm: Unrecognised md component device - /dev/sdf

Это значит, что диски никогда не были объединены в RAID.


3.) 
----Создаем RAID-6 из пяти дисков:

api027@otus1:~$ sudo mdadm --create --verbose /dev/md0 -l 6 -n 5 /dev/sd{b,c,d,e,f}
mdadm: layout defaults to left-symmetric
mdadm: layout defaults to left-symmetric
mdadm: chunk size defaults to 512K
mdadm: size set to 10476544K
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.


4.)
-----Смотрим данные(статус) RAID-массива:

cat /proc/mdstat
api027@otus1:~$ sudo cat /proc/mdstat
Personalities : [linear] [raid0] [raid1] [raid6] [raid5] [raid4] [raid10]
md0 : active raid6 sdf[4] sde[3] sdd[2] sdc[1] sdb[0]
      31429632 blocks super 1.2 level 6, 512k chunk, algorithm 2 [5/5] [UUUUU]
      [=======>.............]  resync = 35.4% (3713980/10476544) finish=3.1min speed=36279K/sec

unused devices: <none>


----Смотрим статус массива ещё раз:
api027@otus1:~$ sudo cat /proc/mdstat
Personalities : [linear] [raid0] [raid1] [raid6] [raid5] [raid4] [raid10]
md0 : active raid6 sdf[4] sde[3] sdd[2] sdc[1] sdb[0]
      31429632 blocks super 1.2 level 6, 512k chunk, algorithm 2 [5/5] [UUUUU]

unused devices: <none>

а теперь через утилиту mdstat :
 api027@otus1:~$ mdadm -D /dev/md0
mdadm: must be super-user to perform this action
api027@otus1:~$ sudo mdadm -D /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Thu May 15 09:06:24 2025
        Raid Level : raid6
        Array Size : 31429632 (29.97 GiB 32.18 GB)
     Used Dev Size : 10476544 (9.99 GiB 10.73 GB)
      Raid Devices : 5
     Total Devices : 5
       Persistence : Superblock is persistent

       Update Time : Thu May 15 09:11:25 2025
             State : clean
    Active Devices : 5
   Working Devices : 5
    Failed Devices : 0
     Spare Devices : 0

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : resync

              Name : otus1:0  (local to host otus1)
              UUID : 56bb521f:067382a0:91d8c616:34bc7bea
            Events : 17

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       8       32        1      active sync   /dev/sdc
       2       8       48        2      active sync   /dev/sdd
       3       8       64        3      active sync   /dev/sde
       4       8       80        4      active sync   /dev/sdf


5.) Выводим из строя наш RAID-6, задавая статус одного из дисков "failed".

api027@otus1:~$ sudo mdadm /dev/md0 --fail /dev/sde
mdadm: set /dev/sde faulty in /dev/md0

---Наслаждаемся произведенным эффектом:
sudo cat /proc/mdstat
Personalities : [linear] [raid0] [raid1] [raid6] [raid5] [raid4] [raid10] 
md0 : active raid6 sdf[4] sde[3](F) sdd[2] sdc[1] sdb[0]
      31429632 blocks super 1.2 level 6, 512k chunk, algorithm 2 [5/4] [UUU_U]
      unused devices: <none> 

---Ищем выведенный из строя диск:
api027@otus1:~$ mdadm -D /dev/md0
mdadm: must be super-user to perform this action
api027@otus1:~$ sudo mdadm -D /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Thu May 15 09:06:24 2025
        Raid Level : raid6
        Array Size : 31429632 (29.97 GiB 32.18 GB)
     Used Dev Size : 10476544 (9.99 GiB 10.73 GB)
      Raid Devices : 5
     Total Devices : 5
       Persistence : Superblock is persistent

       Update Time : Thu May 15 09:29:59 2025
             State : clean, degraded
    Active Devices : 4
   Working Devices : 4
    Failed Devices : 1
     Spare Devices : 0

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : resync

              Name : otus1:0  (local to host otus1)
              UUID : 56bb521f:067382a0:91d8c616:34bc7bea
            Events : 19

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       8       32        1      active sync   /dev/sdc
       2       8       48        2      active sync   /dev/sdd
       -       0        0        3      removed
       4       8       80        4      active sync   /dev/sdf

       3       8       64        -      faulty   /dev/sde

6.) Восстанавливаем RAID-6
-----Удаляем диск из массива:
api027@otus1:~$ sudo mdadm /dev/md0 --remove /dev/sde
mdadm: hot removed /dev/sde from /dev/md0


---Добавили новый диск на 13GB
api027@otus1:~$ sudo lshw -short | grep disk
/0/2/0.0.0    /dev/sda   disk       136GB Virtual Disk
/0/2/0.0.3    /dev/sdb   disk       10GB Virtual Disk
/0/2/0.0.4    /dev/sdc   disk       10GB Virtual Disk
/0/2/0.0.5    /dev/sdd   disk       10GB Virtual Disk
/0/2/0.0.1    /dev/sde   disk       10GB Virtual Disk
/0/2/0.0.2    /dev/sdf   disk       10GB Virtual Disk
/0/2/0.0.6    /dev/sdg   disk       13GB Virtual Disk

------Это диск /dev/sdg


-------Подключаем его в RAID-6:
api027@otus1:~$ sudo mdadm /dev/md0 --add /dev/sdg
mdadm: added /dev/sdg

--------Смотрим статус операции:
api027@otus1:~$ cat /proc/mdstat
Personalities : [linear] [raid0] [raid1] [raid6] [raid5] [raid4] [raid10]
md0 : active raid6 sdg[5] sdf[4] sdd[2] sdc[1] sdb[0]
      31429632 blocks super 1.2 level 6, 512k chunk, algorithm 2 [5/4] [UUU_U]
      [=======>.............]  recovery = 38.8% (4072420/10476544) finish=2.3min speed=45415K/sec

mdadm -D /dev/md0

api027@otus1:~$ sudo mdadm -D /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Thu May 15 09:06:24 2025
        Raid Level : raid6
        Array Size : 31429632 (29.97 GiB 32.18 GB)
     Used Dev Size : 10476544 (9.99 GiB 10.73 GB)
      Raid Devices : 5
     Total Devices : 5
       Persistence : Superblock is persistent

       Update Time : Thu May 15 09:52:46 2025
             State : clean, degraded, recovering
    Active Devices : 4
   Working Devices : 5
    Failed Devices : 0
     Spare Devices : 1

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : resync

    Rebuild Status : 76% complete

              Name : otus1:0  (local to host otus1)
              UUID : 56bb521f:067382a0:91d8c616:34bc7bea
            Events : 34

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       8       32        1      active sync   /dev/sdc
       2       8       48        2      active sync   /dev/sdd
       5       8       96        3      spare rebuilding   /dev/sdg
       4       8       80        4      active sync   /dev/sdf

7.) 
Создать GPT таблицу, пять разделов и смонтировать их в системе


----Создаем раздел GPT на нашем RAID-массиве:
sudo parted -s /dev/md0 mklabel gpt

---Нарезаем партиции:

api027@otus1:~$ sudo parted /dev/md0 mkpart primary ext4 0% 20%
Information: You may need to update /etc/fstab.

api027@otus1:~$ sudo parted /dev/md0 mkpart primary ext4 20% 40%
Information: You may need to update /etc/fstab.

api027@otus1:~$ sudo parted /dev/md0 mkpart primary ext4 40% 60%
Information: You may need to update /etc/fstab.

api027@otus1:~$ sudo parted /dev/md0 mkpart primary ext4 60% 80%
Information: You may need to update /etc/fstab.

api027@otus1:~$ sudo parted /dev/md0 mkpart primary ext4 80% 100%
Information: You may need to update /etc/fstab.

--Оформатировали их в ext4:
for i in $(seq 1 5); do sudo mkfs.ext4 /dev/md0p$i; done
Creating filesystem with 1570944 4k blocks and 393216 inodes
Filesystem UUID: 8fa5dce4-4196-4eaf-ad12-edb05df94b1a
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736

Allocating group tables: done
Writing inode tables: done
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done

mke2fs 1.45.5 (07-Jan-2020)
Creating filesystem with 1571712 4k blocks and 393216 inodes
Filesystem UUID: 269d2fb9-f8b8-461f-b976-2091c3619087
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736

Allocating group tables: done
Writing inode tables: done
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done

mke2fs 1.45.5 (07-Jan-2020)
Creating filesystem with 1571328 4k blocks and 393216 inodes
Filesystem UUID: 7f1343a2-de9d-498a-8228-9f2c33f85712
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736

Allocating group tables: done
Writing inode tables: done
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done

mke2fs 1.45.5 (07-Jan-2020)
Creating filesystem with 1571712 4k blocks and 393216 inodes
Filesystem UUID: cd33bceb-3772-4c48-8015-b187896ff406
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736

Allocating group tables: done
Writing inode tables: done
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done

mke2fs 1.45.5 (07-Jan-2020)
Creating filesystem with 1570944 4k blocks and 393216 inodes
Filesystem UUID: 4a94a667-3338-449f-93fb-cb4e2e418c82
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736

Allocating group tables: done
Writing inode tables: done
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done



------Создали точки монтирования:
mkdir -p /raid/part{1,2,3,4,5}


-------Смонтировали логические диски
api027@otus1:~$ for i in $(seq 1 5); do sudo mount /dev/md0p$i /raid/part$i; done


----Скопировали какую-то инфу на один из разделов:
api027@otus1:~$ sudo cp -r /home/api027/* /raid/part1
api027@otus1:~$ ls /raid/part1
kernel                                                          linux-image-unsigned-6.14.4-061404-generic_6.14.4-061404.202504251003_amd64.deb  lost+found
linux-headers-6.14.4-061404_6.14.4-061404.202504251003_all.deb  linux-modules-6.14.4-061404-generic_6.14.4-061404.202504251003_amd64.deb
api027@otus1:~$
 









  












 














