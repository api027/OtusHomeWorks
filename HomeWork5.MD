#### *Домашнее задание.*
#### *Работа с  NFS.*
##


**Цель домашнего задания:**  
	Научиться самостоятельно разворачивать сервис NFS и подключать к нему клиентов.
Задачи: 
 1. Запустить 2 виртуальных машины (сервер NFS и клиента).
 2. На сервере NFS подготовить и экспортировать директорию. В экспортированной директории должна быть поддиректория с именем upload с правами на запись в неё.
 3. Реализовать атоматическое монтирование на клиенте при старте виртуальной машины (systemd, autofs или fstab — любым способом) с использованием NFSv3.


#### *Выполнение домашнего задания.* 
##
**1. Установлены 2 виртуальных машины Ubuntu 22.04**  
**ВМ OTUS2 - будет сервером NFS.** 
```
root@otus2:~# lsb_release -a
Description:    Ubuntu 24.04.2 LTS
Release:        24.04
```
  
  
**OTUS2 IP 10.0.1.166/24**
```root@otus2:~# ifconfig -a | grep inet
        inet 10.0.1.166  netmask 255.255.255.0  broadcast 10.0.1.255

```
**ВМ OTUS3 будет клиентом NFS.**
		

```api027@otus3:~$ lsb_release -a
No LSB modules are available.
Description:    Ubuntu 24.04.2 LTS
Release:        24.04
```

**OTUS3 IP: 10.0.1.167/24**  
```
api027@otus3:~$ ifconfig -a | grep inet
        inet 10.0.1.167  netmask 255.255.255.0  broadcast 10.0.1.255
```    
##

**2. На сервере NFS  OTUS2 подготовить и экспортировать директорию.  
 В экспортированной директории должна быть поддиректория с именем upload с правами на запись в неё.**

**Устанавливаем nfs-сервер c silent подтверждением:**
 ```
 root@otus2:~# apt install -y nfs-kernel-server
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  libnfsidmap1 nfs-common
Suggested packages:
  watchdog
The following NEW packages will be installed:
  libnfsidmap1 nfs-common nfs-kernel-server
0 upgraded, 3 newly installed, 0 to remove and 64 not upgraded.
Need to get 466 kB of archives.
After this operation, 1,695 kB of additional disk space will be used.
Get:1 http://ru.archive.ubuntu.com/ubuntu noble-updates/main amd64 libnfsidmap1 amd64 1:2.6.4-3ubuntu5.1 [48.3 kB]
Get:2 http://ru.archive.ubuntu.com/ubuntu noble-updates/main amd64 nfs-common amd64 1:2.6.4-3ubuntu5.1 [248 kB]
Get:3 http://ru.archive.ubuntu.com/ubuntu noble-updates/main amd64 nfs-kernel-server amd64 1:2.6.4-3ubuntu5.1 [169 kB]
Fetched 466 kB in 1s (323 kB/s)
Selecting previously unselected package libnfsidmap1:amd64.
(Reading database ... 88064 files and directories currently installed.)
Preparing to unpack .../libnfsidmap1_1%3a2.6.4-3ubuntu5.1_amd64.deb ...
Unpacking libnfsidmap1:amd64 (1:2.6.4-3ubuntu5.1) ...
Selecting previously unselected package nfs-common.
Preparing to unpack .../nfs-common_1%3a2.6.4-3ubuntu5.1_amd64.deb ...
Unpacking nfs-common (1:2.6.4-3ubuntu5.1) ...
Selecting previously unselected package nfs-kernel-server.
Preparing to unpack .../nfs-kernel-server_1%3a2.6.4-3ubuntu5.1_amd64.deb ...
Unpacking nfs-kernel-server (1:2.6.4-3ubuntu5.1) ...
Setting up libnfsidmap1:amd64 (1:2.6.4-3ubuntu5.1) ...
Setting up nfs-common (1:2.6.4-3ubuntu5.1) ...
Creating config file /etc/idmapd.conf with new version
Creating config file /etc/nfs.conf with new version
info: Selecting UID from range 100 to 999 ...
info: Adding system user `statd' (UID 111) ...
info: Adding new user `statd' (UID 111) with group `nogroup' ...
```

**Смотрим настройки сервера в файле /etc/nfs.conf:**  

```
root@otus2:~# cat /etc/nfs.conf
#
# This is a general configuration for the
# NFS daemons and tools
#
[general]
pipefs-directory=/run/rpc_pipefs
#
[nfsrahead]
# nfs=15000
# nfs4=16000
#
[exports]
# rootdir=/export
#
[exportfs]
# debug=0
#
[gssd]
# verbosity=0
# rpc-verbosity=0
# use-memcache=0
# use-machine-creds=1
# use-gss-proxy=0
# avoid-dns=1
# limit-to-legacy-enctypes=0
# context-timeout=0
# rpc-timeout=5
# keytab-file=/etc/krb5.keytab
# cred-cache-directory=
# preferred-realm=
# set-home=1
# upcall-timeout=30
# cancel-timed-out-upcalls=0
#
[lockd]
# port=0
# udp-port=0
#
[exportd]
# debug="all|auth|call|general|parse"
# manage-gids=n
# state-directory-path=/var/lib/nfs
# threads=1
# cache-use-ipaddr=n
# ttl=1800
[mountd]
# debug="all|auth|call|general|parse"
manage-gids=y
# descriptors=0
# port=0
# threads=1
# reverse-lookup=n
# state-directory-path=/var/lib/nfs
# ha-callout=
# cache-use-ipaddr=n
# ttl=1800
#
[nfsdcld]
# debug=0
# storagedir=/var/lib/nfs/nfsdcld
#
[nfsdcltrack]
# debug=0
# storagedir=/var/lib/nfs/nfsdcltrack
#
[nfsd]
# debug=0
# threads=8
# host=
# port=0
# grace-time=90
# lease-time=90
# udp=n
# tcp=y
# vers3=y
# vers4=y
# vers4.0=y
# vers4.1=y
# vers4.2=y
# rdma=n
# rdma-port=20049

