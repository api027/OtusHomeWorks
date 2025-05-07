Домашнее задание по теме:
Обновление ядра системы

Цель:
Научиться обновлять ядро в ОС Linux.

Описание домашнего задания
1) Запустить ВМ c Ubuntu.
2) Обновить ядро ОС на новейшую стабильную версию из mainline-репозитория.
3) Оформить отчет в README-файле в GitHub-репозитории.


Описание выполнения домашнего задания:
1) Из дистрибутива  ubuntu-20.04.6-live-server-amd64.iso  установили на  Windows 2012 R2 Hyper-V  виртуальную машину otus1
2)  Выяснили версию ядра:
api027@otus1:~$ uname -r
5.4.0-144-generic

3) Скачали с сайта новое ядро:
# wget  https://kernel.ubuntu.com/mainline/v6.14.4/amd64/linux-headers-6.14.4-061404-generic_6.14.4-061404.202504251003_amd64.deb
wget  https://kernel.ubuntu.com/mainline/v6.14.4/amd64/linux-headers-6.14.4-061404_6.14.4-061404.202504251003_all.deb
wget  https://kernel.ubuntu.com/mainline/v6.14.4/amd64/linux-image-unsigned-6.14.4-061404-generic_6.14.4-061404.202504251003_amd64.deb
wget  https://kernel.ubuntu.com/mainline/v6.14.4/amd64/linux-modules-6.14.4-061404-generic_6.14.4-061404.202504251003_amd64.deb

4) Установили его командой:
   sudo dpkg -i *.deb

Результаты выполнения команды:
   sudo dpkg -i *.deb
