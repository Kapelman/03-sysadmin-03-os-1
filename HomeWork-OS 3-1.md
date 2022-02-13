#_Домашнее задание занятию "3.3. Операционные системы, лекция 1"_ #
##Выполнил  - Каплин Владимир ##



1. Какой системный вызов делает команда cd? В прошлом ДЗ мы выяснили, что cd не является самостоятельной программой, это shell builtin, 
поэтому запустить strace непосредственно на cd не получится. 
Тем не менее, вы можете запустить strace на /bin/bash -c 'cd /tmp'. В этом случае вы увидите полный список системных вызовов, которые делает сам bash при старте. 
Вам нужно найти тот единственный, который относится именно к cd. Обратите внимание, что strace выдаёт результат своей работы в поток stderr, а не в stdout.

Решение: запустим трассировку strace  strace /bin/bash -c 'cd /tmp' 2>&1
```
vagrant@vagrant:~$ strace /bin/bash -c 'cd /tmp' 2>&1
```
Нужные нам строки stat и chdir
```
stat("/tmp", {st_mode=S_IFDIR|S_ISVTX|0777, st_size=4096, ...}) = 0
chdir("/tmp")
```
Узнаем, что это за системные вызовы.

Stat возвращает информацию о файле в виде структуры. Сама функция возвращает 0 в случае успеха.
```
vagrant@vagrant:~$ man 2 stat

DESCRIPTION
       These  functions  return  information  about a file, in the buffer pointed to by statbuf.  No permissions are required on the file itself, but—in the case of stat(), fstatat(), and lstat()—execute
       (search) permission is required on all of the directories in pathname that lead to the file.
RETURN VALUE
       On success, zero is returned.  On error, -1 is returned, and errno is set appropriately.
```

Chdir()  меняет текущую рабочую директорию процесса на директорию, указанную в аргументе

vagrant@vagrant:~$ man 2 chdir
DESCRIPTION
       chdir() changes the current working directory of the calling process to the directory specified in path.

2. Попробуйте использовать команду file на объекты разных типов на файловой системе. Например:
```
vagrant@netology1:~$ file /dev/tty
/dev/tty: character special (5/0)
vagrant@netology1:~$ file /dev/sda
/dev/sda: block special (8/0)
vagrant@netology1:~$ file /bin/bash
/bin/bash: ELF 64-bit LSB shared object, x86-64
```
Используя strace выясните, где находится база данных file на основании которой она делает свои догадки.

Решение: Запустим strace c фильтром на основные системные вызовы работы с файлами и чтением\записью в FD
Увидим, что единственным подходящим файлом может быть только  "/usr/share/misc/magic.mgc", т.к. остальные файлы либо 
являются системными библиотеками, либо не существуют (/home/vagrant/.magic.mgc), либо имеет небольшой размер "/etc/magic".

Ответ - "/usr/share/misc/magic.mgc"

