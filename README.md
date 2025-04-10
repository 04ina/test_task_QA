# 2. Задание по основам баз данных
### 1. Установить PostgreSQL
Устанавливать СУБД PostgreSQL мы будем через Docker.

Сначала нам нужно загрузить образ СУБД PostgreSQL из репозитория Docker Hub:
```bash
docker pull postgres
```

Затем мы должны запустить контейнер из образа с помощью команды docker run:

```
docker run -itd -e POSTGRES_USER=o4ina -e POSTGRES_PASSWORD=qwerty -p 5432:5432 -v /data:/var/lib/postgresql/data --name postgresql postgres
```

Работать с Postgres-ом мы будем посредством терминального клиента psql. Запуск клиента будет производиться посредством команды docker exec

```
$ docker exec -ti postgresql psql -U o4ina
psql (17.4 (Debian 17.4-1.pgdg120+2))
Type "help" for help.

o4ina=#
```
### 2. Создать БД academy
Создаем базу данных academy и подключаемся к ней.
```sql
CREATE DATABASE academy;
\c academy
```
### 3. Создать необходимые таблицы
```sql
CREATE TABLE Students (
	s_id SERIAL PRIMARY KEY,
 	name text,
	start_year interval YEAR
);

CREATE TABLE Courses (
	c_no SERIAL PRIMARY KEY,
	title text,
	hours interval HOUR
);

CREATE TABLE Exams (
	s_id integer,
	c_no integer,
	score integer CHECK (score BETWEEN 0 AND 100),
		
	FOREIGN KEY (s_id) REFERENCES Students (s_id),
	FOREIGN KEY (c_no) REFERENCES Courses (c_no)
);
```
### 4. Вставить записи в таблицу
```sql
INSERT INTO students(name, start_year) VALUES 
	('Гилев Михаил Вячеславович', '2020 YEARS'),
	('Гусев Антон Николаевич', '2019 YEARS'),
	('Пахомов Артур Данилович', '2020 YEARS'),
	('Ефремов Рубен Платонович', '2023 YEARS'),
	('Лобанов Егор Федотович', '2024 YEARS'),
	('Щукин Роберт Анатольевич', '2022 YEARS'),
	('Попов Вилли Степанович', '2015 YEARS'),
	('Жданов Венедикт Евгеньевич', '2016 YEARS'),
	('Новиков Соломон Кириллович', '2018 YEARS'),
	('Карпов Яков Константинович', '2020 YEARS');

INSERT INTO courses(title, hours) VALUES 
	('Дискретная математика', '144 HOURS'),
	('Ядерная физика', '120 HOURS'),
	('Атомная энергетика', '72 HOURS'),
	('Волновая оптика', '200 HOURS'),
	('Квантовая Механика', '100 HOURS'),
	('Вычислительная техника', '122 HOURS'),
	('Электромагнитная теория', '100 HOURS'),
	('Теоретическая химия', '72 HOURS'),
	('Молекулярная биология', '50 HOURS'),
	('Гравитационная астрономия', '80 HOURS'),
	('Дореволюционная грамматика', '100 HOURS');

INSERT INTO exams VALUES 
	(1, 1, 80),
	(2, 1, 50),
	(2, 2, 100),
	(3, 11, 100),
	(4, 10, 99),
	(5, 5, 55),
	(8, 1, 100),
	(8, 2, 100),
	(8, 3, 100),
	(9, 6, 10);
```
### 5. Написать запрос, который возвращает всех студентов, которые еще не сдали ни одного экзамена
```sql
SELECT 
	s.s_id AS id,
	s.name name
FROM students AS s WHERE NOT EXISTS (
	SELECT * FROM Exams e WHERE e.s_id = s.s_id
);
 id |            name            
----+----------------------------
 10 | Карпов Яков Константинович
  6 | Щукин Роберт Анатольевич
  7 | Попов Вилли Степанович
(3 rows)
```
### 6. Написать запрос, который возвращает список студентов и количество сданных им экзаменов. Только для студентов, у которых есть сданные экзамены
```sql
SELECT 
	s.s_id AS id, 
	s.name AS name, 
	count(*) AS "cnt passed exams" 
FROM students AS s INNER JOIN exams AS e ON s.s_id = e.s_id 
GROUP by s.s_id, s.name;
 id |            name            | cnt passed exams 
----+----------------------------+------------------
  2 | Гусев Антон Николаевич     |                2
  3 | Пахомов Артур Данилович    |                1
  5 | Лобанов Егор Федотович     |                1
  8 | Жданов Венедикт Евгеньевич |                3
  4 | Ефремов Рубен Платонович   |                1
  9 | Новиков Соломон Кириллович |                1
  1 | Гилев Михаил Вячеславович  |                1
(7 rows)
```