[sudo] password for api027: 
Selecting previously unselected package linux-headers-6.14.4-061404.
(Reading database ... 
(Reading database ... 5%
(Reading database ... 10%
(Reading database ... 15%
(Reading database ... 20%
(Reading database ... 25%
(Reading database ... 30%
(Reading database ... 35%
(Reading database ... 40%
(Reading database ... 45%
(Reading database ... 50%
(Reading database ... 55%
(Reading database ... 60%
(Reading database ... 65%
(Reading database ... 70%
(Reading database ... 75%
(Reading database ... 80%
(Reading database ... 85%
(Reading database ... 90%
(Reading database ... 95%
(Reading database ... 100%
(Reading database ... 72296 files and directories currently installed.)
Preparing to unpack linux-headers-6.14.4-061404_6.14.4-061404.202504251003_all.deb ...
Unpacking linux-headers-6.14.4-061404 (6.14.4-061404.202504251003) ...
Selecting previously unselected package linux-image-unsigned-6.14.4-061404-generic.
Preparing to unpack linux-image-unsigned-6.14.4-061404-generic_6.14.4-061404.202504251003_amd64.deb ...
Unpacking linux-image-unsigned-6.14.4-061404-generic (6.14.4-061404.202504251003) ...
Selecting previously unselected package linux-modules-6.14.4-061404-generic.
Preparing to unpack linux-modules-6.14.4-061404-generic_6.14.4-061404.202504251003_amd64.deb ...
Unpacking linux-modules-6.14.4-061404-generic (6.14.4-061404.202504251003) ...
Setting up linux-headers-6.14.4-061404 (6.14.4-061404.202504251003) ...
Setting up linux-modules-6.14.4-061404-generic (6.14.4-061404.202504251003) ...
Setting up linux-image-unsigned-6.14.4-061404-generic (6.14.4-061404.202504251003) ...
I: /boot/vmlinuz is now a symlink to vmlinuz-6.14.4-061404-generic
I: /boot/initrd.img is now a symlink to initrd.img-6.14.4-061404-generic
Processing triggers for linux-image-unsigned-6.14.4-061404-generic (6.14.4-061404.202504251003) ...
/etc/kernel/postinst.d/initramfs-tools:
update-initramfs: Generating /boot/initrd.img-6.14.4-061404-generic
/etc/kernel/postinst.d/zz-update-grub:
Sourcing file `/etc/default/grub'
Sourcing file `/etc/default/grub.d/init-select.cfg'
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-6.14.4-061404-generic
Found initrd image: /boot/initrd.img-6.14.4-061404-generic
Found linux image: /boot/vmlinuz-5.4.0-144-generic
Found initrd image: /boot/initrd.img-5.4.0-144-generic
done


5) В /boot  получили:

api027@otus1:~/kernel$ ls -al /boot
total 238692
drwxr-xr-x  5 root root      4096 May  7 08:07 .
drwxr-xr-x 20 root root      4096 May  6 10:48 ..
-rw-r--r--  1 root root    237887 Feb  3  2023 config-5.4.0-144-generic
-rw-r--r--  1 root root    295497 Apr 25 10:03 config-6.14.4-061404-generic
drwxr-xr-x  3 root root      4096 Jan  1  1970 efi
drwxr-xr-x  4 root root      4096 May  7 08:08 grub
lrwxrwxrwx  1 root root        32 May  7 08:07 initrd.img -> initrd.img-6.14.4-061404-generic
-rw-r--r--  1 root root  86678877 May  6 10:49 initrd.img-5.4.0-144-generic
-rw-r--r--  1 root root 113006342 May  7 08:07 initrd.img-6.14.4-061404-generic
lrwxrwxrwx  1 root root        28 May  6 10:48 initrd.img.old -> initrd.img-5.4.0-144-generic
drwx------  2 root root     16384 May  6 10:47 lost+found
-rw-------  1 root root   4749781 Feb  3  2023 System.map-5.4.0-144-generic
-rw-------  1 root root   9981345 Apr 25 10:03 System.map-6.14.4-061404-generic
lrwxrwxrwx  1 root root        29 May  7 08:07 vmlinuz -> vmlinuz-6.14.4-061404-generic
-rw-------  1 root root  13680904 Feb  3  2023 vmlinuz-5.4.0-144-generic
-rw-------  1 root root  15737344 Apr 25 10:03 vmlinuz-6.14.4-061404-generic
lrwxrwxrwx  1 root root        25 May  6 10:48 vmlinuz.old -> vmlinuz-5.4.0-144-generic
api027@otus1:~/kernel$ 

После перезагрузки в меню Advanced появилась возможность выбрать  новое ядро. Загрузились.

api027@otus1:~$ uname -r
6.14.4-061404-generic

6) Обновили загрузчик:
   
api027@otus1:~$ sudo update-grub
[sudo] password for api027:
Sourcing file `/etc/default/grub'
Sourcing file `/etc/default/grub.d/init-select.cfg'
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-6.14.4-061404-generic
Found initrd image: /boot/initrd.img-6.14.4-061404-generic
Found linux image: /boot/vmlinuz-5.4.0-144-generic
Found initrd image: /boot/initrd.img-5.4.0-144-generic

   sudo grub-set-default 0
 
 7)После перезагрузки проверили версию ядра:
 api027@otus1:~$ uname -r
6.14.4-061404-generic

 

Замечание:
При установке пакета: wget https://kernel.ubuntu.com/mainline/v6.14.4/amd64/linux-headers-6.14.4-061404-generic_6.14.4-061404.202504251003_amd64.deb
Возникали следующие ошибки:
Unpacking linux-modules-6.14.4-061404-generic (6.14.4-061404.202504251003) ...
Setting up linux-headers-6.14.4-061404 (6.14.4-061404.202504251003) ...
dpkg: dependency problems prevent configuration of linux-headers-6.14.4-061404-generic:
 linux-headers-6.14.4-061404-generic depends on libc6 (>= 2.38); however:
  Version of libc6:amd64 on system is 2.31-0ubuntu9.17.
 linux-headers-6.14.4-061404-generic depends on libdw1t64 (>= 0.171); however:
  Package libdw1t64 is not installed.
 linux-headers-6.14.4-061404-generic depends on libelf1t64 (>= 0.144); however:
  Package libelf1t64 is not installed.
 linux-headers-6.14.4-061404-generic depends on libssl3t64 (>= 3.0.0); however:
  Package libssl3t64 is not installed.

dpkg: error processing package linux-headers-6.14.4-061404-generic (--install):
 dependency problems - leaving unconfigured


 


