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

            Физический экстент (PE). Разделение физических томов. Размер 4Mb
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


#                                                       1.7 Snapshot LVM

  <span style="color:black">**_Snapshot - моментальный снимок тома LVM_**</span>

---
  **При создании снимка LVM объем резервного копирования метаданных файловой системы копируется в только что созданный снимок тома. Блоки файлов остаются на исходном томе, однако, до тех пор, пока ничего не изменилось в метаданных снимка, все указатели в блоках по исходному объему остаются правильными. <span style="color:red">Когда файл изменяется на исходном томе, оригинальные блоки копируются в снимок тома перед изменением в файловой системе.**

  ---

  Т.е в снимок будут записываться те блоки которые изменились на исходном томе. В snapshot пишется оригинальный блок, и далее он уже в snapshot-e не меняется, как бы не менялся этот блок в оригинальном томе. При откает будет восстановлен этот оригинальный блок, на момент создания снимка. Поэтому важно следить за размером snapshot-a, если изменённые блоки будет некуда писать, snapshot станет непригоден для отката изменений. Или создавать snapshot размером с исходный том, или использовать функцию автоувеличения размера snapshota ( в конце пункта "Домашнее задание", раздел -  "Автоматическое увеличение размера снапшота LVM")



#                                                       2. Домашнне задание

Для уменьшения корневого раздела работающей машины,  можно загрузится с LiveCD и сделать в нём (создать временный раздел, скопировать туда весь корневой раздел, удалить корневой раздел на существующем LVM томе, создать новый lv раздел нужного размера, создать остальные необходимые разделы по ДЗ, вернуть данные обратно). Или создать новый временный раздел на рабочей машине, перенести туда данные, сконфигурировать загрузчик для запуска с нового раздела, загрузится с него, удалить старый раздел, создать новые разделы с нужными параметрами, вернуть данные, перегрузиться удалить временный раздел

1.      Создаём временный раздел с нужной файловой системой

            [root@lvm ~]# lsblk                                                                                                               │

            NAME                    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT                                                                       │
            sda                       8:0    0   40G  0 disk                                                                                  │
            ├─sda1                    8:1    0    1M  0 part                                                                                  │
            ├─sda2                    8:2    0    1G  0 part /boot                                                                            │
            └─sda3                    8:3    0   39G  0 part                                                                                  │
            ├─VolGroup00-LogVol00 253:0    0 37.5G  0 lvm  /                                                                                │
            └─VolGroup00-LogVol01 253:1    0  1.5G  0 lvm  [SWAP]                                                                           │
            sdb                       8:16   0   10G  0 disk                                                                                  │
            sdc                       8:32   0    2G  0 disk                                                                                  │
            sdd                       8:48   0    1G  0 disk                                                                                  │
            sde                       8:64   0    1G  0 disk

            [root@lvm ~]# pvcreate /dev/sdb                                                                                                   │
            Physical volume "/dev/sdb" successfully created.                                                                                │

            [root@lvm ~]# vgcreate vghw /dev/sdb                                                                                              │
            Volume group "vghw" successfully created                                                                                        │

            [root@lvm ~]# lvcreate -n lv_hw_root -l+100%FREE vghw                                                                             │
            Logical volume "lv_hw_root" created.                                                                                            │

            [root@lvm ~]# mkfs.xfs /dev/vghw/lv_hw_root                                                                                       │
            meta-data=/dev/vghw/lv_hw_root   isize=512    agcount=4, agsize=655104 blks                                                       │
            =                       sectsz=512   attr=2, projid32bit=1                                                               │
            =                       crc=1        finobt=0, sparse=0                                                                  │
            data     =                       bsize=4096   blocks=2620416, imaxpct=25                                                          │
            =                       sunit=0      swidth=0 blks                                                                       │
            naming   =version 2              bsize=4096   ascii-ci=0 ftype=1                                                                  │
            log      =internal log           bsize=4096   blocks=2560, version=2                                                              │
            =                       sectsz=512   sunit=0 blks, lazy-count=1                                                          │
            realtime =none                   extsz=4096   blocks=0, rtextents=0


2.      Перенос данных со старого раздела на новый


            [root@lvm ~]# mount /dev/vghw/lv_hw_root /mnt/

*      Перед копированием данных установим xfsdump - "yum install xfsdump"

            копирование данных:

            [root@lvm ~]# xfsdump -l0 -J - /dev/VolGroup00/LogVol00 | xfsrestory -l0 -J - /mnt/

