# Инструкции
● Для проверки надо скачать репозиторий 

	cd ~
    git clone git@github.com:Timurka4080/zfs.git
	
● После этого зайдите в директорию **zfs** и запускаем виртуальные машины с помощью vagrant

	cd ~/zfs/
	vagrant up 

## 1. Определить алгоритм с наилучшим сжатием
● Для начала настросим создадим pool а потом по смонтируем фыйловую систему. 

    zpool create poolm sbd sbc
    zfs create poolm/src
    zfs create poolm/src/gzip6
    zfs create poolm/src/gzip9
    zfs create poolm/src/lz4
    zfs create poolm/src/lzjb

● Теперь для каждой ФС настроим свое сжатие 

    zfs set compress=lz4 poolm/src/lz4
    zfs set compress=gzip-9 poolm/src/gzip9
    zfs set compress=gzip-6 poolm/src/gzip6
    zfs set compress=lzjb poolm/src/lzjb

● Теперь в директорию poolm/src скачаем модуль ядра

    wget https://kernel.googlesource.com/pub/scm/boot/dracut/dracut.git/+archive/refs/heads/master.tar.gz

● Переносим фал в наши ФС и разархоивируем его. смотрим что получилось и делаем выводы окомпрессии 

    zfs get compression,compressratio | grep poolm
    poolm            compression    off       default
    poolm            compressratio  1.78x     -
    poolm/src        compression    off       default
    poolm/src        compressratio  1.85x     -
    poolm/src/gzip6  compression    on        local
    poolm/src/gzip6  compressratio  1.88x     -
    poolm/src/gzip9  compression    gzip-9    local
    poolm/src/gzip9  compressratio  2.29x     -
    poolm/src/lz4    compression    lz4       local
    poolm/src/lz4    compressratio  1.89x     -
    poolm/src/lzjb   compression    lzjb      local
    poolm/src/lzjb   compressratio  1.76x     -

● И еще можем посмотреть реальый размер файлов команодой 

    [root@server src]# zfs list
    NAME              USED  AVAIL     REFER  MOUNTPOINT
    data              246K  2.68G     35.9K  /home
    data/doc         35.9K  2.68G     35.9K  /home/doc
    data/media       32.9K  2.68G     32.9K  /home/media
    poolm            8.42M  1.74G     25.5K  /poolm
    poolm/src        6.93M  1.74G      674K  /poolm/src
    poolm/src/gzip6  1.61M  1.74G     1.61M  /poolm/src/gzip6
    poolm/src/gzip9  1.35M  1.74G     1.35M  /poolm/src/gzip9
    poolm/src/lz4    1.60M  1.74G     1.60M  /poolm/src/lz4
    poolm/src/lzjb   1.71M  1.74G     1.71M  /poolm/src/lzj

## 2.    
