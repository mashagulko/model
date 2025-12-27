INSERT INTO ваша_таблица (k_vod) VALUES
(101), (102), (103), (104), (105),
(101), (102), (103), (104), (105),
(101), (102), (103), (104), (105),
(101), (102), (103), (104), (105),
(101), (102), (103), (104), (105),
(101), (102), (103), (104), (105),
(101), (102), (103), (104), (105),
(101), (102), (103), (104), (105),
(101), (102), (103), (104), (105),
(101), (102), (103), (104), (105);
Вы сказали:

ЭКЗАМЕН КВАЛИФИКАЦИОННЫЙ 

ПМ.11 Разработка,  администрирование и защита баз данных

специальность 09.02.07 «Информационные системы и программирование»
 


КОМПЕТЕНТНОСТНО-ОРИЕНТИРОВАННОЕ ЗАДАНИЕ 

Вы работаете техником-программистом в компании, занимающейся разработкой и внедрением программного обеспечения. В компанию поступил заказ на разработку нового программного продукта. В рамках проекта по созданию данного ПП, Вам поручили выполнить следующие работы:


Билет №1: Тема «Учет в книжном магазине»
Контрольно-оценочное задание 1
Спроектировать базу данных методом нормализации и методом семантического моделирования. Информация представлена в виде одной таблицы:
a. Код издательства (k_izd)
b. Название издательства (n_izd)
c. Код книги (ISBN)
d. Название книги (n_book)
e. Автор (author)
f. Жанр (genre)
g. Закупочная цена у издательства (cena_zak)
h. Дата поступления (data_post)
i. Количество поступивших экземпляров (kol_post)
j. Количество на складе (kol_sklad)
k. Цена продажи (cena_prod) (Наценка 25%)
l. Дата продажи (data_prod)
m. Количество проданных экземпляров (kol_prod)
n. Код продавца (k_seller)
o. ФИО продавца (fio_seller)
p. Процент от продажи продавцу (procent)
Построить ER-диаграмму базы данных.

Контрольно-оценочное задание 2
Создать БД, таблицы и заполнить их. Справочник книг - не менее 15 наименований, продавцов - не менее 4. Данные о поставках и продажах - не менее 25 записей. Реализовать ограничения целостности (NOT NULL, PRIMARY KEY, FOREIGN KEY).

Контрольно-оценочное задание 3
Создать запросы:
Отчет о продажах за выбранный месяц (поля: дата, название книги, автор, количество, выручка (кол-во*цена), ФИО продавца).
Показать книги, цена продажи которых выше средней цены по магазину.
Показать книги, которые есть в каталоге магазина, но отсутствуют на складе (кол-во = 0).
Увеличить закупочную цену для всех книг заданного издательства на 10%.

Контрольно-оценочное задание 4
Создать триггер, который при регистрации продажи книги будет автоматически уменьшать ее остаток на складе. Если остаток станет отрицательным, откатывать транзакцию с выводом сообщения об ошибке.

Контрольно-оценочное задание 5
Выполнить резервное копирование базы данных. Результат сохранить на рабочем столе.


ChatGPT сказал:
Ниже — готовое решение билета №1 (MySQL). Кратко и по делу.

Задание 1. Нормализация + семантическая модель
1НФ → 3НФ (итоговые таблицы)
Издательства

k_izd (PK)

n_izd

Книги

ISBN (PK)

n_book

author

genre

k_izd (FK)

cena_zak

cena_prod

Поставки

id_post (PK)

ISBN (FK)

data_post

kol_post

Склад

ISBN (PK, FK)

kol_sklad

Продавцы

k_seller (PK)

fio_seller

procent

Продажи

id_prod (PK)

ISBN (FK)

k_seller (FK)

data_prod

kol_prod

ER-диаграмма (текстово)
Издательство 1 — M Книга

Книга 1 — M Поставка

Книга 1 — 1 Склад

Книга 1 — M Продажа

Продавец 1 — M Продажа

Задание 2. Создание БД и таблиц
CREATE DATABASE bookshop;
USE bookshop;

CREATE TABLE publishers (
  k_izd INT PRIMARY KEY,
  n_izd VARCHAR(100) NOT NULL
);

