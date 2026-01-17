# Лабораторная работа 2. Инсталляция БД на сервере
## Цель: Практическое развертывание базы данных и работа с SQL.
## class
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
## company
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
## flight
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
## passenger
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
## ticket
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
