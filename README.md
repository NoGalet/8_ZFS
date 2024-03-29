Практические навыки работы с ZFS

Цели домашнего задания
Научится самостоятельно устанавливать ZFS, настраивать пулы, изучить основные возможности ZFS. 


Описание домашнего задания:
Определить алгоритм с наилучшим сжатием:
Определить какие алгоритмы сжатия поддерживает zfs (gzip, zle, lzjb, lz4);
создать 4 файловых системы на каждой применить свой алгоритм сжатия;
для сжатия использовать либо текстовый файл, либо группу файлов.
Определить настройки пула.
С помощью команды zfs import собрать pool ZFS.
Командами zfs определить настройки:
    - размер хранилища;
    - тип pool;
    - значение recordsize;
    - какое сжатие используется;
    - какая контрольная сумма используется.
Работа со снапшотами:
скопировать файл из удаленной директории;
восстановить файл локально. zfs receive;
найти зашифрованное сообщение в файле secret_message.
_______________________

1) Смотрим список всех дисков, которые есть в виртуальной машине:
[vagrant@ubuntu2 ~]$ lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0   40G  0 disk
`-sda1   8:1    0   40G  0 part /
sdb      8:16   0  512M  0 disk
sdc      8:32   0  512M  0 disk
sdd      8:48   0  512M  0 disk
sde      8:64   0  512M  0 disk
sdf      8:80   0  512M  0 disk
sdg      8:96   0  512M  0 disk
sdh      8:112  0  512M  0 disk
sdi      8:128  0  512M  0 disk

2) Переходим в root пользователя:
[vagrant@ubuntu2 ~]$ sudo -i

3) Создаём пул из двух дисков в режиме RAID 1:
[root@ubuntu2 ~]# zpool create otus1 mirror /dev/sdb /dev/sdc

4) Создадим ещё 3 пула: 
[root@ubuntu2 ~]# zpool create otus2 mirror /dev/sdd /dev/sde
[root@ubuntu2 ~]# zpool create otus3 mirror /dev/sdf /dev/sdg
[root@ubuntu2 ~]# zpool create otus4 mirror /dev/sdh /dev/sdi

5) Смотрим информацию о пулах:
[root@ubuntu2 ~]# zpool list
NAME    SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
otus1   480M   106K   480M        -         -     0%     0%  1.00x    ONLINE  -
otus2   480M   106K   480M        -         -     0%     0%  1.00x    ONLINE  -
otus3   480M   106K   480M        -         -     0%     0%  1.00x    ONLINE  -
otus4   480M   106K   480M        -         -     0%     0%  1.00x    ONLINE  -

6) Добавим разные алгоритмы сжатия в каждую файловую систему:
[root@ubuntu2 ~]# zfs set compression=lzjb otus1
[root@ubuntu2 ~]# zfs set compression=lz4 otus2
[root@ubuntu2 ~]# zfs set compression=gzip-9 otus3
[root@ubuntu2 ~]# zfs set compression=zle otus4

7) Проверим, что все файловые системы имеют разные методы сжатия:
[root@ubuntu2 ~]# zfs get all | grep compression
otus1  compression           lzjb                   local
otus2  compression           lz4                    local
otus3  compression           gzip-9                 local
otus4  compression           zle                    local

8) Скачаем один и тот же текстовый файл во все пулы: 
[root@ubuntu2 ~]# for i in {1..4}; do wget -P /otus$i https://gutenberg.org/cache/epub/2600/pg2600.converter.log; done
--2024-02-11 13:02:57--  https://gutenberg.org/cache/epub/2600/pg2600.converter.log
Resolving gutenberg.org (gutenberg.org)... 152.19.134.47, 2610:28:3090:3000:0:bad:cafe:47
Connecting to gutenberg.org (gutenberg.org)|152.19.134.47|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 41016061 (39M) [text/plain]
Saving to: '/otus1/pg2600.converter.log'

100%[============================================================>] 41,016,061  1.03MB/s   in 35s

2024-02-11 13:03:33 (1.12 MB/s) - '/otus1/pg2600.converter.log' saved [41016061/41016061]

--2024-02-11 13:03:34--  https://gutenberg.org/cache/epub/2600/pg2600.converter.log
Resolving gutenberg.org (gutenberg.org)... 152.19.134.47, 2610:28:3090:3000:0:bad:cafe:47
Connecting to gutenberg.org (gutenberg.org)|152.19.134.47|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 41016061 (39M) [text/plain]
Saving to: '/otus2/pg2600.converter.log'

100%[============================================================>] 41,016,061   957KB/s   in 65s

2024-02-11 13:04:40 (616 KB/s) - '/otus2/pg2600.converter.log' saved [41016061/41016061]

--2024-02-11 13:04:40--  https://gutenberg.org/cache/epub/2600/pg2600.converter.log
Resolving gutenberg.org (gutenberg.org)... 152.19.134.47, 2610:28:3090:3000:0:bad:cafe:47
Connecting to gutenberg.org (gutenberg.org)|152.19.134.47|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 41016061 (39M) [text/plain]
Saving to: '/otus3/pg2600.converter.log'

97% [==========================================================>  ] 40,108,615  1.40MB/s   in 29s

2024-02-11 13:05:11 (1.30 MB/s) - Connection closed at byte 40108615. Retrying.

--2024-02-11 13:05:13--  (try: 2)  https://gutenberg.org/cache/epub/2600/pg2600.converter.log
Connecting to gutenberg.org (gutenberg.org)|152.19.134.47|:443... connected.
HTTP request sent, awaiting response... 206 Partial Content
Length: 41016061 (39M), 907446 (886K) remaining [text/plain]
Saving to: '/otus3/pg2600.converter.log'

100%[+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++=>] 41,016,061   131KB/s   in 8.8s

2024-02-11 13:05:25 (101 KB/s) - '/otus3/pg2600.converter.log' saved [41016061/41016061]

--2024-02-11 13:05:25--  https://gutenberg.org/cache/epub/2600/pg2600.converter.log
Resolving gutenberg.org (gutenberg.org)... 152.19.134.47, 2610:28:3090:3000:0:bad:cafe:47
Connecting to gutenberg.org (gutenberg.org)|152.19.134.47|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 41016061 (39M) [text/plain]
Saving to: '/otus4/pg2600.converter.log'

100%[============================================================>] 41,016,061  1.73MB/s   in 27s

2024-02-11 13:05:53 (1.45 MB/s) - '/otus4/pg2600.converter.log' saved [41016061/41016061]

9) Проверим, что файл был скачан во все пулы:
[root@ubuntu2 ~]# ls -l /otus*
/otus1:
total 22067
-rw-r--r--. 1 root root 41016061 Feb  2 08:53 pg2600.converter.log

/otus2:
total 17994
-rw-r--r--. 1 root root 41016061 Feb  2 08:53 pg2600.converter.log

/otus3:
total 10959
-rw-r--r--. 1 root root 41016061 Feb  2 08:53 pg2600.converter.log

/otus4:
total 40091
-rw-r--r--. 1 root root 41016061 Feb  2 08:53 pg2600.converter.log

10) Проверим, сколько места занимает один и тот же файл в разных пулах и проверим степень сжатия файлов:
[root@ubuntu2 ~]# zfs list
NAME    USED  AVAIL     REFER  MOUNTPOINT
otus1  21.7M   330M     21.6M  /otus1
otus2  17.7M   334M     17.6M  /otus2
otus3  10.8M   341M     10.7M  /otus3
otus4  39.3M   313M     39.2M  /otus4

[root@ubuntu2 ~]# zfs get all | grep compressratio | grep -v ref
otus1  compressratio         1.81x                  -
otus2  compressratio         2.22x                  -
otus3  compressratio         3.65x                  -
otus4  compressratio         1.00x                  -

11) Скачиваем архив в домашний каталог: 
[root@ubuntu2 ~]# wget -O archive.tar.gz --no-check-certificate 'https://drive.usercontent.google.com/download?id=1MvrcEp-WgAQe57aDEzxSRalPAwbNN1Bb&export=download
> '
--2024-02-11 13:18:34--  https://drive.usercontent.google.com/download?id=1MvrcEp-WgAQe57aDEzxSRalPAwbNN1Bb&export=download%0A
Resolving drive.usercontent.google.com (drive.usercontent.google.com)... 64.233.163.132, 2a00:1450:4010:c06::84
Connecting to drive.usercontent.google.com (drive.usercontent.google.com)|64.233.163.132|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 7275140 (6.9M) [application/octet-stream]
Saving to: 'archive.tar.gz'

100%[============================================================>] 7,275,140   2.39MB/s   in 2.9s

2024-02-11 13:18:43 (2.39 MB/s) - 'archive.tar.gz' saved [7275140/7275140]

12) Разархивируем его:
[root@ubuntu2 ~]# tar -xzvf archive.tar.gz
zpoolexport/
zpoolexport/filea
zpoolexport/fileb

13) Проверим, возможно ли импортировать данный каталог в пул:
[root@ubuntu2 ~]# zpool import -d zpoolexport/
   pool: otus
     id: 6554193320433390805
  state: ONLINE
 action: The pool can be imported using its name or numeric identifier.
 config:

        otus                         ONLINE
          mirror-0                   ONLINE
            /root/zpoolexport/filea  ONLINE
            /root/zpoolexport/fileb  ONLINE

14) Сделаем импорт данного пула к нам в ОС:
[root@ubuntu2 ~]# zpool status
  pool: otus
 state: ONLINE
  scan: none requested
config:

        NAME                         STATE     READ WRITE CKSUM
        otus                         ONLINE       0     0     0
          mirror-0                   ONLINE       0     0     0
            /root/zpoolexport/filea  ONLINE       0     0     0
            /root/zpoolexport/fileb  ONLINE       0     0     0

errors: No known data errors

15) Далее нам нужно определить настройки:
[root@ubuntu2 ~]# zpool get all otus
NAME  PROPERTY                       VALUE                          SOURCE
otus  size                           480M                           -
otus  capacity                       0%                             -
otus  altroot                        -                              default
otus  health                         ONLINE                         -
otus  guid                           6554193320433390805            -
otus  version                        -                              default
otus  bootfs                         -                              default
otus  delegation                     on                             default
otus  autoreplace                    off                            default
otus  cachefile                      -                              default
otus  failmode                       wait                           default
otus  listsnapshots                  off                            default
otus  autoexpand                     off                            default
otus  dedupditto                     0                              default
otus  dedupratio                     1.00x                          -
otus  free                           478M                           -
otus  allocated                      2.09M                          -
otus  readonly                       off                            -
otus  ashift                         0                              default
otus  comment                        -                              default
otus  expandsize                     -                              -
otus  freeing                        0                              -
otus  fragmentation                  0%                             -
otus  leaked                         0                              -
otus  multihost                      off                            default
otus  checkpoint                     -                              -
otus  load_guid                      10325597362898850483           -
otus  autotrim                       off                            default
otus  feature@async_destroy          enabled                        local
otus  feature@empty_bpobj            active                         local
otus  feature@lz4_compress           active                         local
otus  feature@multi_vdev_crash_dump  enabled                        local
otus  feature@spacemap_histogram     active                         local
otus  feature@enabled_txg            active                         local
otus  feature@hole_birth             active                         local
otus  feature@extensible_dataset     active                         local
otus  feature@embedded_data          active                         local
otus  feature@bookmarks              enabled                        local
otus  feature@filesystem_limits      enabled                        local
otus  feature@large_blocks           enabled                        local
otus  feature@large_dnode            enabled                        local
otus  feature@sha512                 enabled                        local
otus  feature@skein                  enabled                        local
otus  feature@edonr                  enabled                        local
otus  feature@userobj_accounting     active                         local
otus  feature@encryption             enabled                        local
otus  feature@project_quota          active                         local
otus  feature@device_removal         enabled                        local
otus  feature@obsolete_counts        enabled                        local
otus  feature@zpool_checkpoint       enabled                        local
otus  feature@spacemap_v2            active                         local
otus  feature@allocation_classes     enabled                        local
otus  feature@resilver_defer         enabled                        local
otus  feature@bookmark_v2            enabled                        local

16) Запрос сразу всех параметром файловой системы:
[root@ubuntu2 ~]# zfs get all otus
NAME  PROPERTY              VALUE                  SOURCE
otus  type                  filesystem             -
otus  creation              Fri May 15  4:00 2020  -
otus  used                  2.04M                  -
otus  available             350M                   -
otus  referenced            24K                    -
otus  compressratio         1.00x                  -
otus  mounted               yes                    -
otus  quota                 none                   default
otus  reservation           none                   default
otus  recordsize            128K                   local
otus  mountpoint            /otus                  default
otus  sharenfs              off                    default
otus  checksum              sha256                 local
otus  compression           zle                    local
otus  atime                 on                     default
otus  devices               on                     default
otus  exec                  on                     default
otus  setuid                on                     default
otus  readonly              off                    default
otus  zoned                 off                    default
otus  snapdir               hidden                 default
otus  aclinherit            restricted             default
otus  createtxg             1                      -
otus  canmount              on                     default
otus  xattr                 on                     default
otus  copies                1                      default
otus  version               5                      -
otus  utf8only              off                    -
otus  normalization         none                   -
otus  casesensitivity       sensitive              -
otus  vscan                 off                    default
otus  nbmand                off                    default
otus  sharesmb              off                    default
otus  refquota              none                   default
otus  refreservation        none                   default
otus  guid                  14592242904030363272   -
otus  primarycache          all                    default
otus  secondarycache        all                    default
otus  usedbysnapshots       0B                     -
otus  usedbydataset         24K                    -
otus  usedbychildren        2.01M                  -
otus  usedbyrefreservation  0B                     -
otus  logbias               latency                default
otus  objsetid              54                     -
otus  dedup                 off                    default
otus  mlslabel              none                   default
otus  sync                  standard               default
otus  dnodesize             legacy                 default
otus  refcompressratio      1.00x                  -
otus  written               24K                    -
otus  logicalused           1020K                  -
otus  logicalreferenced     12K                    -
otus  volmode               default                default
otus  filesystem_limit      none                   default
otus  snapshot_limit        none                   default
otus  filesystem_count      none                   default
otus  snapshot_count        none                   default
otus  snapdev               hidden                 default
otus  acltype               off                    default
otus  context               none                   default
otus  fscontext             none                   default
otus  defcontext            none                   default
otus  rootcontext           none                   default
otus  relatime              off                    default
otus  redundant_metadata    all                    default
otus  overlay               off                    default
otus  encryption            off                    default
otus  keylocation           none                   default
otus  keyformat             none                   default
otus  pbkdf2iters           0                      default
otus  special_small_blocks  0                      default

17)C помощью команды grep можно уточнить конкретный параметр, например:
a) Размер:
[root@ubuntu2 ~]# zfs get available otus
NAME  PROPERTY   VALUE  SOURCE
otus  available  350M   -

b) Тип:
[root@ubuntu2 ~]# zfs get readonly otus
NAME  PROPERTY  VALUE   SOURCE
otus  readonly  off     default

c) Значение recordsize:
[root@ubuntu2 ~]# zfs get recordsize otus
NAME  PROPERTY    VALUE    SOURCE
otus  recordsize  128K     local

d) Тип сжатия (или параметр отключения):
[root@ubuntu2 ~]# zfs get compression otus
NAME  PROPERTY     VALUE     SOURCE
otus  compression  zle       local

e) Тип контрольной суммы:
[root@ubuntu2 ~]# zfs get checksum otus
NAME  PROPERTY  VALUE      SOURCE
otus  checksum  sha256     local

18) Скачаем файл, указанный в задании:
[root@ubuntu2 ~]# --2024-02-11 13:31:59--  https://drive.usercontent.google.com/download?id=1wgxjih8YZ-cqLqaZVa0lA3h3Y029c3oI
Saving to: 'otus_task2.file'
100%[============================================================>] 5,432,736   2.44MB/s   in 2.1s
2024-02-11 13:32:07 (2.44 MB/s) - 'otus_task2.file' saved [5432736/5432736]
[1]+  Done 

19) Восстановим файловую систему из снапшота:
[root@ubuntu2 ~]# zfs receive otus/test@today < otus_task2.file

20) Далее, ищем в каталоге /otus/test файл с именем “secret_message”:
[root@ubuntu2 ~]# find /otus/test -name "secret_message"
/otus/test/task1/file_mess/secret_message

21) Смотрим содержимое найденного файла:
[root@ubuntu2 ~]# cat /otus/test/task1/file_mess/secret_message
\https://otus.ru/lessons/linux-hl/