3.     Изменение конфигурации grub, для загрузки с нового тома


            [root@lvm ~]# for i in /proc /sys /dev  /run /boot ; do mount --bind $i /mnt/$i; done <- биндим необходимые каталоги для имитации корневого раздела в папке где смонтирован временный том

            [root@lvm ~]# chroot /mnt <- делаем темповый раздел корневым


       Поверяем, что находимся в нужной точке:

            [root@lvm /]# mount                                                                                                               │
            /dev/mapper/vghw-lv_hw_root on / type xfs (rw,relatime,seclabel,attr2,inode64,noquota)     <-    Номер инода, отличный от 2, указывает на то, что видимый корень не является фактическим корнем файловой системы   (для виртуальных машин не работает)                             │
            proc on /proc type proc (rw,nosuid,nodev,noexec,relatime)                                                                         │
            sysfs on /sys type sysfs (rw,nosuid,nodev,noexec,relatime,seclabel)                                                               │
            devtmpfs on /dev type devtmpfs (rw,nosuid,seclabel,size=110948k,nr_inodes=27737,mode=755)                                         │
            tmpfs on /run type tmpfs (rw,nosuid,nodev,seclabel,mode=755)                                                                      │
            /dev/sda2 on /boot type xfs (rw,relatime,seclabel,attr2,inode64,noquota)                                                          │
       или

            [root@lvm /]# ls -di /                                                                         │
             64 /           <-             Номер инода, отличный от 2, указывает на то, что видимый корень не является фактическим корнем файловой системы (для виртуальных машин не работает)

4.      Обновление образа initrd:

            [root@lvm ~]# cd /boot

            [root@lvm boot]# for i in `ls initramfs-*img`; do dracut -v $i `echo $i|sed "s/initramfs-//g;s/.img//g"` --force; done

            ...........
            ** Created microcode section ***                                                                                             
            *** Creating image file done ***                                                                                                 
            *** Creating initramfs image file '/boot/initramfs-3.10.0-862.2.3.el7.x86_64.img' done ***


        Меняем значение корневого раздела в файле /boot/grub2/grub.cfg

        Генерируем новый grub, с новым корневым разделом. Для этого меняем значение в файле  /etc/default/grub:

-      rd.lvm.lv=VolGroup00/LogVol00  - на значение vghw/vghw_root (наш новый корневой раздел)


            [root@lvm /]# grub2-mkconfig -o /boot/grub2/grub.cfg  <------- генерируем новый конфиг                                                                              │
            Generating grub configuration file ...                                                                                            │
            Found linux image: /boot/vmlinuz-3.10.0-862.2.3.el7.x86_64                                                                        │
            Found initrd image: /boot/initramfs-3.10.0-862.2.3.el7.x86_64.img

       Отключаемся от виртуальной машины (ctrl+D) и перезагружаем её - vagrant reload


