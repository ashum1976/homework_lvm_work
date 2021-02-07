#                                                    Homework_lvm_work

#                                                       1 Начало работы с LVM

            1.1 Введение
            
LVM — менеджер, позволяющий управлять логическими томами в системах Linux. Сами логические тома можно собрать из нескольких дисков или разделов дисков. LVM расшифровывается как Logical Volume Manager — менеджер логических томов.

Возможности:

* Thin provisioning.
* Использование RAID.
* Снимки диска (snapshot).
* Простое добавление нового физического диска.
* Уменьшение тома.
* Расширение тома (в том числе и корневого) без необходимости отмонтирования раздела.


Работа с томами при помощи LVM организована на 3-х уровнях:

* Физический том (PV). Физический диск или раздел.
* Группа томов (VG). Объединение физических томов.
* Логический том (LV). Раздел группы томов. Может содержать файловую систему.
* Физические и логические тома, в свою очередь, делятся на фрагменты (экстенты):

            Физический экстент (PE). Разделение физических томов.
            Логический экстент (LE). Разделение логических томов.
   
#                                                       1.2 Установка
            
Для работы с LVM необходима установка одноименной утилиты lvm2. В Vagrantfile, в секции provision добавляем для установки с помощью yum (yum install ..... lvm2)
Перед созданием томов, необходимо определиться, на каких дисках это делаем. 

    
            [root@lvm ~]# lsblk                                                                                                    
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

            
или утилита lvmdiskscan

            [root@lvm ~]# lvmdiskscan                                                                  
            /dev/sda2 [       1.00 GiB]                                                                
            /dev/sda3 [     <39.00 GiB] LVM physical volume                                                
            /dev/sdb  [      10.00 GiB]                                                                     
            /dev/sdc  [       2.00 GiB]                                                                   
            /dev/sdd  [       1.00 GiB]                                                                    
            /dev/sde  [       1.00 GiB]                                                                    
            4 disks                                                                                         
            1 partition                                                                                                                           
            0 LVM physical volume whole disks                                                             
            1 LVM physical volume

            
Есть диски /dev/sdb, /dev/sdc, /dev/sdd, /dev/sde  на них будм создавать новый том LVM

            1.2.1 Создание разделов

Помечаем диски, что они будут использоваться для LVM:

            pvcreate /dev/sdb
            Physical volume "/dev/sdc" successfully created.

            
команда pvdisplay покажет диски, которые можно использовать для создания LVM 
            
            [root@lvm ~]# pvdisplay
            
            --- Physical volume ---
            PV Name               /dev/sdb
            VG Name               vghw
            PV Size               10.00 GiB / not usable 4.00 MiB
            Allocatable           yes 
            PE Size               4.00 MiB
            Total PE              2559
            Free PE               511
            Allocated PE          2048
            PV UUID               rb8XQi-jZxk-eG5i-ofO6-15c7-aQVn-t96le8


* где 

- PV Name — имя диска. 
- VG Name — группа томов, в которую входит данный диск (в нашем случае уже создана группа и диск добавлен в неё).
- PV Size — размер диска.
- Allocatable — распределение по группам. Если NO, то диск еще не задействован и его необходимо для использования включить в группу.
- PE Size — размер физического фрагмента (экстента). Пока диск не добавлен в группу, значение будет 0.
- Total PE — количество физических экстентов.
- Free PE — количество свободных физических экстентов.
- Allocated PE — распределенные экстенты.
- PV UUID — идентификатор физического раздела.
        
        
        
            1.2.2 Создание групп томов

            
Инициализированные на первом этапе диски должны быть объединены в группы.

Группа может быть создана:

            vgcreate vghw /dev/sdb


* где vghw — произвольное имя создаваемой группы; /dev/sdb - добавляемый диск в группу.