```
vagrant@vagrant:~$ strace -e stat,lstat,fstat,openat,read,write file /dev/sda
openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=35401, ...}) = 0
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libmagic.so.1", O_RDONLY|O_CLOEXEC) = 3
read(3, "\177ELF\2\1\1\0\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0 N\0\0\0\0\0\0"..., 832) = 832
fstat(3, {st_mode=S_IFREG|0644, st_size=158080, ...}) = 0
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libc.so.6", O_RDONLY|O_CLOEXEC) = 3
read(3, "\177ELF\2\1\1\3\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0\360q\2\0\0\0\0\0"..., 832) = 832
fstat(3, {st_mode=S_IFREG|0755, st_size=2029224, ...}) = 0
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/liblzma.so.5", O_RDONLY|O_CLOEXEC) = 3
read(3, "\177ELF\2\1\1\0\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0\3003\0\0\0\0\0\0"..., 832) = 832
fstat(3, {st_mode=S_IFREG|0644, st_size=162264, ...}) = 0
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libbz2.so.1.0", O_RDONLY|O_CLOEXEC) = 3
read(3, "\177ELF\2\1\1\0\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0@\"\0\0\0\0\0\0"..., 832) = 832
fstat(3, {st_mode=S_IFREG|0644, st_size=74848, ...}) = 0
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libz.so.1", O_RDONLY|O_CLOEXEC) = 3
read(3, "\177ELF\2\1\1\3\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0\200\"\0\0\0\0\0\0"..., 832) = 832
fstat(3, {st_mode=S_IFREG|0644, st_size=108936, ...}) = 0
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libpthread.so.0", O_RDONLY|O_CLOEXEC) = 3
read(3, "\177ELF\2\1\1\0\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0\220\201\0\0\0\0\0\0"..., 832) = 832
fstat(3, {st_mode=S_IFREG|0755, st_size=157224, ...}) = 0
openat(AT_FDCWD, "/usr/lib/locale/locale-archive", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=3035952, ...}) = 0
stat("/home/vagrant/.magic.mgc", 0x7ffcd3d09560) = -1 ENOENT (No such file or directory)
stat("/home/vagrant/.magic", 0x7ffcd3d09560) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/etc/magic.mgc", O_RDONLY) = -1 ENOENT (No such file or directory)
stat("/etc/magic", {st_mode=S_IFREG|0644, st_size=111, ...}) = 0
openat(AT_FDCWD, "/etc/magic", O_RDONLY) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=111, ...}) = 0
read(3, "# Magic local data for file(1) c"..., 4096) = 111
read(3, "", 4096)                       = 0
openat(AT_FDCWD, "/usr/share/misc/magic.mgc", O_RDONLY) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=5811536, ...}) = 0
openat(AT_FDCWD, "/usr/lib/x86_64-linux-gnu/gconv/gconv-modules.cache", O_RDONLY) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=27002, ...}) = 0
fstat(1, {st_mode=S_IFCHR|0620, st_rdev=makedev(0x88, 0), ...}) = 0
lstat("/dev/sda", {st_mode=S_IFBLK|0660, st_rdev=makedev(0x8, 0), ...}) = 0
write(1, "/dev/sda: block special (8/0)\n", 30/dev/sda: block special (8/0)
) = 30
+++ exited with 0 +++
```


3. Предположим, приложение пишет лог в текстовый файл. Этот файл оказался удален (deleted в lsof), 
однако возможности сигналом сказать приложению переоткрыть файлы или просто перезапустить приложение – нет. 
Так как приложение продолжает писать в удаленный файл, место на диске постепенно заканчивается. 
Основываясь на знаниях о перенаправлении потоков предложите способ обнуления открытого удаленного файла 
(чтобы освободить место на файловой системе).

Моделируем ситуацию:
- Запускаем бесконечный пинг, который пишет в файл в одном терминале

```
vagrant@vagrant:~$ ping 127.0.0.1 >file_ping 2>&1
```
- Файл file_ping начал расти
```
vagrant@vagrant:~$ ls -al
total 100
drwxr-xr-x 6 vagrant vagrant  4096 Feb  2 20:45 .
drwxr-xr-x 3 root    root     4096 Dec 19 19:42 ..
-rw-rw-r-- 1 vagrant vagrant  1084 Feb  2 20:45 file_ping
```
- Удаляем файл
```
vagrant@vagrant:~$ rm file_ping
```
- Проверим командой ls, что файл отсутствует.

```
vagrant@vagrant:~$ ls -al
total 96
drwx------ 3 vagrant vagrant  4096 Jan 18 20:36 .config
-rw-rw-r-- 1 vagrant vagrant    30 Feb  2 20:08 file
-rw-rw-r-- 1 vagrant vagrant    13 Jan 26 21:22 file1
-rw-rw-r-- 1 vagrant vagrant  9022 Feb  2 20:08 file3
-rw------- 1 vagrant vagrant   317 Jan 28 21:37 .lesshst
drwxrwxr-x 3 vagrant vagrant  4096 Jan 18 20:36 .local
-rw-r--r-- 1 vagrant vagrant   866 Jan 12 21:11 .profile
```
- Находим по Pstree -p PID процесса - 1806, видим метку deleted

