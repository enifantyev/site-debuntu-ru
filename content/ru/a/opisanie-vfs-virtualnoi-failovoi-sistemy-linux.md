---
title: "Описание VFS Виртуальной Файловой Системы Linux"
date: 2020-01-18
weight: 10
description: >
  Мой неполный перевод [Overview of the Linux Virtual File System](https://www.kernel.org/doc/html/latest/filesystems/vfs.html).
tags:
  - VFS
  - FileSystem
  - translation
---

Мой неполный перевод [Overview of the Linux Virtual File System](https://www.kernel.org/doc/html/latest/filesystems/vfs.html). Некоторые моменты, по различным причинам, я пока не понимаю, поэтому кое-что привожу в оригинале, как повод для дальнейших размышлений. Другие же моменты, как мне кажется, я понимаю, поэтому дополнил их некоторой информацией.

2020-01-18

## Введение
Виртуальная Файловая Система (называемая также Коммутатором Виртуальной Файловой Системы) является программным слоем, который предоставляет программам, выполняющимся в *пользовательском пространстве*, интерфейс взаимодействия с файловой системой. VFS-коммутатор также реализует в ядре Linux уровень абстракции для унификации подключения к нему разнообразных файловых систем.

Системные вызовы VFS `open(2)`, `stat(2)`, `read(2)`, `write(2)`, `chmod(2)` и так далее, вызываются из контекста работающего процесса. *Filesystem locking* описана в `Documentation/filesystems/locking.rst`. (*VFS system calls open(2), stat(2), read(2), write(2), chmod(2) and so on are called from a process context. Filesystem locking is described in the document Documentation/filesystems/locking.rst.*)

### Кэш записей каталога / Directory Entry Cache (dcache)
VFS обеспечивает системные вызовы `open(2)`, `stat(2)`, `chmod(2)` и другие похожие. Аргумент `pathname`, который передаётся на вход этим системным вызовам, используется VFS-системой для поиска вхождений в *кэше записей каталога* (называемым также *dentry cache*, или ещё короче *dcache*). Это обеспечивает сверхбыстрый механизм трансляции `pathname` (filename) в определённый `dentry`. *Кэш записей каталога* необходим только для увеличения производительности, поэтому он хранится в оперативной памяти и никогда не записывается на диск.

*Dentry-кэш* предназначен для обзора всего файлового пространства. А так как большинство компьютеров не могут вместить все записи dentry в оперативную память, то некоторое количество записей в *dentry-кэше* может отсутствовать. Поэтому, иногда в процессе сопоставления частей *pathname* с соответствующими dentry, VFS-коммутатору, возможно, придётся заодно создать некоторое количество dentry, и только после этого загрузить требуемый inode.

### Объект Inode / The Inode Object
Отдельный dentry обычно указывает на **inode**, который является специальной структурой файловой системы, описывающей её файлы-объекты. Объект inode предоставляет различные сведения о файле (`man 2 stat`). В частности:

- `st_dev`&nbsp;&ndash; *id* устройства, содержащего файл;
- `st_ino`&nbsp;&ndash; номер этой inode в таблице inodes;
- `st_mode`&nbsp;&ndash; тип объекта (`man 7 inode`):
    - обычный файл с данными;
    - directory;
    - block device;
    - ...
- `st_nlink`&nbsp;&ndash; количество жёстких ссылок (имён файлов) на этот *inode*.
> Объект `inode` не содержит имени файла! Так как в пределах одной файловой системы (POSIX) на один файл может ссылаться больше одной именованной *жёсткой ссылки*, то имя файла не сохраняется в `inode`. Вместо этого, в файлах-каталогах (тип directory), в списке содержимого данного каталога, создаётся запись: **имя файла** (то есть имя жёсткой ссылки)&nbsp;&mdash; **номер inode**. Отсюда следует, что при обнулении поля `st_nlink` в структуре какого-либо `inode`, соответствующие блоки файловой системы будут помечены свободными и файл можно считать удалённым.

*Inodes* могут находиться на дисках (на файловых системах блочных устройств), либо в оперативной памяти (на псевдо-файловых системах). *Inodes*, находящиеся на дисках, при необходимости копируются в оперативную память, а при каких-либо в них изменениях, обновлённая копия записывается обратно на диск. Несколько *dentry* могут указывать на один *inode* (в случае жёстких ссылок и тому подобное).

Для поиска какого-либо *inode* необходимо, чтобы для его родительского inode-каталога, VFS вызвала метод `lookup()`. Этот метод установлен конкретной реализацией ФС, к которой относится требуемый *inode*. (*This method is installed by the specific filesystem implementation that the inode lives in.*) С момента получения VFS-коммутатором *dentry* (а следовательно и соответствующего *inode*), мы можем делать с ним всякие скучные вещи, типа открыть файл посредством системного вызова `open(2)`, или получить данные inode через `stat(2)`. Из пользовательского пространства системный вызов `stat(2)` работает довольно просто: как только VFS заимел соответствующий *dentry*, он передаёт данные из *inode* обратно в *userspace* (*The stat(2) operation is fairly simple: once the VFS has the dentry, it peeks at the inode data and passes some of it back to userspace.*).

### Объект File / The File Object
Открытие какого-либо файла требует ещё одной операции: выделение *файловой структуры* (это реализация *файловых дескрипторов* на стороне ядра). Свеже выделенная *файловая структура* инициализируется с указателем на соответствующий dentry и на набор файловых операций функций-членов. (*The freshly allocated file structure is initialized with a pointer to the dentry and a set of file operation member functions.*) Они берутся из сведений, содержащихся в соответствующей inode. Применяемый к файлу метод `open()` вызывается тогда, как только конкретная реализация файловой системы сможет сделать это. Вы можете видеть, что это ещё одна функция выполняемая VFS-коммутатором. *Файловая структура* помещается в *таблицу дескрипторов файлов* для вызывающего процесса.

Чтение, запись и закрытие файла (и другие подобные операции VFS) осуществляются с помощью соответствующих методов, перечисленных в подходящей к требуемому файлу *файловой структуре*, каковая структура становится доступной из *пространства пользователя* через использование конкретного *файлового дескриптора*. *Dentry* остаётся в использовании, пока файл открыт, что, в свою очередь, означает, что VFS inode остаётся в использовании.

## Регистрация и монтирование файловой системы
Процедура регистрации типа файловой системы в ядре Linux, и процедура отмены регистрации, используют следующие функции API:
```
#include <linux/fs.h>

extern int register_filesystem(struct file_system_type *);
extern int unregister_filesystem(struct file_system_type *);
```

Передающийся `struct file_system_type` описывает вашу ФС. Когда запрашивается монтирование тома какой-либо ФС к каталогу в вашем *пространстве имён*, то VFS вызывет подходящий метод монтирования для конкретной ФС. Новый vfsmount, ссылающийся на дерево, возвращаемое ->mount(), будет подсоединён к точке монтирования, так что когда в процессе разбора *pathname* будет достигнута точка монтирования, то будет совершён переход внутрь vfsmount, на его корень.

Вы можете узнать о всех зарегистрированных в ядре типах файловых систем в файле `/proc/filesystems`/

### struct file_system_type
Эта структура описывает файловую систему. Начиная с kernel 2.6.39, определены следующие элементы `struct`:
```
struct file_system_operations {
        const char *name;
        int fs_flags;
        struct dentry *(*mount) (struct file_system_type *, int,
                                 const char *, void *);
        void (*kill_sb) (struct super_block *);
        struct module *owner;
        struct file_system_type * next;
        struct list_head fs_supers;
        struct lock_class_key s_lock_key;
        struct lock_class_key s_umount_key;
};
```
**name**
- наименование типа файловой системы, например как "ext2", "iso9660", "msdos" и прочее.

**fs_flags**
- различные флаги (то есть, FS_REQUIRES_DEV, FS_NO_DCACHE, и т.д.)

**mount**
- метод, вызов которого производится при монтировании нового экземпляра файловой системы

**kill_sb**
- метод, вызов которого производится при размонтировании экземпляра файловой системы

**owner**
- для внутреннего использования VFS: в большинстве случаев вы должны инициализировать это через THIS_MODULE

**next**
- для внутреннего использования VFS: вы должны инициализировать это в NULL

&nbsp;&nbsp;s_lock_key, s_umount_key: lockdep-specific

Метод `mount()` имеет следующие аргументы:
**struct file_system_type \*fs_type**
- описывает файловую систему, частично инициализируется определённым кодом файловой системы

**int flags**
- флаги монтирования

**const char \*dev_name**
- монтируемое имя устройства

**void \*data**
- различные опции в виде ASCII-строки, указываемые при монтировании (смотри раздел “Mount Options”)

При вызове, метод mount() должен возвращать корневой dentry запрошенного дерева каталогов. Вызывающий процесс должен захватить ссылку на *суперблок* ФС, и *суперблок* должен быть заблокирован. При ошибке, метод возвращает указатель ERR_PTR (ошибка).

Аргументы соответствуют таковым из mount(2) и их интерпретация зависит от типа ФС. Например, для блочной файловой системы, `dev_name` интерпретируется как имя блочного устройства, и устойство это открыто, и оно содержит подходящую файловую систему, для которой метод `mount()` создаёт и инициализирует соответствующий `struct super_block`, и возвращает корневой dentry вызывающему процессу.

...

## Объект super_block
Объект superblock представляет монтированную файловую систему.

### struct super_operations
Эта структура описывает как VFS может манипулировать суперблоком вашей файловой системы. Начиная с kernel 2.6.22, определены следующие элементы:
```
struct inode *(*alloc_inode)(struct super_block *sb);
void (*destroy_inode)(struct inode *);

void (*dirty_inode) (struct inode *, int flags);
int (*write_inode) (struct inode *, int);
void (*drop_inode) (struct inode *);
void (*delete_inode) (struct inode *);
void (*put_super) (struct super_block *);
int (*sync_fs)(struct super_block *sb, int wait);
int (*freeze_fs) (struct super_block *);
int (*unfreeze_fs) (struct super_block *);
int (*statfs) (struct dentry *, struct kstatfs *);
int (*remount_fs) (struct super_block *, int *, char *);
void (*clear_inode) (struct inode *);
void (*umount_begin) (struct super_block *);

int (*show_options)(struct seq_file *, struct dentry *);

ssize_t (*quota_read)(struct super_block *, int, char *, size_t, loff_t);
ssize_t (*quota_write)(struct super_block *, int, const char *, size_t, loff_t);
int (*nr_cached_objects)(struct super_block *);
void (*free_cached_objects)(struct super_block *, int);
};
```
Все методы вызываются без каких-либо удерживаемых? (being held) блокировок, если не указано иное. Это означает, что большинство методов можно безопасно блокировать. Все методы вызываются только из контекста процесса (то есть не из обработчика прерываний или ниже (bottom half)).

...

## Объект Inode
Объект inode представляет объект внутри файловой системы.

### struct inode_operations
Эта структура описывает, как VFS может манипулировать inode'ом в вашей файловой системе. Начиная с kernel 2.6.22, определены следующие элементы:
```
int (*create) (struct inode *,struct dentry *, umode_t, bool);
struct dentry * (*lookup) (struct inode *,struct dentry *, unsigned int);
int (*link) (struct dentry *,struct inode *,struct dentry *);
int (*unlink) (struct inode *,struct dentry *);
int (*symlink) (struct inode *,struct dentry *,const char *);
int (*mkdir) (struct inode *,struct dentry *,umode_t);
int (*rmdir) (struct inode *,struct dentry *);
int (*mknod) (struct inode *,struct dentry *,umode_t,dev_t);
int (*rename) (struct inode *, struct dentry *,
               struct inode *, struct dentry *, unsigned int);
int (*readlink) (struct dentry *, char __user *,int);
const char *(*get_link) (struct dentry *, struct inode *,
                         struct delayed_call *);
int (*permission) (struct inode *, int);
int (*get_acl)(struct inode *, int);
int (*setattr) (struct dentry *, struct iattr *);
int (*getattr) (const struct path *, struct kstat *, u32, unsigned int);
ssize_t (*listxattr) (struct dentry *, char *, size_t);
void (*update_time)(struct inode *, struct timespec *, int);
int (*atomic_open)(struct inode *, struct dentry *, struct file *,
                   unsigned open_flag, umode_t create_mode);
int (*tmpfile) (struct inode *, struct dentry *, umode_t);
};
```
И опять, все методы вызываются без каких-либо блокировок, если не указано иное.

...

## Объект Адресное Пространство
Объект *адресное пространство* используется для группирования и управления страницами в страничном кэше (page cache). Он может быть использован для отслеживания страниц в файле и отслеживания отображения секция файла в адресное пространство процесса.

...

### Обработка ошибок в течении writeback
Большинство приложений, которые буферизуют операции чтения/записи, периодически синхронизируют файл вызовом (fsync, fdatasync, msync or sync_file_range), чтобы быть уверенным, что данные в буфере сброшены обратно на устройство хранения. ...

### struct address_space_operations
Эта структура описывает, как VFS может манипулировать отображением файла в *страничном кэше* в вашей файловой системе. Определены следующие элементы структуры:
```
int (*writepage)(struct page *page, struct writeback_control *wbc);
int (*readpage)(struct file *, struct page *);
int (*writepages)(struct address_space *, struct writeback_control *);
int (*set_page_dirty)(struct page *page);
int (*readpages)(struct file *filp, struct address_space *mapping,
                 struct list_head *pages, unsigned nr_pages);
int (*write_begin)(struct file *, struct address_space *mapping,
                   loff_t pos, unsigned len, unsigned flags,
                struct page **pagep, void **fsdata);
int (*write_end)(struct file *, struct address_space *mapping,
                 loff_t pos, unsigned len, unsigned copied,
                 struct page *page, void *fsdata);
sector_t (*bmap)(struct address_space *, sector_t);
void (*invalidatepage) (struct page *, unsigned int, unsigned int);
int (*releasepage) (struct page *, int);
void (*freepage)(struct page *);
ssize_t (*direct_IO)(struct kiocb *, struct iov_iter *iter);
/* isolate a page for migration */
bool (*isolate_page) (struct page *, isolate_mode_t);
/* migrate the contents of a page to the specified target */
int (*migratepage) (struct page *, struct page *);
/* put migration-failed page back to right list */
void (*putback_page) (struct page *);
int (*launder_page) (struct page *);

int (*is_partially_uptodate) (struct page *, unsigned long,
                              unsigned long);
void (*is_dirty_writeback) (struct page *, bool *, bool *);
int (*error_remove_page) (struct mapping *mapping, struct page *page);
int (*swap_activate)(struct file *);
int (*swap_deactivate)(struct file *);
};
```
**writepage**
- вызывается *Виртуальной Машиной* для сброса черновых страниц обратно в хранилище. Это может случаться для сохранения целостности данных (`sync`), или для освобождения памяти (flush)....

**readpage**
- вызывается *Виртуальной Машиной* при чтении страницы из хранилища. Эта страница будет заблокирована во время вызова `readpage`, и должна быть разблокирована и промаркирована, как *uptodate*, с момента завершения чтения. Если `->readpage` обнаруживает, что требуется разблокировка страницы по какой-либо причине, то он может сделать это, но вернув AOP_TRUNCATED_PAGE. В этом случае страница будет relocated, relocked и, если всё успешно будет выполнено, то вызов `->readpage` будет повторён.

**writepages**
- вызывается *Виртуальной Машиной* для сброса на диск всех страниц, ассоциированных с *адресным пространством* объекта.

...

## Объект Файл
Файловый объект представляет файл, открытый процессом. Он также известен, как *файловый дескриптор* (open file description) в POSIX-диалекте.

### struct file_operations
Эта структура описывает, как VFS может манипулировать открытым файлом. Начиная с kernel 4.18, определены следующие элементы:
```
struct file_operations {
        struct module *owner;
        loff_t (*llseek) (struct file *, loff_t, int);
        ssize_t (*read) (struct file *, char __user *, size_t, loff_t *);
        ssize_t (*write) (struct file *, const char __user *, size_t, loff_t *);
        ssize_t (*read_iter) (struct kiocb *, struct iov_iter *);
        ssize_t (*write_iter) (struct kiocb *, struct iov_iter *);
        int (*iopoll)(struct kiocb *kiocb, bool spin);
        int (*iterate) (struct file *, struct dir_context *);
        int (*iterate_shared) (struct file *, struct dir_context *);
        __poll_t (*poll) (struct file *, struct poll_table_struct *);
        long (*unlocked_ioctl) (struct file *, unsigned int, unsigned long);
        long (*compat_ioctl) (struct file *, unsigned int, unsigned long);
        int (*mmap) (struct file *, struct vm_area_struct *);
        int (*open) (struct inode *, struct file *);
        int (*flush) (struct file *, fl_owner_t id);
        int (*release) (struct inode *, struct file *);
        int (*fsync) (struct file *, loff_t, loff_t, int datasync);
        int (*fasync) (int, struct file *, int);
        int (*lock) (struct file *, int, struct file_lock *);
        ssize_t (*sendpage) (struct file *, struct page *, int, size_t, loff_t *, int);
        unsigned long (*get_unmapped_area)(struct file *, unsigned long, unsigned long, unsigned long, unsigned long);
        int (*check_flags)(int);
        int (*flock) (struct file *, int, struct file_lock *);
        ssize_t (*splice_write)(struct pipe_inode_info *, struct file *, loff_t *, size_t, unsigned int);
        ssize_t (*splice_read)(struct file *, loff_t *, struct pipe_inode_info *, size_t, unsigned int);
        int (*setlease)(struct file *, long, struct file_lock **, void **);
        long (*fallocate)(struct file *file, int mode, loff_t offset,
                          loff_t len);
        void (*show_fdinfo)(struct seq_file *m, struct file *f);
#ifndef CONFIG_MMU
        unsigned (*mmap_capabilities)(struct file *);
#endif
        ssize_t (*copy_file_range)(struct file *, loff_t, struct file *, loff_t, size_t, unsigned int);
        loff_t (*remap_file_range)(struct file *file_in, loff_t pos_in,
                                   struct file *file_out, loff_t pos_out,
                                   loff_t len, unsigned int remap_flags);
        int (*fadvise)(struct file *, loff_t, loff_t, int);
};
```
Again, all methods are called without any locks being held, unless otherwise noted.

**llseek**
- вызывается, когда VFS нуждается в перемещении file position index

**read**
- называется `read(2)` и относится к системным вызовам
...

## Directory Entry Cache (dcache)
### struct dentry_operations
Эта структура описывает, как файловая система может перегружать стандартные *dentry* операции.

```
struct dentry_operations {
        int (*d_revalidate)(struct dentry *, unsigned int);
        int (*d_weak_revalidate)(struct dentry *, unsigned int);
        int (*d_hash)(const struct dentry *, struct qstr *);
        int (*d_compare)(const struct dentry *,
                         unsigned int, const char *, const struct qstr *);
        int (*d_delete)(const struct dentry *);
        int (*d_init)(struct dentry *);
        void (*d_release)(struct dentry *);
        void (*d_iput)(struct dentry *, struct inode *);
        char *(*d_dname)(struct dentry *, char *, int);
        struct vfsmount *(*d_automount)(struct path *);
        int (*d_manage)(const struct path *, bool);
        struct dentry *(*d_real)(struct dentry *, const struct inode *);
};
```
...

### Directory Entry Cache API
There are a number of functions defined which permit a filesystem to manipulate dentries:

## Mount Options
### Parsing options
При монтировании или размонтировании файловой системы, передаётся строка, содержащая список опций монтирования, разделённых запятыми. Эти опции могут иметь любую из этих форм:
```
option option=value
```

The <linux/parser.h> header defines an API that helps parse these options. There are plenty of examples on how to use it in existing filesystems.

### Showing options
Если файловая система приняла опции, то она должна определить show_options(), для показа всех актуальных текущих опций. Правила показа:

- обязаны быть показаны тех опции, значения которых отличаются от принимаемых по умолчанию;
- могут быть показаны те опции, которые включены (по умолчанию) или имеют значения по умолчанию.

Опции, внутренее используемые между маунт хэлпером и ядром (такие как дескриптор файла), или которые значимы только во время монтирования (такие, как управляющие созданием журнала), исключены от вышеоуказанных правил.

The underlying reason for the above rules is to make sure, that a mount can be accurately replicated (e.g. umounting and mounting again) based on the information found in /proc/mounts.

Resources
(Note some of these resources are not up-to-date with the latest kernel
version.)
Creating Linux virtual filesystems. 2002
<http://lwn.net/Articles/13325/>
The Linux Virtual File-system Layer by Neil Brown. 1999
<http://www.cse.unsw.edu.au/~neilb/oss/linux-commentary/vfs.html>
A tour of the Linux VFS by Michael K. Johnson. 1996
<http://www.tldp.org/LDP/khg/HyperNews/get/fs/vfstour.html>
A small trail through the Linux kernel by Andries Brouwer. 2001
<http://www.win.tue.nl/~aeb/linux/vfs/trail.html>