CREATE TABLE books (
  ISBN VARCHAR(20) PRIMARY KEY,
  n_book VARCHAR(150) NOT NULL,
  author VARCHAR(100) NOT NULL,
  genre VARCHAR(50),
  k_izd INT NOT NULL,
  cena_zak DECIMAL(8,2) NOT NULL,
  cena_prod DECIMAL(8,2) NOT NULL,
  FOREIGN KEY (k_izd) REFERENCES publishers(k_izd)
);

CREATE TABLE warehouse (
  ISBN VARCHAR(20) PRIMARY KEY,
  kol_sklad INT NOT NULL,
  FOREIGN KEY (ISBN) REFERENCES books(ISBN)
);

CREATE TABLE supplies (
  id_post INT AUTO_INCREMENT PRIMARY KEY,
  ISBN VARCHAR(20) NOT NULL,
  data_post DATE NOT NULL,
  kol_post INT NOT NULL,
  FOREIGN KEY (ISBN) REFERENCES books(ISBN)
);

CREATE TABLE sellers (
  k_seller INT PRIMARY KEY,
  fio_seller VARCHAR(100) NOT NULL,
  procent DECIMAL(5,2) NOT NULL
);

CREATE TABLE sales (
  id_prod INT AUTO_INCREMENT PRIMARY KEY,
  ISBN VARCHAR(20) NOT NULL,
  k_seller INT NOT NULL,
  data_prod DATE NOT NULL,
  kol_prod INT NOT NULL,
  FOREIGN KEY (ISBN) REFERENCES books(ISBN),
  FOREIGN KEY (k_seller) REFERENCES sellers(k_seller)
);
Заполнение:
– книги ≥15
– продавцы ≥4
– поставки + продажи ≥25
(обычные INSERT INTO, без особенностей)

Задание 3. Запросы
1. Отчет о продажах за месяц
SELECT s.data_prod, b.n_book, b.author,
       s.kol_prod,
       s.kol_prod * b.cena_prod AS vyruchka,
       sel.fio_seller
FROM sales s
JOIN books b ON s.ISBN = b.ISBN
JOIN sellers sel ON s.k_seller = sel.k_seller
WHERE MONTH(s.data_prod) = 5 AND YEAR(s.data_prod) = 2025;
2. Книги с ценой выше средней
SELECT n_book, cena_prod
FROM books
WHERE cena_prod > (SELECT AVG(cena_prod) FROM books);
3. Книги без остатка
SELECT b.n_book
FROM books b
JOIN warehouse w ON b.ISBN = w.ISBN
WHERE w.kol_sklad = 0;
4. Увеличить закупочную цену издательства на 10%
UPDATE books
SET cena_zak = cena_zak * 1.10
WHERE k_izd = 1;
Задание 4. Триггер
DELIMITER $$

CREATE TRIGGER trg_sale_after_insert
AFTER INSERT ON sales
FOR EACH ROW
BEGIN
  UPDATE warehouse
  SET kol_sklad = kol_sklad - NEW.kol_prod
  WHERE ISBN = NEW.ISBN;

  IF (SELECT kol_sklad FROM warehouse WHERE ISBN = NEW.ISBN) < 0 THEN
    SIGNAL SQLSTATE '45000'
    SET MESSAGE_TEXT = 'Ошибка: недостаточно книг на складе';
  END IF;
END$$

DELIMITER ;
Задание 5. Резервное копирование
mysqldump -u root -p bookshop > C:\Users\Имя_пользователя\Desktop\bookshop_backup.sql
Если нужно:

ER-диаграмма в Draw.io

готовые INSERT на 25+ записей

вариант под PostgreSQL

Скажи, что именно.

Вы сказали:
Билет №2: Тема «Поликлиника. Учет пациентов и приемов»
Контрольно-оценочное задание 1
Спроектировать БД. Исходная таблица:
a. Код пациента (k_pac)
b. ФИО пациента (fio_pac)
c. Дата рождения (dr)
d. Пол (pol)
e. Код врача (k_doc)
f. ФИО врача (fio_doc)
g. Специальность врача (spec)
h. Категория врача (kat)
i. Табельный номер врача (tab_nom)
j. Дата приема (data_priem)
k. Код услуги (k_usl)
l. Название услуги (n_usl)
m. Стоимость услуги (stoim)
n. Диагноз (diag)
o. Назначенное лечение (lech)
Построить ER-диаграмму.

