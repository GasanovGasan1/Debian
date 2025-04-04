# Отчет по лабораторной работе №1 базовая настройка PostgreSQL на Debian
### Гасанов Г.Ю. ИС-21

### 1.  Подготовка среды

1.1  Обновляем список доступных пакетов из репозиториев: 
>```sql
> apt-get update;
>```
1.2  Устанавливаем обновления для всех имеющихся пакетов:
>```sql
>apt-get upgrade
>```


### 2.  Установка PostgreSQL
2.1  Устанавливаем PostgreSQL: 
>```bash
>apt-get install postgresql


### 2.2  Устанавливаем клиентский пакет: 
> ```sql
> apt-get install PostgreSQL-client
> ```


### 2.3  Проверяем статус службы: 
>```bash
>systemctl status PostgreSQL
>```


### 3. Создание служебной учётной записи
3.1  Выводим содержимое файла `/etc/passwd`, где хранятся сведения о
    пользователях системы: 
>```bash 
> cat /etc/passwd
>```


3.2  Для администрирования баз данных переходим на учётную запись
    администратора: 
>```bash
>sudo -i -u postgres
>``` 

3.3  Запускаем оболочку PostgreSQL для выполнения SQL-запросов: 
>```bash
>psql
>```
    
3.4  Выходим из psql (завершаем работу оболочки): 
>```bash
>\q
>``` 

3.5  Выходим из учётной записи администратора БД и возвращаемся к
    обычному пользователю: 
>```bash
>exit
>```


### Пользователь postgres в системе:

-  Не имеет привилегий `root`, но управляет `PostgreSQL`.
-  Обладает полными правами на базы данных `PostgreSQL`.
-  Не может использовать `sudo`, так как это обычный системный
   пользователь.
-  Может запускать psql без пароля, так как является владельцем сервера БД.


### 4.  Первичная настройка конфигурационных файлов

4.1  Открываем каталог PostgreSQL версии 15 и просматриваем его содержимое: 
>```bash
>ls /etc/postgresql/15
>``` 

4.2  Открываем каталог `main`, чтобы проверить, какие файлы находятся внутри: 
>```bash
>sudo ls /etc/postgresql/15/main
>```


4.3 Редактируем конфигурационный файл `PostgreSQL`, изменяя порт с 5432 на 5433:
>```sql
>sudo nano /etc/postgresql/15/main/postgresql.conf
>```

4.4 Перезапускаем PostgreSQL, чтобы изменения вступили в силу: 
> ```sql
> sudo systemctl restart postgresql
>```


### Основные файлы конфигурации:
`postgresql.conf` - Основные настройки сервера. Этот файл управляет параметрами работы PostgreSQL, такими как:

>-   порт (`port`)
>-   логирование (`logging_collector`, `log_statement`)
>-   настройки памяти (`shared_buffers`, `work_mem`)
>-   количество подключений (`max_connections`)
>-   параметры сетевого доступа (`listen_addresses`)

`pg_hba.conf` - настройки аутентификации. Этот файл определяет, какие
пользователи могут подключаться, с каких адресов и с каким методом
аутентификации.

`pg_ident.conf` - связывает системных и базовых пользователей. Этот файл
позволяет привязать системные учётные записи Linux к пользователям
PostgreSQL

### 5.  Управление сервисом

-   Проверить статус:
>```bash
> systemctl status postgresql
>```
-   Запустить сервер: 
>```bash
> sudo systemctl start postgresql
>```
-   Остановить сервер:
>```bash
> sudo systemctl stop postgresql
>```
-   Перезапустить сервер:
>```bash
> sudo systemctl restart postgresql
>```
-   Добавить в автозапуск:
>```bash
> sudo systemctl enable postgresql
>```
>


### 6.  Создание тестовой базы данных

6.1  Запускаем PostgreSQL: 
>```bash\
>psql
>```

6.2 Создаём пользователя с паролем: 
>```sql
>CREATE USER tsimbaliukA WITH PASSWORD '1234';
>```

6.3 Создаём базу данных и назначаем владельца: 
>```sql
>CREATE DATABASE gasanov_db OWNER algasanov;
>```

6.4 Выводим список пользователей: 
>```bash
>\du
>```

6.5 Выводим список баз данных: 
>```bash
>\l
>```


6.6 Входим в базу данных под созданным пользователем:
>```sql
> psql -U algasanov -d gasanov_db -W;
>```
>

6.7 Получаем ошибку: `FATAL: Peer authentication
    failed for user "algasanov"`. Это происходит из-за одноранговой
    аутентификации (`peer`), которая требует, чтобы имя пользователя в
    системе `Linux` совпадало с именем пользователя в `PostgreSQL`.

6.8 Изменяем метод аутентификации с `peer` на `md5` в файле
    `pg_hba.conf`, чтобы разрешить вход по паролю: 
> ```sql
> sudo nano /etc/postgresql/15/main/pg_hba.conf
> ```


6.9 Находим строку, относящуюся к локальным подключениям, и заменяем peer на md5:

6.10 Сохраняем изменения и перезапускаем PostgreSQL: 
> ```sql
> sudo systemctl restart postgresql
>```


### 7.  Знакомство со схемами

7.1 Создаём схему test_schema: 
>```bash
>CREATE SCHEMA test_schema
>```

7.2 Даём пользователю `algasanov` права на использование схемы: 
>```sql
>GRANT USAGE ON SCHEMA test_schema TO algasanov;
>```

7.3 Разрешаем пользователю algasanov создавать объекты в схеме:
>```sql
>GRANT CREATE ON SCHEMA test_schema TO algasanov;
>```

7.4 Пробуем выбрать данные из test_schema.public: 
>```sql
>SELECT * FROM test_schema.public;
>```

7.5 Создаём таблицу test_table в test_schema: 
>```sql
> CREATE TABLE test_schema.test_table (id SERIAL PRIMARY KEY, name TEXT NOT NULL);
>```

7.6 Просматриваем содержимое test_table: 
> ```sql
> SELECT * FROM test_schema.test_table;
> ```


В `PostgreSQL` схема - это логическая структура внутри базы данных,
которая группирует объекты, такие как таблицы, представления, индексы,
функции и т. д. Схема в `PostgreSQL` похожа на папку внутри базы данных.
Она позволяет организовывать объекты и управлять доступом к ним.

Допустим есть база данных `company_db`. Внутри неё можно создать разные
схемы:

>- `public` - стандартная схема, куда по умолчанию попадают все объекты.
>- `sales` - таблицы, связанные с продажами.
>- `hr` - таблицы, связанные с персоналом.

Вместо создания отдельных баз данных для разных отделов, можно
использовать схемы, что упрощает управление и доступ к данным.

### 8. Использование утилиты psql для базовых операций

-- 1.1. Создаём таблицу в схеме public
CREATE TABLE public.test_table (
    id SERIAL PRIMARY KEY,
    name TEXT,
    age INTEGER
);

-- 1.2. Вставляем несколько записей
INSERT INTO public.test_table (name, age) VALUES
('Alice', 30),
('Bob', 25),
('Charlie', 35);

-- 1.3. SELECT: выводим все записи
SELECT * FROM public.test_table;

-- 1.4. UPDATE: изменяем значение (например, возраст для Alice)
UPDATE public.test_table
SET age = 31
WHERE name = 'Alice';

-- 1.5. DELETE: удаляем запись (например, Bob)
DELETE FROM public.test_table
WHERE name = 'Bob';

-- Проверяем изменения
SELECT * FROM public.test_table;

-- ========================================================
-- Часть 2. Работа в схеме test_schema
-- ========================================================

-- Предполагаем, что схема test_schema уже создана.
-- Если нет, её можно создать командой:
-- CREATE SCHEMA test_schema;

-- 2.1. Создаём таблицу в схеме test_schema с явной привязкой к схеме
CREATE TABLE test_schema.another_table (
    id SERIAL PRIMARY KEY,
    description TEXT
);

-- 2.2. Вставляем несколько записей в таблицу test_schema.another_table
INSERT INTO test_schema.another_table (description) VALUES
('Первый тестовый запрос'),
('Второй тестовый запрос');

-- 2.3. Обращение к таблице в схеме test_schema
-- Явное указание схемы:
SELECT * FROM test_schema.another_table;

-- Дополнительно:
-- Чтобы не указывать схему каждый раз, можно изменить параметр search_path.
-- Например, сначала ищем в test_schema, затем в public:
SET search_path TO test_schema, public;

-- Теперь можно обращаться к таблице another_table без указания схемы:
SELECT * FROM another_table;



### 9.  Настройка локальных и сетевых подключений

1. Изменение файла postgresql.conf
Нужно открыть  файл (путь может отличаться, например, для Ubuntu/Debian):

bash
Копировать
sudo nano /etc/postgresql/14/main/postgresql.conf
 строку с параметром listen_addresses  изменить её следующим образом:

listen_addresses = '*'

2. Изменение файла pg_hba.conf

sudo nano /etc/postgresql/14/main/pg_hba.conf
Нужно добавить  строку, разрешающую подключения с  локальной сети (например, подсеть 192.168.1.0/24):

host    all    all    192.168.1.0/24    md5


3. Перезапуск PostgreSQL
Примените изменения, перезапустив сервер PostgreSQL:

sudo systemctl restart postgresql

4. Настройка подключения в pgAdmin или DBeaver
При создании нового подключения укажите следующие параметры:

Хост (Server): IP-адрес сервера PostgreSQL (например, 192.168.1.10).

Порт: 5432

База данных: gasanov_db

Пользователь: (например, algasanov или postgres)

Пароль: (укажите пароль для пользователя)


### 10. Журналирование (logging)

10.1  Изменили настройки журналирования в `postgresql.conf`. Включили `logging_collector = on`, задали каталог логов (`pg_log`), формат имени файла и параметры логирования (`log_statement = 'all'`, `log_connections = on`, `log_disconnections = on`, `log_duration = on`).
10.2  Перезапустили сервис PostgreSQL.
10.3  Проверили, что сервис работает.
10.4  Нашли файлы логов в Debian и посмотрели их список: 
>```bash
>ls /var/lib/postgresql/15/main/pg_log/
>```

10.5  Открыли лог и проверили записи: 
Пример вывода из лога может выглядеть так:
2025-03-30 10:15:32.123 UTC [23456]: [1-1] user=postgres,db=gasanov_db LOG:  statement: CREATE TABLE test (id SERIAL);
2025-03-30 10:15:32.125 UTC [23456]: [2-1] user=postgres,db=gasanov_db LOG:  duration: 1.234 ms  sta


В логах видно SQL-запросы, подключения, отключения и время выполнения
запросов. Это подтверждает, что журналирование работает и новые записи
появляются.

### 11. Назначение ролей и прав

### 1. Создание ограниченной роли limited_user

```sql
-- Создать роль без возможности создавать базы и без суперправ
CREATE ROLE limited_user WITH LOGIN PASSWORD '1234' NOSUPERUSER NOCREATEDB NOCREATEROLE;
```

### 2. Создание тестовой базы или использование существующей


CREATE DATABASE test_db;
GRANT CONNECT ON DATABASE test_db TO limited_user;
```

### 3. Тестирование операций с limited_user

```sql
-- Подключившись к базе test_db
\c test_db limited_user

Попытка создания таблицы
CREATE TABLE test_table (id SERIAL PRIMARY KEY, data TEXT);
```

Предоставляем права пользователю
GRANT CREATE ON SCHEMA public TO limited_user;
```

После этого limited_user сможет создавать таблицы:
```sql
CREATE TABLE public.test_table (id SERIAL PRIMARY KEY, data TEXT);
```

### 4. Пояснение выдачи прав через GRANT

- **GRANT <привилегия> ON <объект> TO <роль>**  
  Выдаёт конкретные права (например, SELECT, INSERT, UPDATE, DELETE, CREATE) на объект (таблицу, схему, базу данных) указанной роли или ролям.  
  Пример:
  ```sql
 

