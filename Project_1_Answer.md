<h1 style="text-align:center">Project_1_QuestionAnswer</h1>
<p style="text-align:center">By Hanzhi Yang</p>

# 1. Question 1
```sql
select movie
from appeared_in
where star = 'Edward Norton'
```

| movie |
|:-|
|'Fight Club'|
|'The Illusionist'|
|'The Incredible Hulk'|

# 2. Question 2
```sql
select star
from appeared_in
where star <> 'Brad Pitt' and movie in (
    select movie
    from appeared_in
    where star = 'Brad Pitt')
```
| star | 
|:-|
|'Edward Norton'|
|'Angelina Jolie'|
|'George Clooney'|
|'Matt Damon'|
|'Vincent Cassel'|

# 3. Question 3
```sql
select sum(how_much)
from made_money
where movie in (
    select movie 
    from appeared_in 
    where star = 'Tom Hanks' and movie in (
        select movie
        from appeared_in
        where star = 'Rita Wilson'
        )
    )
```
Totoal money is : **51444736.00**

# 4. Question 4
```sql
select star
from in_couple
where star <> 'Ben Affleck' and couple_num in (
    select couple_num 
    from divorced 
    where couple_num in (
        select couple_num
        from in_couple
        where star = 'Ben Affleck'
    )
)
```
|star|
|:-|
|Jennifer Garner|


# 5. Question 5
```sql
select star
from in_couple
where couple_num in (
    select couple_num
    from married natural join divorced
)
```
|star|
|:-|
|'Angelina Jolie'|
|'Brad Pitt'|


# 6. Question 6

```sql
select A.star, B.star
from appeared_in A, appeared_in B, in_couple AC, in_couple BC, married, made_money
where 
    A.movie = B.movie  and
    A.star > B.star and # keypoint
    AC.couple_num = BC.couple_num and
    AC.star = A.star and
    BC.star = B.star and
    AC.couple_num = married.couple_num and 
    A.movie = made_money.movie and
    married.day > made_money.day_opened
group by A.star, B.star

/* or */
with star_star_num ( A, B, couple_num) as (
select A.star, B.star, min(AC.couple_num)
from appeared_in A, appeared_in B, in_couple AC, in_couple BC, married, made_money
where 
    A.movie = B.movie  and
    A.star > B.star and # keypoint
    AC.couple_num = BC.couple_num and
    AC.star = A.star and
    BC.star = B.star and
    AC.couple_num = married.couple_num and 
    A.movie = made_money.movie and
    married.day > made_money.day_opened
group by A.star, B.star
)
select A, B
from star_star_num 
```
|star 1|star 2|
|:-|:-|
|'Brad Pitt'|'Angelina Jolie'|
|'Jennifer Garner'|'Ben Affleck'|
|'Tom Hanks'|'Rita Wilson'|
|'Vincent Cassel'|'Monica Bellucci'|

# 7. Question 7
```sql
with star_freq(star, freq) as (
    select star, count(movie)
    from appeared_in
    group by star
)
select star 
from star_freq
where freq in (
    select max(freq)
    from star_freq
)
```
|star|
|:-|
|'Brad Pitt'|
|'Matt Damon'|
|'Tom Hanks'|

# 8. Question 8
```sql
with married_times(A, B, times) as (
select A.star, B.star, count(A.couple_num) as marriedtimes
from in_couple A, in_couple B
where 
    A.couple_num = B.couple_num and
    A.star > B.star
group by A.star, B.star)
select A, B
from married_times
where times > 1
```
|star 1|star 2|
|:-|:-|
|'Brad Pitt'|'Angelina Jolie'|
|'Tom Hanks'|'Rita Wilson'|

# 9. Question 9
```sql
select star
from in_couple natural join divorced
group by star
having ( count(couple_num) > 1)
```
|star|
|:-|
|'Angelina Jolie'|
|'Brad Pitt'|
|'Tom Hanks'|

# 10. Question 10
```sql
with star_avg(star, aver) as (
	select star, avg(how_much) as aver
	from appeared_in natural join made_money
	group by star
)
select star from star_avg
where aver = (select max(aver) from star_avg)
```
|star|
|:-|
|'Scarlett Johansson'|
# 11. Credit Question

```sql
with couple_pair(couple_num, DayBegin, DayEnd) as (
select married.COUPLE_NUM, married.DAY as DayMarried, divorced.DAY as DayDivorcedwih
from married left join divorced
on married.COUPLE_NUM = divorced.COUPLE_NUM
)
select A_star, B_star from (
    select A.star as A_star, B.star as B_star, avg(made_money.HOW_MUCH) as average
    from in_couple A, in_couple B, couple_pair, appeared_in AA, appeared_in BA, made_money
    where 
        A.star > B.star and
        A.couple_num = B.couple_num and
        A.couple_num = couple_pair.couple_num and
        A.star = AA.star and 
        B.star = BA.star and 
        AA.movie = BA.movie and 
        AA.movie = made_money.MOVIE and
        (isnull(couple_pair.dayEnd) or made_money.DAY_OPENED < couple_pair.dayEnd) and 
        made_money.DAY_OPENED > couple_pair.daybegin
        group by A.star, B.star
) as myalias
order by average desc
limit 1

/* or */
with pair_average (A_star, B_star, average) as (
	select A.star as A_star, B.star as B_star, avg(made_money.HOW_MUCH) as average#A.couple_num, couple_pair.daybegin, couple_pair.dayend, AA.Movie, made_money.HOW_MUCH as money, made_money.DAY_OPENED as DayOpen
	from in_couple A, in_couple B, appeared_in AA, appeared_in BA, made_money, 
    (select married.COUPLE_NUM, married.DAY as DayBegin, divorced.DAY as DayEnd
		from married left join divorced
		on married.COUPLE_NUM = divorced.COUPLE_NUM) couple_pair
	where 
		A.star > B.star and
		A.couple_num = B.couple_num and
		A.couple_num = couple_pair.couple_num and
		A.star = AA.star and 
		B.star = BA.star and 
		AA.movie = BA.movie and 
		AA.movie = made_money.MOVIE and
		(isnull(couple_pair.dayEnd) or made_money.DAY_OPENED < couple_pair.dayEnd) and 
		made_money.DAY_OPENED > couple_pair.daybegin
		group by A.star, B.star
) 
select A_star, B_star from pair_average
where average = (select max(average) from pair_average)
```
|star 1|star 2|
|:-|:-|
|'Tom Hanks'|'Rita Wilson'|