Контрольно-оценочное задание 2
Создать БД. Справочник врачей - не менее 6, пациентов - не менее 20, услуг - не менее 10. Записи о приемах - не менее 30. Реализовать ограничения целостности.

Контрольно-оценочное задание 3
Создать запросы:
Показать всех пациентов, принятых указанным врачом за период.

Показать врачей, чья средняя стоимость приема выше средней по поликлинике.
Показать услуги, которые есть в прейскуранте, но ни разу не назначались.
Повысить стоимость всех услуг для врачей высшей категории на 5%.

Контрольно-оценочное задание 4
Создать триггер, который не позволяет назначить прием пациенту на одну и ту же дату и время к разным врачам.

Контрольно-оценочное задание 5
Выполнить резервное копирование БД.


ChatGPT сказал:
Ниже — готовое решение билета №2 (MySQL). Кратко, структурировано, как на экзамене.

Задание 1. Проектирование БД (нормализация)
Итоговые таблицы (3НФ)
Пациенты

k_pac (PK)

fio_pac

dr

pol

Врачи

k_doc (PK)

fio_doc

spec

kat

tab_nom

Услуги

k_usl (PK)

n_usl

stoim

Приемы

id_priem (PK)

k_pac (FK)

k_doc (FK)

k_usl (FK)

data_priem

diag

lech

ER-диаграмма (словесно)
Пациент 1 — M Прием

Врач 1 — M Прием

Услуга 1 — M Прием

Задание 2. Создание БД и таблиц
CREATE DATABASE clinic;
USE clinic;

CREATE TABLE patients (
  k_pac INT PRIMARY KEY,
  fio_pac VARCHAR(100) NOT NULL,
  dr DATE NOT NULL,
  pol CHAR(1) NOT NULL
);

CREATE TABLE doctors (
  k_doc INT PRIMARY KEY,
  fio_doc VARCHAR(100) NOT NULL,
  spec VARCHAR(100) NOT NULL,
  kat VARCHAR(50) NOT NULL,
  tab_nom VARCHAR(20) NOT NULL
);

CREATE TABLE services (
  k_usl INT PRIMARY KEY,
  n_usl VARCHAR(100) NOT NULL,
  stoim DECIMAL(8,2) NOT NULL
);

CREATE TABLE priems (
  id_priem INT AUTO_INCREMENT PRIMARY KEY,
  k_pac INT NOT NULL,
  k_doc INT NOT NULL,
  k_usl INT NOT NULL,
  data_priem DATETIME NOT NULL,
  diag VARCHAR(200),
  lech VARCHAR(200),
  FOREIGN KEY (k_pac) REFERENCES patients(k_pac),
  FOREIGN KEY (k_doc) REFERENCES doctors(k_doc),
  FOREIGN KEY (k_usl) REFERENCES services(k_usl)
);
Заполнение:
– врачи ≥ 6
– пациенты ≥ 20
– услуги ≥ 10
– приемы ≥ 30

Задание 3. SQL-запросы
1. Пациенты, принятые врачом за период
SELECT DISTINCT p.fio_pac, pr.data_priem
FROM priems pr
JOIN patients p ON pr.k_pac = p.k_pac
WHERE pr.k_doc = 1
AND pr.data_priem BETWEEN '2025-05-01' AND '2025-05-31';
2. Врачи, у кого средняя стоимость приема выше средней по поликлинике
SELECT d.fio_doc
FROM doctors d
JOIN priems pr ON d.k_doc = pr.k_doc
JOIN services s ON pr.k_usl = s.k_usl
GROUP BY d.k_doc, d.fio_doc
HAVING AVG(s.stoim) >
       (SELECT AVG(stoim) FROM services);
3. Услуги, которые ни разу не назначались
SELECT s.n_usl
FROM services s
LEFT JOIN priems pr ON s.k_usl = pr.k_usl
WHERE pr.k_usl IS NULL;
4. Повысить стоимость услуг для врачей высшей категории на 5%
UPDATE services
SET stoim = stoim * 1.05
WHERE k_usl IN (
  SELECT DISTINCT pr.k_usl
  FROM priems pr
  JOIN doctors d ON pr.k_doc = d.k_doc
  WHERE d.kat = 'Высшая'
);
Задание 4. Триггер (запрет двойного приема)
DELIMITER $$