Просмотреть информацию о созданных группах можно командой:

            vgdisplay vghw

            --- Volume group ---
            VG Name               vghw
            System ID             
            Format                lvm2
            Metadata Areas        2
            Metadata Sequence No  7
            VG Access             read/write
            VG Status             resizable
            MAX LV                0
            Cur LV                1
            Open LV               0
            Max PV                0
            Cur PV                2
            Act PV                2
            VG Size               11.99 GiB
            PE Size               4.00 MiB
            Total PE              3070
            Alloc PE / Size       2048 / 8.00 GiB
            Free  PE / Size       1022 / 3.99 GiB
            VG UUID               7YMmA6-ENcd-zC1h-kUow-coXW-zKY1-S0RhfN
            
            
            
            --- Volume group ---                                                                           
  VG Name               vghw                                                                     
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
  Alloc PE / Size       2048 / 8.00 GiB                                                       
  Free  PE / Size       511 / <2.00 GiB                                                                                
  VG UUID               7YMmA6-ENcd-zC1h-kUow-coXW-zKY1-S0RhfN 

* где:

- VG Name — имя группы.
- Format — версия подсистемы, используемая для создания группы.
- Metadata Areas — область размещения метаданных. Увеличивается на единицу с созданием каждой группы.
- VG Access — уровень доступа к группе томов.
- VG Size — суммарный объем всех дисков, которые входят в группу.
- PE Size — размер физического фрагмента (экстента).
- Total PE — количество физических экстентов.
- Alloc PE / Size — распределенное пространство: колическтво экстентов / объем.
- Free  PE / Size — свободное пространство: колическтво экстентов / объем.
- VG UUID — идентификатор группы.


            1.2.3 Создание логических томов

Последний этап — создание логического раздела их группы томов командой lvcreate. Ее синтаксис:

Cоздания логических томов:

            lvcreate -L 8G vghw -n rootvghw

* создание тома на 8 Гб из группы vghw, с именем rootvghw

            lvcreate -l 80%VG vghw

* при создании тома используется 80% от дискового пространства группы vghw

* также можно использовать %PVS — процент места от физического тома (PV); %ORIGIN — размер оригинального тома (применяется для снапшотов).



            [root@lvm ~]# lvdisplay /dev/vghw/rootvghw                                                       
            --- Logical volume ---                                                                         
            LV Path                /dev/vghw/rootvghw                                                     
            LV Name                rootvghw                                                            
            VG Name                vghw                                                                 
            LV UUID                emmhfo-uX0Z-Grna-D87m-qdTi-OP31-5udcug                              
            LV Write Access        read/write                                                           
            LV Creation host, time lvm, 2021-02-07 18:39:31 +0000                                       
            LV Status              available                                                           
            LV Size                8.00 GiB                                                              
            Current LE             2048                                                                
            Segments               1                                                                                                                
            Allocation             inherit                                                              
            Read ahead sectors     auto                                                                                                   
            Block device           253:2


* где:

- LV Path — путь к устройству логического тома.
- LV Name — имя логического тома.
- VG Name — имя группы томов.
- LV UUID — идентификатор.
- LV Write Access — уровень доступа.
- LV Creation host, time — имя компьютера и дата, когда был создан том.
- LV Size — объем дискового пространства, доступный для использования.
- Current LE — количество логических экстентов.


#                                                         1.3 Создание файловой системы и монтирование тома

Чтобы начать использовать созданный том, необходимо его отформатировать, создав файловую систему и примонтировать раздел в каталог. Процесс создания файловой системы на томах LVM ничем не отличается от работы с любыми другими разделами.
Для создания файловой системы ext4 вводим:

            [root@lvm ~]# mkfs.ext4 /dev/vghw/rootvghw
            
Для разового монтирования

            [root@lvm ~]# mkdir /mnt/data
            [root@lvm ~]# mount /dev/vghw/rootvghw /mnt/data/

Для постоянного монтирования созданного раздела добавляем строку в fstab:

            /dev/vghw/rootvghw  /mnt/data   ext4    defaults        1 2
            
#                                                       1.4 Расширение физического тома

Расширение физического раздела можно сделать за счет добавление нового диска или увеличение дискового пространства имеющегося (например, если диск виртуальный).

Если добавляем еще один диск.

Инициализируем новый диск:

            pvcreate /dev/sdc
            [root@lvm ~]# pvcreate /dev/sdc
            Physical volume "/dev/sdc" successfully created. 
            
* в данном примере мы инициализировали диск sdc.