[statd]
# debug=0
# port=0
# outgoing-port=0
# name=
# state-directory-path=/var/lib/nfs/statd
# ha-callout=
# no-notify=0
#
[sm-notify]
# debug=0
# force=0
# retry-time=900
# outgoing-port=
# outgoing-addr=
# lift-grace=y
#
[svcgssd]
# principal=
```

**Просматриваем открытые порты на NFS сервере(интересуют 111 и 2049):**  
```
Параметры  команды  ss:
	-t(tcp)
	-n(no resolving)
	-p(show process using socket)
	-l(display only listening sockets)
	-u(udp)
	
root@otus2:~# ss -tnplu | grep 111
udp   UNCONN 0      0            0.0.0.0:111        0.0.0.0:*    users:(("rpcbind",pid=12384,fd=5),("systemd",pid=1,fd=239))
udp   UNCONN 0      0               [::]:111           [::]:*    users:(("rpcbind",pid=12384,fd=7),("systemd",pid=1,fd=241))
tcp   LISTEN 0      4096         0.0.0.0:111        0.0.0.0:*    users:(("rpcbind",pid=12384,fd=4),("systemd",pid=1,fd=238))
tcp   LISTEN 0      4096            [::]:111           [::]:*    users:(("rpcbind",pid=12384,fd=6),("systemd",pid=1,fd=240))
root@otus2:~# ss -tnplu | grep 2049
tcp   LISTEN 0      64           0.0.0.0:2049       0.0.0.0:*
tcp   LISTEN 0      64              [::]:2049          [::]:*
```

**Создаем каталог для публикации через NFS:**  
```
mkdir -p /srv/share/upload    
-p: создаем дочерние каталоги
```

**Рекурсивно удаляем  владельцев на подкаталоги NFS:**  
```
chown -R nobody:nogroup /srv/share
```

**Устанавливаем для  upload полный доступ на чтение, ЗАПИСЬ и выполнение для владельца, группы и всех остальных пользователей:**  
```
chmod 0777 /srv/share/upload 
```

**Редактируем файл exports, здесь задаются права доступа:**  
```
root@otus2:~# cat /etc/exports
/etc/exports: the access control list for filesystems which may be exported


** Example for NFSv2 and NFSv3:
 /srv/homes       hostname1(rw,sync,no_subtree_check) hostname2(ro,sync,no_subtree_check)**

root@otus2:~# cat << EOF > /etc/exports
/srv/share 10.0.1.167/32(rw,sync,root_squash)
EOF
root@otus2:~# cat /etc/exports
/srv/share 10.0.1.167/32(rw,sync,root_squash)
-------Root_squash prevents the root user on a client machine from having superuser privileges on the NFS server
```

**Экспортируем директорию сначала с предупреждением:**
```
root@otus2:~# exportfs -r
exportfs: /etc/exports [1]: Neither 'subtree_check' or 'no_subtree_check' specified for export "10.0.1.167/32:/srv/share".
  Assuming default behaviour ('no_subtree_check').
```

**а потом успешно, добавив параметр subtree_check:**
```root@otus2:~# cat << EOF > /etc/exports
/srv/share 10.0.1.167/32(rw,sync,root_squash,subtree_check)
EOF
root@otus2:~# exportfs -r
```

**Просматриваем текущий export list на NFS сервере:**  
```
root@otus2:~# exportfs -s
/srv/share  10.0.1.167/32(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,root_squash,no_all_squash)