CREATE TRIGGER trg_no_double_priem
BEFORE INSERT ON priems
FOR EACH ROW
BEGIN
  IF EXISTS (
    SELECT 1
    FROM priems
    WHERE k_pac = NEW.k_pac
      AND data_priem = NEW.data_priem
  ) THEN
    SIGNAL SQLSTATE '45000'
    SET MESSAGE_TEXT = 'Ошибка: пациент уже записан на это время';
  END IF;
END$$

DELIMITER ;
Задание 5. Резервное копирование
mysqldump -u root -p clinic > C:\Users\Имя_пользователя\Desktop\clinic_backup.sql
Если нужно:

ER-диаграмма в draw.io

готовые INSERT (30+ приемов)

адаптация под PostgreSQL

Напиши, что именно.

Вы сказали:
Контрольно-оценочное задание 3
Создать запросы:
Показать всех пациентов, принятых указанным врачом за период.

Показать врачей, чья средняя стоимость приема выше средней по поликлинике.
Показать услуги, которые есть в прейскуранте, но ни разу не назначались.
Повысить стоимость всех услуг для врачей высшей категории на 5%.

Контрольно-оценочное задание 4
Создать триггер, который не позволяет назначить прием пациенту на одну и ту же дату и время к разным врачам.

Контрольно-оценочное задание 5
Выполнить резервное копирование БД.



















Билет №3: Тема «Автопредприятие. Учет рейсов и водителей»
Контрольно-оценочное задание 1
Спроектировать БД. Исходная таблица:
a. Код автомобиля (k_avto)
b. Госномер (gosnom)
c. Марка (marka)
d. Год выпуска (god)
e. Код водителя (k_vod)
f. ФИО водителя (fio_vod)
g. Категория прав (kat_prav)
h. Оклад водителя (oklad)
i. Код рейса (k_reis)
j. Пункт назначения (punkt)
k. Дата рейса (data_reis)
l. Пробег, км (probeg)
m. Расход топлива, л (rashod)
n. Стоимость перевозки для клиента (stoim_perev)
o. Комиссия водителя (% от стоимости) (procent_vod)
Построить ER-диаграмму.

Контрольно-оценочное задание 2
Создать БД. Справочник автомобилей - не менее 5, водителей - не менее 7, рейсов - не менее 25. Реализовать ограничения целостности.

Контрольно-оценочное задание 3
Создать запросы:
Отчет по рейсам за месяц (дата, марка, госномер, ФИО водителя, пункт, пробег, доход).

Показать водителей, общая сумма комиссий которых за период выше среднего.
Показать автомобили, которые не совершали рейсов в текущем месяце.
Увеличить оклад водителям со стажем более 10 лет на 10%.

Контрольно-оценочное задание 4
Создать триггер, который при добавлении рейса проверяет наличие у водителя необходимой категории прав для управления данным автомобилем (например, категория "C" для грузовиков). В случае несоответствия - запрещать вставку.

Контрольно-оценочное задание 5
Выполнить резервное копирование БД.
ChatGPT сказал:
Даю готовое решение Билета №3 (MySQL). Кратко, экзаменационный формат.

Билет №3. Автопредприятие
Задание 1. Проектирование БД (3НФ)
Таблицы
Автомобили

k_avto (PK)

gosnom

marka

god

kat_prav_req — требуемая категория прав

Водители

k_vod (PK)

fio_vod

kat_prav

oklad

stazh — стаж (лет)

Рейсы

k_reis (PK)

k_avto (FK)

k_vod (FK)

punkt

data_reis

probeg

rashod

stoim_perev

procent_vod

ER-диаграмма (словесно)
Автомобиль 1 — M Рейс

Водитель 1 — M Рейс

Задание 2. Создание БД и таблиц
CREATE DATABASE transport;
USE transport;

CREATE TABLE autos (
  k_avto INT PRIMARY KEY,
  gosnom VARCHAR(15) NOT NULL,
  marka VARCHAR(50) NOT NULL,
  god INT NOT NULL,
  kat_prav_req CHAR(1) NOT NULL
);

CREATE TABLE drivers (
  k_vod INT PRIMARY KEY,
  fio_vod VARCHAR(100) NOT NULL,
  kat_prav CHAR(1) NOT NULL,
  oklad DECIMAL(10,2) NOT NULL,
  stazh INT NOT NULL
);

