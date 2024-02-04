## Postgres in docker

### Подготовка
``` sql
--Cоздать файл с названием docker-compose.yml и следующим содержимым: 
version: "3.1"

volumes:
    pg_project:

services:
  postgres15:
    container_name: postgres15
    image: postgres:15
    restart: always
    environment:
      - POSTGRES_PASSWORD=postgres
      - POSTGRES_USER=postgres
      - POSTGRES_DB=postgres
    volumes:
        - pg_project:/var/lib/postgresql/data
    ports:
        - "${POSTGRES_PORT:-5439}:5432"
```
### TERMINAL 1
``` sql
--Поднять container с postgres
docker compose up -d

--Убедиться, что он поднялся
docker ps -a

Результат: 
CONTAINER ID   IMAGE         COMMAND                  CREATED          STATUS          PORTS                                       NAMES
08cf84d2662c   postgres:15   "docker-entrypoint.s…"   14 minutes ago   Up 14 minutes   0.0.0.0:5439->5432/tcp, :::5439->5432/tcp   postgres15
```
### TERMINAL 2
``` sql
-- Подключиться к БД через psql
psql -h localhost -U postgres -p 5439 -d postgres

-- Удаление таблицы, если она существует
DROP TABLE IF EXISTS my_table;
-- Создание новой таблицы
CREATE TABLE IF NOT EXISTS my_table(
    text TEXT
);
-- Вставка данных в таблицу
insert into my_table values('table1', 'table2');

-- Проверить, что данные добавились
select * from my_table;

-- Результат: 
  text  
--------
 table1
 table2
(2 rows)

```
### TERMINAL 1
``` sql
--удаляем контейнер с postgres
docker stop 08cf84d2662c
docker rm 08cf84d2662c
```
### TERMINAL 2
``` sql
--убедимся, что контейнер упал
psql -h localhost -U postgres -p 5439 -d postgres 
--Результат: 
psql: error: connection to server at "localhost" (127.0.0.1), port 5439 failed: Connection refused
 Is the server running on that host and accepting TCP/IP connections?
```
### TERMINAL 1
``` sql
--поднимаем новый контейнер с postgres
docker compose up -d

--Результат: 
[+] Running 1/1
 
￼
 Container postgres15  Started  
 
 docker ps -a
 
 result: 
 
 CONTAINER ID   IMAGE         COMMAND                  CREATED          STATUS          PORTS                                       NAMES
5ef3133237d1   postgres:15   "docker-entrypoint.s…"   40 seconds ago   Up 39 seconds   0.0.0.0:5439->5432/tcp, :::5439->5432/tcp   postgres15
```
### TERMINAL 2
``` sql
psql -h localhost -U postgres -p 5439 -d postgres 

select * from my_table;

--Результат: 
  text  
--------
 table1
 table2
(2 rows)
```
### Вывод
``` sql
-- данные сохранились на HDD основной ОС, падение контейнера docker не повлияло потерей данных
```
