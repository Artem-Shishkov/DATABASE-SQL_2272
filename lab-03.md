# Лабораторная работа 3. Представления и процедуры
## Цель: Освоение механизмов абстракции данных и программных модулей.
## Задачи:
* Создание представлений для выходных документов
* Разработка хранимых процедур с параметрами
* Представление сложных запросов при помощи представления
## Представление “Отчёт по проданным билетам”, оно показывает информацию о проданных билетах: кто летит, куда, когда, какой класс и какая компания выполняет рейс.
```sql
CREATE OR REPLACE VIEW shishkov_02272.ticket_report
 AS
 SELECT t.ticket_id,
    (p.name::text || ' '::text) || p.surname::text AS passenger,
    (f.from_city::text || ' → '::text) || f.to_city::text AS route,
    f.flight_date,
    c.name AS class_name,
    c.price AS class_price,
    comp.contact AS company_contact
   FROM shishkov_02272.ticket t
     JOIN shishkov_02272.passenger p ON t.passenger_id = p.passenger_id
     JOIN shishkov_02272.flight f ON t.flight_id = f.flight_id
     JOIN shishkov_02272.class c ON t.class_id = c.class_id
     JOIN shishkov_02272.company comp ON f.company_id = comp.company_id;

ALTER TABLE shishkov_02272.ticket_report
    OWNER TO student;
```
## Посмотреть, что оно выводит можно с помощью:
```sql
SELECT * FROM shishkov_02272.ticket_report;
```
## Второе представление: “Расписание рейсов по компаниям”, оно показывает компанию, откуда и куда полет, дату полета и количество проданных билетов.
```sql
CREATE OR REPLACE VIEW shishkov_02272.flight_schedule
 AS
 SELECT comp.contact AS company_contact,
    f.flight_id,
    f.from_city,
    f.to_city,
    f.flight_date,
    count(t.ticket_id) AS sold_tickets
   FROM shishkov_02272.flight f
     JOIN shishkov_02272.company comp ON f.company_id = comp.company_id
     LEFT JOIN shishkov_02272.ticket t ON f.flight_id = t.flight_id
  GROUP BY comp.contact, f.flight_id, f.from_city, f.to_city, f.flight_date;

ALTER TABLE shishkov_02272.flight_schedule
    OWNER TO student;
```
## Посмотреть, что оно выводит можно с помощью:
```sql
SELECT * FROM shishkov_02272.flight_schedule;
```
## Хранимая процедура(функция) с параметрами - получает все билеты по фамилии пассажира.
```sql
CREATE OR REPLACE FUNCTION shishkov_02272.get_tickets_by_surname(
	surname_input character varying)
    RETURNS TABLE(passenger_name text, flight_date date, route text, class_name character varying, price numeric) 
    LANGUAGE 'plpgsql'
    COST 100
    VOLATILE PARALLEL UNSAFE
    ROWS 1000

AS $BODY$
BEGIN
    RETURN QUERY
    SELECT 
        p.name || ' ' || p.surname AS passenger_name,
        f.flight_date,
        f.from_city || ' → ' || f.to_city AS route,
        c.name AS class_name,
        c.price
    FROM shishkov_02272.ticket t
    JOIN shishkov_02272.passenger p ON t.passenger_id = p.passenger_id
    JOIN shishkov_02272.flight f ON t.flight_id = f.flight_id
    JOIN shishkov_02272.class c ON t.class_id = c.class_id
    WHERE p.surname ILIKE surname_input;
END;
$BODY$;

ALTER FUNCTION shishkov_02272.get_tickets_by_surname(character varying)
    OWNER TO student;
```
## Пример вызова:
```sql
SELECT * FROM shishkov_02272.get_tickets_by_surname('Шишков');
```
## Вторая функция - для фильтрации рейсов по городу.
```sql
CREATE OR REPLACE FUNCTION shishkov_02272.get_flights_from_city(
	city_name character varying)
    RETURNS TABLE(company_contact character varying, flight_date date, from_city character varying, to_city character varying) 
    LANGUAGE 'plpgsql'
    COST 100
    VOLATILE PARALLEL UNSAFE
    ROWS 1000

AS $BODY$
BEGIN
    RETURN QUERY
    SELECT 
        comp.contact,
        f.flight_date,
        f.from_city,
        f.to_city
    FROM shishkov_02272.flight f
    JOIN shishkov_02272.company comp ON f.company_id = comp.company_id
    WHERE f.from_city ILIKE city_name;
END;
$BODY$;

ALTER FUNCTION shishkov_02272.get_flights_from_city(character varying)
    OWNER TO student;
```
## Пример вызова:
```sql
SELECT * FROM shishkov_02272.get_flights_from_city('Москва');
```