5.      Загрузка с новым корневым разделом, создание разделов с нужными параметрами


        смотрим, что загрузились с нужным корневым разделом:

                [root@lvm ~]# lsblk                                                                                                                                                        
                NAME                    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT                                             
                sda                       8:0    0   40G  0 disk                                                                                
                ├─sda1                    8:1    0    1M  0 part                                                                           
                ├─sda2                    8:2    0    1G  0 part /boot                                                                      
                └─sda3                    8:3    0   39G  0 part                                                                               
                ├─VolGroup00-LogVol00 253:0    0 37.5G  0 lvm                                                                                   │
                └─VolGroup00-LogVol01 253:2    0  1.5G  0 lvm  [SWAP]                                                                           │
                sdb                       8:16   0   10G  0 disk                                                                                  │
                └─vghw-lv_hw_root       253:1    0   10G  0 lvm  /                                                                                │
                sdc                       8:32   0    2G  0 disk                                                                                  │
                sdd                       8:48   0    1G  0 disk                                                                                  │
                sde                       8:64   0    1G  0 disk                                                                                  │

                [root@lvm ~]# lvremove /dev/VolGroup00/LogVol00                                                                                   │
                Do you really want to remove active logical volume VolGroup00/LogVol00? [y/n]: y                                                  │
                Logical volume "LogVol00" successfully removed                                                                                  │

                [root@lvm ~]# lvcreate -n VolGroup00/LogVol00 -L 8G /dev/VolGroup00          <---- Создали "/" раздел 8G размера                                                     │
                WARNING: xfs signature detected on /dev/VolGroup00/LogVol00 at offset 0. Wipe it? [y/n]: y                                        │
                Wiping xfs signature on /dev/VolGroup00/LogVol00.                                                                               │
                Logical volume "LogVol00" created.


        Далее создаём файловую систему, монтируем LogVol00 раздел в папку mnt и восстанавливаем данные как в предыдущем варианте:

                root@lvm ~]# mkfs.xfs /dev/VolGroup00/LogVol00

               meta-data=/dev/VolGroup00/LogVol00 isize=512    agcount=4, agsize=524288 blks
                =                       sectsz=512   attr=2, projid32bit=1
                =                       crc=1        finobt=0, sparse=0
                data     =                       bsize=4096   blocks=2097152, imaxpct=25
                =                       sunit=0      swidth=0 blks
                naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
                log      =internal log           bsize=4096   blocks=2560, version=2
                =                       sectsz=512   sunit=0 blks, lazy-count=1
                realtime =none                   extsz=4096   blocks=0, rtextents=0

                [root@lvm ~]# mount /dev/VolGroup00/LogVol00 /mnt

                [root@lvm ~]# xfsdump -l0 -J - /dev/vghw/lv_hw_root | xfsrestore -l0 -J  - /mnt


        Изменение конфигурации grub, для загрузки со старого тома:


                [root@lvm ~]# for i in /proc /sys /dev  /run /boot ; do mount --bind $i /mnt/$i; done <- биндим необходимые каталоги для имитации корневого раздела в папке где смонтирован временный том

                [root@lvm ~]# chroot /mnt <- делаем темповый раздел корневым


                [root@lvm /]# cd /boot          <----- переходим в boot каталог                       

                [root@lvm boot]# for i in `ls initramfs-*img`; do dracut -v $i `echo $i|sed "s/initramfs-//g;s/.img//g"` --force; done            <----- генерируем новый initramfs

        Меняем значение корневого раздела в файле /boot/grub2/grub.cfg

        Генерируем новый grub, с новым корневым разделом. Для этого меняем значение в файле  /etc/default/grub:

-      rd.lvm.lv= vghw/vghw_root  - на значение  rd.lvm.lv=VolGroup00/LogVol00 (наш старый  корневой раздел, размером 8Gb )


            [root@lvm /]# grub2-mkconfig -o /boot/grub2/grub.cfg  <------- генерируем новый конфиг                                                                              │
            Generating grub configuration file ...                                                                                            │
            Found linux image: /boot/vmlinuz-3.10.0-862.2.3.el7.x86_64                                                                        │
            Found initrd image: /boot/initramfs-3.10.0-862.2.3.el7.x86_64.img



        Создание зеркала на свободных дисках для отдельного раздела var:

                [root@lvm boot]# pvcreate /dev/sdd /dev/sdc   <-------- готовим диски для LVM

                Physical volume "/dev/sdd" successfully created.
                Physical volume "/dev/sdc" successfully created.

                [root@lvm boot]# vgcreate vghw_var /dev/sdd /dev/sdc    <----------- создаём новую volume group с именем vghw_var

                Volume group "vghw_var" successfully created


                [root@lvm boot]# lvcreate -L 900M -m1 -n lv_hw_var vghw_var          <----------- создали зеркальный том, размером 900M потому что один диск 2G а второй 1G.

                Logical volume "lv_hw_var" created.

                [root@lvm boot]# mkfs.ext4 /dev/vghw_var/lv_hw_var   <-------------- создали файловую систему

                mke2fs 1.42.9 (28-Dec-2013)

                mount  /dev/vghw_var/lv_hw_var /mnt            <----------- монтируем новый раздел и копируем на него данные со старой папки /var


                [root@lvm boot]# cp -aR /var/* /mnt/   <------ копируем содержимое /var каталог в новый раздел                                                                                                                                                                                                        

                [root@lvm boot]# rsync -avHPSAX /var /mnt   


        Удаляем содержимое старой папки /var и монтируем новый том в папку:

                [root@lvm boot]# rm -rf /var/*

                [root@lvm boot]# mount /dev/vghw_var/lv_hw_var /var  


        Изменяем файл /etc/fstab, для автоматического монтирования /var при запуске сисетмы

                [root@lvm boot]# blkid        

                /dev/mapper/vghw_var-lv_hw_var_rimage_0: UUID="dd8f0ac1-2591-4dc4-b49c-aa55bea29ae5" TYPE="ext4"
                /dev/mapper/vghw_var-lv_hw_var_rimage_1: UUID="dd8f0ac1-2591-4dc4-b49c-aa55bea29ae5" TYPE="ext4"
                /dev/mapper/vghw_var-lv_hw_var: UUID="dd8f0ac1-2591-4dc4-b49c-aa55bea29ae5" TYPE="ext4"                       <------------ получаем UUID и вносим в файл fstab

                UUID=dd8f0ac1-2591-4dc4-b49c-aa55bea29ae5       /var    ext4    defaults        0 0        <----- монтирование по UUID можно по устройству "/dev/vghw_var/lv_hw_var"

         После перезагрузки очищаем ненужный том, группу, и выводим диск из LVM  (lv_hw_root, vghw, /dev/sdb )


                [root@lvm ~]# lvremove /dev/vghw/lv_hw_root
                Do you really want to remove active logical volume vghw/lv_hw_root? [y/n]: y
                Logical volume "lv_hw_root" successfully removed

                root@lvm ~]# vgremove vghw
                Volume group "vghw" successfully removed

                [root@lvm ~]# pvremove /dev/sdb
                Labels on physical volume "/dev/sdb" successfully wiped.