```
##

**3. Реализовать атоматическое монтирование на клиенте при старте виртуальной машины OTUS3 (systemd, autofs или fstab — любым способом) с использованием NFSv3.** 

**Инсталлируем клиент из репозитория:** 
```
sudo apt install nfs-common
```

**Для автоматического монтирования правим fstab:**
```
echo "10.0.1.166:/srv/share/ /mnt nfs vers=3,noauto,x-systemd.automount 0 0" >> /etc/fstab
```

**Рассматриваем fstab:**  
```
root@otus3:~# cat /etc/fstab | grep 166
10.0.1.166:/srv/share/ /mnt nfs vers=3,noauto,x-systemd.automount 0 0
```
**Перегрузим конфигурацию systemd:**
```
systemctl daemon-reload
```

**Перемонтируем fstab:**
```
systemctl restart remote-fs.target
```

***Перезагрузили и смотрим вывод mount:***  
```
root@otus3:/mnt/upload# mount | grep 166
10.0.1.166:/srv/share/ on /mnt type nfs (rw,relatime,vers=3,rsize=1048576,wsize=1048576,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,mountaddr=10.0.1.166,mountvers=3,mountport=45272,mountproto=udp,local_lock=none,addr=10.0.1.166)
```

##### Видим, что mountvers=3. Т.е. NFSv3, что и необходимо согласно заданию.

##### На клиенте otus3 в подключенном каталоге NFS сервера создали файл:  

```root@otus3:/mnt/upload# touch test_upload_from_otus3```

**На сервере OTUS2 новосозданный файл увидели. Вот он:**
```
root@otus2:/srv/share/upload# ls
test1  test_upload_from_otus3
```

**На сервере otus2 проверяем состояние служб NFS для "публикации" файлов:**  

```root@otus2:/srv/share/upload# showmount -a 10.0.1.166
All mount points on 10.0.1.166:
10.0.1.167:/srv/share

root@otus2:/srv/share/upload# exportfs -s
/srv/share  10.0.1.167/32(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,root_squash,no_all_squash)
```
**После перезагрузки  otus2 и otus 3  мы смогли на клиенте создать файл на сервере, значит каталоги отдаются и монтируются в автоматическом режиме:**

```
api027@otus2:/srv/share$ cd /srv/share/upload/
api027@otus2:/srv/share/upload$ ls -lha
-rw-r--r-- 1 nobody nogroup    0 May 29 10:17 final_test_file
```

##
**3.1. Создание скриптов для настройки сервера и клиента.**
```
Перед запуском делаем chmod +x
```


**Скрипт для настройки NFS-сервера.** 


```
root@otus2:/home/api027# cat nfss_script.sh
#!/bin/bash
apt install -y nfs-kernel-server
mkdir -p /srv/share/upload
chown -R nobody:nogroup /srv/share
chmod 0777 /srv/share/upload
#Pass multi-line string to a file in Bash
cat << EOF > /etc/exports
/srv/share 10.0.1.167/32(rw,sync,root_squash,subtree_check)
EOF
# End of Pass multi-line string to a file in Bash
exportfs -r
```

**Запускается без ошибок:**

```
root@otus2:/home/api027# ./nfss_script.sh
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
nfs-kernel-server is already the newest version (1:2.6.4-3ubuntu5.1).
0 upgraded, 0 newly installed, 0 to remove and 80 not upgraded.
```




**Скрипт для настройки NFS-клиента.**
```
root@otus3:~# cat nfsc_script.sh
#!/bin/bash
sudo apt install -y nfs-common
echo "10.0.1.166:/srv/share/ /mnt nfs vers=3,noauto,x-systemd.automount 0 0" >> /etc/fstab
echo "daemon reloaded"
systemctl daemon-reload
echo "restart remote-fs.target"
systemctl restart remote-fs.target
```

***Проверка запуска:***
```
root@otus3:~# ./nfsc_script.sh
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
nfs-common is already the newest version (1:2.6.4-3ubuntu5.1).
0 upgraded, 0 newly installed, 0 to remove and 79 not upgraded.
daemon reloaded
restart remote-fs.target
```

**Ошибок не выдает.**

**Конец домашней работы.**




**Заметки:**
1.) Unmount nfs раздела на клиенте  получился только с **umount -l**
-l, --lazy
           Lazy unmount. Detach the filesystem from the file hierarchy now, and clean up all references to this filesystem as soon as it is not busy anymore.

2.) **“>”** is the output redirection operator used for **overwriting** files that already exist in the directory. 
While, the “**>>**” is an output operator as well, but, it **appends** the data of an existing file
**
