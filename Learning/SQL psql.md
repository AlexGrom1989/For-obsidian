### Университетские конспекты

> [!abstract] SQL - (Structured Query Language)
> Включает в себя: 
> - DDL (Data Definition Language) - определение структур данных
> - DML (Data Manipulation Language) - управление данными
> - DQL (Data Query Language) - извлечение данных из БД

Примеры смотреть в `/home/alex/Desktop/Learn_DevOps_sql/Learn_sql_git_linux` или ниже в этом файле.

```sql
|*----------------*|
Создание и Удаление{

CREATE DATABASE ....; --=> создание бд (CREATE DATABASE shop;)
DROP DATABASE ....; --=> удаление бд (DROP DATABASE shop;)
}

|*--------*|
Типы данных{

INT
VARCHAR --=> маленький текст(255 симв)
TEXT --=> большой текст(65535 симв)
DATE
JSON
}

|*-----------------------*|
Создание/удаление таблицы {

CREATE TABLE users (
    id INT NOT NULL AUTO_INCREMENT, 	#id типа integer, отличное от пустого, автоувеличивающийся
    name VARCHAR(30),	  #максимум 30 символов
    bio TEXT,
    birth DATE,
    PRIMARY KEY(id) 	#обеспечивает уникальность полей в скобках (в данном случае id)
); --=> создание

DROP TABLE users; --=> удаление
}

|*-----------------------------------*|
Добавление/удаление столбца в таблице {

ALTER TABLE users ADD password VARCHAR (32); --=> добавление 	#добавление столбца password в таблицу users
ALTER TABLE users DROP password VARCHAR (32); --=> удаление
}

|*---------------------------------------------*|
Добавление/обновление/удаление записей в ячейках{

INSERT INTO users (name, bio, birth) VALUES
    ('Alex', 'My Way', '2005-04-04'),
    ('Alexey', 'My Way2', '2005-05-05'),
    ('Alexandr', 'My Way3', '2005-06-06'); --=> добавление
    
UPDATE users SET name = 'Maxim', bio = 'Maxim Way' WHERE id >= 2 AND name = 'Alex'; --=> обновление

ALTER TABLE users CHANGE birth birth2 DATE NOT NULL; --=> изменение столбца и его параметров 	#birth превратился в birth2, но теперь отличный от паустого

DELETE FROM users WHERE id = 2 OR name IS NULL; --=> удаление строки (важно, что IS null, = null не работает!)

TRUNCATE users; --=> очистка всей таблицы
}

|*-------------------*|
Сортировка со сдвигом {

SELECT age FROM clients ORDER BY age DESC OFFSET 1 LIMIT 1; --=> offset n <=> пропустить первые n записей. Т.о. получим второго по старости клиента.
}

|*-------------------*|
Агрегирующие функции {

SELECT car_mark, name, count(*) FROM clients GROUP BY car_mark, name; --=> группировка клиентов по марке машины и имени. Фукнция агрегации - count(*).

SELECT car_mark, name, count(*) OVER (PARTITION BY car_mark) FROM clients;
--=> partition - аналог group by, но данные не группируются, а просто каждому классу группировки добавляется результат агрегирующей функции. (заметим, что не обязательно группировать по всем выводимым колонкам)  

Приятнее смотреть: ...OVER (PARTITION BY car_mark ORDER BY car_mark)...

SELECT min(age) FROM clients 
	UNION SELECT max(age) FROM clients; --=> объединение/склейка таблиц (в данном случае результат - две строчки)
}

|*-------------------------*|
Ранжирующие оконные функции {

SELECT ROW_NUMBER() OVER (PARTITION BY client_id ORDER BY check_in_date) booking_num FROM bookings;  --=> получим для каждого клиента отсортированный и ПРОНУМЕРОВАННЫЙ список его ЗАСЕЛЕНИЙ по дате.

RANK() -- если строки i и (i+1) совпадают,то их ранг =i, а ранг (i+2)ой строки(отличной от двух предыдущих) =(i+2).

DENSE_RANK() -- если строки i и (i+1) совпадают,то их ранг =i, а ранг (i+2)ой строки(отличной от двух предыдущих) =(i+1).
}

|*--------------*|
Функции смещения {

SELECT LAG(room_number) OVER (PARTITION BY client_id ORDER BY check_in_date) previous_room FROM bookings; --=> получим для каждого клиента поле с указанием номера комнаты, в которую он заселялся в ПРОШЛЫЙ раз. (если он ранее не заселялся, то поле будет NULL)
SELECT LEAD(room_number) ...; --=> ... в СЛЕДУЮЩИЙ раз.

SELECT FIRST_VALUE(room_number) ...; --=> ПЕРВОЕ значение столбца в указанной группе\партиции.
SELECT LAST_VALUE(room_number) ...; --=> ПОСЛЕДНЕЕ ... .
SELECT NTH_VALUE(room_number, 6) ...; --=> ШЕСТОЕ ... .
 
}

|*-------------------------*|
Операторы CASE WHEN ELSE END{

SELECT 
	room_number,
	room_price,
	CASE 
		WHEN room_price > 5000 THEN 'Expensive room'
		WHEN room_price > 2500 AND room_price < 5000 THEN 'Normal room'
		ELSE 'Cheap room'
	END as room_price_category
FROM rooms;
}

|*-------------------------------*|
Подзапросы и СТЕ(конструкция WITH){ --вложенные select'ы 

SELECT room_number, с, DENSE_RANK() over (
    ORDER BY c DESC
) AS booking_rank FROM (
    SELECT room_number, count(*) AS c FROM bookings
    GROUP BY room_number
    HAVING c IN (
        SELECT distinct count(*) AS c FROM bookings
        GROUP BY room_number ORDER BY c DESC LIMIT 5
    )
) --=> Возвращает номера комнат(room_number), которые бронировали чаще всего, а также место занимаемое в топе по кол-ву(с) бронирований(booking_rank):
room_number  c           booking_rank
-----------  ----------  ------------
41           470         1
4            462         2
49           462         2
15           454         3
32           451         4
6            441         5


WITH rooms_with_mb AS 
(SELECT room_number FROM rooms WHERE has_minibar=1) --=> выделили select в новую таблицу, которую потом можно использовать в рамках исполняемого файла
}

|*--------------*|
Объединение JOIN {

SELECT c.name, a.age FROM clients c JOIN ages a ON c.client_id = a.client_id; --=> INNER JOIN (возвращает только пересечение множеств)

SELECT ... LEFT JOIN ... --=> возвращает пересечение множеств и оставшиеся записи из левой таблицы (для них вместо соотв-его значения из правой таблицы будет NULL)

SELECT ... RIGHT JOIN ... --=> возвращает пересечение множеств и осатавшиеся записи из правой таблицы


SELECT ... FULL JOIN ... --=> по сути это LEFT JOIN объединенный с RIGHT JOIN. (Иногда наз FULL OUTER JOIN)
}






-- Выше лишь малая часть всего синтаксиса SQL, дальше круче...


/* ======================================================================== */
/* Далее то, что я узнал в унике. По большей части - работа в консоли psql. */
/* Пример реального проектного кода есть в файле simpleSBD.sql.             */
/* Там описана подготовка .sql файла для дальнейшего его выполнения.        */
/* ======================================================================== */

/* ========================================== */
/* psql. Подключение и создание базы данных   */
/* ========================================== */

sudo -iu postgres -- подключение к psql

createdb <db_name> -- создание новой бд (createdb example_db)
psql <db_name> -- переход в базу данных с именем <db_name> (psql example_db)
dropdb <db_name> -- удаление бд (dropdb example_db)

history -- история команд
exit -- выйти из psql


/* ========================================== */
/* Работа внутри БД (example_db=# ...)        */
/* ========================================== */

\s -- история команд
\l -- список баз данных
\q -- вернуться в psql аналогично exit
\! clear -- очистить терминал (через \! впринципе выполняется любой вызов консольной команды)
\d -- список таблиц

\d <table_name> -- описание таблицы <table_name> (\d students)

\COPY <table_name> FROM /home/<table_name>.csv CSV; -- Копирование данных из csv на компе в sql таблицу <table_name>

\i <path_to_sql_file> -- выполнение скрипта из файла .sql по пути <path_to_sql_file>

-- В этой консоли работают все известные команды для работы с таблицами, единственное требование - каждую команду заканчивать ";".

/* ========================================== */
/* Роли. Все еще работа внутри БД             */
/* ========================================== */

\du -- вывод списка владельцев и их права / список ролей (alexpsql и postgres в моем случае)
\dp -- вывод таблиц с правами доступа к ним

-- Добавление, изменение и удаление пользователей и ролей (роль можно присваивать нескольким пользователям, но в целом функционал похожий. Т.е. если только один пользователь, то и роль не нужна и наоборот):

CREATE USER <user_name> WITH PASSWORD '1234567890'; -- создаем пользователя
GRANT CONNECT ON DATABASE <db_name> to <user_name>; -- подключаем его к БД

-- CREATE ROLE <role_name> LOGIN; -- не понял зачем таки писать login, но вроде как разрешение данной роли подключение к БД. Нах надо как-будто.
CREATE ROLE <role_name> WITH password '12345';
ALTER ROLE <role_name> WITH superuser;
DROP ROLE <role_name>;

SELECT current_user, session_user; -- узнать текущую роль, на которой работаешь
SET ROLE <role_name>; -- переключиться на новую роль
SET SESSION AUTHORIZATION <role_name>; -- поменять пользователя текущей сессии

GRANT SELECT ON <table_name> TO <role_name>; -- предоставление права чтения таблицы <table_name> роли <role_name>
REVOKE SELECT ON <table_name> FROM <role_name>; -- удаление права чтения
-- Вместо SELECT можно писать другие привелегии, например UPDATE, INSERT, DELETE, TRUNCATE, EXECUTE(относится к функциям в бд) и др.

GRANT <role_name> to <user_name>; -- присвоить роль пользователю

-- Пример:
GRANT SELECT, UPDATE (name, surname) ON students TO testRole;
GRANT SELECT ON myView to testRole; -- выдадим доступ к представлению myView для роли testRole
/* ========================================== */
/* Транзакции. Все еще работа внутри БД       */
/* ========================================== */

BEGIN TRANSACTION ISOLATION LEVEL <isolation_level>; -- начало транзакции 
-- isolation_level глобально всего 4x видов:
READ COMMITTED -- при параллельной работе ждет окончания работы в нужной таблице.
READ UNCOMMITTED -- то же что committed
REPEATABLE READ -- работа только с сохраненной в начале версией бд, при параллельном изменении одной и той же таблицы не ждет завершения более раннего процесса в другой транзакции, а просто выдает ошибку.
SERIALISABLE -- то же что repeatable

SAVEPOINT <point_name>; -- установка точки сохранения. В случае ошибки вернемся сюда
ROLLBACK TO SAVEPOINT <point_name>; -- Откат транзакции к точке возврата
ROLLBACK; -- завершение транзакции с отменой изменений
END; -- завершение транзакции с сохранением изменений в оригинальную бд


/* ========================================== */
/* Триггеры. Все еще внутри БД                */
/* ========================================== */

CREATE OR REPLACE FUNCTION <func_name>() RETURNS TRIGGER as $$ BEGIN ... END; $$ language plpgsql; -- Создание функции, которая возвращает триггер. 

CREATE TRIGGER <trigger_name> 
BEFORE UPDATE or INSERT on <table_name> 
FOR EACH ROW
	EXECUTE FUNCTION <func_name>(); -- конструкция, которая присваивает триггер <triger_name>, возвращаемый функцией <func_name>, всем записям в таблице <table_name>


-- Пример функции (NEW это очередная запись в таблице):
$$
begin
	if NEW.student_amount < 75 then
		raise exception 'ERROR: amount must be >= 75';
	end if;
	return NEW;
end;
$$ language plpgsql;


/* ========================================== */
/* Представления VIEW. Все еще внутри БД      */
/* ========================================== */
-- Представление - это замена часто исопльзующегося запроса к БД на кодовое слово / команду 
CREATE VIEW <view_name> AS ... ; -- создание представления 
SELECT * FROM <view_name>; -- вызов представления
DROP VIEW <view_name>; -- удаление представления

CREATE OR REPLACE VIEW <view_name> AS ... ; -- тоже создание представления. Говорит само за себя
DROP VIEW IF EXISTS <view_name>; -- тоже удаление представления. Говорит само за себя

-- Пример:
CREATE VIEW poisk AS 
SELECT groupID, count(*) from students group by groupID;

SELECT * FROM poisk;


/* ========================================== */
/* Работа с функциями, курсорами и индексами  */
/* ========================================== */

-- см. файл script_for_4_lb_bd.sql или пример 2 ниже

```
---

