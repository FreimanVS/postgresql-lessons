## Уровни изоляции

### read commited (default)
``` sql
/* session A */
\set AUTOCOMMIT off
show transaction isolation level; --read commited
create table persons(id serial, first_name text, second_name text);
insert into persons(first_name, second_name) values('ivan', 'ivanov');
insert into persons(first_name, second_name) values('petr', 'petrov');
commit;
insert into persons(first_name, second_name) values('sergey', 'sergeev');
```
``` sql
/* session B */
\set AUTOCOMMIT off
show transaction isolation level; --read commited
select from persons; -- не видно новой записи, потому что session A ещё не закоммитила изменения
```
``` sql
/* session A */
commit;
```
``` sql
/* session B */
select from persons; -- теперь запись видно, потому что в session A был выполнен commit
```

### repeatable read
``` sql
/* session A */
set transaction isolation level repeatable read;
```
``` sql
/* session B */
set transaction isolation level repeatable read;
```
``` sql
/* session A */
insert into persons(first_name, second_name) values('sveta', 'svetova');
```
``` sql
/* session B */
select* from persons; --не видно новой записи, потому что session A и session B не закоммитили изменения
```
``` sql
/* session A */
commit;
```
``` sql
/* session B */
select* from persons; --не видно новой записи, потому что session A и session B не закоммитили изменения
commit;
select* from persons; --теперь видно новую запись
```