CREATE TABLE trips (
  k_reis INT AUTO_INCREMENT PRIMARY KEY,
  k_avto INT NOT NULL,
  k_vod INT NOT NULL,
  punkt VARCHAR(100) NOT NULL,
  data_reis DATE NOT NULL,
  probeg INT NOT NULL,
  rashod DECIMAL(8,2),
  stoim_perev DECIMAL(10,2) NOT NULL,
  procent_vod DECIMAL(5,2) NOT NULL,
  FOREIGN KEY (k_avto) REFERENCES autos(k_avto),
  FOREIGN KEY (k_vod) REFERENCES drivers(k_vod)
);
Задание 3. SQL-запросы
1. Отчет по рейсам за месяц
SELECT t.data_reis, a.marka, a.gosnom, d.fio_vod,
       t.punkt, t.probeg,
       t.stoim_perev AS dohod
FROM trips t
JOIN autos a ON t.k_avto = a.k_avto
JOIN drivers d ON t.k_vod = d.k_vod
WHERE MONTH(t.data_reis) = 5 AND YEAR(t.data_reis) = 2025;
2. Водители, чья сумма комиссий выше средней
SELECT d.fio_vod
FROM drivers d
JOIN trips t ON d.k_vod = t.k_vod
GROUP BY d.k_vod, d.fio_vod
HAVING SUM(t.stoim_perev * t.procent_vod / 100) >
       (
         SELECT AVG(sum_comm)
         FROM (
           SELECT SUM(stoim_perev * procent_vod / 100) AS sum_comm
           FROM trips
           GROUP BY k_vod
         ) x
       );
3. Автомобили без рейсов в текущем месяце
SELECT a.marka, a.gosnom
FROM autos a
LEFT JOIN trips t
  ON a.k_avto = t.k_avto
  AND MONTH(t.data_reis) = MONTH(CURDATE())
  AND YEAR(t.data_reis) = YEAR(CURDATE())
WHERE t.k_reis IS NULL;
4. Увеличить оклад водителям со стажем > 10 лет
UPDATE drivers
SET oklad = oklad * 1.10
WHERE stazh > 10;
Задание 4. Триггер (проверка категории прав)
DELIMITER $$

CREATE TRIGGER trg_check_license
BEFORE INSERT ON trips
FOR EACH ROW
BEGIN
  IF (
    SELECT kat_prav
    FROM drivers
    WHERE k_vod = NEW.k_vod
  ) <> (
    SELECT kat_prav_req
    FROM autos
    WHERE k_avto = NEW.k_avto
  ) THEN
    SIGNAL SQLSTATE '45000'
    SET MESSAGE_TEXT = 'Ошибка: недостаточная категория прав у водителя';
  END IF;
END$$

DELIMITER ;
Задание 5. Резервное копирование БД
mysqldump -u root -p transport > C:\Users\Имя_пользователя\Desktop\transport_backup.sql
Если нужно:

готовые INSERT (25 рейсов)

ER-диаграмма в draw.io

вариант под PostgreSQL

Пиши, что добавить.

Вы сказали:
Билет №4: Тема «Образовательный центр. Учет курсов и студентов»
Контрольно-оценочное задание 1
Спроектировать БД. Исходная таблица:
a. Код студента (k_stud)
b. ФИО студента (fio_stud)
c. Контактный телефон (phone)
d. Код курса (k_kurs)
e. Название курса (n_kurs)
f. Код преподавателя (k_prep)
g. ФИО преподавателя (fio_prep)
h. Стоимость курса (stoim_kurs)
i. Дата начала курса (data_start)
j. Дата окончания курса (data_end)
k. Дата посещения (data_pos)
l. Статус посещения (явка/неявка) (status)
m. Оценка за итоговый тест (ocenka)
n. Ставка преподавателя за час (stavka)
o. Количество часов курса (chasy)
Построить ER-диаграмму.

Контрольно-оценочное задание 2
Создать БД. Справочник курсов - не менее 8, преподавателей - не менее 5, студентов - не менее 25. Записи о посещениях - не менее 100. Реализовать ограничения целостности.

Контрольно-оценочное задание 3
Создать запросы:
Показать посещаемость конкретного студента на выбранном курсе.

Показать курсы, стоимость которых выше средней стоимости всех курсов.
Показать студентов, зачисленных на курсы, но ни разу не посетивших занятия.
Повысить ставку преподавателям, чьи студенты в среднем имеют оценку выше 4.5, на 7%.

