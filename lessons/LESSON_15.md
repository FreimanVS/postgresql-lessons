## ИНДЕКСЫ И ОПТИМИЗАЦИЯ ЗАПРОСОВ

#### ПОДГОТОВКА ДАННЫХ
``` sql
-- Удаление таблицы, если она существует
DROP TABLE IF EXISTS my_table;
-- Создание новой таблицы
CREATE TABLE IF NOT EXISTS my_table(
    random_num INTEGER,
    random_text TEXT,
    bool_field BOOLEAN
);
-- Вставка данных в таблицу
INSERT INTO my_table(random_num, random_text, bool_field)
SELECT
    s.id,
    chr((32 + random() * 94)::INTEGER),
    random() < 0.01
FROM generate_series(1, 100000) AS s(id)
ORDER BY random();
```
### ОБЫЧНЫЙ ИНДЕКС
#### по числовому полю
``` sql
-- Создание индекса по числовому полю
CREATE INDEX ON my_table(random_num);
-- Анализ таблицы для обновления статистики
ANALYZE my_table;
-- План запроса для выборки по точному значению
EXPLAIN SELECT * FROM my_table WHERE random_num = 1;

-- Результат: 
|QUERY PLAN                                                                                            |
|------------------------------------------------------------------------------------------------------|
|Index Scan using my_table_random_num_idx on my_table  (cost=0.29..8.31 rows=1 width=7)|
|  Index Cond: (random_num = 1)                                                                        |

DROP INDEX my_table_random_num_idx;
```
#### по текстовому полю
``` sql
-- Создание индекса по текстовому полю
CREATE INDEX ON my_table(random_text);

ANALYZE my_table;

EXPLAIN SELECT * FROM my_table WHERE
random_text = 'a';

-- Результат: 
Index Scan using my_table_random_text_idx on my_table  (cost=0.29..30.67 rows=1050 width=7)
  Index Cond: (random_text = 'a'::text)

DROP INDEX my_table_random_text_idx;
```
### СОСТАВНОЙ ИНДЕКС
``` sql
-- Создание составного индекса
CREATE INDEX ON my_table(random_num, random_text);

ANALYZE my_table;

-- План запроса с использованием составного индекса
EXPLAIN SELECT * FROM my_table WHERE random_num <= 100 AND
random_text = 'a';

-- Результат: 
Index Scan using my_table_random_num_random_text_idx on my_table  (cost=0.29..9.30 rows=1 width=7)
  Index Cond: ((random_num <= 100) AND (random_text = 'a'::text))

DROP INDEX my_table_random_num_random_text_idx;
```
### ИНДЕКС ПО ФУНКЦИИ
``` sql
-- Создание индекса по функции
CREATE INDEX ON my_table(LOWER(random_text));
-- Анализ таблицы для обновления статистики
ANALYZE my_table;
-- Экспланация анализируемого запроса с условием на текстовое поле
EXPLAIN SELECT * FROM my_table WHERE LOWER(random_text) =
'a';

-- Результат: 
Bitmap Heap Scan on my_table  (cost=24.23..498.09 rows=2057 width=7)
  Recheck Cond: (lower(random_text) = 'a'::text)
  ->  Bitmap Index Scan on my_table_lower_idx  (cost=0.00..23.72 rows=2057 width=0)
        Index Cond: (lower(random_text) = 'a'::text)

DROP INDEX my_table_lower_idx;
```
### ИНДЕКС ДЛЯ ПОЛНОТЕКСТОВОГО ПОИСКА
``` sql
-- Создание таблицы статей
CREATE TABLE articles (
    id SERIAL PRIMARY KEY,
    title TEXT,
    content TEXT
);

-- Вставка данных в таблицу статей
INSERT INTO articles (title, content) VALUES
('PostgreSQL Tutorial', 'This tutorial covers the basics of PostgreSQL.'),
('Full-Text Search in PostgreSQL', 'Learn how to use full-text search in
PostgreSQL.'),
('Advanced PostgreSQL Features', 'Explore advanced features of PostgreSQL,
including GIN indexes and full-text search.');

-- Добавление столбца tsvector с автоматическим заполнением для полнотекстового поиска
ALTER TABLE articles ADD COLUMN content_tsvector TSVECTOR GENERATED ALWAYS
AS (to_tsvector('english', content)) STORED;

-- Просмотр содержимого таблицы статей
SELECT * FROM articles;

-- Создание GIN индекса для столбца tsvector
CREATE INDEX idx_articles_content_tsvector ON articles USING gin
(content_tsvector);

-- Отключение последовательного сканирования для демонстрации использования индекса, 
-- так как данных в таблице слишком мало и последовательный поиск выполнится быстрее */
SET enable_seqscan = OFF;

-- Экспланация запроса полнотекстового поиска по статьям
EXPLAIN SELECT title, content FROM articles WHERE content_tsvector @@
to_tsquery('english', 'PostgreSQL & full-text');

-- Результат: 
Bitmap Heap Scan on articles  (cost=20.00..24.01 rows=1 width=64)
  Recheck Cond: (content_tsvector @@ '''postgresql'' & ''full-text'' <-> ''full'' <-> ''text'''::tsquery)
  ->  Bitmap Index Scan on idx_articles_content_tsvector  (cost=0.00..20.00 rows=1 width=0)
        Index Cond: (content_tsvector @@ '''postgresql'' & ''full-text'' <-> ''full'' <-> ''text'''::tsquery)
        
-- Включение последовательного сканирования обратно
SET enable_seqscan = ON;
```
#### УТИЛИЗАЦИЯ ДАННЫХ
``` sql
DROP TABLE IF EXISTS my_table;
```

