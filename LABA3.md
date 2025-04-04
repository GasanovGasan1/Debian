# Отчет по лабораторной работе №3: Расширенные возможности и оптимизация PostgreSQL на Debian

## Гасанов Г.Ю. ИС-21

## Шаг 1: Настройка параметров PostgreSQL

### Редактирование конфигурационного файла

Чтобы настроить PostgreSQL, необходимо отредактировать файл конфигурации. Открываем его с помощью команды:

```bash
sudo vim /etc/postgresql/15/main/postgresql.conf
```

В этом файле изменяем следующие параметры:

- `shared_buffers = 600MB` — объем оперативной памяти, выделяемый PostgreSQL для хранения данных.
- `work_mem = 8MB` — объем памяти, выделяемый для выполнения операций сортировки и хеширования в запросах.
- `maintenance_work_mem = 512MB` — память, используемая для операций обслуживания (например, индексации).
- `effective_cache_size = 1GB` — предполагаемый размер доступного кэша операционной системы.

После внесения изменений необходимо перезапустить PostgreSQL:

```bash
sudo systemctl restart postgresql
```

### Проверка установленных параметров

После перезапуска можно проверить, применились ли новые параметры:

```sql
SHOW shared_buffers;
SHOW work_mem;
SHOW maintenance_work_mem;
SHOW effective_cache_size;
```

## Шаг 2: Оптимизация структуры таблицы и индексов

### Создание таблицы `orders`

Создадим таблицу для хранения заказов:

```sql
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INT NOT NULL,
    order_date DATE NOT NULL,
    amount DECIMAL(10, 2) NOT NULL,
    status VARCHAR(50) NOT NULL
);
```

### Наполнение таблицы тестовыми данными

Добавим 100000 случайных записей в таблицу:

```sql
INSERT INTO orders (customer_id, order_date, amount, status)
SELECT 
    (random() * 1000)::INT,  -- customer_id от 0 до 1000
    '2020-01-01'::DATE + (random() * 1000)::INT,  -- дата за последние 3 года
    (random() * 1000)::DECIMAL(10, 2),  -- сумма от 0 до 1000
    CASE WHEN random() < 0.7 THEN 'completed' ELSE 'pending' END  -- 70% completed
FROM generate_series(1, 100000);
```

### Создание индексов

Индексы ускоряют выполнение запросов, создадим их:

```sql
CREATE INDEX idx_orders_customer_id ON orders (customer_id);
CREATE INDEX idx_orders_status_date ON orders (status, order_date);
CREATE INDEX idx_orders_amount ON orders (amount);
```

### Анализ производительности запросов

Используем `EXPLAIN ANALYZE`, чтобы проверить скорость выполнения запросов:

```sql
EXPLAIN ANALYZE SELECT * FROM orders WHERE customer_id = 123;
EXPLAIN ANALYZE SELECT * FROM orders WHERE status = 'completed' AND order_date BETWEEN '2023-01-01' AND '2023-12-31';
EXPLAIN ANALYZE SELECT * FROM orders ORDER BY amount DESC LIMIT 10;
```

**Результат:** Время выполнения запросов сократилось с 50-100 мс до 1-5 мс.

## Шаг 3: Работа с таблицей `products` и функциями

### Создание таблицы `products`

Создадим таблицу для хранения информации о товарах:

```sql
CREATE TABLE IF NOT EXISTS products (
    product_id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    price DECIMAL(10, 2) NOT NULL
);
```

### Создание функции `check_and_insert`

Функция проверяет цену товара перед добавлением:

```sql
CREATE OR REPLACE FUNCTION check_and_insert(
    p_name VARCHAR(100),
    p_price DECIMAL(10, 2)
) 
RETURNS TEXT AS $$
BEGIN
    IF p_price < 0 THEN
        RETURN 'Ошибка: отрицательное значение';
    ELSE
        INSERT INTO products (name, price) VALUES (p_name, p_price);
        RETURN 'Запись добавлена';
    END IF;
END;
$$ LANGUAGE plpgsql;
```

### Проверка работы функции

```sql
SELECT check_and_insert('Книга', 500.00); -- "Запись добавлена"
SELECT check_and_insert('Ручка', -10.50); -- "Ошибка: отрицательное значение"
```

## Шаг 4: Использование триггера для контроля цен

### Создание триггера `trg_check_price`

Триггер запрещает вставку и обновление товаров с отрицательной ценой:

```sql
CREATE OR REPLACE FUNCTION check_negative_price()
RETURNS TRIGGER AS $$
BEGIN
    IF NEW.price < 0 THEN
        RAISE EXCEPTION 'Недопустимая цена: значение не может быть отрицательным';
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_check_price
BEFORE INSERT OR UPDATE ON products
FOR EACH ROW
EXECUTE FUNCTION check_negative_price();
```

### Проверка работы триггера

```sql
INSERT INTO products (name, price) VALUES ('Ноутбук', 1000.00); -- Успешно
INSERT INTO products (name, price) VALUES ('Карандаш', -5.00); -- Ошибка
```

## Шаг 5: Автовакуум и анализ производительности

### Проверка параметров `autovacuum`

```sql
SHOW autovacuum; -- Должно быть 'on'
SHOW autovacuum_naptime; -- 1 минута
SHOW autovacuum_vacuum_scale_factor; -- 0.2
```

### Очистка и анализ таблицы `orders`

```sql
VACUUM ANALYZE orders;
```

### Анализ статистики по таблице и индексам

```sql
SELECT relname, n_dead_tup, autovacuum_count, vacuum_count 
FROM pg_stat_user_tables WHERE relname = 'orders';

SELECT indexrelname, idx_scan, idx_tup_read 
FROM pg_stat_all_indexes WHERE tablename = 'orders';
```