6.      Home разде со снапшотами

                [root@lvm ~]# lvcreate -n LogVol_Home -L 2G VolGroup00  <--------------- создали новый раздел в LVM группе  VolGroup00
                Logical volume "LogVol_Home" created.

            [root@lvm ~]# mkfs.xfs /dev/VolGroup00/LogVol_Home  < ------------- Создаём файловую систему на разделе

            meta-data=/dev/VolGroup00/LogVol_Home isize=512    agcount=4, agsize=131072 blks
            =                       sectsz=512   attr=2, projid32bit=1
            =                       crc=1        finobt=0, sparse=0
            data     =                       bsize=4096   blocks=524288, imaxpct=25
            =                       sunit=0      swidth=0 blks
            naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
            log      =internal log           bsize=4096   blocks=2560, version=2
            =                       sectsz=512   sunit=0 blks, lazy-count=1
            realtime =none                   extsz=4096   blocks=0, rtextents=0

        Монтируем новый раздел в промежуточную папку, копируем со старой папки Home в эту промежуточную папку, удаляем содержимое старой, отмонтируем новый раздел, примонтировать по новой и прописать в fstab по UUID

            [root@lvm ~]# mount /dev/VolGroup00/LogVol_Home /mnt/
            [root@lvm ~]# cp -aR /home/* /mnt/
            [root@lvm ~]# rm -rf /home/*
            [root@lvm ~]# umount /mnt
            [root@lvm ~]# mount /dev/VolGroup00/LogVol_Home /home/
            [root@lvm ~]# blkid
            .......................
            /dev/mapper/VolGroup00-LogVol_Home: UUID="bb3bddf8-546b-4a80-a5c0-a10d7a570b4f" TYPE="xfs"

        Проверка работы снапшотов:

            [root@lvm ~]# touch /home/test{1..10}  < -------------- сгенерировали 10 тестовых файлов
            [root@lvm ~]# lvcreate -L 100M -s -n home_snap /dev/VolGroup00/LogVol_Home < ------------- создали снапшот-раздел раздела LogVol_Home
            Rounding up size to full physical extent 128.00 MiB
            Logical volume "home_snap" created.


            [root@lvm ~]# rm -f /home/test{5..9}

            [root@lvm ~]# ls /home/
            test1  test10  test2  test3  test4


            root@lvm ~]# lvconvert --merge /dev/VolGroup00/
            home_snap    LogVol00     LogVol01     LogVol_Home  

            [root@lvm ~]# lvconvert --merge /dev/VolGroup00/home_snap
            Merging of volume VolGroup00/home_snap started.
            VolGroup00/LogVol_Home: Merged: 100.00%

            [root@lvm ~]# ls /home/

            [root@lvm ~]# mount /home

            [root@lvm ~]# ls /home/
            test1  test10  test2  test3  test4  test5  test6  test7  test8  test9

###         Автоматическое увеличение размера снапшота LVM

- Нужно изменить файлик /etc/lvm/lvm.conf:


      snapshot_autoextend_threshold = 80
      snapshot_autoextend_percent = 20
      monitoring = 1

**Эти опции включают мониторинг, а так же увеличивают размер снапшота на 20% (от текущего размера) каждый раз, когда он оказывается занят изменениями на 80%.**

** Установить dmeventd, который может не стоять, но который нужен чтобы все работало!**
