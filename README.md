# full_stack_project1_log_analysis
# List of all relational database
List of relations
 Schema |   Name   | Type  |  Owner
 --------+----------+-------+---------
 public | articles | table | vagrant
 public | authors  | table | vagrant
 public | log      | table | vagrant
(3 rows)

## 1.	Public.articles
   Table "public.articles"
 Column |           Type           |                       Modifiers           
--------+--------------------------+-------------------------------------------------------
 author | integer                  | not null
 title  | text                     | not null
 slug   | text                     | not null
 lead   | text                     |
 body   | text                     |
 time   | timestamp with time zone | default now()
 id     | integer                  | not null default nextval('articles_id_seq'::regclass)
Indexes:
    "articles_pkey" PRIMARY KEY, btree (id)
    "articles_slug_key" UNIQUE CONSTRAINT, btree (slug)
Foreign-key constraints:
    "articles_author_fkey" FOREIGN KEY (author) REFERENCES authors(id)




## 2.	Public.authors
Table "public.authors"
 Column |  Type   |                      Modifiers
--------+---------+------------------------------------------------------
 name   | text    | not null
 bio    | text    |
 id     | integer | not null default nextval('authors_id_seq'::regclass)
Indexes:
    "authors_pkey" PRIMARY KEY, btree (id)
Referenced by:
    TABLE "articles" CONSTRAINT "articles_author_fkey" FOREIGN KEY (author) REFERENCES authors(id)



## 3. public.log
Table "public.log"
 Column |           Type           |                    Modifiers              
--------+--------------------------+--------------------------------------------------
 path   | text                     |
 ip     | inet                     |
 method | text                     |
 status | text                     |
 time   | timestamp with time zone | default now()
 id     | integer                  | not null default nextval('log_id_seq'::regclass)


# project:
## 1.	What are the most popular three articles of all time?
select path, count(path) from public.log group by path order by count desc limit 4;

**this will return:**
/article/candidate-is-jerk  | 338647
 /article/bears-love-berries | 253801
 /article/bad-things-gone    | 170098


## 2.	Who are the most popular article authors of all time?
select id,author,title from public.articles order by author;

this will return:
id | author |               title
----+--------+------------------------------------
 29 |      1 | There are a lot of bears
 25 |      1 | Bears love berries, alleges bear
 27 |      1 | Goats eat Google's lawn
 28 |      1 | Media obsessed with bears
 30 |      2 | Trouble for troubled troublemakers
 26 |      2 | Candidate is jerk, alleges rival
 23 |      3 | Bad things gone, say good people
 24 |      4 | Balloon goons doomed
(8 rows)

Next:
select id, name from public.authors;

this will return:
id |          name
----+------------------------
  1 | Ursula La Multa
  2 | Rudolf von Treppenwitz
  3 | Anonymous Contributor
  4 | Markoff Chaney
(4 rows)

Next: join the above tables (real query starts from here):
select public.authors.name, public.articles.title from public.articles join public.authors on public.authors.id=public.articles.author order by name desc;

this will return:
name          |               title
------------------------+------------------------------------
 Ursula La Multa        | There are a lot of bears
 Ursula La Multa        | Bears love berries, alleges bear
 Ursula La Multa        | Goats eat Google's lawn
 Ursula La Multa        | Media obsessed with bears
 Rudolf von Treppenwitz | Trouble for troubled troublemakers
 Rudolf von Treppenwitz | Candidate is jerk, alleges rival
 Markoff Chaney         | Balloon goons doomed
 Anonymous Contributor  | Bad things gone, say good people
(8 rows)

Next step is to know the number of views of each title and add the column to the right according to title name:

select row_number() over(), substring(path from 10), count(path) from public.log group by path order by count desc limit 9;

this will return:
row_number |         substring         | count
------------+---------------------------+--------
         28 |                           | 479121
        103 | candidate-is-jerk         | 338647
        110 | bears-love-berries        | 253801
        117 | bad-things-gone           | 170098
         33 | goats-eat-googles         |  84906
         91 | trouble-for-troubled      |  84810
         41 | balloon-goons-doomed      |  84557
        150 | so-many-bears             |  84504
        158 | media-obsessed-with-bears |  84383