Проверяем объем физического тома:

            [root@lvm ~]# pvdisplay
            
            "/dev/sdc" is a new physical volume of "2.00 GiB"
            --- NEW Physical volume ---
            PV Name               /dev/sdc
            VG Name               
            PV Size               2.00 GiB
            Allocatable           NO
            PE Size               0   
            Total PE              0
            Free PE               0
            Allocated PE          0
            PV UUID               VLlt80-2xBS-y0bg-Ap2I-nvms-ONJB-TBTymR
            

б) Если увеличиваем дисковое пространство имеющегося диска.

Увеличиваем размер физического (виртуального диска) диска командой:

            pvresize /dev/sdc

* где /dev/sdc — диск, который был увеличен.


            1.4.1 Добавление нового диска к группе томов

Расширяем группу томов командой:

            [root@lvm ~]# vgextend vghw /dev/sdc
            Volume group "vghw" successfully extended


* данная команда расширит группу vghw за счет добавленого sdc.

Результат можно увидеть командой:

            [root@lvm ~]# vgdisplay vghw
            
            Act PV                2
            VG Size               11.99 GiB
            PE Size               4.00 MiB
            Total PE              3070
            Alloc PE / Size       2048 / 8.00 GiB
            Free  PE / Size       1022 / 3.99 GiB

            1.4.2 Увеличение логического раздела

Выполняется одной командой.

а) все свободное пространство:

            [root@lvm ~]# lvextend -l+100%FREE /dev/vghw/rootvghw 
            
            Size of logical volume vghw/rootvghw changed from 8.00 GiB (2048 extents) to 11.99 GiB (3070 extents).
            Logical volume vghw/rootvghw successfully resized.
        
        
* данной командой мы выделяем все свободное пространство группы томов vghw разделу rootvghw.

б) определенный объем:



            lvresize -L+2G /dev/vghw/rootvghw 
            
            Size of logical volume vghw/rootvghw changed from 8.00 GiB (2048 extents) to 10.00 GiB (2560 extents).
            Logical volume vghw/rootvghw successfully resized.

* данной командой мы добавляем 2 Гб от группы томов vghw разделу rootvghw.

в) до нужного объема:



            [root@lvm ~]# lvextend -L11.99G /dev/vghw/rootvghw 
            
            Rounding size to boundary between physical extents: 11.99 GiB.
            Size of logical volume vghw/rootvghw changed from 8.00 GiB (2048 extents) to 11.99 GiB (3070 extents).
            Logical volume vghw/rootvghw successfully resized.

* данной командой мы доводим наш раздел до объема в 11.99 Гб.

Результат можно увидеть командой:

            [root@lvm ~]# lvdisplay /dev/vghw/rootvghw 
            --- Logical volume ---
            LV Path                /dev/vghw/rootvghw
            LV Name                rootvghw
            VG Name                vghw
            LV UUID                emmhfo-uX0Z-Grna-D87m-qdTi-OP31-5udcug
            LV Write Access        read/write
            LV Creation host, time lvm, 2021-02-07 18:39:31 +0000
            LV Status              available
            # open                 0
            LV Size                11.99 GiB
            Current LE             3070
            Segments               2
            Allocation             inherit
            Read ahead sectors     auto
            - currently set to     8192
            Block device           253:2

Обратить внимание нужно на опцию:
    
            LV Size                11.99 GiB

            1.4.3 Расширение файловой системы

            
Размер раздела увеличен до 11.99 Gb, но созданная ранее файловая система осталась на размере 8Gb 
        
            df -Th  
            
            Filesystem                      Type      Size  Used Avail Use% Mounted on
            .........
            /dev/mapper/vghw-rootvghw       ext4      7.8G   36M  7.3G   1% /mnt/data
            
Произведем resize файловой системы:


            [root@lvm ~]# resize2fs /dev/vghw/rootvghw 
            resize2fs 1.42.9 (28-Dec-2013)
            Filesystem at /dev/vghw/rootvghw is mounted on /mnt/data; on-line resizing required
            old_desc_blocks = 1, new_desc_blocks = 2
            The filesystem on /dev/vghw/rootvghw is now 3143680 blocks long.
            
            [root@lvm ~]# df -Th
            Filesystem                      Type      Size  Used Avail Use% Mounted on
            ................
            /dev/mapper/vghw-rootvghw       ext4       12G   40M   12G   1% /mnt/data