### 7. Вывести список курсов со средним баллом по экзамену. Список отсортирован по убыванию среднего балла
```sql
SELECT 
	c.c_no AS no, 
	c.title AS title, 
	round(avg(e.score), 2) AS "avg score" 
FROM courses AS c LEFT JOIN exams AS e ON c.c_no = e.c_no 
GROUP BY c.c_no, c.title 
ORDER BY "avg score" DESC NULLS LAST;
 no |           title            | avg score 
----+----------------------------+-----------
 11 | Дореволюционная грамматика |    100.00
  2 | Ядерная физика             |    100.00
  3 | Атомная энергетика         |    100.00
 10 | Гравитационная астрономия  |     99.00
  1 | Дискретная математика      |     76.67
  5 | Квантовая Механика         |     55.00
  6 | Вычислительная техника     |     10.00
  4 | Волновая оптика            |          
  8 | Теоретическая химия        |          
  9 | Молекулярная биология      |          
  7 | Электромагнитная теория    |          
(11 rows)
```
### 8. Сгенерировать скрипт, который наполняет таблицы произвольными данными (можно с помощью psql, можно с помощью любого языка программирования)

Скрипт наполнения таблицы полностью написан на SQL. Получение разнообразных ФИО и названий дисциплин происходит за счет случайного комбинирования частей последних. 
#### заполнение таблицы students
```sql
WITH names AS (
	SELECT * FROM (VALUES 
		('Гилев','Михаил','Вячеславович'),
		('Гусев','Антон','Николаевич'),
		('Пахомов','Артур','Данилович'),
		('Ефремов','Рубен','Платонович'),
		('Лобанов','Егор','Федотович'),
		('Щукин','Роберт','Анатольевич'),
		('Попов','Вилли','Степанович'),
		('Жданов','Венедикт','Евгеньевич'),
		('Новиков','Соломон','Кириллович'),
		('Карпов','Яков','Константинович')
	) AS names(surname, name, patronymic)
)
INSERT INTO students(name, start_year)
	SELECT 
		(SELECT surname FROM names ORDER BY random()*gs LIMIT 1) || ' ' ||
		(SELECT name FROM names ORDER BY random()*gs LIMIT 1) || ' ' ||
		(SELECT patronymic FROM names ORDER BY random()*gs LIMIT 1) AS name,
		((EXTRACT(YEAR FROM now()) - round(random()*20))::text || ' years'::text)::interval AS start_year
	FROM generate_series(1, 10) AS gs
RETURNING *;
 s_id |             name             | start_year 
------+------------------------------+------------
   11 | Лобанов Рубен Николаевич     | 2022 years
   12 | Новиков Рубен Константинович | 2016 years
   13 | Попов Антон Анатольевич      | 2007 years
   14 | Щукин Антон Федотович        | 2020 years
   15 | Новиков Соломон Федотович    | 2015 years
   16 | Щукин Антон Николаевич       | 2010 years
   17 | Щукин Егор Евгеньевич        | 2024 years
   18 | Жданов Яков Федотович        | 2015 years
   19 | Гилев Рубен Анатольевич      | 2006 years
   20 | Карпов Рубен Степанович      | 2011 years
(10 rows)

INSERT 0 10
```
#### заполнение таблицы courses
```sql
WITH course_names AS (
	SELECT * FROM (VALUES 
		('Дискретная','математика'),
		('Ядерная','физика'),
		('Атомная','энергетика'),
		('Волновая','оптика'),
		('Квантовая','Механика'),
		('Вычислительная','техника'),
		('Электромагнитная','теория'),
		('Теоретическая','химия'),
		('Молекулярная','биология'),
		('Гравитационная','астрономия'),
		('Дореволюционная', 'грамматика')
		
	) AS names(cn_part_1, cn_part_2)
)
INSERT INTO courses(title, hours)
	SELECT 
		(SELECT cn_part_1 FROM course_names ORDER BY random()*gs LIMIT 1) || ' ' ||
		(SELECT cn_part_2 FROM course_names ORDER BY random()*gs LIMIT 1),
		((72 + round(random()*120))::text || ' hours'::text)::interval AS start_year
	FROM generate_series(1, 10) AS gs
RETURNING *;
 c_no |            title            |   hours   
------+-----------------------------+-----------
   12 | Молекулярная математика     | 138:00:00
   13 | Молекулярная химия          | 127:00:00
   14 | Гравитационная физика       | 95:00:00
   15 | Атомная биология            | 157:00:00
   16 | Молекулярная физика         | 150:00:00
   17 | Квантовая астрономия        | 120:00:00
   18 | Электромагнитная астрономия | 152:00:00
   19 | Дореволюционная теория      | 80:00:00
   20 | Квантовая энергетика        | 129:00:00
   21 | Молекулярная оптика         | 110:00:00
(10 rows)

INSERT 0 10
```
#### заполнение таблицы exams
```sql
INSERT INTO exams(s_id, c_no, score) 
	SELECT * FROM (
		SELECT DISTINCT ON (s.s_id, c.c_no)
			s.s_id, 
			c.c_no,
			round(random()*100) AS score
		FROM students AS s, courses AS c
		ORDER BY s.s_id, c.c_no
	)
	ORDER BY random() LIMIT 10
RETURNING *;
  s_id | c_no | score 
------+------+-------
   20 |   17 |    53
    5 |    6 |    19
    5 |   11 |    67
   18 |   15 |    53
    3 |    4 |    80
    1 |   18 |    84
   10 |    4 |    63
    7 |   21 |    64
    6 |   13 |    25
   16 |   13 |    92
(10 rows)

INSERT 0 10
```
