
Q2
我的答案
```sql
SELECT
	primary_title,
	type,
	runtime_minutes 
FROM
	( SELECT * , RANK( ) OVER ( PARTITION BY type ORDER BY runtime_minutes DESC ) AS rank FROM titles ) AS ranking 
WHERE
	ranking.rank = 1
```

标准答案
```sql
WITH types(type, runtime_minutes) AS ( 
  SELECT type, MAX(runtime_minutes)
    FROM titles
    GROUP BY type
)
SELECT titles.type, titles.primary_title, titles.runtime_minutes
  FROM titles
  JOIN types
  ON titles.runtime_minutes == types.runtime_minutes AND titles.type == types.type
  ORDER BY titles.type, titles.primary_title
  ;
```

Q3
我的答案
```sql
SELECT type, count(1) as count FROM titles GROUP BY type ORDER BY count ASC
```

标准答案
```sql
SELECT type, count(*) AS title_count FROM titles GROUP BY type ORDER BY title_count ASC;
```

Q4
我的答案
```sql
SELECT
	( premiered / 10 ) || '0s' AS decade,
	count( * ) AS count 
FROM
	titles 
WHERE
	premiered IS NOT NULL 
GROUP BY
	decade 
ORDER BY
	count DESC
```

标准答案
```sql
SELECT 
  CAST(premiered/10*10 AS TEXT) || 's' AS decade,
  COUNT(*) AS num_movies
  FROM titles
  WHERE premiered is not null
  GROUP BY decade
  ORDER BY num_movies DESC
  ;

```

Q5
我的答案
```sql
WITH allCount as (
	SELECT COUNT(*) as count FROM titles
)
SELECT
	( premiered / 10 ) || '0s' AS decade,
	ROUND(count(*) * 100.0 / allCount.count, 4)  AS count
FROM
	titles, allCount
WHERE
	premiered IS NOT NULL 
GROUP BY
	decade 
ORDER BY
	count DESC
```

标准答案
```sql
SELECT
  CAST(premiered/10*10 AS TEXT) || 's' AS decade,
  ROUND(CAST(COUNT(*) AS REAL) / (SELECT COUNT(*) FROM titles) * 100.0, 4) as percentage
  FROM titles
  WHERE premiered is not null
  GROUP BY decade
  ORDER BY percentage DESC, decade ASC
  ;

```

Q6
我的答案
```sql
SELECT
	titles.primary_title,
	akas.count 
FROM
	( SELECT title_id, COUNT( * ) AS count FROM akas GROUP BY title_id ORDER BY count DESC LIMIT 10 ) AS akas
	LEFT JOIN ( SELECT title_id, primary_title FROM titles ) AS titles ON titles.title_id = akas.title_id
```

标准答案
```sql
WITH translations AS (
  SELECT title_id, count(*) as num_translations 
    FROM akas 
    GROUP BY title_id 
    ORDER BY num_translations DESC, title_id 
    LIMIT 10
)
SELECT titles.primary_title, translations.num_translations
  FROM translations
  JOIN titles
  ON titles.title_id == translations.title_id
  ORDER BY translations.num_translations DESC
  ;
```

Q7
我的答案
```sql
WITH cInfo as(
	SELECT sum(rating * votes) / sum(votes) as c FROM ratings
), mInfo as (
	SELECT 25000.0 as m
), WRInfo as (
	SELECT (votes/(votes + mInfo.m)) * rating  + (mInfo.m/(votes + mInfo.m)) * cInfo.c as wr, title_id from ratings,mInfo,cInfo
)
SELECT titles.primary_title, top250.wr FROM (SELECT * from WRInfo ORDER BY wr DESC LIMIT 250) as top250
JOIN (SELECT title_id, primary_title FROM titles) as titles
on titles.title_id = top250.title_id
```

标准答案
```sql
WITH
  av(average_rating) AS (
    SELECT SUM(rating * votes) / SUM(votes)
      FROM ratings
      JOIN titles
      ON titles.title_id == ratings.title_id AND titles.type == 'movie' 
  ),
  mn(min_rating) AS (SELECT 25000.0)
SELECT
  primary_title,
  (votes / (votes + min_rating)) * rating + (min_rating / (votes + min_rating)) * average_rating as weighed_rating
  FROM ratings, av, mn
  JOIN titles
  ON titles.title_id == ratings.title_id and titles.type == 'movie'
  ORDER BY weighed_rating DESC
  LIMIT 250
  ;

```

Q8
我的答案
```sql
SELECT
	count(DISTINCT(person_id))
FROM
	crew 
WHERE
	title_id IN ( SELECT title_id FROM crew WHERE person_id = ( SELECT person_id FROM people WHERE name = 'Mark Hamill' AND born = 1951 ) ) 
	AND (category = 'actor' 
	OR category = 'actress')
```

标准答案
```sql
WITH hamill_titles AS (
  SELECT DISTINCT(crew.title_id)
    FROM people
    JOIN crew
    ON crew.person_id == people.person_id AND people.name == 'Mark Hamill' AND people.born == 1951
)
SELECT COUNT(DISTINCT(crew.person_id))
  FROM crew
  WHERE (crew.category == 'actor' OR crew.category == 'actress') AND crew.title_id in hamill_titles
```

Q9
我的答案
```sql
WITH movies1 as (
	SELECT title_id FROM crew
	join people on crew.person_id = people.person_id AND name = 'Mark Hamill' AND born = 1951
), movies2 as (
	SELECT title_id FROM crew
	join people on crew.person_id = people.person_id AND name = 'George Lucas' AND born = 1944
)
SELECT * FROM titles, movies1, movies2
WHERE titles.title_id in (SELECT movies1.title_id INTERSECT SELECT movies2.title_id) and titles.type = 'movie'
```

标准答案
```sql
WITH hamill_movies(title_id) AS (
  SELECT crew.title_id
    FROM crew
    JOIN people
    ON crew.person_id == people.person_id AND people.name == 'Mark Hamill' AND people.born == 1951
)
SELECT titles.primary_title
  FROM crew
  JOIN people
  ON crew.person_id == people.person_id AND people.name == 'George Lucas' AND people.born == 1944 AND crew.title_id IN hamill_movies
  JOIN titles
  ON crew.title_id == titles.title_id AND titles.type == 'movie'
  ORDER BY titles.primary_title
;

```

Q10
我的答案
```sql
WITH split(word, str) AS (
    SELECT '', genres ||',' FROM titles
    UNION ALL SELECT
    substr(str, 0, instr(str, ',')),
    substr(str, instr(str, ',')+1)
    FROM split WHERE str!=''
)

SELECT word, count(word) as count FROM split WHERE word!='\N' and word != '' GROUP BY word ORDER BY count desc;
```

标准答案
```sql
WITH RECURSIVE split(genre, rest) AS (
  SELECT '', genres || ',' FROM titles WHERE genres != '\N'
   UNION ALL
  SELECT substr(rest, 0, instr(rest, ',')),
         substr(rest, instr(rest, ',')+1)
    FROM split
   WHERE rest != ''
)
SELECT genre, count(*) as genre_count
  FROM split 
 WHERE genre != ''
 GROUP BY genre
 ORDER BY genre_count DESC;
```