(9 rows)

select slug from public.articles;
this will return:

---------------------------
 bad-things-gone
 balloon-goons-doomed
 bears-love-berries
 candidate-is-jerk
 goats-eat-googles
 media-obsessed-with-bears
 trouble-for-troubled
 so-many-bears
(8 rows)

select authors.name, articles.slug from articles join authors on authors.id=articles.author order by name desc;
this will return:
name          |           slug
------------------------+---------------------------
 Ursula La Multa        | so-many-bears
 Ursula La Multa        | bears-love-berries
 Ursula La Multa        | goats-eat-googles
 Ursula La Multa        | media-obsessed-with-bears
 Rudolf von Treppenwitz | trouble-for-troubled
 Rudolf von Treppenwitz | candidate-is-jerk
 Markoff Chaney         | balloon-goons-doomed
 Anonymous Contributor  | bad-things-gone
(8 rows)

Name the above 2 tables as new tables:
create table mytable1 as select authors.name, articles.slug from articles join authors on authors.id=articles.author order by name desc;

create table mytable2 as select row_number() over(), substring(path from 10), count(path) from public.log group by path order by count desc limit 9;



select * from mytable1;
this will return:
          name          |           slug
------------------------+---------------------------
 Ursula La Multa        | so-many-bears
 Ursula La Multa        | bears-love-berries
 Ursula La Multa        | goats-eat-googles
 Ursula La Multa        | media-obsessed-with-bears
 Rudolf von Treppenwitz | trouble-for-troubled
 Rudolf von Treppenwitz | candidate-is-jerk
 Markoff Chaney         | balloon-goons-doomed
 Anonymous Contributor  | bad-things-gone
(8 rows)

select * from mytable2;
this will return:
row_number |         substring         | count
------------+---------------------------+--------
         28 |                           | 479121
        103 | candidate-is-jerk         | 338647
        110 | bears-love-berries        | 253801
        117 | bad-things-gone           | 170098
         33 | goats-eat-googles         |  84906
         91 | trouble-for-troubled      |  84810
         41 | balloon-goons-doomed      |  84557
        150 | so-many-bears             |  84504
        158 | media-obsessed-with-bears |  84383
(9 rows)



Next, we join the above two tables together:
create table mytable3 as select name,slug,count from mytable1 join mytable2 on mytable1.slug = mytable2.substring;
name          |           slug            | count
------------------------+---------------------------+--------
 Anonymous Contributor  | bad-things-gone           | 170098
 Markoff Chaney         | balloon-goons-doomed      |  84557
 Ursula La Multa        | bears-love-berries        | 253801
 Rudolf von Treppenwitz | candidate-is-jerk         | 338647
 Ursula La Multa        | goats-eat-googles         |  84906
 Ursula La Multa        | media-obsessed-with-bears |  84383
 Ursula La Multa        | so-many-bears             |  84504
 Rudolf von Treppenwitz | trouble-for-troubled      |  84810
(8 rows)