Размер файловой системы теперь 12Gb


#                                                        1.5 Уменьшение томов
            
            
* Размер некоторый файловых систем, например, XFS уменьшить нельзя. Из положения можно выйти, создав новый уменьшенный том с переносом на него данных и                 последующим удалением.

LVM также позволяет уменьшить размер тома. Для этого необходимо выполнить его отмонтирование, проверить  на ошибки и уменьшить размер:

Отмонтируем раздел, который нужно уменьшить:

            [root@lvm ~]# umount /dev/mapper/vghw-rootvghw

Проверим на ошибки с автоматическим исправлением:

            [root@lvm ~]# e2fsck -fy /dev/vghw/rootvghw 
            
            e2fsck 1.42.9 (28-Dec-2013)
            Pass 1: Checking inodes, blocks, and sizes
            Pass 2: Checking directory structure
            Pass 3: Checking directory connectivity
            Pass 4: Checking reference counts
            Pass 5: Checking group summary information
            /dev/vghw/rootvghw: 11/786432 files (0.0% non-contiguous), 92368/3143680 blocks  


Уменьшаем файловую систему раздела:
            
            [root@lvm ~]# resize2fs /dev/vghw/rootvghw 8G
            
            resize2fs 1.42.9 (28-Dec-2013)
            Resizing the filesystem on /dev/vghw/rootvghw to 2097152 (4k) blocks.
            The filesystem on /dev/vghw/rootvghw is now 2097152 blocks long.
            
Уменьшаем сам раздел:

            [root@lvm ~]# lvreduce /dev/vghw/rootvghw -L 8G
            
            WARNING: Reducing active logical volume to 8.00 GiB.
            THIS MAY DESTROY YOUR DATA (filesystem etc.)
            Do you really want to reduce vghw/rootvghw? [y/n]: y
            Size of logical volume vghw/rootvghw changed from 11.99 GiB (3070 extents) to 8.00 GiB (2048 extents).
            Logical volume vghw/rootvghw successfully resized.
            
Монтируем раздел и проверяем его размер:

            [root@lvm ~]# mount /dev/vghw/rootvghw /mnt/data/
            
                      
            
            [root@lvm ~]# df -hT
            
            Filesystem                      Type      Size  Used Avail Use% Mounted on
            /dev/mapper/VolGroup00-LogVol00 xfs        38G  755M   37G   2% /
            devtmpfs                        devtmpfs  109M     0  109M   0% /dev
            tmpfs                           tmpfs     118M     0  118M   0% /dev/shm
            tmpfs                           tmpfs     118M  4.6M  114M   4% /run
            tmpfs                           tmpfs     118M     0  118M   0% /sys/fs/cgroup
            tmpfs                           tmpfs      50M  4.0K   50M   1% /tmp
            /dev/sda2                       xfs      1014M   63M  952M   7% /boot
            tmpfs                           tmpfs      24M     0   24M   0% /run/user/1000
            /dev/mapper/vghw-rootvghw       ext4      7.8G   36M  7.4G   1% /mnt/data

###  Очень важно, чтобы сначала был уменьшен размер файловой системы, затем тома. Также важно не уменьшить размер тома больше, чем файловой системы. В противном случае данные могут быть уничтожены. Перед выполнением операции, обязательно создаем копию важных данных.        
     
#                                                       1.6 Удаление томов


Если необходимо полностью разобрать LVM тома, выполняем следующие действия.

Отмонтируем разделы:

            [root@lvm ~]# umount /dev/mapper/vghw-rootvghw


Удаляем соответствующую запись из fstab если есть (в противном случае наша система может не загрузиться после перезагрузки):

Теперь удаляем логический том:

            [root@lvm ~]# lvremove /dev/vghw/rootvghw 
            
            Do you really want to remove active logical volume vghw/rootvghw? [y/n]: y
            Logical volume "rootvghw" successfully removed


* если система вернет ошибку Logical volume contains a filesystem in use, необходимо убедиться, что мы отмонтировали том.

Удаляем группу томов:

            [root@lvm ~]# vgremove vghw 
            Volume group "vghw" successfully removed