```
vagrant@vagrant:~$ sudo lsof -p 1806
COMMAND  PID    USER   FD   TYPE DEVICE SIZE/OFF    NODE NAME
ping    1806 vagrant  cwd    DIR  253,0     4096 1051845 /home/vagrant
ping    1806 vagrant  rtd    DIR  253,0     4096       2 /
ping    1806 vagrant  txt    REG  253,0    72776 1835881 /usr/bin/ping
ping    1806 vagrant  mem    REG  253,0  3035952 1835290 /usr/lib/locale/locale-archive
ping    1806 vagrant  mem    REG  253,0   137584 1841525 /usr/lib/x86_64-linux-gnu/libgpg-error.so.0.28.0
ping    1806 vagrant  mem    REG  253,0  2029224 1841468 /usr/lib/x86_64-linux-gnu/libc-2.31.so
ping    1806 vagrant  mem    REG  253,0   101320 1841650 /usr/lib/x86_64-linux-gnu/libresolv-2.31.so
ping    1806 vagrant  mem    REG  253,0  1168056 1835853 /usr/lib/x86_64-linux-gnu/libgcrypt.so.20.2.5
ping    1806 vagrant  mem    REG  253,0    31120 1841471 /usr/lib/x86_64-linux-gnu/libcap.so.2.32
ping    1806 vagrant  mem    REG  253,0   191472 1841428 /usr/lib/x86_64-linux-gnu/ld-2.31.so
ping    1806 vagrant    0u   CHR  136,0      0t0       3 /dev/pts/0
ping    1806 vagrant    1w   REG  253,0    44014 1048629 /home/vagrant/file_ping (deleted)
ping    1806 vagrant    2w   REG  253,0    44014 1048629 /home/vagrant/file_ping (deleted)
ping    1806 vagrant    3u  icmp             0t0   34871 00000000:0001->00000000:0000
ping    1806 vagrant    4u  sock    0,9      0t0   34872 protocol: PINGv6
```
- Воспользуемся командой tee (также можно и просто перенаправить вывод, например, echo stop >file_ping )
видим, что файл file_ping снова появился, размер файла 7 байт.
```
echo string | sudo tee /home/vagrant/file_ping

vagrant@vagrant:~$ ls -al
total 100
drwx------ 3 vagrant vagrant  4096 Jan 18 20:36 .config
-rw-rw-r-- 1 vagrant vagrant    30 Feb  2 20:08 file
-rw-rw-r-- 1 vagrant vagrant    13 Jan 26 21:22 file1
-rw-rw-r-- 1 vagrant vagrant  9022 Feb  2 20:08 file3
-rw-rw-r-- 1 vagrant vagrant     7 Feb  2 21:16 file_ping
-rw------- 1 vagrant vagrant   317 Jan 28 21:37 .lesshst
```

- единственно не могу понять почему в запущенном процессе нет происходит обновления информации.
```
vagrant@vagrant:~$ sudo lsof -p 1806
COMMAND  PID    USER   FD   TYPE DEVICE SIZE/OFF    NODE NAME
ping    1806 vagrant  cwd    DIR  253,0     4096 1051845 /home/vagrant
ping    1806 vagrant  rtd    DIR  253,0     4096       2 /
ping    1806 vagrant  txt    REG  253,0    72776 1835881 /usr/bin/ping
ping    1806 vagrant  mem    REG  253,0  3035952 1835290 /usr/lib/locale/locale-archive
ping    1806 vagrant  mem    REG  253,0   137584 1841525 /usr/lib/x86_64-linux-gnu/libgpg-error.so.0.28.0
ping    1806 vagrant  mem    REG  253,0  2029224 1841468 /usr/lib/x86_64-linux-gnu/libc-2.31.so
ping    1806 vagrant  mem    REG  253,0   101320 1841650 /usr/lib/x86_64-linux-gnu/libresolv-2.31.so
ping    1806 vagrant  mem    REG  253,0  1168056 1835853 /usr/lib/x86_64-linux-gnu/libgcrypt.so.20.2.5
ping    1806 vagrant  mem    REG  253,0    31120 1841471 /usr/lib/x86_64-linux-gnu/libcap.so.2.32
ping    1806 vagrant  mem    REG  253,0   191472 1841428 /usr/lib/x86_64-linux-gnu/ld-2.31.so
ping    1806 vagrant    0u   CHR  136,0      0t0       3 /dev/pts/0
ping    1806 vagrant    1w   REG  253,0   164541 1048629 /home/vagrant/file_ping (deleted)
ping    1806 vagrant    2w   REG  253,0   164541 1048629 /home/vagrant/file_ping (deleted)
ping    1806 vagrant    3u  icmp             0t0   34871 00000000:0001->00000000:0000
ping    1806 vagrant    4u  sock    0,9      0t0   34872 protocol: PINGv6
```
4. Занимают ли зомби-процессы какие-то ресурсы в ОС (CPU, RAM, IO)?

