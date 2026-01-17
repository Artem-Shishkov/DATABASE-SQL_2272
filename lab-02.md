# Лабораторная работа 2. Инсталляция БД на сервере
## Цель: Практическое развертывание базы данных и работа с SQL.
## Таблица class
```sql
CREATE TABLE IF NOT EXISTS shishkov_02272.class
(
    class_id integer NOT NULL DEFAULT nextval('shishkov_02272.class_class_id_seq'::regclass),
    name character varying(255) COLLATE pg_catalog."default" NOT NULL,
    price numeric(10,2) NOT NULL,
    CONSTRAINT class_pkey PRIMARY KEY (class_id)
)

TABLESPACE pg_default;

ALTER TABLE IF EXISTS shishkov_02272.class
    OWNER to student;
```
## Таблица company
```sql
CREATE TABLE IF NOT EXISTS shishkov_02272.company
(
    company_id integer NOT NULL DEFAULT nextval('shishkov_02272.company_company_id_seq'::regclass),
    contact character varying(255) COLLATE pg_catalog."default" NOT NULL,
    CONSTRAINT company_pkey PRIMARY KEY (company_id)
)

TABLESPACE pg_default;

ALTER TABLE IF EXISTS shishkov_02272.company
    OWNER to student;
```
## Таблица flight
```sql
CREATE TABLE IF NOT EXISTS shishkov_02272.flight
(
    flight_id integer NOT NULL DEFAULT nextval('shishkov_02272.flight_flight_id_seq'::regclass),
    company_id integer NOT NULL,
    flight_date date NOT NULL,
    from_city character varying(255) COLLATE pg_catalog."default" NOT NULL,
    to_city character varying(255) COLLATE pg_catalog."default" NOT NULL,
    CONSTRAINT flight_pkey PRIMARY KEY (flight_id),
    CONSTRAINT flight_company_id_fkey FOREIGN KEY (company_id)
        REFERENCES shishkov_02272.company (company_id) MATCH SIMPLE
        ON UPDATE NO ACTION
        ON DELETE CASCADE
)

TABLESPACE pg_default;

ALTER TABLE IF EXISTS shishkov_02272.flight
    OWNER to student;
```
## Таблица passenger
```sql
CREATE TABLE IF NOT EXISTS shishkov_02272.passenger
(
    passenger_id integer NOT NULL DEFAULT nextval('shishkov_02272.passenger_passenger_id_seq'::regclass),
    name character varying(255) COLLATE pg_catalog."default" NOT NULL,
    surname character varying(255) COLLATE pg_catalog."default" NOT NULL,
    date_of_birth date NOT NULL,
    CONSTRAINT passenger_pkey PRIMARY KEY (passenger_id)
)

TABLESPACE pg_default;

ALTER TABLE IF EXISTS shishkov_02272.passenger
    OWNER to student;
```
## Таблица ticket
```sql
CREATE TABLE IF NOT EXISTS shishkov_02272.ticket
(
    ticket_id integer NOT NULL DEFAULT nextval('shishkov_02272.ticket_ticket_id_seq'::regclass),
    passenger_id integer NOT NULL,
    flight_id integer NOT NULL,
    class_id integer NOT NULL,
    CONSTRAINT ticket_pkey PRIMARY KEY (ticket_id),
    CONSTRAINT ticket_class_id_fkey FOREIGN KEY (class_id)
        REFERENCES shishkov_02272.class (class_id) MATCH SIMPLE
        ON UPDATE NO ACTION
        ON DELETE RESTRICT,
    CONSTRAINT ticket_flight_id_fkey FOREIGN KEY (flight_id)
        REFERENCES shishkov_02272.flight (flight_id) MATCH SIMPLE
        ON UPDATE NO ACTION
        ON DELETE CASCADE,
    CONSTRAINT ticket_passenger_id_fkey FOREIGN KEY (passenger_id)
        REFERENCES shishkov_02272.passenger (passenger_id) MATCH SIMPLE
        ON UPDATE NO ACTION
        ON DELETE CASCADE
)

TABLESPACE pg_default;

ALTER TABLE IF EXISTS shishkov_02272.ticket
    OWNER to student;
```
## Вставка данных в таблицы (по 4 строки в каждую таблицу):
```sql
-- Вставка в company 
INSERT INTO shishkov_02272.company (company_id, contact) VALUES 
(1, 'contact@aeroflot.ru'),
(2, 'support@s7.ru'),
(3, '6-7answers.ru'),
(4, 'sigmamail.ru');
-- Вставка в passenger 
INSERT INTO shishkov_02272.passenger (passenger_id, name, surname, date_of_birth) VALUES 
(1, 'Иван', 'Петров', '1990-05-15'),
(2, 'Мария', 'Сидорова', '1985-12-20'),
(3, 'Алексей', 'Иванов', '1995-03-10'),
(4, 'Артём', 'Шишков', '2006-08-18');
-- Вставка в class 
INSERT INTO shishkov_02272.class (class_id, name, price) VALUES 
(1, 'Economy', 15000.00),
(2, 'Business', 50000.00),
(3, 'Premium', 75000.00),
(4, 'First', 100000.00);
-- Вставка в flight 
INSERT INTO shishkov_02272.flight (flight_id, company_id, flight_date, from_city, to_city) VALUES 
(1, 1, '2025-10-01', 'Москва', 'Лондон'),
(2, 2, '2025-10-05', 'Новосибирск', 'Рим'),
(3, 3, '2025-10-05', 'Санкт-Петербург', 'Париж'),
(4, 4, '2025-11-09', 'Иркутск', 'Токио');
-- Вставка в ticket 
INSERT INTO shishkov_02272.ticket (ticket_id, passenger_id, flight_id, class_id) VALUES 
(1, 4, 4, 3),  
(2, 1, 3, 2),  
(3, 3, 2, 4),  
(4, 2, 1, 1); 
```
## Выполнение содержательных SELECT-запросов с JOIN
* Пример 1: JOIN passenger, ticket, class (3 таблицы) — пассажиры с классами и ценой:
```sql
SELECT 
    p.name,
    p.surname,
    cl.name AS class_name,
    cl.price
FROM shishkov_02272.passenger p
JOIN shishkov_02272.ticket t ON p.passenger_id = t.passenger_id
JOIN shishkov_02272.class cl ON t.class_id = cl.class_id
ORDER BY cl.price DESC;
```
* Пример 2: JOIN passenger, ticket, flight (3 таблицы) — список пассажиров с их рейсами:
```sql
SELECT 
    p.name || ' ' || p.surname AS full_name, f.flight_date,
    f.from_city || ' → ' || f.to_city AS route
FROM shishkov_02272.passenger p
JOIN shishkov_02272.ticket t ON p.passenger_id = t.passenger_id
JOIN shishkov_02272.flight f ON t.flight_id = f.flight_id
ORDER BY f.flight_date;
```
