# Лекции 8 семестр

- [Лекция 11](#Лекция-11)
	- [Flashback](#Flashback)
	- [Recycle Bin](#Recycle-Bin)

- [Лекция 12](#Лекция-12)
	- [Flashback transaction](#Flashback-transaction)
	- [Flashback database](#Flashback-database)

- [Лекция 13](#Лекция-13)
	- [ASM](#ASM)
	- [Failure group](#Failure-group)
	- [Зеркалирование](#Зеркалирование)

- [Лекция 14](#Лекция-14)
	- [Запуск RMAN](#Запуск-RMAN)

- [Лекция 15](#Лекция-15)
	- [Каталог восстановления](#Каталог-восстановления)
	- [Dublicate database](#Dublicate-database)
	- [Операции с каталогом](#Операции-с-каталогом)
	- [Virtual private catalog](#Virtual-private-catalog)

- [Лекция 16](#Лекция-16)
	- [Бекапы](#Бекапы)

- [Лекция 17](#Лекция-17)
	- [Создание backup sets](#Создание-backup-sets)
	- [Fast incremental backup](#Fast-incremental-backup)
	- [Read-only tablespace copy](#Read--only-tablespace-copy)

- [Лекция 18](#Лекция-18)
	- [Restore Recover](#Restore-Recover)
	- [Redo log group](#Redo-log-group)
	- [Типы recovery](#Типы-recovery)
	- [Пользовательское восстановление](#Пользовательское-восстановление)

- [Лекция 19](#Лекция-19)
	- [Восстановления в режиме ARCHIVELOG и без него](#Восстановления-в-режиме-ARCHIVELOG-и-без-него)

- [Лекция 20](#Лекция-20)
	- [Дубликат БД](#Дубликат-БД)
	- [Dispatcher](#Dispatcher)

- [Лекция 21](#Лекция-21)
	- [TSPITR](#TSPITR)

- [Лекция 22](#Лекция-22)
	- [Monitoring RMAN](#Monitoring-RMAN)
	- [Опция DEBUG](#Опция-DEBUG)
	- [ОПЕРАЦИИ ВВОДА и ВЫВОДА](#ОПЕРАЦИИ-ВВОДА-и-ВЫВОДА)
	- [Мультиплекирование](#Мультиплекирование)
	- [Скорость бекапа на TAPE](#Скорость-бекапа-на-TAPE)
	- [Настройка команды BACKUP](#Настройка-команды-BACKUP)

- [Лекция 23](#Лекция-23)
	- [Automatic diagnostic repository](#Automatic-diagnostic-repository)
	- [Проблема](#Проблема)
	- [Инцидент](#Инцидент)
	- [HEALTH MONITOR](#HEALTH-MONITOR)

- [Лекция 24](#Лекция-24)
	- [Corrupted blocks](#Corrupted-blocks)
	- [DB_BLOCK_CHECKING](#DB_BLOCK_CHECKING)

# Лекция 11
## Flashback

- Flashback query - запросы к старому состоянию таблицы в каком-то моменте времени.  
- Flashback version query - как изменялись данные в определненный момент времени - не используется для внешних, временных, sys таблицами и представлениями. Не может захватить интервал при наличии DDL команды и сжатие сегмента.        
- Flashback transaction query - изменения выполненные за транзакцию - DDL - изменения в системных таблицах.  
- Flashback transaction - восстановление данных по транзакции с учетом и без учета зависимостей.   

1. Уровень БД (Flashback log - flashback recovery area)
	- Можно восстановить бд на конкретный момент времени
	- По умолчанию выключена, так как ресуросемкая
2. Уровень таблицы 
	- Drop - восстановление таблицы и данных (recycle bin)
	- Table - восстановление из undo 
	- Query - сравнение текущих данных и данных в прошлом из Undo
	- Version - из Undo
	- Data archive - получить исторические данные по транзакциям (Archive log)
3. Уровень транзакций - восстанавливает подозрительные транзакции и зависимые undo/redo archive log

## Recycle Bin

Recyclebin=ON; - не работает.   
Flashback table <name> to before drop.   
BIN$ - приставка, данные переименовываются, но не перемещаются, DBA_FREE_SPACE - не изменяется.   

Данные хранятся, пока не удалим или хватает места на диске.   
В случае восстановления данных с одинаковым именем, то для восстановления можно использовать имена системы, иначе будет восстановлена последняя удаленная.    
При недостатке места, корзина будет расширяться, если это невозможно, то удаляются элементы корзины.   
Для очищения корзины можно выполнить команду purge:
1. Purge table|index
2. Purge Tablespace [user]
3. Purge user_recyclebin (мои объекты), dba_recyclebin (вся корзина).
Удаление минуя корзину:
1. Drop table <> purge
2. Drop tablespace <> including contents
3. Drop user <> cascade - удаление юзера вместе с его объектами
*can_undrop* - могут быть восстановлены.  
*show recyclebin*.   

Для включения Flashback включить:
1. Undo_managment='Auto'
2. Undo_tablespace='UNDOTBS1' - активно в данный момент только одно
3. Undo_retention=900

Ретроспективный запрос.  

Logminer - для упрощения восстановления - определяет undo sql.     

Предустановки:
1. Иметь разрешение на выполнение пакета dbms_flashback
2. Доп логирование для первичного ключа
3. Select any transaction привелегия
4. Дополнительные журналы логирования 

Опции восстановления:
1. NoconflictOnly - неконфликтующие изменения целевой транзакции будут откатаны 
2. Nocascade force - все изменения в целевой транзакции будут откатаны
3. Cascade - будут откатаны изменения в зависимых и целевой транзакциях

Отчеты об откатах - dba_flashback_txn_state, dba_fkashback_txn_report.  

# Лекция 12
## Flashback transaction

Осуществляется онлайн и требует:
1. Наличие прав Flashback any table или наличии привилегии Flashback для конкретной таблицы.
2. Наличие привилегий select, insert, delete, alter - dml 
3. Включить row movement (alter table <name> enable row movement)

Ограничения:
1. Статистика не откатывается
2. Невозможно для системных таблиц, не включает ddl операции, не генерирует undo/redo
3. Поддерживаются текщие индексы и зависимости
4. Выполняется как единая транзакция с exclusive DML lock.

## Flashback database

Подготовительный этап:
1. SHUTDOWN IMMEDIATE;
2. STARTUP MOUNT EXCLUSIVE;
3. ALTER SYSTEM SET DB_FLASHBACK_RETENTION_TARGET=2880 SCOPE=BOTH;
4. ALTER DATABASE FLASHBACK ON;
5. ALTER DATABASE OPEN;

Когда не можем использовать Flashback database:
1. Контрольные файлы были восстановлены или заново созданы
2. Табличичное пространство было удалено
3. Файлы данных увеличились в размере
  
Гарантированная точка восстановления гарантирует, что вы можете выполнить команду FLASHBACK DATABASE на этот SCN в любое время.

# Лекция 13
## ASM
ASM - automatic storage management.  
Фоновые процессы на сторне ASM:
- RBAL - координирует балансировку активности для дисклвых групп
- ARB0 - Перемещают allocation unit между дисками ASM по 1, 2, 4, 8, 16, 64 мб обычно
- GMON - Group monitor - работает со спец системой встроенной таблицы, которая зранит инфу о стаусе дисковых групп

Фоновые процессы на сторне БД:
- RBAL - открыть диски в дисковых группах ASM
- ASMB - осуществляет подключение FG - "моста" к экземпляру ASM

Диски ASM объединяются в дисковые группы.   

Процесс создания ASM экземпляра:
- Создаем дисковую группу
- Какая избыточность (делать ли копии или нет)
- Выбираем разделы для включения в дисковую группу
- Есть скрипт local_config, который сконфигурирует кластерную систему
- Создастся файл параметров и файл паролей
- ASM_POWER_LIMIT - контролирует скорость перебалансировки (от 1 до 11 - max)
- ASM_DISKSTRING - список дисков
- ASM_DISKGROUPS - список группов дисков
- SPFILE - отдельный spfile+asm.ora  
Можно коннектится как sysasm или sysoper (может лишь остановить или запустить).   
Экземпляр не монтируем и не открываем.   
SYSASM - роль для обслуживания ASM экземпляров.   
В ASM - нет словаря данных.   
Для остановки необходимо - необходимо сначала отключить бд, а потом ASM.   
Структура:
- ASM файлы состоят из ASM дисковых групп, которые состоят из дисков, которые состоят из allocation unit, которые отображаются на физические блоки.
- ASM файлы - хранят инфу о файлах данных, контрольных файлах, spfiles.

Дисковая группа - набор дисков, который управляется, как одна логическая единица.   

## Failure group

Failure group:
- Группа ошибок - набор дисков, обеспечивающий дублирование информации - дисковую группу разделяют на группу дисков и данные в каждой группе дублируются. Теперь когда одна группа упадет, информация останется на других.

## Зеркалирование

Зеркалирование - disk group mirroring:
- Зеркалирование осуществляется на уровне экстентов (1 или несколько allocation units)
- На каждом диске одновременно находятся исходные и зеркальные au
- Уровни избыточности (зеркалированности):
	- Внешняя избыточность - при помощи оборудования
	- Нормальная избыточность - два зеркала - по крайней мере две группы ошибок
	- Высока избыточность - как минимум 3 failure group
По умолчанию каждый диск это failure group.  
Динамическая перебалансировка - добавили/удалили дисковую группу - автоматически происходит перебалансировка данных - перемещается то количество даннх, которое пропорционально добавленному хранилищу.    
Скорость перебалансировки определяется параметров ASM_POWER_LIMIT (1 -> 11).   

Версии формата обмена между бд и ASM:
1. Версия бд должна быть больше, чем Compatible.RDBMS - формат сообщений между бд и ASM (минимальная версия для монтирования дисковой группы).
2. Версия asm должна быть больше, чем Compatible.ASM - формат хранения мета информации на ASM.
3. Полезно при работе с гетерогенными системами
4. Этот параметр нельзя отменить

Параметры:
- au_size - 1, 2, 4, 8, 16, 32, 64 mb
- Compatible.RDBMS
- Compatible.ASM
- disk_repair_time - 0 - 2^32 - время перед отключением диска при выключении
- template.redundancy - шаблон избыточности - unprotected|mirror|high
- template.stripe - расслоение данных - coarse (для перераспределения данных)|fine (распределение по 128 кб для файлов управления, redo logs)

## Механизм быстрой перебалансировки

Механизм быстрой перебалансировки - fast mirror resync - упал какой-то диск - переходим на другой - диск восстановился - скопировали на него только модифицированные экстенты.   
V$ASM_DISK and V$ASM_DISK_STAT - представления с информациях о дисках.

Ограничения ASM:
- 63 дисковых группы
- 10 000 дисков ASM
- 4 петабайта данных для каждого диска
- 40 экзабайт хранилища
- 1 миллион файлов

# Лекция 14

## Запуск RMAN

Запуск RMAN:
`rmat target (целевая база) (имя пользователя/пароль@индетификатор бд)`, если больше ничего не указывать, то вся мета инфа о восстановлении будет сохранятся в файле управления.    
log_archive_min_success_dest - минимальное возможное количество успешного архивирования.  
RUN - запуск пакетов (job).   
Бекапы могут быть сделаны на диске, ленточном устройстве (Media management library), FRA.   
Политика удежрания - сколько необходимо зранить резервные копии:
- Recovery window - пероид времени, в который восстановление доступно
- Redundancy - фиксированное число бекапов, которые должны храниться
REPORT OBSOLETE - отчет по устаревшим файлам.   
DELETE OBSOLETE - удаление устаревших копий.   

# Лекция 15

## Каталог восстановления

Каталог восстановления содержит метаданные об операциях RMAN для каждой зарегистрированной целевой базы данных.

Rman + controlfile:
- Простота администрирования
- По умолчанию

Rman + recovery catalog:
- Копии controlfiles
- Может обслуживаться нексолько бд
- Сохранение скриптов
- Отчеты для таргетов
- Возможность хранить бекапы вечно

Метаданные: backup set list, image copy list
В Recovery catalog database хранится:
- Инфомрация о структуре бд
- Архивы журналов повторов
- Информация по Backup sets
- Информация по Data file copies

## Dublicate database

Если сделали дубликат бд и пытаемся ее зарегистрировать в каталоге (clone), то будет ошибка, так как бд уже есть. В клоне не меняется id (dbid) не изменяется (dbid v$database). Чтобы изменить dbid утилита - dbnewid - `nid target=user/pass@svc_name [DBNAME=new_dbname]` - база должна быть в режиме mount.

Чтобы будет после смены:
- Предыдущие бекапы и логи недоступны
- Надо открыть бд с resetlogs
- Сделать копию бд

Unregister database - отвязать бд от каталога.

## Операции с каталогом

В каталог можно добавить:
- Копии controlfile
- Data file
- Backup
- Archived redo log.   

Пересинхронизация:
- Частичная: журналы повторов, backup sets, data file copies.
- Полная: частичная + структура бд (создается снепшот файла управления).   

Resync:
- Выполнение нечастых бекпов
- Изменена структура бд

Скрипты:
- Локальные - ассоциированы с конкретной бд (CREATE SCRIPT)
- Глобальные - работает с любой бд (CREATE GLOBAL SCRIPT).   

Резервное копирование каталога:
- На диск
- На ленточное устройство
- Копия файлов управления

Resync catalog (на основии файла управления) или catalog start with.   
Экспрот и импорт (export, import) - создаем логическое резервное копирование (набор команд для выполнения)

## Virtual private catalog

Virtual private catalog - кусок из основного каталога. (GRANT REGISTER [Catalog] DATABSE [NAME] TO user).   
`create virtual catalog;`.  

# Лекция 16

## Бекапы

Типы бекапов:
- Full: всё что есть в бд - файлы данных, control file + archive log и spfile [optional] + опция delete input - удаление archive log после копирования - работа не с fra - DISK.
- Incremental: только те изменения, которые были выполнены после последнего бекапа

Определить файлы для бекапа: вся бд, журналы повторов, control files, spfile.   
Image copy - bit to bit (только на диск копирование).  

Место бекапа:
- Диск
- Media Management Library - бекап если работаем с rman для ленточных устройств (или Oracle secure backup)
- FRA (файлы OMF, сами удаляются и управляются)

Параметры rman:
- Каналы - по которым идет запись
- Число backup copies
- Определение максимального размера и числа файла копии
- Оптимизиция
- Автоматическое копирование control file
- Определение параллелизма (sbt - serial backup tape - `PARALLELISM 3`)
- Установка параметров шифрования и компрессии (COMPRESSED - только backup set - удаление пустых экстентов + над HWM)
- Исключение табличного пространства из бекапа
- Device type 

Есть галочка в policy - автоматическое копирование файлов управления и spfile при каждом бекапе и изменении структуры бд.    

Опция CLEAR - установка параметра в значение по умолчанию.

show exclude - те табличные пространства, которые исключены из бекапов.   

Оптимизации backup - пропуск файлов, которые были backup'ed:
- Когда включена оптимизация
- ALL | LIKE option
- Один тип канала выделен
- Используется FRA
   
Как мы видим, команда crosscheck RMAN сравнивает записи каталога RMAN с фактическими файлами операционной системы и сообщает о местонахождении "истекших" или "устаревших" записей каталога RMAN.

# Лекция 17

BACKUP AS [BACKUPSET | COPY]

## Создание backup sets

Формат файла:
- d - имя базы данных
- s - номер backup set
- p - номер куска backup set (piece)
- D - день месяца

Уровни инкрементального бекапа:
- 0: аналогичен полному бекапу
- 1: куммулятивный - только модифицированные после бекапа уровня 0, differential - блоки после последнего инкрементального бекапа.

## Fast incremental backup

Fast incremental backup - block change tracking - все блоки, которые изменены с последнего бекапа, сохранются в спец файл. - CTWR (v$block_change_tracking)

## Read-only tablespace copy

Read-only tablespace copy:
- Копия только, если ее нет
- Изменили формат доступа
- Можно использовать SKIP_READONLY

KEEP FOREVER (или до момента времени или точка востановления) - только реализуется в каталогах.

TAG - имя бекапа.

Шифрование - с помощью пароля и кошелька.

# Лекция 18

## Restore Recover

Restore - восстановление файла из резервной копии.   
Recover - накат информации из redo logs

Некритические потери:
- Бд может продолжнать функционировать
- Надо создать новый файл, перестроить файл (индексы на нем), произмести операцию recover
- Примеры: tempfile, indexes

## Redo log group

Состояние Redo log group:
- Current - в группу идет запись (LGWR) - пока switch не произойдет (нельзя удалить)
- Active - содержит информацию для восстановления - до выполнения чепоинта - пока все данные не запишутся в файл
- Inactive - не записывается и не требуется для восстановления

Если бд состоит только из индексов, то не надо recover - просто пересоздаем их.   

Для пересоздания файла паролей необходимо:
- Использовать утилиту orapwd (password - пароль, entries - максимальное количество пользователей с привилегией sysdba)

## Типы recovery

Complete recovery (например, если crash):
- Возвращает бд в текущее состояние (на текущий момент), включая все закомиченные изменения
- Восстанавливаются файлы данных из копии, накатывает изменения из online, archive redo log - получаем инфу о закомиченных транзакция и наоборот, открываем БД,  незакомиченные транзакции откатываются. 

Incomplete recovery:
- Возвращает бд в некий момент времени в прошлом
- Восстанавливаются файлы данных из копии, накатывает изменения из archive redo log - получаем инфу о закомиченных транзакция и наоборот, открываем БД, незакомиченные транзакции откатываются.

Способы восстановдения:
- Пользовательское восстановление (использование OS)
- Используя RMAN

## Пользовательское восстановление

При пользовательском восстановлении необходимо перед копированием файлов, если включен Archivelog, перевести табличные пространства в backup mode.   

BACKUP MODE: DML команда начинает изменение блока БД, тогда разные части блока будут записываться в разные моменты времени, и в этом режиме оракл понимает эти блоки и можно копировать файлы в режиме archive log.

Алгоритм с BACKUP MODE:
- alter tablespace users begin backup;
- copy (ручное копирование);
- alter tablespace users end backup.

CONTROLFILE может быть как IMAGE-копи или в виде трассировки (получим команду на создание контролфайла – логическая копия).  

Ручное complete db recovery:

1)	recovery восстанавливаем БД до последнего возможного SCN
2)	может быть выполнено целиком для всей БД, или для отдельного файла данных или табличного пространства
3)	требует текущего файла управления или копии
4)	требует бэкап всех восстанавливаемых файлов
5)	требует всех archive log до настоящего момент.   

Определение файлов, которые требуют восстановления: `select file#, error from v$recover_file;`.   
Определение archive log, которые требуют восстановления: `select archive_name from v$recovery_log;`.  
Применение информации из redo log:
`recover automatic from ‘альтернативное расположение восстановленных archive log’  database/tablespace/datafile;`.   

Ручное incomplete db recovery необходимо в следующих ситуациях:
1)	хотите откатить бд на момент в прошлом, до того, как была сделана ошибка пользователем или администратором;
2)	база содержит разрушенные блоки после попытки recovery;
3)	не можете выполнить complete recovery, потому что некоторые redo log файлы утеряны 
4)	создание тестовой бд, которая соответствуют состоянию бд в прошлом;
5)	один или более unarchive redo log файлов потеряны.   

Методы incomplete db recovery: по времени, SCN, cancel – во время recovery.

Recovering Read-only табличного пространства:
1) НЕ нужен backup mode для создания копии этих файлов; 
2) НЕ нужно делать их offline.   

Recovering NOLOGGING – нет записи в redo log - объектов БД. 
/*+ append */ - вставка без логирования, выше high watermark

# Лекция 19

## Восстановления в режиме ARCHIVELOG и без него

RMAN RESTORE – восстанавливает фалы БД из бэкапа.   
RMAN RECOVER – накат данных, используя инкрементальные бэкапы и redo log файлы.    

**Все ниже в РЕЖИМЕ ARCHIVELOG**

Некритичные файлы: применяем restore + recover.   
Complete recovery: loss System-critical data file in archivelog - для табличных пространств system / undo.

Восстановление images копии. rman может восстановить, используя инкрементальные бэкапы:
- можно обновить image копии на основе инкрементального бэкапа;
- использование инкрементальных бэкапов уменьшает время восстановления;
- не требуется заново делать image копию после инкрементального восстановления.

**Fast switch** (быстрое переключение с файлов данных на image copy) 
1)	текущие файлы данных переводим в оффлайн;
2)	switch datafile ‘name’ to copy; (переключение на копию)
3)	recover файла данных
4)	перевод файлов в режим онлайн.  
опционально:
5)	создание image копии в оригинальной локации
6)	перевод файлов данных в оффлайн
7)	switch обратно
8)	recover файлов данных
9)	перевод файлов данных в онлайн.

**set newname** для команды switch.

**Все ниже в РЕЖИМЕ NOARCHIVELOG**

При потере файла данных:
1)	закрыть экземпляр;
2)	восстановить полностью БД из бэкапа;
3)	открыть БД.   

Пользователь должен сам сделать все изменения с последнего бэкапа.   
Создания restore point – задание имени какому-то моменту времени. ТОЛЬКО ПРИ ИСПОЛЬЗОВАНИИ С КАТАЛОГОМ

Серверное incomplete recovery - использование rman.   
Использование инкрементальных бэкапов – ограниченное восстановление в режиме noarchivelog (восстановление без информации из archive log, будут использоваться инкрементальные бэкапы).    

Preparing to restore DB to new host:
-	записать dbid
-	скопировать pfile на новый хост;
-	проверьте, что бэкапы, включая control file autobackup, доступны на восстанавливаемом хосте.

Performing Disaster Recovery – аварийное восстановление. Требует: бэкапы файлов данных, archived redo logs файлы, один control file autobackup.

# Лекция 20

## Дубликат БД

DUPLICATE – для создание дубля БД, используя бэкапы БД и archive redo log. Может быть нужен для:
- тест бэкапов и восстановление;
- восстановление объектов, которые были неправильно удалены из целевой БД, путем экспорта и импорта в целевую БД.   

Определение параметров: 
- db_name должны быть отличные, если находится в том oracle_home; db_block_size – то же значение. что и у целевой БД (определяет размер блока пространства system, который нельзя изменить).

Параметры для управления именованием файлов: 
- control_files;
- db_file_name_convert – преобразование файлов данных;
- log_file_name_convert – преобразование redo log file.

Бекапы целевой бд должны быть доступны для дубликата.  
Бекапы могут быть комбинацией полных и инкреиментальных.  
Архивные журналы повторов целевой бд должны быть доступны для дубликата.  
Архивные журналы могут быть на media manager, image copy, текущие archive redo logs.

Какие операции выполняются при клонировании:
1)	создание control file для клона;
2)	restore целевые файлы данных на клоне;
3)	выполняет incomplete recovery, используя всевозможные инкрементальные бэкапы и archive redo log;
4)	закрывает и открывает клона;
5)	открывает клона с опцией resetlogs;
6)	создает файлы redo log;
7)	генерирует новый dbid для клона.

skip readonly – исключение read-only табличных пространств.   
skip tablespace – перечислить табличные пространства, которые не должны попасть в клона (нельзя исключить system и undo).   
nofilenamecheck – отключение проверки имен файлов (при создании клона на одном и том же хосте, когда одинаковые файлы или redo log. будет ошибка).   
open restricted – БД открыта в режиме restricted session (только для отдельных пользователей).   

## Dispatcher

Dispatcher - процесс, позволяющий многим клиентам подключаться к одному и тому же серверу без необходимости выделенного процесса для каждого клиента. Диспетчер обрабатывает и направляет несколько входящих запросов на сетевые сеансы к процессам общего сервера.


# Лекция 21

## TSPITR

TSPITR - восстановление табличного на заданный момент времени:
- Быстро восстановить одно или более табличных пространств 
- Не влияет на состояние других таюличных пространств

Target time - точка во времени или SCN, на которое булет восстановлено табличное прсотраноство.    
Recovery set - файлы данных, которые образуют табличные пространства, которые будут восстанавливаться.    
Auxiliary - файлы данных для выполнения TSPITR - доп файлы:
- System tablespace
- Ubdo segments
- Временное табличное пространство
Auxiliary destination - пространство на диске для хранения данных.   

Когда использовать:
- Для восстановления при выполнениии truncate table
- Логические ошибки в таблице (метаинфа в словаре данных)
- Отмена действий при выполнении пакетных операций или DML команд, которые действовали на часть бд (отдельные tablespaces)
- Для восстановления на момент времени в прошлом

Нельзя использовать:
- Для удаленного табличного пространства
- Переименнового табличного пространства перед тем, как оно было переименновано

Подготовка:
- Определить момент времени
- Набор файлов для восстановления
- Определить объекты, которые будут утеряны, чтобы заранее сохранить

Ограничения:
- Не можем делать операцию второй раз пока используется recovery catalog
- Выполнить операцию и передем tablespace в онлайн, не можем использовать бекап для более раннего момента времени
- Для определения времени восстановления можно использовать Flashback query, transaction query, version query.

TS_PITR_Check - определяет наличие связей между табличными пространствами.  
Если есть связь:
- Добавить зависимое табличное пространство к восстанавливаемым
- Отказаться от связи на время операции
- Удалить связь

TS_PITR_OBJECTS_TO_BE_DROPPED - определяет объекты, которые будут утеряны.   
Export и Import - для бекапа и восстановления потерянных объектов.  

Fully automated:
- Опеределение расположения доп экземпляра
- RMAN управляет операцией

Customized with automatic auxiliary instance:
- Кастомное расположение файлов
- Указываем параметры инициализации
- Указываем параметры конфигурации каналов
- Базируется на fully

TSPITR with own auxiliary instance:
- Сами управляем доп инстансом

Пример работы RMAN: 
- создаем доп инстанс
- переводим табл пространства в offline
- выполняем restore control file для доп инстанса
- выполняем restore data fiels из recovery set и auxiliary set для доп инстанса
- выполняем recover и открываем доп инстанс с resetlogs.
- экспорт метаинфы из доп инстанса
- выключение доп инстанса
- импорт словаря метаданных в целевую бд
- удаление доп файлов


# Лекция 22

Параллелизация Backup sets - PARALLELISM - установить значение больше 1:
- Создание нескольких каналов и сопоставление им файлов данных для 
- Нельзя распределить один backup set на несколько каналов

## Monitoring RMAN

Monitoring RMAN:
- V$SESSION V$PROCESS - утсановить связь между сессиями и каналами RMAN: p.addr = s.paddr
- Для отслеживания нескольких сессий можно воспользоваться SET COMMAND ID для остлеживания конкретной сессии, чтобы скореллировать процесс и канал во время бекапа
- SPID колонка из V$PROCESS указывает ID потока (WINDOWS), ID процесса (UNIX)
- Процесс - ресурс, используемый сессией
- PADDR - адрес процесса, которым владеет сессия
- SID - номер сессии
- CLIENT_INFO - rman channel='ORAC_SDT_TAPE_1'

Мониторинг долгих задач:
- V$SESSION_LONGOPS - определяет статус операций с временем более 6с
- Detail rows - описывают файлы, обрабатываемые на одном шаге работы - более гранулированы
- Aggregate rows - файлы, обрабатываемые на всех шагах
- Job step - создание или восстановление backup set или копии файла данных

Колонки V$SESSION_LONGOPS:
- SOFAR:
	- Для image copy - число блоков, которое было прочитано
	- Для backup input rows - число блоков, которое было прочитано перед бекапом
	- Для backup output rows - число блоков, которые были записаны в backup piece
	- Для восстановления - число блоков, которое было обоработано во время восстановления файлов
	- Для proxy copy - число файлов, которые были скопировано
- TOTALWORK:
	- Для image copy - число блоков в файле
	- Для backup input rows - число блоков, которое было должно быть прочитано
	- Для backup output rows - число блоков, которое должно быть записано (0)
	- Для восстановления - число блоков, которое должно быть обоработано во время восстановления файлов
	- Для proxy copy - число файлов, которые должно быть скопировано
- OPNAME:
	- Текстовое описание строки - RMAN: datafile copy | datafile backup | full datafile retore
- CONTEXT:
	- Для выходных строк бекапов - значение 2, для остальных - 1, кроме proxy copy - ничего.    

## Опция DEBUG

Опция DEBUG `rman target ... debug | run { debug on; ... debug off; }` :
- Используется для просмотра работы PL/SQL и определения зависания команды RMAN
- Можно задать эту опцию в строке ввода команды RMAN или в RUN блоке
- DEBUG - создает ограмное количество данных, которую лучше перенаправлять в trace файл (без ошибок - 516 кбайт).    

TUNING - настройка RMAN:
- Настройка RMAN - регулировка затрат на backup и recovery: определить, что важнее backup (read/write data) или restore(copy/validating blocks)

## ОПЕРАЦИИ ВВОДА и ВЫВОДА

ОПЕРАЦИИ ВВОДА/ВЫВОДА:
- Два вида буффера - выделяются в SGA:
	- Дисковый и внешнее хранилище (обычно при работе с tape)
- Когда выполняется backup - RMAN читает файлы используя дисковые буфферы, а записывается выходные файлы бекапа используя любой из двух.
- Когда выполняется восстановление - наоборот
- Ввод/вывод синхронный:
	- Одна задача в один момент: либо backup, либо restore.
- Ввод/вывод асинхронный:
	- Несколько задач в один момент

## Мультиплекирование

RMAN мультиплекирование:
- Определяет, как RMAN распределяет дисковые буфферы
- Multiplexing - количество файлов бекапа, которые читаются одновременно и затем записываются в один и тот же backup piece
- Степень мультиплекирования зависит от параметра: FILESPRESET (default 8) - параметр команды BACKUP - max количество файлов на set, MAXOPENFILES - в conigurate channel / allocate channel.
- Для записи каждый канал выделяет 4 буффера по 1 мб. Эти буффера выделяются из PGA, пока DBWR_IO_SLAVES не равен нулю, иначе из SGA

TAPE BUFFERS:
- Выделяется 4 буффера на канал для записи на TAPE (или чтения при восстановлении) по 256 кбайт.
- Если BACKUP_TAP_IO_SLAVES установлен в true, то буфферы беруться из SGA (Shared Pool / Large Pool - если выделен)
- Если BACKUP_TAP_IO_SLAVES установлен в false, то буфферы беруться из PGA

Для мониторинга могут использоваться представления:
- V$BACKUP_SYNC_IO - обратимся к колонке DISCRETE_BYTES_PER_SECOND для просмтора скорости ввода/вывода - сравниваем с максимумом, если меньше, то настраиваем
- V$BACKUP_ASYNC_IO - файл, который имеет большое отношение LONG_WAITS к IO_COUNT узкое место:
	- IO_COUNT - число I/O операций примененных к файлу
	- LONG_WAITS - количество раз, когда backup/restore говорил ОС, что ждет завершения ввода/вывода
	- Время ожидания должно быть равно 0
	
## Скорость бекапа на TAPE

Скорость бекапа на TAPE:
- Скорость записи без сжатия
- Уровень сжатия (чем больше, тем больше скорость)
- Скорость перемотки устройства
- Физический размер блока - объем данных записываемый за одну операцию записи (чем больше, тем больше скорость) - BLKSIZE (около 1 мб)
- Скорость сети, если запись через сеть

Правила повышение просиводительности TAPE:
- Увеличение числа tape
- Один канал - они девайс
- Делать копии файлов в один бекап сет, если они нужны оба
- Делать меньше FILESPERSET

Настройка каналов `configure channel | allocate channel` :
- Предельный размер backup piece - MAXPIECESIZE
- Предохранять RMAN от большой пропускной способности - настраивать параметр Rate
- Уровень мультиплекирования для каждого канала
- Сконфигурировать несколько дисков для повышения I/O
- Сконфигурировать множественные каналы для SBT девайса

## Настройка команды BACKUP

Настройка команды BACKUP:
- MAXPIECESIZE - предельный размер backup piece
- FILESPERSET - предохранять RMAN от чтения с множества дисков за раз
- MAXOPENFILES - не считывалось много файлов за раз для tape
- BACKUP DURATION - уменьшение числа нагрузки при выполнении backup операции:
	- Определить самое короткое время, которое надо для backup. Потом его использовать для задания временного окна:
		- MINIMIAZE TIME: бекап выполняется как можно быстрее
		- MINIMIZE LOAD: время бекапа распределяется по всему временному интервалу - уменьшение нагрузки на систему

Опция VALIDATE - определение узких мест при записи на tape или диск.

# Лекция 23

## Automatic diagnostic repository

Automatic diagnostic repository - набор файлов, спец. образом структурированный - V$DIAG_INFO / утилита ADRCI:
- trace файлы
- dump инцедентов
- alert log
- отчеты Health monitor.     
    
Каждый инстанс хранит инфу в своей папке.      
Корневая папка определяется в DIAGNOSTIC_DEST:
- Если в нем null, то устанавливается ORACLE_BASE, иначе ORACLE_HOME/log.    

Структура репозитория (иерархия):
- ADR Base
- diag
- rdbms
- DB Name
- SID -- ADR Home
- alert, sdump, trace ...    

*ADRCI* - обеспечение взаимодействия с репозиторием ADR (не требуется особых прав).   

V$DIAG_INFO - представление ADR (пути к папкам в полях):
- ADR Base
- ADR Home
- diag trace
- diag alert
- diag incident
- ...    

## Проблема

*Проблема* - критическая ошибка в БД, идентифицируется ID и ключом - набор атрибутов для описания ошибки.  
Ключ включает в себя - ORA номер ошибки, значение параметров ошибки и др.   

## Инцидент

*Инцидент* - единичное проявление проблемы, идентифицируется ID и является уникальным.   
При инциденте:
- делается отметка в alert log
- собирается диагностическую информацию об инциденте
- задает теги
- сохраняет информацию в спец. директории в ADR.    

Инцидент соответствует одной проблеме. Так как информации много, то не все инциденты сохраняются - не более 5 dump'ов в час - называется потоковым управлением.   

## HEALTH MONITOR

*HEALTH MONITOR*
Проверяет различные компоненты БД, включая файлы, память, метаданные, транзакции и пользовательским действиям.   
Генерирует отчеты.   
Запускается автоматически при критической ошибки, можно запустить DBA с помощью пакета DBMS_HM.   
Может осуществляться в online / offline (nomount) режиме БД.
Информацию о проверках можно посмотреть в V$HM_CHECK.

# Лекция 24

## Corrupted blocks

Разрушенные (corrupted) data блоки - блоки, которые находятся в не распознаваемом формате оракла или содержимое блока внутренне несогласовано. Обычно возникает из-за аппаратной части или ОС.    
Два вида:
- Логическое - внутренняя ошибка оракла, информация несогласованная в блоке - информация в заголовке не сходится с реальностью.
- Media - формат блока некореектен и нельзя прочитать информацию. Можно восстановить или удалить объект с этим блоком.   

При чтении / записи блока проверяется:
- Версия блока
- Адрес блока данных (номер файла и блока) в кэше и в буффере блоков на диске
- Контрольная сумма для блока.   

## DB_BLOCK_CHECKING

Можно проверить ошибку: dbverify или проверить в log файлах (alert).   
*DB_BLOCK_CHECKING* - проверка блоков в реальном времени. Всегда включен для system tablespace.  
Значения:
- OFF - блоки не проверяются нигде кроме system
- LOW - проверяется заголовки блоков, когда блок изменился в памяти
- MEDIUM - всё кроме индексно-организованных таблиц + LOW
- FULL - проверка индексных блоков + MEDIUM.  

V$DATABASE_BLOCK_CORRUPTION - отображение блоков, помеченных как разрушенные.    

*SYSTEM DEFERRED* - параметр установится для новой сессии, перезагрузка не требуется.    