Зомби не занимают памяти, CPU или IO, но блокируют записи в таблице процессов, размер которой ограничен для каждого пользователя и системы в целом.
Поэтому проблемой может стать большое кол-во зомби процессов, занявшив всю таблицу процессов. В такой ситуации новые 
процессы не смогут появляться.

5. В iovisor BCC есть утилита opensnoop:
```
root@vagrant:~# dpkg -L bpfcc-tools | grep sbin/opensnoop
/usr/sbin/opensnoop-bpfcc
```
На какие файлы вы увидели вызовы группы open за первую секунду работы утилиты? Воспользуйтесь пакетом bpfcc-tools 
для Ubuntu 20.04. 

Решение: Установим пакет с помощью инструкции:

```
sudo apt-get install bpfcc-tools linux-headers-$(uname -r)
```


```
vagrant@vagrant:/usr/sbin$ dpkg -L bpfcc-tools | grep sbin/opensnoop
/usr/sbin/opensnoop-bpfcc

vagrant@vagrant:/usr/sbin$ sudo /usr/sbin/opensnoop-bpfcc
PID    COMM               FD ERR PATH
1195   vminfo              4   0 /var/run/utmp
677    dbus-daemon        -1   2 /usr/local/share/dbus-1/system-services
677    dbus-daemon        20   0 /usr/share/dbus-1/system-services
677    dbus-daemon        -1   2 /lib/dbus-1/system-services
677    dbus-daemon        20   0 /var/lib/snapd/dbus-1/system-services/
683    irqbalance          6   0 /proc/interrupts
683    irqbalance          6   0 /proc/stat
683    irqbalance          6   0 /proc/irq/20/smp_affinity
683    irqbalance          6   0 /proc/irq/0/smp_affinity
683    irqbalance          6   0 /proc/irq/1/smp_affinity
683    irqbalance          6   0 /proc/irq/8/smp_affinity
683    irqbalance          6   0 /proc/irq/12/smp_affinity
683    irqbalance          6   0 /proc/irq/14/smp_affinity
683    irqbalance          6   0 /proc/irq/15/smp_affinity
1195   vminfo              4   0 /var/run/utmp
677    dbus-daemon        -1   2 /usr/local/share/dbus-1/system-services
677    dbus-daemon        20   0 /usr/share/dbus-1/system-services
677    dbus-daemon        -1   2 /lib/dbus-1/system-services
677    dbus-daemon        20   0 /var/lib/snapd/dbus-1/system-services/
1195   vminfo              4   0 /var/run/utmp
```

6. Какой системный вызов использует uname -a? Приведите цитату из man по этому системному вызову, 
где описывается альтернативное местоположение в /proc, где можно узнать версию ядра и релиз ОС.

Решение: запустим команду uname -a с трассировкой
 
```
uname({sysname="Linux", nodename="vagrant", ...}) = 0
fstat(1, {st_mode=S_IFCHR|0620, st_rdev=makedev(0x88, 0x1), ...}) = 0
uname({sysname="Linux", nodename="vagrant", ...}) = 0
uname({sysname="Linux", nodename="vagrant", ...}) = 0
write(1, "Linux vagrant 5.4.0-91-generic #"..., 106Linux vagrant 5.4.0-91-generic #102-Ubuntu SMP Fri Nov 5 16:31:28 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux
) = 106
close(1)                                = 0
close(2)                                = 0
exit_group(0)                           = ?
```
По выводу находим системный вызов uname. Именно этот системный вызов обспечивает информацией о версии системы.

```
NAME
       uname - get name and information about current kernel
```

Читая help, находим строку о файле, в котором можно увидеть аналогичную информацию.
Это папка /proc/sys/kernel/ и файлы ostype, hostname, osrelease, version, domainname
```
Part of the utsname information is also accessible via /proc/sys/kernel/{ostype, hostname, osrelease, version, domainname}.
```
Проверяем, данные совпадают.

```
vagrant@vagrant:/proc/sys/kernel$ uname -a
Linux vagrant 5.4.0-91-generic #102-Ubuntu SMP Fri Nov 5 16:31:28 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux
vagrant@vagrant:/proc/sys/kernel$ cat /proc/sys/kernel/osrelease
5.4.0-91-generic
```

7. Чем отличается последовательность команд через ; и через && в bash? Например:
```
root@netology1:~# test -d /tmp/some_dir; echo Hi
Hi
root@netology1:~# test -d /tmp/some_dir && echo Hi
root@netology1:~#
```
Есть ли смысл использовать в bash &&, если применить set -e?