Убираем пометку с дисков на использование их для LVM:

            [root@lvm ~]# pvremove /dev/sdb
            Labels on physical volume "/dev/sdb" successfully wiped.
            
            [root@lvm ~]# pvremove /dev/sdc
            Labels on physical volume "/dev/sdc" successfully wiped.




Создание зеркала
С помощью LVM мы может создать зеркальный том — данные, которые мы будем на нем сохранять, будут отправляться на 2 диска. Таким образом, если один из дисков выходит из строя, мы не потеряем свои данные.

Зеркалирование томов выполняется из группы, где есть, минимум, 2 диска.

1. Сначала инициализируем диски:

pvcreate /dev/sd{d,e}

* в данном примере sdd и sde.

2. Создаем группу:

vgcreate vg02 /dev/sd{d,e}

3. Создаем зеркальный том: 

lvcreate -L200 -m1 -n lv-mir vg02

* мы создали том lv-mir на 200 Мб из группы vg02.

В итоге:

lsblk

... мы увидим что-то на подобие:

sdd                       8:16   0    1G  0 disk
  vg02-lv--mir_rmeta_0  253:2    0    4M  0 lvm
    vg02-lv--mir        253:6    0  200M  0 lvm
  vg02-lv--mir_rimage_0 253:3    0  200M  0 lvm
    vg02-lv--mir        253:6    0  200M  0 lvm
sde                       8:32   0    1G  0 disk
  vg02-lv--mir_rmeta_1  253:4    0    4M  0 lvm
    vg02-lv--mir        253:6    0  200M  0 lvm
  vg02-lv--mir_rimage_1 253:5    0  200M  0 lvm
    vg02-lv--mir        253:6    0  200M  0 lvm

* как видим, на двух дисках у нас появились разделы по 200 Мб.

Работа со снапшотами
Снимки диска позволят нам откатить состояние на определенный момент. Это может послужить быстрым вариантом резервного копирования. Однако нужно понимать, что данные хранятся на одном и том же физическом носителе, а значит, данный способ не является полноценным резервным копированием.

Создание снапшотов для тома, где уже используется файловая система XFS, имеет некоторые нюансы, поэтому разберем разные примеры.

Создание для не XFS:

lvcreate -L500 -s -n sn01 /dev/vg01/lv01

* данная команда помечает, что 500 Мб дискового пространства устройства /dev/vg01/lv01 (тома lv01 группы vg01) будет использоваться для snapshot (опция -s).

Создание для XFS:

xfs_freeze -f /mnt; lvcreate -L500 -s -n sn01 /dev/vg01/lv01; xfs_freeze -u /mnt

* команда xfs_freeze замораживает операции в файловой системе XFS.

Посмотрим список логических томов:

lvs

Получим что-то на подобие:

LV   VG   Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
lv01 vg01 owi-aos---   1,00g
sn01 vg01 swi-a-s--- 500,00m      lv01   2,07

* поле Origin показывает, к какому оригинальному логическому тому относится LV, например, в данной ситуации наш раздел для снапшотов относится к lv01.

Также можно посмотреть изменения в томах командой:

lsblk

Мы должны увидеть что-то подобное:

sdc                8:32   0    1G  0 disk 
  vg01-lv01-real 253:3    0    1G  0 lvm  
    vg01-lv01    253:2    0    1G  0 lvm  /mnt
    vg01-sn01    253:5    0    1G  0 lvm  
  vg01-sn01-cow  253:4    0  500M  0 lvm  
    vg01-sn01    253:5    0    1G  0 lvm 

С этого момента все изменения пишутся в vg01-sn01-cow, а vg01-lv01-real фиксируется только для чтения и мы может откатиться к данному состоянию диска в любой момент.

Содержимое снапшота можно смонтировать и посмотреть, как обычный раздел:

mkdir /tmp/snp

Монтирование не XFS:

mount /dev/vg01/sn01 /tmp/snp

Монтирование XFS:

mount -o nouuid,ro /dev/vg01/sn01 /tmp/snp

Для выполнения отката до снапшота, выполняем команду:

lvconvert --merge /dev/vg01/sn01
            
            
            
            
            
            
            
            
            
            
            
            
