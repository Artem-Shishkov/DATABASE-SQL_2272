# Лабораторная работа 5. Триггеры и аудит
## Создание таблицы аудита
```sql
CREATE TABLE IF NOT EXISTS shishkov_02272.audit_log
(
    audit_id integer NOT NULL DEFAULT nextval('shishkov_02272.audit_log_audit_id_seq'::regclass),
    table_name text COLLATE pg_catalog."default" NOT NULL,
    operation_type text COLLATE pg_catalog."default" NOT NULL,
    operation_time timestamp without time zone NOT NULL DEFAULT now(),
    old_data jsonb,
    CONSTRAINT audit_log_pkey PRIMARY KEY (audit_id)
)

TABLESPACE pg_default;

ALTER TABLE IF EXISTS shishkov_02272.audit_log
    OWNER to student;
```
## Функция триггера для каскадного удаления для таблицы company (связь один-ко-многим - company (1) → flight (N), при удалении авиакомпании: удаляются все её рейсы; факт удаления пишется в аудит):
```sql
CREATE OR REPLACE FUNCTION shishkov_02272.cascade_delete_company()
    RETURNS trigger
    LANGUAGE 'plpgsql'
    COST 100
    VOLATILE NOT LEAKPROOF
AS $BODY$
BEGIN
    DELETE FROM shishkov_02272.flight
    WHERE company_id = OLD.company_id;

    INSERT INTO shishkov_02272.audit_log
    (table_name, operation_type, old_data)
    VALUES
    ('company', 'DELETE', to_jsonb(OLD));

    RETURN OLD;
END;
$BODY$;

ALTER FUNCTION shishkov_02272.cascade_delete_company()
    OWNER TO student;
```
## Добавление триггера в таблицу company:
```sql
CREATE TRIGGER trg_company_cascade_delete
BEFORE DELETE ON shishkov_02272.company
FOR EACH ROW
EXECUTE FUNCTION shishkov_02272.cascade_delete_company();
```
## Функция триггера для аудита изменений при добавлении, обновлении или удалении:
```sql
CREATE OR REPLACE FUNCTION shishkov_02272.audit_ticket_changes()
    RETURNS trigger
    LANGUAGE 'plpgsql'
    COST 100
    VOLATILE NOT LEAKPROOF
AS $BODY$
BEGIN
    IF (TG_OP = 'DELETE') THEN
        INSERT INTO shishkov_02272.audit_log
        (table_name, operation_type, old_data)
        VALUES
        ('ticket', 'DELETE', to_jsonb(OLD));

    ELSIF (TG_OP = 'UPDATE') THEN
        INSERT INTO shishkov_02272.audit_log
        (table_name, operation_type, old_data)
        VALUES
        ('ticket', 'UPDATE', to_jsonb(OLD));

    ELSIF (TG_OP = 'INSERT') THEN
        INSERT INTO shishkov_02272.audit_log
        (table_name, operation_type, old_data)
        VALUES
        ('ticket', 'INSERT', to_jsonb(NEW));
    END IF;

    RETURN NULL; 
END;
$BODY$;

ALTER FUNCTION shishkov_02272.audit_ticket_changes()
    OWNER TO student;
```
## Проверки:
* Добавление в ticket:
```sql
INSERT INTO shishkov_02272.ticket
(ticket_id, passenger_id, flight_id, class_id)
VALUES (30001, 1, 1, 1);
```
* Обновление в ticket:
```sql
UPDATE shishkov_02272.ticket
SET class_id = 2
WHERE ticket_id = 30001;
```
* Удаление из ticket:
```sql
DELETE FROM shishkov_02272.ticket
WHERE ticket_id = 30001;
```
* Каскадное удаление одной строки из company -> удаление всех строк с таким же company_id во flight:
```sql
DELETE FROM shishkov_02272.company
WHERE company_id = 5507;
```
* Вызов журнала аудита:
```sql
SELECT *
FROM shishkov_02272.audit_log
ORDER BY operation_time DESC;
```