### Университетские конспекты

Примеры смотреть в `/home/alex/Desktop/Learn_DevOps_sql/Learn_sql_git_linux`

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
}

|*-------------------------*|
Ранжирующие оконные функции {

SELECT ROW_NUMBER() OVER (PARTITION BY client_id ORDER BY check_in_date) booking_num FROM bookings;  --=> получим для каждого клиента отсортированный и ПРОНУМЕРОВАННЫЙ список его ЗАСЕЛЕНИЙ по дате.

RANK() -- если строки i и (i+1) совпадают,то их ранг =i, а ранг (i+2)ой строки(отличной от двух предыдущих) =(i+1).

DENSE_RANK() -- если строки i и (i+1) совпадают,то их ранг =i, а ранг (i+2)ой строки(отличной от двух предыдущих) =(i+2).
}

|*--------------*|
Функции смещения {

SELECT LAG(room_number) OVER (PARTITION BY client_id ORDER BY check_in_date) previous_room FROM bookings; --=> получим для каждого клиента поле с указанием номера комнаты, в которую он заселялся в ПРОШЛЫЙ раз. (если он ранее не заселялся, то поле будет NULL)
SELECT LEAD(room_number) ...; --=> ... в СЛЕДУЮЩИЙ раз.

SELECT FIRST_VALUE(room_number) ...; --=> ПЕРВОЕ значение столбца в указанной группе\партиции.
SELECT LAST_VALUE(room_number) ...; --=> ПОСЛЕДНЕЕ ... .
SELECT NTH_VALUE(room_number, 6) ...; --=> ШЕСТОЕ ... .
 
}

|*----------------*|
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

-- см. файл script_for_4_lb_bd.sql





-- Решение второй задачи РК 2.2 БД (база данных fmm)
select distinct с.руковод, сс.фио, сс.руковод from сотрудники с join сотрудники сс on с.руковод = сс.номер_ст where сс.руковод in (select начальник from отделы);

```
---

### Конспекты по Цифровой Кафедре

> [!abstract] SQL - (Structured Query Language)
> Включает в себя: 
> - DDL (Data Definition Language) - определение структур данных
> - DML (Data Manipulation Language) - управление данными
> - DQL (Data Query Language) - извлечение данных из БД





