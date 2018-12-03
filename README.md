## full_stack_project1_log_analysis
#1. install vagrant and Virualbox<br />
#2. download github repository [fullstack-nandegree-vm repository](https://github.com/udacity/fullstack-nanodegree-vm)<br />
#3. download data from (https://d17h27t6h515a5.cloudfront.net/topher/2016/August/57b5f748_newsdata/newsdata.zip) <br />
#4. unzip this file after downloading it. The file inside is called newsdata.sql. Put this file into the vagrant directory <br />
## first we should setup the following steps:
#To load the data, cd into the vagrant directory and then type the followinig in the command line: <br />
#1. vagrant up <br />
#2. vagrant ssh <br />
#3. psql -d news -f newsdata.sql <br />
#4. psql -d news <br />
#5. the command line will change to "news =>", indicating that we're now in the "news" database <br />
#6. then, we enter the following commands for question 2: <br />
# question 2: pre-queries:<br />
### PSQL command to create view in the "psql -d news"
select public.authors.name, public.articles.title from public.articles join public.authors on public.authors.id=public.articles.author order by name desc;<br /><br />
create view mytable1 as select authors.name, articles.slug from articles join authors on authors.id=articles.author order by name desc;<br /><br />
create view mytable2 as select row_number() over(), substring(path from 10), count(path) from public.log group by path order by count desc limit 9;<br /><br />
create view mytable3 as select name,slug,count from mytable1 join mytable2 on mytable1.slug = mytable2.substring;<br /><br />
# question 3: pre-queries:<br />
### PSQL command to create view in the "psql -d news"
create view mytable4 as select time,status from log where status='404 NOT FOUND';<br /><br />
create view mytable5 as select date(time) as date, time, status from mytable4;<br /><br />
create view mytable6 as select date, count(date) from mytable5 group by date order by count(date);<br /><br />
create view mytable7 as select time,status from log;<br /><br />
create view mytable8 as select date(time) as date, time, status from mytable7;<br /><br />
create view mytable9 as select date, count(date) as all_count from mytable8 group by date order by count(date);<br /><br />
create view mytable10 as select mytable6.date,mytable6.count,mytable9.all_count from mytable6 join mytable9 on mytable9.date = mytable6.date;<br /><br />
create view mytable_final as select mytable10.date, count/all_count::float as div from mytable10;<br /><br />
