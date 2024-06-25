---
title: "Описание файловой системы OverlayFS"
date: 2020-02-01
weight: 10
description: >
  Мой неполный перевод статьи [Overlay Filesystem](https://www.kernel.org/doc/Documentation/filesystems/overlayfs.txt) из https://www.kernel.org
tags:
  - VFS
  - FileSystem
  - translation
aliases:
  - /a/opisanie-failovoi-sistemy-overlayfs
---
2020-02-01

> Не закончен, так как не хватает знаний. По мере изучения, постараюсь дополнить.

> Данная файловая системы используется, например, в Docker. Часто её можно наблюдать на роутерах или других подобных устройствах, где оперативная запись информации ведётся во временную наложенную ФС, доступную до момента выключения устройства.

> С помощью OverlayFS я объединяю домашние фото и видео каталоги для удобного просмотра. Так как архивы исходных фотографий доступны только для чтения, наложением рабочей папки, для сохранения временных или промежуточных файлов обработки, я добиваюсь удобства в обработке фоток.

> Каталог = Директория = Папка

> Перед пониманием темы Overlay Filesystem, мне необходимо провентилировать [Linux Virtual File System](/a/opisanie-vfs-virtualnoi-failovoi-sistemy-linux).

## Overlay Filesystem
Этот документ описывает прототип нового подхода к обеспечению функциональности **overlay filesystem** (*наложенная файловая система*) в Линукс (некоторые предпочитают называть **union-filesystems** *объединённая файловая система*). **Overlay filesystem** пытается показать файловую систему так, как результат наложения одной ФС поверх другой ФС.

### Overlay объекты
Метод наложения файловой системы является смешанным, так как объекты, которые видны в этой ФС не всегда ей принадлежат. Во многих случаях, доступ к объекту в объединении будет неразличимым от доступа к соответствующему объекту из обычной файловой системы. Это наиболее очевидно из поля 'st_dev' возвращаемого системным вызовом `stat` (`man 2 stat`).
> Почитав stat(2), я выяснил, что поле 'st_dev' описывает устройство на котором находится файл. 'st_ino' — номер inode.

В то время как директории будут сообщать 'st_dev' из *наложенной* файловой системы, объекты, не являющиеся каталогами, могут сообщать 'st_dev' из *нижней* или *верхней* файловой системы, которая предоставляет объект. Аналогично 'st_ino' будет уникальным только в сочетании с 'st_dev', и оба они могут изменяться в течение всего времени жизни объекта не-каталога. Многие приложения и инструменты игнорируют эти значения.

В особом случае, когда все наложенные слои принадлежат одной подлежащей файловой системе, то все объекты будут сообщать 'st_dev' из наложенной файловой системы, а 'st_ino' из подлежащей файловой системы. Это делает монтирование наложения более совместимым со сканерами файловой системы, и оверлей-объекты будут отличаться от соответствующих объектов в оригинальной файловой системе.

On 64bit systems, even if all overlay layers are not on the same underlying filesystem, the same compliant behavior could be achieved with the "xino" feature. The "xino" feature composes a unique object identifier from the real object st_ino and an underlying fsid index. If all underlying filesystems support NFS file handles and export file handles with 32bit inode number encoding (e.g. ext4), overlay filesystem will use the high inode number bits for fsid.  Even when the underlying filesystem uses 64bit inode numbers, users can still enable the "xino" feature with the "-o xino=on" overlay mount option.  That is useful for the case of underlying filesystems like xfs and tmpfs, which use 64bit inode numbers, but are very unlikely to use the high inode number bit.

### Верхний и нижний слои
Наложенная файловая система комбинируется из двух ФС - *верхней* и *нижней*. Когда имя объекта существует в обеих ФС, тогда объект из *верхней* ФС виден, а объект из *нижней* ФС скрыт. В случае с совпадением имён каталогов, их содержимое будет объединено.

Было бы правильней употреблять термин верхнее и нижнее *дерево каталогов* вместо термина *файловая система*, поскольку вполне возможно, что оба *дерева каталогов* могут находиться в одной и той же файловой системе, и здесь не требуется указывать корень файловой системы для верхней или нижней файловой системы.

Нижняя файловая система может быть любой файловой системой, поддерживаемой Linux, и для неё не требуется прав на запись. Нижняя файловая система может быть даже другим оверлеем. Верхняя файловая система обычно доступна для записи, и в таком случае она должна поддерживать создание расширенных атрибутов "trusted.*", и также должна предоставлять допустимый 'd_type' в 'readdir' ответах, так что NFS не подходит.

> readdir&nbsp;&mdash; чтение директории
описание поля "d_type"...
<details><p>
"d_type" Это поле содержит значение, обозначающее тип файла, что позволяет избежать затрат на вызов lstat(2), если дальнейшие действия зависят от типа файла.

When a suitable feature test macro is defined (_DEFAULT_SOURCE on glibc versions since 2.19, or _BSD_SOURCE on glibc versions 2.19 and earlier), glibc defines the following macro constants for the value returned in d_type:
DT_BLK      This is a block device.
DT_CHR      This is a character device.
DT_DIR      This is a directory.
DT_FIFO     This is a named pipe (FIFO).
DT_LNK      This is a symbolic link.
DT_REG      This is a regular file.
DT_SOCK     This is a UNIX domain socket.
DT_UNKNOWN  The file type could not be determined.

На данный момент только некоторые файловые системы (среди них, такие как: Btrfs, ext2, ext3, and ext4) умеют возвращать правильное значение типа файла в поде "d_type". Все приложения обязаны должным образом обрабатывать возвращаемое значение DT_UNKNOWN.
</p></details><br>

Только в варианте, когда наложенная ФС имеет атрибут "только для чтения" и составлена из двух "только для чтения" файловых систем, могут использоваться любые типы ФС.

### Каталоги
В оверлее, в основном, участвуют каталоги. Если *имя* объекта ссылается не на каталог, и объект проявляется как в верхней, так и в нижней файловых системах, то нижний объект скрыт и *имя* относится только к верхнему объекту.

Если верхний и нижний объекты являются каталогами, то формируется объединенный каталог.

В момент монтирования две директории, указанные в опциях как "upperdir" и "lowerdir", соединяются в одну объединённую директорию:

```
# mount -t overlay overlay -olowerdir=/lower,upperdir=/upper,workdir=/work /merged
```

Опция "workdir" должна указывать на пустой каталог в той же файловой системе, где находится "upperdir".

> *dentry* — объект VFS, содержащий информацию о директориях ФС и существующий только в памяти файловой системы и не хранится на диске. Если я правильно понял, то предназначен для уменьшения обращений к ФС при перечитывании содержимого каталогов.

Тогда всякий раз, когда поиск запрашивается в такой объединенной директории, просмотр выполняется в каждой фактической директории и объединенный результат кэшируется в *dentry*, принадлежащем наложенной файловой системе. Если оба актуальных поиска находят каталоги, оба сохраняются и создается объединенный каталог, в противном случае сохраняется только один: верхний, если он существует, иначе нижний.

Объединяются только списки имён из директорий. Другое содержимое, такое как метаданные и расширенные атрибуты, отображается только для директорий из верхней ФС. Эти атрибуты для директории из нижней ФС скрыты.

### "Выбеленные" и "непрозрачные" каталоги
> Не уверен, что подобрал правильный перевод для "whiteouts and opaque directories".

Чтобы выполнять `rm` и `rmdir` без изменения нижней файловой системы, наложенная файловая система должна записать в верхнюю файловую систему информацию о том, что файлы были удалены. Это делается с помощью "выбеленных" и "непрозрачных" каталогов (объекты не-каталоги всегда непрозрачны).

"Выбеленная" директория создаётся как символьное устройство с номером устройства 0/0. Когда в верхнем слое объединенного каталога обнаруживается "выбеленная" директория, любое совпадающее имя на нижнем уровне игнорируется, и сама "выбеленная" директория также скрывается.

Каталог объявляется "непрозрачным" установкой расширенного атрибута "trust.overlay.opaque" в значение "y". Если верхний слой содержит "непрозрачный" каталог с именем совпадающим с именем каталога из нижнего слоя, тогда соответствующий каталог в нижнем слое игнорируется.

### Системный запрос readdir (Чтение каталога)
Когда `readdir` запрашивается в объединенном каталоге, то каждый верхний и нижний каталог читается, а списки имён объединяются очевидным образом (сначала читается верхний, затем нижний, уже существующие имена не добавляются повторно). Этот объединенный список имён кэшируется в 'struct file' и таким остаётся так долго, пока файл остаётся открытым. Если директория открыта и одновременно читается двумя процессами, то каждый из них будет иметь отдельный кэш. A `seekdir` to the start of the directory (offset 0) followed by a `readdir` will cause the cache to be discarded and rebuilt.

Это означает, что изменения в объединённой директории не проявятся пока каталог читается. Вряд ли это будет замечено каким-либо программами.

seek offsets are assigned sequentially when the directories are read.
Thus if

  - read part of a directory
  - remember an offset, and close the directory
  - re-open the directory some time later
  - seek to the remembered offset

there may be little correlation between the old and new locations in the list of filenames, particularly if anything has changed in the directory.

Readdir on directories that are not merged is simply handled by the underlying directory (upper or lower).

### Переименование каталога
Когда переименовывается каталог, который является нижним или объединённым (то есть этот каталог не был создан на верхнем слое до начала операции), overlayfs может обработать его двумя различными способами:

1. возврат EXDEV ошибки: эта ошибка возвращается вызовом `rename`, когда при попытке перемещения файла или директории нарушаются границы файловой системы. Приложение обычно готово обработать эту ошибку (`mv`, для примера, рекурсивно копирует дерево директорий). Это поведение по умолчанию.
2. Если свойство "redirect_dir" включено, тогда директория будет скопирована (без своего содержимого). Потом будет *установлен* расширенный атрибут "trusted.overlay.redirect" на *путь* оригинального местоположения от корня *наложения*. В заключении директория перемещается в новое место.

Есть несколько способов настроить "redirect_dir" свойство.

Kernel config options:

- OVERLAY_FS_REDIRECT_DIR:
    - If this is enabled, then redirect_dir is turned on by  default.
- OVERLAY_FS_REDIRECT_ALWAYS_FOLLOW:
    - If this is enabled, then redirects are always followed by default. Enabling this results in a less secure configuration.  Enable this option only when worried about backward compatibility with kernels that have the redirect_dir feature and follow redirects even if turned off.

Module options (can also be changed through `/sys/module/overlay/parameters/*`) :

- "redirect_dir=BOOL":
    - See OVERLAY_FS_REDIRECT_DIR kernel config option above.
- "redirect_always_follow=BOOL":
    - See OVERLAY_FS_REDIRECT_ALWAYS_FOLLOW kernel config option above.
- "redirect_max=NUM":
    - The maximum number of bytes in an absolute redirect (default is 256)



Mount options:

- "redirect_dir=on":
    Redirects are enabled.
- "redirect_dir=follow":
    Redirects are not created, but followed.
- "redirect_dir=off":
    Redirects are not created and only followed if "redirect_always_follow"
    feature is enabled in the kernel/module config.
- "redirect_dir=nofollow":
    Redirects are not created and not followed (equivalent to "redirect_dir=off"
    if "redirect_always_follow" feature is not enabled).

When the NFS export feature is enabled, every copied up directory is indexed by the file handle of the lower inode and a file handle of the upper directory is stored in a "trusted.overlay.upper" extended  attribute on the index entry.  On lookup of a merged directory, if the upper directory does not match the file handle stores in the index, that is an indication that multiple upper directories may be redirected to the same lower directory.  In that case, lookup returns an error and warns about a possible inconsistency.

Because lower layer redirects cannot be verified with the index, enabling NFS export support on an overlay filesystem with no upper layer requires turning off redirect follow (e.g. "redirect_dir=nofollow").

### Прочие объекты не-Каталоги
Объекты, не являющиеся каталогами (файлы, симлинки, device-special files, и т.д.), представлены в *объединённой* ФС из *верхней*, либо из *нижней* файловой системы, по обстоятельствам. Когда требуется доступ на запись к файлу, представленному в нижней файловой системе, то сначала файл копируется из нижней ФС в верхнюю (copy_up). Обратите внимание, что создание жёсткой ссылки также требует copy_up, тогда как создание симлинка такой операции конечно не требует.

Если файл, открытый на запись, не был изменён, то операция copy_up выполнена не будет.

Процесс copy_up начинается с проверки наличия в *верхней* ФС соответствующих каталогов, которые создаются при необходимости. После чего в существующий или созданный каталог копируется объект с такими же метаданными (owner, mode, mtime, symlink-target etc.). Далее, в случае если объект это файл, то копируется содержимое нижнего файла в только что созданный файл в верхней ФС. В заключении копируются расширенные атрибуты.

После завершения процесса copy_up, overlay ФС предоставит прямой доступ к только что созданному файлу из *верхней* файловой системы. Последующие операции над файлом будут едва заметны для overlay ФС (тогда как операции над именем файла, вроде переименования или unlink, конечно будут замечены и обработаны).

### Множество нижних слоёв
Множество нижних слоёв можно задать с помощью символа двоеточие (":"), используя его как разделитель между именами каталогово. Для примера:
```
mount -t overlay overlay -olowerdir=/lower1:/lower2:/lower3 /merged
```

Как видно в примере, опции "upperdir=" и "workdir=" могут не указываться. И в этом случае наложение будет доступно только для чтения.

Указанные нижние каталоги будут стекированы справа налево, то есть lower1 будет на самом верху, lower2 посередине, а lower3 на нижнем уровне.

![](/img/opisanie-failovoi-sistemy-overlayfs/ro-ro-overlay.png)

> *В пустом каталоге `RawArchive`, после монтирования доступном только для чтения, мы увидим объединённое отображение двух папок c исходниками фото- и видеоматериалов. Все каталоги, задействованные в этой схеме, могут находиться, как на одном носителе, так и на разных (даже сетевых?).*

### Копирование только метаданных
Когда включена опция копирования только метаданных, overlayfs скопирует только метаданные (в отличие от всего файла), в том случае когда выполняется определённая операция, типа `chown/chmod`. Полностью файл будет скопирован после того, как файл будет открыт для **записи**.

Другими словами, это операция с задержкой копирования данных. Данные будут скопированы в том случае, если понадобится их изменение.

Есть несколько способов включения/отключния этой опции. Опция CONFIG_OVERLAY_FS_METACOPY может быть установлена/снята для включения/отключения этой функции по умолчанию. Или можно включить/отключить эту опцию во время загрузки модуля с параметром metacopy=on/off. И наконец, использовать опцию metacopy=on/off во время монтирования.

Не стоит использовать *metacopy=on* с ненадёжными верхним/нижним каталогом. Otherwise it is possible that an attacker can create a handcrafted file with appropriate REDIRECT and METACOPY xattrs, and gain access to file on lower pointed by REDIRECT. This should not be possible on local system as setting "trusted." xattrs will require CAP_SYS_ADMIN. But it should be possible for untrusted layers like from a pen drive.

Note: redirect_dir={off|nofollow|follow(*)} conflicts with metacopy=on, and results in an error.

(*) redirect_dir=follow only conflicts with metacopy=on if upperdir=... is given.

### Расшаривание и копирование слоёв
Нижние слои могут совместно использоваться при монтировании нескольких разных overlay-стеков, и это обычная практика. При монтировании какой-нибудь Overlay может использовать тот же путь до каталога нижнего слоя, что и другой overlay mount.