Контрольно-оценочное задание 4
Создать триггер, который автоматически рассчитывает и заносит в отдельную таблицу "Зарплата" сумму к выплате преподавателю по итогам месяца (ставка * отработанные часы) при добавлении записи об окончании курса.

Контрольно-оценочное задание 5
Выполнить резервное копирование БД.
ChatGPT сказал:
Ниже — готовое решение Билета №4 (MySQL). Формат — как на экзамене, без лишнего.

Билет №4. Образовательный центр
Задание 1. Проектирование БД (3НФ)
Таблицы
Студенты

k_stud (PK)

fio_stud

phone

Преподаватели

k_prep (PK)

fio_prep

stavka

Курсы

k_kurs (PK)

n_kurs

stoim_kurs

chasy

data_start

data_end

k_prep (FK)

Зачисления

id_enr (PK)

k_stud (FK)

k_kurs (FK)

ocenka

Посещения

id_pos (PK)

id_enr (FK)

data_pos

status

Зарплата

id_zp (PK)

k_prep (FK)

mesyac

summa

ER-диаграмма (словесно)
Студент 1 — M Зачисление

Курс 1 — M Зачисление

Зачисление 1 — M Посещение

Преподаватель 1 — M Курс

Преподаватель 1 — M Зарплата

Задание 2. Создание БД и таблиц
CREATE DATABASE education;
USE education;

CREATE TABLE students (
  k_stud INT PRIMARY KEY,
  fio_stud VARCHAR(100) NOT NULL,
  phone VARCHAR(20) NOT NULL
);

CREATE TABLE teachers (
  k_prep INT PRIMARY KEY,
  fio_prep VARCHAR(100) NOT NULL,
  stavka DECIMAL(8,2) NOT NULL
);

CREATE TABLE courses (
  k_kurs INT PRIMARY KEY,
  n_kurs VARCHAR(100) NOT NULL,
  stoim_kurs DECIMAL(10,2) NOT NULL,
  chasy INT NOT NULL,
  data_start DATE NOT NULL,
  data_end DATE NOT NULL,
  k_prep INT NOT NULL,
  FOREIGN KEY (k_prep) REFERENCES teachers(k_prep)
);

CREATE TABLE enrollments (
  id_enr INT AUTO_INCREMENT PRIMARY KEY,
  k_stud INT NOT NULL,
  k_kurs INT NOT NULL,
  ocenka DECIMAL(3,2),
  FOREIGN KEY (k_stud) REFERENCES students(k_stud),
  FOREIGN KEY (k_kurs) REFERENCES courses(k_kurs)
);

CREATE TABLE attendance (
  id_pos INT AUTO_INCREMENT PRIMARY KEY,
  id_enr INT NOT NULL,
  data_pos DATE NOT NULL,
  status ENUM('явка','неявка') NOT NULL,
  FOREIGN KEY (id_enr) REFERENCES enrollments(id_enr)
);

CREATE TABLE salary (
  id_zp INT AUTO_INCREMENT PRIMARY KEY,
  k_prep INT NOT NULL,
  mesyac DATE NOT NULL,
  summa DECIMAL(10,2) NOT NULL,
  FOREIGN KEY (k_prep) REFERENCES teachers(k_prep)
);
Задание 3. SQL-запросы
1. Посещаемость студента по курсу
SELECT a.data_pos, a.status
FROM attendance a
JOIN enrollments e ON a.id_enr = e.id_enr
WHERE e.k_stud = 1 AND e.k_kurs = 2;
2. Курсы дороже средней стоимости
SELECT n_kurs, stoim_kurs
FROM courses
WHERE stoim_kurs > (SELECT AVG(stoim_kurs) FROM courses);
3. Студенты, не посетившие ни одного занятия
SELECT s.fio_stud
FROM students s
JOIN enrollments e ON s.k_stud = e.k_stud
LEFT JOIN attendance a ON e.id_enr = a.id_enr
WHERE a.id_pos IS NULL;
4. Повысить ставку преподавателям (ср. оценка > 4.5)
UPDATE teachers
SET stavka = stavka * 1.07
WHERE k_prep IN (
  SELECT c.k_prep
  FROM courses c
  JOIN enrollments e ON c.k_kurs = e.k_kurs
  GROUP BY c.k_prep
  HAVING AVG(e.ocenka) > 4.5
);
Задание 4. Триггер (расчет зарплаты)
DELIMITER $$

