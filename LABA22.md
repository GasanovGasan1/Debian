# Отчет по лабораторной работе №2 Резервное копирование, восстановление и мониторинг в Debian и PostgreSQL
### Гасанов Г.Ю. ИС-21
2. Создание резервной копии
1. pg_dump -U algasanov -F c -b -v -f ~/backups/gasanov_db.backup gasanov_db
(бэкап всей БД)
В PostgreSQL есть две основные утилиты для резервного копирования: pg_dump и pg_basebackup. Они предназначены для разных задач и используются в зависимости от требований к бэкапу:
1.1. pg_dump:
Создаёт резервную копию отдельной базы данных в виде SQL-файла или архива. Этот файл содержит команды (CREATE, INSERT и т. д.), позволяющие воссоздать базу данных с нуля. Используется, когда:
•	нужно сохранить копию только одной базы, а не всего сервера;
•	требуется перенести базу на другой сервер (например, с одной версии PostgreSQL на другую);
•	нужно получить резервную копию в виде SQL-скрипта, который можно редактировать перед восстановлением.
1.2. pg_basebackup:
Выполняет полное копирование всего сервера PostgreSQL, включая базы данных, настройки, файлы данных и журналы транзакций. Используется, когда:
•	требуется создать полную копию сервера с сохранением всех данных;
•	нужно настроить репликацию базы данных для отказоустойчивости;
•	необходимо быстро восстановить базу в том же состоянии, в каком она была на момент создания копии;
3. Частичное (выборочное) резервное копирование
1. pg_dump -U algasanov -n public -Fc -b -v -f ~/backups/gasanov_db_public.backup gasanov_db
(бэкап только схемы public)

```bash
pg_dump: Это утилита командной строки PostgreSQL для создания резервных копий баз данных.
```

-U algasanov: Опция указывает имя пользователя ,под которым будет выполнено резервное копирование.
-n public: Опция указывает на схему, которая будет включена в резервное копирование.
-Fc: Опция указывает формат резервной копии (custom format), который обеспечивает более гибкие возможности восстановления по сравнению с текстовым форматом.
-b: Опция включает создание архива базы данных (dump of the complete transaction log), что полезно для точного восстановления в определенный момент времени.
-v: Опция включает подробный вывод (verbose), который отображает процесс создания резервной копии на экране.
-f ~/backups/gasanov_db_public.backup: Опция указывает имя и путь к файлу, в который будет сохранена резервная копия базы данных.
gasanov_db: Это имя базы данных, которая будет скопирована.
2. pg_dump -U algasanov -n public -t test_table -t test_table_new -F c -b -v -f ~/backups/gasanov_db_public_tables.backup gasanov_db
(бэкап определенных таблиц из public)

```bash
pg_dump: Та же утилита для создания резервных копий PostgreSQL.
```

-U algasanov: Имя пользователя для подключения к базе данных.
-n public: Указание на схему public для резервного копирования.
-t test_table -t test_table_new: Опция -t используется для указания таблиц, которые нужно включить в резервную копию. 
-F c: Опция указывает формат резервной копии (custom format), который обеспечивает компактный и быстрый формат для резервных копий, оптимизированный для быстрого восстановления.
-b: Включает создание архива базы данных (dump of the complete transaction log), что полезно для точного восстановления в определенный момент времени.
-v: Выводит подробную информацию о процессе на экран.
-f ~/backups/gasanov_db_public_tables.backup: Указание пути и имени файла, в который будет сохранена резервная копия с выбранными таблицами.
gasanov_db: Имя базы данных для резервного копирования.

4. Восстановление из резервной копии
1. pg_restore -U algasanov -d gasanov_db -v ~/backups/gasanov_db.backup
(восставливаем таблицу из бэкапа)
2. psql -U algasanov -d gasanov_db -c "
```sql
SELECT * FROM test_table LIMIT 10;
```
"
(проверяем)
5. Автоматизация бэкапов с помощью cron
1. Скрипт резервного копирования (`backup_script.sh`)
Этот скрипт делает бэкап базы `gasanov_db`, добавляет к имени файла текущую дату и удаляет резервные копии старше 7 дней.  
BACKUP_DIR="/path/to/backup"  # Папка для хранения бэкапов
DATABASE="gasanov_db"         # Имя базы данных
USER="algasanov"              # Пользователь PostgreSQL
TIMESTAMP=$(date +"%Y-%m-%d_%H-%M-%S")  # Метка времени
# Создание бэкапа

```bash
pg_dump -U $USER -F c -b -v -f ${BACKUP_DIR}/${DATABASE}_${TIMESTAMP}.backup $DATABASE
```

