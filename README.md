# Домашнее задание №12

Секционировать большую таблицу из демо базы flights
1. Скачиваем базу
   
>wget https://edu.postgrespro.com/demo-medium-en.zip
>

![image](https://github.com/blaidex2/Postgres_Homework-12/assets/130083589/daf863d5-5eee-489b-b57f-af956519286d)

2. Устанавливаем архиватор
   
>sudo apt install unzip
>

![image](https://github.com/blaidex2/Postgres_Homework-12/assets/130083589/4ce92dd5-1b2a-4b06-8e42-b64e8e8a2648)

3. Распаковываем файл и переместим полученный файл с базой в папку с Postgres
   
>unzip demo-medium-en.zip
>
>sudo mv demo-medium-en-20170815.sql /var/lib/postgresql/15/main/
>

![image](https://github.com/blaidex2/Postgres_Homework-12/assets/130083589/94c7de5d-acb4-459b-8610-8f745399d6b5)

4. Импортируем демо базу в Postgres
   
>sudo -u postgres psql
>
>\i /var/lib/postgresql/15/main/demo-medium-en-20170815.sql
>
![image](https://github.com/blaidex2/Postgres_Homework-12/assets/130083589/008e0887-856b-493f-b2eb-825d7b56884e)


5. Создадим секционированную таблицу flights_demo по диапазону поля scheduled_ departure со структурой, аналогичной flights, но без констрейнтов. Затем создадим на ней все констрейнты от оригинальной таблицы, кроме уникальных (иначе не получится секционировать, уникальные ограничения должны быть в ключе секционирования).
   
>create table bookings.flights_demo(like bookings.flights excluding constraints) partition by range (scheduled_departure);
>

Добавим констрейнты:

>alter table flights_demo add constraint flights_check check ((scheduled_arrival > scheduled_departure));
>
>alter table flights_demo add constraint flights_check1 check (((actual_arrival is null) or ((actual_departure is not null) and (actual_arrival is not null) and
>
>(actual_arrival > actual_departure))));
>
>alter table flights_demo add constraint flights_status_check CHECK (status::text = ANY (ARRAY['On Time'::character varying::text, 'Delayed'::character varying::text,
>
>'Departed'::character varying::text, 'Arrived'::character varying::text, 'Scheduled'::character varying::text, 'Cancelled'::character varying::text]));
>
>alter table flights_demo add constraint flights_aircraft_code_fkey FOREIGN KEY (aircraft_code) references aircrafts_data (aircraft_code);
>
>alter table flights_demo add constraint flights_arrival_airport_fkey FOREIGN KEY (arrival_airport) references airports_data (airport_code);
>
>alter table flights_demo add constraint flights_departure_airport_fkey FOREIGN KEY (departure_airport) references airports_data (airport_code);
>

![image](https://github.com/blaidex2/Postgres_Homework-12/assets/130083589/4a9cd540-94a2-434e-b664-c0f0c87d67ed)

6. Подключим таблицу flights как дефолтную секцию к новой таблице flights_demo. Переименуем таблицы. В результате получаем секционированную таблицу flights с одной секцией.
    
>alter table bookings.flights_demo attach partition bookings.flights default;
>
>alter table bookings.flights rename to flights_origin;
>
>alter table bookings.flights_demo rename to flights;
>

![image](https://github.com/blaidex2/Postgres_Homework-12/assets/130083589/50f24a28-c823-4b10-8bea-ea31fd6a3507)

7. Выполним анализ

>select DATE_PART('year', scheduled_departure) year_flight, DATE_PART('month', scheduled_departure), count(flight_id) cnt_flight from flights group by DATE_PART('year',
>
> scheduled_departure), DATE_PART('month', scheduled_departure);
>
>create table bookings.flights_201705 (like bookings.flights_origin including all);
>
>create table bookings.flights_201706 (like bookings.flights_origin including all);
>
>create table bookings.flights_201707 (like bookings.flights_origin including all);
>
>create table bookings.flights_201708 (like bookings.flights_origin including all);
>
>create table bookings.flights_201709 (like bookings.flights_origin including all);
>
>insert into flights_201705 select * from flights_origin where DATE_PART('year', scheduled_departure) = 2017 and DATE_PART('month', scheduled_departure) = 5;
>
>insert into flights_201706 select * from flights_origin where DATE_PART('year', scheduled_departure) = 2017 and DATE_PART('month', scheduled_departure) = 6;
>
>insert into flights_201707 select * from flights_origin where DATE_PART('year', scheduled_departure) = 2017 and DATE_PART('month', scheduled_departure) = 7;
>
>insert into flights_201708 select * from flights_origin where DATE_PART('year', scheduled_departure) = 2017 and DATE_PART('month', scheduled_departure) = 8;
>
>insert into flights_201709 select * from flights_origin where DATE_PART('year', scheduled_departure) = 2017 and DATE_PART('month', scheduled_departure) = 9;
>
>truncate table only bookings.flights_origin;
>
 
![image](https://github.com/blaidex2/Postgres_Homework-12/assets/130083589/55fc9ac6-6767-4605-b284-1dd33b9193cb)

8. Удалим ограничение с таблицы ticket_flights и очистим данные в дефолтной секции.
    
>\d ticket_flights
>
>alter table ticket_flights drop constraint ticket_flights_flight_id_fkey ;
>
>truncate table only bookings.flights_old;

![image](https://github.com/blaidex2/Postgres_Homework-12/assets/130083589/539e8c74-0761-42b9-a352-4845d30548b1)

9. Присоединим созданные секции к таблице flights.
    
>alter table flights attach partition flights_201705 for values from (date '2017-05-01') to (date '2017-06-01');
>
>alter table flights attach partition flights_201706 for values from (date '2017-06-01') to (date '2017-07-01');
>
>alter table flights attach partition flights_201707 for values from (date '2017-07-01') to (date '2017-08-01');
>
>alter table flights attach partition flights_201708 for values from (date '2017-08-01') to (date '2017-09-01');
>
>alter table flights attach partition flights_201709 for values from (date '2017-09-01') to (date '2017-10-01');
>

![image](https://github.com/blaidex2/Postgres_Homework-12/assets/130083589/f0ccff54-e8e6-4bae-93b5-305224499726)

>\d+ flights
>


![image](https://github.com/blaidex2/Postgres_Homework-12/assets/130083589/d81a9674-0dac-4fe9-bf00-9b101e23fcae)