CREATE TRIGGER trg_salary_after_course
AFTER UPDATE ON courses
FOR EACH ROW
BEGIN
  IF NEW.data_end <> OLD.data_end THEN
    INSERT INTO salary (k_prep, mesyac, summa)
    VALUES (
      NEW.k_prep,
      LAST_DAY(NEW.data_end),
      NEW.chasy * (SELECT stavka FROM teachers WHERE k_prep = NEW.k_prep)
    );
  END IF;
END$$

DELIMITER ;
Задание 5. Резервное копирование
mysqldump -u root -p education > C:\Users\Имя_пользователя\Desktop\education_backup.sql
Если нужно:

готовые INSERT (100+ посещений)

ER-диаграмма в draw.io

упрощённый вариант для защиты

Напиши, что именно добавить.

Вы сказали:
Билет №5: Тема «Стройматериалы. Учет поставок и отгрузок»
Контрольно-оценочное задание 1
Спроектировать БД. Исходная таблица:
a. Код поставщика (k_post)
b. Наименование поставщика (n_post)
c. Код материала (k_mat)
d. Наименование материала (n_mat)
e. Единица измерения (ed_izm)
f. Цена за единицу у поставщика (cena_post)
g. Дата поставки (data_post)
h. Объем поставки (v_post)
i. Количество на складе (kol_sklad)
j. Цена отгрузки для клиента (cena_otgr) (Наценка 20%)
k. Дата отгрузки (data_otgr)
l. Объем отгрузки (v_otgr)
m. Код клиента (k_klient)
n. Наименование клиента (n_klient)
o. Код менеджера по продажам (k_man)
p. ФИО менеджера (fio_man)
q. Процент менеджера (procent)
Построить ER-диаграмму.

Контрольно-оценочное задание 2
Создать БД. Справочник материалов - не менее 12, поставщиков - не менее 4, клиентов - не менее 6, менеджеров - не менее 3. Данные о поставках и отгрузках - не менее 30 записей. Реализовать ограничения целостности.

Контрольно-оценочное задание 3
Создать запросы:

Отчет об отгрузках за квартал (клиент, материал, объем, сумма, менеджер).
Показать материалы, цена отгрузки которых выше средней цены на материалы той же единицы измерения.
Показать материалы, которые есть в каталоге, но баланс на складе ниже минимального запаса (минимальный запас - отдельное поле в справочнике материалов).
Уменьшить цену поставки для материалов от конкретного поставщика на 5% в связи с договором.

Контрольно-оценочное задание 4
Создать триггер, который при отгрузке материала клиенту проверяет достаточность остатка на складе. Если остаток достаточен - списывает его. Если нет - переводит требуемое количество в поле "Резерв" и создает запись в таблице "Заявки на закупку".

Контрольно-оценочное задание 5
Выполнить резервное копирование БД.
ChatGPT сказал:
Ниже — готовое решение Билета №5 (MySQL). Экзаменационный стиль, без лишнего.

Билет №5. Стройматериалы
Задание 1. Проектирование БД (3НФ)
Таблицы
Поставщики

k_post (PK)

n_post

Материалы

k_mat (PK)

n_mat

ed_izm

min_zapas — минимальный запас

kol_sklad

rezerv

Клиенты

k_klient (PK)

n_klient

Менеджеры

k_man (PK)

fio_man

procent

Поставки

id_postavki (PK)

k_post (FK)

k_mat (FK)

data_post

v_post

cena_post

Отгрузки

id_otgr (PK)

k_mat (FK)

k_klient (FK)

k_man (FK)

data_otgr

v_otgr

cena_otgr

Заявки_на_закупку

id_zak (PK)

k_mat (FK)

kolvo

data_zak

ER-диаграмма (словесно)
Поставщик 1 — M Поставка

Материал 1 — M Поставка

Материал 1 — M Отгрузка

Клиент 1 — M Отгрузка

Менеджер 1 — M Отгрузка

Материал 1 — M Заявка на закупку

Задание 2. Создание БД и таблиц
CREATE DATABASE building;
USE building;

CREATE TABLE suppliers (
  k_post INT PRIMARY KEY,
  n_post VARCHAR(100) NOT NULL
);