Next, we sum the count of each author:
We should use aggregate (http://www.postgresqltutorial.com/postgresql-grouping-sets/):

select name, sum(count) from mytable3 group by name order by sum desc;

**this will return:**
name          |  sum
------------------------+--------
 Ursula La Multa        | 507594
 Rudolf von Treppenwitz | 423457
 Anonymous Contributor  | 170098
 Markoff Chaney         |  84557
(4 rows)






## 3.	On which days did more than 1% of requests lead to errors?

select status,count(status) from log group by status;
this will return:
status     |  count
---------------+---------
 404 NOT FOUND |   12908
 200 OK        | 1664827
(2 rows)

create table mytable4 as select time,status from log where status='404 NOT FOUND';     (Note that MUST use single quotes)
select * from mytable4 limit 20;
this will return:
time          |    status
------------------------+---------------
 2016-07-01 07:07:24+00 | 404 NOT FOUND
 2016-07-01 07:07:54+00 | 404 NOT FOUND
 2016-07-01 07:07:59+00 | 404 NOT FOUND
 2016-07-01 07:09:03+00 | 404 NOT FOUND
 2016-07-01 07:11:35+00 | 404 NOT FOUND
 2016-07-01 07:15:27+00 | 404 NOT FOUND
 2016-07-01 07:19:09+00 | 404 NOT FOUND
 2016-07-01 07:25:36+00 | 404 NOT FOUND
 2016-07-01 07:28:53+00 | 404 NOT FOUND
 2016-07-01 07:30:37+00 | 404 NOT FOUND
 2016-07-01 07:31:27+00 | 404 NOT FOUND
 2016-07-01 07:32:32+00 | 404 NOT FOUND
 2016-07-01 07:33:16+00 | 404 NOT FOUND
 2016-07-01 07:33:38+00 | 404 NOT FOUND
 2016-07-01 07:34:33+00 | 404 NOT FOUND
 2016-07-01 07:34:28+00 | 404 NOT FOUND
 2016-07-01 07:44:14+00 | 404 NOT FOUND
 2016-07-01 07:46:27+00 | 404 NOT FOUND
 2016-07-01 07:46:46+00 | 404 NOT FOUND
 2016-07-01 07:55:23+00 | 404 NOT FOUND
(20 rows)


We need to extract date, but ATTENTION, time is a timestamp type, not a string type, so we use date() function to convert (https://stackoverflow.com/questions/6133107/extract-date-yyyy-mm-dd-from-a-timestamp-in-postgresql):

create table mytable5 as select date(time) as date, time, status from mytable4;

select * from mytable5 limit 10;

this will return:
date    |          time          |    status
------------+------------------------+---------------
 2016-07-01 | 2016-07-01 07:07:24+00 | 404 NOT FOUND
 2016-07-01 | 2016-07-01 07:07:54+00 | 404 NOT FOUND
 2016-07-01 | 2016-07-01 07:07:59+00 | 404 NOT FOUND
 2016-07-01 | 2016-07-01 07:09:03+00 | 404 NOT FOUND
 2016-07-01 | 2016-07-01 07:11:35+00 | 404 NOT FOUND
 2016-07-01 | 2016-07-01 07:15:27+00 | 404 NOT FOUND
 2016-07-01 | 2016-07-01 07:19:09+00 | 404 NOT FOUND
 2016-07-01 | 2016-07-01 07:25:36+00 | 404 NOT FOUND
 2016-07-01 | 2016-07-01 07:28:53+00 | 404 NOT FOUND
 2016-07-01 | 2016-07-01 07:30:37+00 | 404 NOT FOUND
(10 rows)

Then we apply the aggregation:
create table mytable6 as select date, count(date) from mytable5 group by date order by count(date);

this will return mytable6:
date    | count
------------+-------
 2016-07-01 |   274
 2016-07-31 |   329
 2016-07-07 |   360
 2016-07-27 |   367
 2016-07-10 |   371
 2016-07-12 |   373
 2016-07-23 |   373
 2016-07-16 |   374
 2016-07-18 |   374
 2016-07-04 |   380
 2016-07-29 |   382
 2016-07-20 |   383
 2016-07-14 |   383
 2016-07-13 |   383
 2016-07-02 |   389
 2016-07-25 |   391
 2016-07-28 |   393
 2016-07-26 |   396
 2016-07-30 |   397
 2016-07-03 |   401
 2016-07-11 |   403
 2016-07-22 |   406
 2016-07-15 |   408
 2016-07-09 |   410
 2016-07-21 |   418
 2016-07-08 |   418
 2016-07-06 |   420
 2016-07-05 |   423
 2016-07-24 |   431
 2016-07-19 |   433
 2016-07-17 |  1265
(31 rows)

The same goes for all links:

create table mytable7 as select time,status from log; 
create table mytable8 as select date(time) as date, time, status from mytable7;
create table mytable9 as select date, count(date) as all_count from mytable8 group by date order by count(date);


this will return mytable9:
date    | all_count
------------+-----------
 2016-07-01 |     38705
 2016-07-31 |     45845
 2016-07-26 |     54378
 2016-07-27 |     54489
 2016-07-10 |     54489
 2016-07-11 |     54497
 2016-07-16 |     54498
 2016-07-20 |     54557
 2016-07-05 |     54585
 2016-07-25 |     54613
 2016-07-07 |     54740
 2016-07-06 |     54774
 2016-07-28 |     54797
 2016-07-12 |     54839
 2016-07-03 |     54866
 2016-07-23 |     54894
 2016-07-04 |     54903
 2016-07-29 |     54951
 2016-07-15 |     54962
 2016-07-30 |     55073
 2016-07-08 |     55084
 2016-07-24 |     55100
 2016-07-13 |     55180
 2016-07-14 |     55196
 2016-07-02 |     55200
 2016-07-22 |     55206
 2016-07-09 |     55236
 2016-07-21 |     55241
 2016-07-19 |     55341
 2016-07-18 |     55589
 2016-07-17 |     55907
(31 rows)


Next, combine mytable6 and mytabl9 together:

create table mytable10 as select mytable6.date,mytable6.count,mytable9.all_count from mytable6 join mytable9 on mytable9.date = mytable6.date;

date    | count | all_count
------------+-------+-----------
 2016-07-01 |   274 |     38705
 2016-07-02 |   389 |     55200
 2016-07-03 |   401 |     54866
 2016-07-04 |   380 |     54903
 2016-07-05 |   423 |     54585
 2016-07-06 |   420 |     54774
 2016-07-07 |   360 |     54740
 2016-07-08 |   418 |     55084
 2016-07-09 |   410 |     55236
 2016-07-10 |   371 |     54489
 2016-07-11 |   403 |     54497
 2016-07-12 |   373 |     54839
 2016-07-13 |   383 |     55180
 2016-07-14 |   383 |     55196
 2016-07-15 |   408 |     54962
 2016-07-16 |   374 |     54498
 2016-07-17 |  1265 |     55907
 2016-07-18 |   374 |     55589
 2016-07-19 |   433 |     55341
 2016-07-20 |   383 |     54557
 2016-07-21 |   418 |     55241
 2016-07-22 |   406 |     55206
 2016-07-23 |   373 |     54894
 2016-07-24 |   431 |     55100
 2016-07-25 |   391 |     54613
 2016-07-26 |   396 |     54378
 2016-07-27 |   367 |     54489
 2016-07-28 |   393 |     54797
 2016-07-29 |   382 |     54951
 2016-07-30 |   397 |     55073
 2016-07-31 |   329 |     45845
(31 rows)

Divided by all_count: (https://stackoverflow.com/questions/1780242/postgres-math-expression-calculcated-for-each-row-in-table)

create table mytable_final as select mytable10.date, count/all_count::float as div from mytable10;

  date    |         div
------------+---------------------
 2016-07-01 | 0.00707918873530552
 2016-07-02 | 0.00704710144927536
 2016-07-03 | 0.00730871578026464
 2016-07-04 | 0.00692129756115331
 2016-07-05 | 0.00774938169826875
 2016-07-06 | 0.00766787161792091
 2016-07-07 | 0.00657654366094264
 2016-07-08 | 0.00758841042771041
 2016-07-09 | 0.00742269534361648
 2016-07-10 | 0.00680871368533098
 2016-07-11 | 0.00739490247169569
 2016-07-12 | 0.00680172869673043
 2016-07-13 | 0.00694092062341428
 2016-07-14 | 0.00693890861656642
 2016-07-15 | 0.00742331065099523
 2016-07-16 | 0.00686263716099673
 2016-07-17 |  0.0226268624680273
 2016-07-18 | 0.00672794977423591
 2016-07-19 | 0.00782421712654271
 2016-07-20 | 0.00702018072841249
 2016-07-21 |  0.0075668434677142
 2016-07-22 | 0.00735427308625874
 2016-07-23 | 0.00679491383393449
 2016-07-24 | 0.00782214156079855
 2016-07-25 | 0.00715946752604691
 2016-07-26 | 0.00728235683548494
 2016-07-27 | 0.00673530437335976
 2016-07-28 | 0.00717192547037247
 2016-07-29 | 0.00695164783170461
 2016-07-30 | 0.00720861402138979
 2016-07-31 | 0.00717635510960846
(31 rows)



select date,div from mytable_final where div>0.01;

**this will return:**
date    |        div
------------+--------------------
 2016-07-17 | 0.0226268624680273
(1 row)

Solution is over.
