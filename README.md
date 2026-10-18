#Практические навыки работы с ZFS

#Задание

# 1.Определить алгоритм с наилучшим сжатием:

# 1.1 определить, какие алгоритмы сжатия поддерживает zfs (gzip, zle, lzjb, lz4);

# 1.2 создать 4 файловых системы, на каждой применить свой алгоритм сжатия;

# 1.3 для сжатия использовать либо текстовый файл, либо группу файлов.

# 2. Определить настройки пула.

# С помощью команды zfs import собрать pool ZFS.

# Командами zfs определить настройки:

# 2.1 размер хранилища;

# 2.2 тип pool;

# 2.3 значение recordsize;

# 2.4 какое сжатие используется;

# 2.5 какая контрольная сумма используется.

# 3. Работа со снапшотами:

# 3.1 скопировать файл из удаленной директории;

# 3.2 восстановить файл локально. zfs receive;

# 3.3 найти зашифрованное сообщение в файле secret_message.



#Первоначальная установка

#установка пакеты утилит zfs

apt install zfsutils-linux

#Задание 1:Определение алгоритма с наилучшим сжатием

#создаем 4 пула по 2 диска в режиме RAID1

zpool create zfs1 mirror /dev/sdb /dev/sdc

zpool create zfs2 mirror /dev/sdd /dev/sde

zpool create zfs3 mirror /dev/sdf /dev/sdg

zpool create zfs4 mirror /dev/sdh /dev/sdi

#добавляем разные алгориты сжатия в кажду из раннее созданных ФС

zfs set compression=lzjb zfs1

zfs set compression=lz4 zfs2

zfs set compression=gzip-9 zfs3

zfs set compression=zle zfs4

#скачаем один и тот же файл во все раннее созданные пулы

for i in {1..4}; do wget -P /zfs$i https://gutenberg.org/cache/epub/2600/pg2600.converter.log; done

#проверим сколько места и какая степень сжатия у каждого из пулов

zfs list

zfs get all | grep compressratio | grep -v ref

#Результат:

#NAME   USED  AVAIL  REFER  MOUNTPOINT

#zfs1  19.1M   813M  19.0M  /zfs1

#zfs2  15.7M   816M  15.6M  /zfs2

#zfs3  9.63M   822M  9.51M  /zfs3

#zfs4  30.6M   801M  30.4M  /zfs4

#zfs1  compressratio         1.60x                  -

#zfs2  compressratio         1.96x                  -

#zfs3  compressratio         3.20x                  -

#zfs4  compressratio         1.00x                  -

#Вывод:самый эффекивный метод сжатия gzip-9,который был применен к 3 пулу

#Задание 2: Определение настроек пула

#скачать и разархивировать архив в домашний каталог

wget -O archive.tar.gz --no-check-certificate 'https://drive.usercontent.google.com/download?id=1MvrcEp-WgAQe57aDEzxSRalPAwbNN1Bb&export=download'
tar -xzvf archive.tar.gz


#делаем импорт данного пула к нам и проверям его статус
zpool import -d zpoolexport/ otus

zpool status

#размер хранилища

zfs get available otus

#NAME  PROPERTY   VALUE  SOURCE

#otus  available  350M   -

#тип pool

zfs get readonly otus

#NAME  PROPERTY  VALUE   SOURCE

#otus  readonly  off     default

#значение recordsize

zfs get recordsize otus

#NAME  PROPERTY    VALUE    SOURCE

#otus  recordsize  128K     local

#какое сжатие используется

zfs get compression otus

#NAME  PROPERTY     VALUE           SOURCE

#otus  compression  zle             local

#какая контрольная сумма используется

zfs get checksum otus

#NAME  PROPERTY  VALUE      SOURCE

#otus  checksum  sha256     local

#Задание 3: Работа со снапшотом

#скачиваем файл из задания

wget -O otus_task2.file --no-check-certificate https://drive.usercontent.google.com/download?id=1wgxjih8YZ-cqLqaZVa0lA3h3Y029c3oI&export=download

#восстанавливаем ФС из снапшота

zfs receive otus/test@today < otus_task2.file

#поиск файлов с сообщением

find /otus/test -name "secret_message"

#смотрим содержимое файла

cat /otus/test/task1/file_mess/secret_message

#Итог: https://otus.ru/lessons/linux-hl/