CREATE TABLE materials (
  k_mat INT PRIMARY KEY,
  n_mat VARCHAR(100) NOT NULL,
  ed_izm VARCHAR(20) NOT NULL,
  min_zapas INT NOT NULL,
  kol_sklad INT NOT NULL,
  rezerv INT DEFAULT 0
);

CREATE TABLE clients (
  k_klient INT PRIMARY KEY,
  n_klient VARCHAR(100) NOT NULL
);

CREATE TABLE managers (
  k_man INT PRIMARY KEY,
  fio_man VARCHAR(100) NOT NULL,
  procent DECIMAL(5,2) NOT NULL
);

CREATE TABLE supplies (
  id_postavki INT AUTO_INCREMENT PRIMARY KEY,
  k_post INT NOT NULL,
  k_mat INT NOT NULL,
  data_post DATE NOT NULL,
  v_post INT NOT NULL,
  cena_post DECIMAL(10,2) NOT NULL,
  FOREIGN KEY (k_post) REFERENCES suppliers(k_post),
  FOREIGN KEY (k_mat) REFERENCES materials(k_mat)
);

CREATE TABLE shipments (
  id_otgr INT AUTO_INCREMENT PRIMARY KEY,
  k_mat INT NOT NULL,
  k_klient INT NOT NULL,
  k_man INT NOT NULL,
  data_otgr DATE NOT NULL,
  v_otgr INT NOT NULL,
  cena_otgr DECIMAL(10,2) NOT NULL,
  FOREIGN KEY (k_mat) REFERENCES materials(k_mat),
  FOREIGN KEY (k_klient) REFERENCES clients(k_klient),
  FOREIGN KEY (k_man) REFERENCES managers(k_man)
);

CREATE TABLE purchase_requests (
  id_zak INT AUTO_INCREMENT PRIMARY KEY,
  k_mat INT NOT NULL,
  kolvo INT NOT NULL,
  data_zak DATE NOT NULL,
  FOREIGN KEY (k_mat) REFERENCES materials(k_mat)
);
Задание 3. SQL-запросы
1. Отчет об отгрузках за квартал
SELECT c.n_klient, m.n_mat, s.v_otgr,
       s.v_otgr * s.cena_otgr AS summa,
       man.fio_man
FROM shipments s
JOIN clients c ON s.k_klient = c.k_klient
JOIN materials m ON s.k_mat = m.k_mat
JOIN managers man ON s.k_man = man.k_man
WHERE QUARTER(s.data_otgr) = 2 AND YEAR(s.data_otgr) = 2025;
2. Материалы с ценой выше средней по единице измерения
SELECT n_mat, cena_otgr
FROM shipments sh
JOIN materials m ON sh.k_mat = m.k_mat
WHERE cena_otgr >
(
  SELECT AVG(sh2.cena_otgr)
  FROM shipments sh2
  JOIN materials m2 ON sh2.k_mat = m2.k_mat
  WHERE m2.ed_izm = m.ed_izm
);
3. Материалы ниже минимального запаса
SELECT n_mat, kol_sklad, min_zapas
FROM materials
WHERE kol_sklad < min_zapas;
4. Уменьшить цену поставки поставщика на 5%
UPDATE supplies
SET cena_post = cena_post * 0.95
WHERE k_post = 1;
Задание 4. Триггер (проверка остатков)
DELIMITER $$

CREATE TRIGGER trg_check_stock
BEFORE INSERT ON shipments
FOR EACH ROW
BEGIN
  DECLARE ostatok INT;

  SELECT kol_sklad INTO ostatok
  FROM materials
  WHERE k_mat = NEW.k_mat;

  IF ostatok >= NEW.v_otgr THEN
    UPDATE materials
    SET kol_sklad = kol_sklad - NEW.v_otgr
    WHERE k_mat = NEW.k_mat;
  ELSE
    UPDATE materials
    SET rezerv = rezerv + NEW.v_otgr
    WHERE k_mat = NEW.k_mat;

    INSERT INTO purchase_requests (k_mat, kolvo, data_zak)
    VALUES (NEW.k_mat, NEW.v_otgr - ostatok, CURDATE());

    SIGNAL SQLSTATE '45000'
    SET MESSAGE_TEXT = 'Недостаточно материала. Создана заявка на закупку';
  END IF;
END$$

DELIMITER ;
Задание 5. Резервное копирование БД
mysqldump -u root -p building > C:\Users\Имя_пользователя\Desktop\building_backup.sql