### Примеры .sql файлов

#### first
> Создание таблиц. Их заполнение. Объявление Функций. Работа с пользователями\ролями\правами доступа.

```sql
-- Создаем студентов
create table students (
	id bigserial primary key,
	name text not null,
	groupID text not null,
	dateOfBirth date,
	city text,
	admission_year integer
);


-- Создаем города
create table cities (
	name text not null
);
-- Заполняем города
insert into cities 
values ('МСК'), ('СПБ'), ('ЕКБ'), ('ЧЛБ'), ('НН'), ('ВН'), ('КРД'), ('УФА'), ('СЕВ'), ('КРОП');


-- Заполняем студентов
create or replace function insert_func() returns void as $$
declare
	i integer;
	j integer;
	fio_code text := '';
	st_date date := '2000-01-01';
	fin_date date := '2006-01-01';
	birth date;
	admis_date integer;
	letter text[] := array['П', 'К', 'З', 'Ж', 'Е', 'Д', 'Г', 'В', 'Б', 'А', 'Ч', 'Т', 'С', 'Р', 'Ф', 'Л', 'М', 'Н', 'О', 'Ш'];
	
begin
	for i in 1..100000000 loop
		
		for j in 1..3 loop
			fio_code := fio_code || letter[1+floor(random()*20)];
		end loop;
		
		birth := st_date + (random() * (fin_date - st_date + 1))::int;
		admis_date := date_part('YEAR', birth) + 18;
		
		insert into students (name, groupID, dateOfBirth, city, admission_year)
		values (
			'Student ' || fio_code || '_' || i,
			letter[1+floor(random()*20)] || '::' || admis_date || '::' || (i % 17000),
			birth,
			(select name from cities order by random() limit 1),
			admis_date
		);
		
		fio_code := '';
		birth := NULL;
		admis_date := NULL;
		
	end loop;
end $$ language plpgsql;

select insert_func();

-- Создаем факультеты
create table faculties (
	faculty_code text primary key,
	name text,
	students_amount integer default 5100000,
	specs jsonb default '{"specs": ["Basic", "Intermediate", "Professional"]}',
	dekan text
);

-- Заполняем факультеты
insert into faculties (faculty_code, name, dekan)
values 
('П', 'Психология', 'Петров Петр Петрович'),
('К', 'Кибернетика', 'Кузнецов Сергей Иванович'),
('З', 'Здравоохранение', 'Зайцева Анна Викторовна'),
('Ж', 'Журналистика', 'Жукова Мария Александровна'),
('Е', 'Экономика', 'Егоров Алексей Дмитриевич'),
('Д', 'Дизайн', 'Дмитриев Денис Сергеевич'),
('Г', 'География', 'Горшков Игорь Николаевич'),
('В', 'Востоковедение', 'Васильев Николай Андреевич'),
('Б', 'Биология', 'Баранов Алексей Владимирович'),
('А', 'Архитектура', 'Александрова Ольга Петровна'),
('Ч', 'Человеческие ресурсы', 'Чернов Илья Владимирович'),
('Т', 'Технологии', 'Тимофеев Андрей Сергеевич'),
('С', 'Социология', 'Смирнова Наталья Васильевна'),
('Р', 'Реклама', 'Рябов Максим Анатольевич'),
('Ф', 'Филология', 'Федорова Татьяна Сергеевна'),
('Л', 'Логистика', 'Лебедев Владислав Константинович'),
('М', 'Медицинские науки', 'Мартынова Екатерина Юрьевна'),
('Н', 'Неврология', 'Никитин Виктор Павлович'),
('О', 'Огневое искусство', 'Овчинникова Светлана Алексеевна'),
('Ш', 'Шумовые технологии', 'Широков Юрий Валентинович');

-- Создаем таблицу обучения
create table edu (
	groupID text primary key,
	students_amount integer,
	mean_stipend integer default 2700,
	faculty_code text references faculties(faculty_code),
	edu_year integer default 2024
);

-- Заполняем таблицу обучения
create or replace function insert_edu() returns void as $$
declare 
	stipends integer[] := array[0, 2700, 3500, 4500];
begin
	insert into edu (groupID, students_amount)
	select groupid, count(*) from students group by groupid order by groupid;
	
	update edu set 
		mean_stipend = stipends[1 + floor(random() * 3)],
		faculty_code = left(groupid, 1);
end
$$ language plpgsql;

select insert_edu();

-- Создадим пользователя test с подключением к нашей БД
create user test with password 'test12345';
grant connect on database "simpleStudBase" to test;

-- Выдадим полные права для пользователя test на таблицу students
grant select, insert, delete, update on students to test;
grant usage, select on sequence students_id_seq to test; -- права к последовательности id-шников студентов

-- Выдадим ограниченные права записи для пользователя на таблицу faculties
grant select, update (specs) on faculties to test;

-- Выдадим только право чтения для пользователя test на таблицу edu
grant select on edu to test;

-- Представление, описывающее фактическое кол-во человек на факультете
create view stud_amount_in_faculty_view as 
select left(students.groupid, 1) as fac_code, count(*) as stud_amount from students inner join edu on edu.groupid = students.groupid group by fac_code;

-- Создадим роль (роль можно присваивать нескольким пользователям)
create role test_role;

--Выдадим особые права этой роли
grant update (mean_stipend, edu_year) on edu to test_role;
grant select on stud_amount_in_faculty_view to test_role;

-- Присвоим роль test_role пользователю test
grant test_role to test;

```
---

