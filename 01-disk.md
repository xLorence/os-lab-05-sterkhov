### Вывод lsblk после разметки
На виртуальном диске `/dev/loop0` создана таблица разделов GPT, диск поделен на два раздела по 511 МБ: `loop0p1` и `loop0p2`.

```
NAME      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
loop0       7:0    0    1G  0 loop
├─loop0p1 259:0    0  511M  0 part
└─loop0p2 259:1    0  511M  0 part
```

### Вывод blkid (UUID и типы)
Разделам присвоены уникальные идентификаторы, метки и разные типы файловых систем.   
- `/dev/loop0p1`: LABEL="DATA1" UUID="145e2133-7c6c-444e-9f87-5fc6a906f1fd" BLOCK_SIZE="1024" TYPE="ext4" PARTLABEL="data1" PARTUUID="1c5f48b5-7249-478f-86ff-2a630798b0c3"
- `/dev/loop0p2`: LABEL="DATA2" UUID="a540104b-72b0-47e6-a517-f1c3ba987b61" SEC_TYPE="ext2" BLOCK_SIZE="1024" TYPE="ext3" PARTLABEL="data2" PARTUUID="a3606d69-3bfa-4ca9-8533-b43e5a9b577e"

### Свободное место (`df -hT`) и занятые проценты

```
Filesystem     Type  Size  Used Avail Use% Mounted on
/dev/loop0p1   ext4  469M   14K  440M   1% /mnt/data1
/dev/loop0p2   ext3  469M   47K  444M   1% /mnt/data2
```
Разделы сразу после создания показывают, что 1% места уже занят, хотя файлы на них не записывались. Это происходит потому, что при форматировании сразу резервируется место под служебные структуры файловой системы: таблицу inode (где хранится информация о файлах), системный журнал и скрытую папку `lost+found`.

### Что пошло не так
При попытке отформатировать второй раздел в xfs система выдала ошибку mkfs.xfs: `command not found`. Установка пакета через `sudo apt install xfsprogs` не удалась из-за недоступности сетевых репозиториев (`Package 'xfsprogs' has no installation candidate`). Чтобы выполнить работу, второй раздел был отформатирован во встроенную файловую систему ext3 командой `sudo mkfs.ext3 -L DATA2 /dev/loop0p2`. 