# Удаление старых бэкапов (старше 7 дней)

```bash
find ${BACKUP_DIR} -name "${DATABASE}_*.backup" -mtime +7 -exec rm {} \;
```

2. Делаем скрипт исполняемым*

```bash
chmod +x ~/backups/backup_script.sh
```

Это дает скрипту права на выполнение.
 3. Добавляем автозапуск через `crontab`
Открываем редактор `crontab`:  
Добавляем строку:  
0 2 * * * /path/to/backup_script.sh
Теперь бэкап будет выполняться **каждый день в 2:00 ночи**.
4. Проверяем задание в `crontab` 
```bash

```bash
crontab -l
```

```

6. Мониторинг состояния системы
1. Включаем расширения для мониторинга
Эти команды включают статистику активности пользователей и базы данных:  

```bash

```sql
CREATE EXTENSION 
```sql
IF NOT EXISTS pg_stat_activity;
```

```

```


```bash

```sql
CREATE EXTENSION 
```sql
IF NOT EXISTS pg_stat_database;
```

```

```

2. Настраиваем логирование в `postgresql.conf`
Добавляем или изменяем параметры:  

```bash
log_statement = 'all'       # Логировать все SQL-запросы  
```


```bash
log_connections = on        # Логировать подключения  
```


```bash
log_disconnections = on     # Логировать отключения  
```


```bash
log_duration = on           # Логировать длительность запросов  
```


```bash
log_lock_waits = on         # Логировать ожидание блокировок  
```

3. Перезапускаем PostgreSQL, чтобы изменения вступили в силу 

```bash

```bash

```bash
sudo systemctl restart postgresql
```

```

```

4. Проверяем активные соединения и статистику базы

```bash

```sql
SELECT * FROM pg_stat_activity;
```
  -- Показывает текущие подключения  
```


```bash

```sql
SELECT * FROM pg_stat_database;
```
  -- Показывает статистику по базе  
```

7. Мониторинг PostgreSQL Настройка мониторинга и логирования в PostgreSQL 
1. Включаем расширение для сбора статистики запросов  

```bash

```sql
CREATE EXTENSION 
```sql
IF NOT EXISTS pg_stat_statements;
```

```

```

2. Настраиваем сбор SQL-статистики в `postgresql.conf` 
Добавляем параметры:  

```bash
shared_preload_libraries = 'pg_stat_statements'
```


```bash
pg_stat_statements.max = 10000
```


```bash
pg_stat_statements.track = all
```

3. Перезапускаем PostgreSQL

```bash

```bash

```bash
sudo systemctl restart postgresql
```

```

```

4. Проверяем статистику запросов  

```bash
SELECT queryid, calls, total_time, min_time, max_time, mean_time, query  
```

FROM pg_stat_statements  
ORDER BY total_time DESC  
LIMIT 10;
5. Включаем логирование в `postgresql.conf`  

```bash
logging_collector = on
```


```bash
log_directory = 'pg_log'
```


```bash
log_filename = 'postgresql-%Y-%m-%d_%H%M%S.log'
```


```bash
log_truncate_on_rotation = on
```


```bash
log_rotation_age = 1d
```


```bash
log_rotation_size = 10MB
```

6. Перезапускаем PostgreSQL 

```bash

```bash

```bash
sudo systemctl restart postgresql
```

```

```

Анализ логов через rsyslog
7. Устанавливаем `rsyslog` 

```bash
sudo apt update  
```


```bash
sudo apt install rsyslog
```

8. Настраиваем `/etc/rsyslog.conf 
Добавляем:  
module(load="imfile" PollingInterval="10")
input(type="imfile"
      File="/var/lib/postgresql/12/main/pg_log/*.log"
      Tag="postgresql"
      Severity="info"
      Facility="local6")
9. Перезапускаем rsyslog и проверяем логи 

```bash
sudo systemctl restart rsyslog  
```


```bash
sudo tail -f /var/log/syslog
```

Автоматическое управление логами через logrotate
10. Создаём конфигурацию logrotate**  

```bash
sudo vim /etc/logrotate.d/postgresql
```

Добавляем:  
/var/lib/postgresql/12/main/pg_log/*.log {
    daily
    missingok
    rotate 7
    compress
    delaycompress
    notifempty
    create 0640 postgres postgres
    sharedscripts
    postrotate
        /usr/bin/invoke-rc.d rsyslog rotate > /dev/null
    endscript
}
11. Проверяем работу logrotate

```bash
sudo logrotate -d /etc/logrotate.d/postgresql
```