#### second

> Определение функций. Оценка работы индексов. Работа с полнотекстовым поиском. Оптимизация вставки/удаления/изменения. Партиционированные таблицы(масштабирование)

```sql
-- Определяем функцию вывода списка группы
create or replace function get_group(gr_id text) returns refcursor as 
$$
declare
	curs1 refcursor := 'curs1';
begin
	open curs1 for select * from students where groupid = gr_id;
	return curs1;
end;
$$ language plpgsql;

-- Вызов созданной функции
select * from get_group('Б::2022::3');

-- Определяем функцию подсчета фактического числа студентов на факультете
create or replace function faculty_amount(fac_id text) returns integer as
$$
declare
	amount integer := 0;
	r edu%rowtype;
	fac_code text := null;
begin
	select faculty_code into fac_code from faculties where faculty_code = fac_id;
	if fac_code is null then
		raise exception 'ERROR: Факультет с кодом % не найден!', fac_id;
	else
		for r in (select * from edu where groupid like (fac_id || '%')) loop
			amount := amount + r.students_amount;
		end loop;
	end if;
	return amount;
end;
$$ language plpgsql;

-- Вызов сделанной функции
select * from faculty_amount('П');

/*
Преимущество функций перед представлениями заключается в их универсальности. Имеется в виду, что функции могут возвращать любые типы данных, в том числе и сущности(таблицы), тем самым включая в себя функционал представлений.
*/

------------------------


-- Оценка работы индексов на запросах с фильтрацией над одной таблицей
select * from edu
where students_amount between 49 and 52 and groupid ~ '::2019::' limit 5;

explain (analyze, costs off)
select * from edu
where students_amount between 49 and 52 and groupid ~ '::2019::' limit 5;

create index idx_edu_49_stud_amount_52_group_2019 on edu (students_amount, groupid);

set enable_seqscan = off;

explain (analyze, costs off)
select * from edu
where students_amount between 49 and 52 and groupid ~ '::2019::' limit 5;

set enable_seqscan = on;

-- Оценка работы индексов на запросах с фильтрацией над несколькими таблицами

-- Число студентов из городов, код которых кончается на "Б"
select c.name, count(id) 
from cities c join students s on c.name = s.city
where s.admission_year in (2018, 2019)
and c.name like '%Б'
group by c.name;

explain (analyze, costs off) select c.name, count(id) 
from cities c join students s on c.name = s.city
where s.admission_year in (2018, 2019)
and c.name like '%Б'
group by c.name;

create index idx_students_adm_year on students(admission_year);
create index idx_cities_name on cities(name);

set enable_seqscan = off;

explain (analyze, costs off) select c.name, count(id) 
from cities c join students s on c.name = s.city
where s.admission_year in (2018, 2019)
and c.name like '%Б'
group by c.name;

set enable_seqscan = on;

------------------
-- Работа с полнотекстовым поиском

-- просто текст

create index idx_faculty_describe_lang
on faculties using gin(to_tsvector('russian', description));

explain analyze
select faculty_code, description from faculties
where to_tsvector('russian', description) @@ to_tsquery('english');

-- массивы

create index idx_faculty_exams 
on faculties using gin (exams);

explain analyze
select faculty_code, exams from faculties
where exists (
        select * from unnest(exams) as elem -- с помощью unnest делаем массив таблицей
        where elem ~ '#tech'
);

-- json

create index idx_faculty_specs 
on faculties using gin(specs);

explain analyze
select faculty_code, specs from faculties
where specs @> '{"rating": "High"}';

----
-- Оптимизируем вставку/удаление/изменение занчений в большую таблицу

-- для чтения добавим популярные индексы
create index idx_students_groupid on students (groupid);
create index idx_students_name on students (name);
create index idx_students_adm_year on students (admission_year);

-- для удаления разделим данные на секции (секционирование)
-- Cоздадим партиционированную таблицу-клон 
create table modern_students (
	id bigserial,
	name text not null,
	groupID text not null,
	dateOfBirth date,
	city text,
	admission_year integer
)partition by range(admission_year);

-- Разбиваем ее на части
create table students_before_2019 partition of modern_students
for values from (0) to (2019);
create table students_after_2019 partition of modern_students
for values from (2019) to (2030);

--запишем старые данные в новую партиционную таблицу
insert into moder_students select * from students;

--для добавления большого объема данных следует очищать ресурсозатратные индексы
drop index idx_students_pkey;

insert into students values <новые данные>;

create index idx_students_pkey on students (id);

```
---
