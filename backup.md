# Домашнее задание к занятию "`Резервное копирование баз данных`" - `Исаков Михаил`

Задание можно выполнить как в любом IDE, так и в командной строке.

### Задание 1
Финансовая компания решила увеличить надёжность работы баз данных и их резервного копирования. 

Необходимо описать, какие варианты резервного копирования подходят в случаях: 

1.1. Необходимо восстанавливать данные в полном объёме за предыдущий день.

1.2. Необходимо восстанавливать данные за час до предполагаемой поломки.

*Приведите ответ в свободной форме.*

#### *Ответ:*

Считаю, что компании может подойти несколко способов резервного копирования. Копания финансовая, значит данные имеют высокую ценность. 

1. Полное резервное копирование.
2. Полное резервное копирование с дифференциальным бэкапом.

p/s Избыточное резервное копирование повлечет за собой требование к свободному дисковому пространству.

1.2. Необходимо восстанавливать данные за час до предполагаемой поломки.

1. Если данных не много и нагрузка на БД невысокая, то полное резервное копирование каждый день с дифференциальным бэкапом каждый час. Вариант потребует больше места для хранения.
2. Если данных много и нагрузка на БД высокая, то полное резервное копирование каждый день с инкрементным бэкапом каждый час. Для варианта потребуется меньше места для хранения.

### Задание 2

2.1. С помощью официальной документации приведите пример команды резервирования данных и восстановления БД (pgdump/pgrestore).

*Приведите ответ в свободной форме.*

#### *Ответ:*

2.1. С помощью официальной документации приведите пример команды резервирования данных и восстановления БД (pgdump/pgrestore).

Команда резервирования данных:
```bash
pg_dump sakila > sakiladb.sql
```
[Ссылка на документацию PostgreSQL, pg_dump](https://www.postgresql.org/docs/current/app-pgdump.html)

Команда восстановления БД:
```bash
pg_restore -d sakila sakiladb.sql
```
[Ссылка на документацию PostgreSQL, pg_restore](https://www.postgresql.org/docs/current/app-pgrestore.html)

---


### Задание 3

3.1. С помощью официальной документации приведите пример команды инкрементного резервного копирования базы данных MySQL. 

*Приведите ответ в свободной форме.*

#### *Ответ:*
Инкрементное резервное копирование требует создание первоначальной полной копии. Далее инкрементное резервное копирование:
```bash
mysqlbackup --defaults-file=/home/dbadmin/my.cnf \
  --incremental --incremental-base=history:last_backup \
  --backup-dir=/home/dbadmin/temp_dir \
  --backup-image=incremental_image1.bi \
   backup-to-image
```
Пример использует опцию --incremental-base=history:last_backup, которая извлекает LSN последней успешной полной или частичной резервной копии (не TTS) из mysql.backup_history таблицы и на этой основе выполняет инкрементную резервную копию.

[Ссылка на документацию MySQL](https://dev.mysql.com/doc/mysql-enterprise-backup/8.2/en/mysqlbackup.incremental.html#meb-incremental-considerations)

---