Решение:
- Точка с запятой позволяет записывать две и более команд в одной строке. Соответственно первая строка просто выполнит
две команды, одна за другой. echo Hi выведет терминал Hi

- && представляет из себя логический оператор "И". Команда 2 будет выполнена, только когда команда 1 вернет статус 0.
```
command2 is executed if, and only if, command1 returns an exit status of zero (success).
```
Соответственно вторая строка не выведет Hi, т.к. деректория /tmp/some_dir отсутствует.

- Опиция set -e закроет сессию, если код возврата команды не равен 0. Но exit не сработает, если за командой с кодом возврата
отличным от 0 используется логическое И (&&), т.е. && можно применять и при set -e.
```
-e  Exit immediately if a command exits with a non-zero status.

The shell does not exit if the command that fails is part of 
the command list immediately following a while or until keyword, 
part of the test in an if statement, part of any command executed in a && or || list except 
the command following the final && or ||, any command in a pipeline but the last, 
or if the command’s return status is being inverted with !
```
8. Из каких опций состоит режим bash set -euxo pipefail и почему его хорошо было бы использовать в сценариях?

Ответ: Согласно описанию, при таком наборе опций bash будет прерывать сессию, если возникнет ошибка (возврат статуса, 
отличного от 0) при выполнении команды, в т.ч. при выполнении любой команды в PIPILINE, также выполниться
exit при обращении к несущетвующей переменной среды. До прерывании сессии будет 
выведет лог выпонения команд, значения и ошибки. 

Т.е. таким образом при таком наборе опций можно отключать неинтерактивные сессии, если при их работе возникли ошибки или
было обращение к несуществующей переменной окружения, вызов логируется, в т.ч. ошибки.

```
-e  Exit immediately if a command exits with a non-zero status. 
-u  Treat unset variables as an error when substituting.
-x  Print commands and their arguments as they are executed.
-o option-name
pipefail     the return value of a pipeline is the status of
             the last command to exit with a non-zero status,
             or zero if no command exited with a non-zero status
```   

```
vagrant@vagrant:~$ set -euxo pipefail
vagrant@vagrant:~$ ls -al /tmp/ 2>&1 | grep 'no' | echo $path
+ grep --color=auto no
-bash: path: unbound variable
+ ls --color=auto -al /tmp/
Connection to 127.0.0.1 closed.
```
```
vagrant@vagrant:~$ set -euxo pipefail
vagrant@vagrant:~$ ls -al /tmp/rrrr 2>&1 | grep 'no'
+ grep --color=auto no
+ ls --color=auto -al /tmp/rrrr
ls: cannot access '/tmp/rrrr': No such file or directory
Connection to 127.0.0.1 closed.
```

9. Используя -o stat для ps, определите, какой наиболее часто встречающийся статус у процессов в системе. 
В man ps ознакомьтесь (/PROCESS STATE CODES) что значат дополнительные к основной заглавной буквы статуса процессов. 
Его можно не учитывать при расчете (считать S, Ss или Ssl равнозначными).


Решение:  с помощью команды ps -Ao stat найдем все процессы.
Больше всего процессов статусе S    interruptible sleep (waiting for an event to complete),
меньше всего в статусе R running or runnable (on run queue)
```
I 54
R 3
S 74
```

С помощью команды man ps найдем описание второй буквы статуса.
```
Here are the different values that the s, stat and state output specifiers (header "STAT" or "S") will display to describe the state of a process:

               D    uninterruptible sleep (usually IO)
               I    Idle kernel thread
               R    running or runnable (on run queue)
               S    interruptible sleep (waiting for an event to complete)
               T    stopped by job control signal
               t    stopped by debugger during the tracing
               W    paging (not valid since the 2.6.xx kernel)
               X    dead (should never be seen)
               Z    defunct ("zombie") process, terminated but not reaped by its parent

       For BSD formats and when the stat keyword is used, additional characters may be displayed:

               <    high-priority (not nice to other users)
               N    low-priority (nice to other users)
               L    has pages locked into memory (for real-time and custom IO)
               s    is a session leader
               l    is multi-threaded (using CLONE_THREAD, like NPTL pthreads do)
               +    is in the foreground process group
```

В системе есть следующие процессы:

```
I
I<
R
R+
S
S+
S<
S<s
Sl
SLsl
SN
Ss
Ssl
```