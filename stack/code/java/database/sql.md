## 统计 top10 畅销书的作家总共写过多少本

> 有三个表：
>
> 作家表（至少三列：主键、姓名、性别）
>
> 图书表（至少三列：主键、书名、 外键）
>
> 畅销书作者（有两列：作家名称、排名、每月统计 top10,该表只保存最新的 top10 作家数据）
>
> 图书表是作家表的从表，畅销书作者表中的数据是作家表的子集。请提供出建 表语句，并编写一个 sql：统计 top10 畅销书的作家总共写过多少本书



```sql
select a.writer_id,c.writer_name,count(b.book_name) as total_books
from
(select distinct writer_id from hot_books_infos) a
left join book_infos b on a.writer_id =b.writer_id
left join writer_infos c on a.writer_id = c.writer_id
group by a.writer_id,c.writer_name
```



## Full  join

![image-20230513191742340](https://article.biliimg.com/bfs/article/33ccfe183e608d94b5b41cb76a6e88b08d5989bf.png)

```
select if(a.`date` is null,b.`date`,a.`date`) as dt ,if(a.v1 is null ,0,a.v1) as v1 ,if(b.v2 is null ,0,b.v2)
as v2 from a full join b 
```



## 计算每个人在哪一天的消费金额最大

![image-20230513191852070](https://article.biliimg.com/bfs/article/9d7c5814be46eeda2fecc9dc2ec9259a4bcf4e1e.png)

```
select a.name,a.time,b.max_money from syc_mianshi a join (select name,max(money) as
max_money from syc_mianshi group by name) b on a.name = b.name and a.money=b.max_money;
或者使用开窗函数实现：
select name ,time ,money from (select name,time,money,row_number() over(partition by name
order by money desc) as rank from syc_mianshi) a
where a.rank =1;
```

## 表数据去重分析

![image-20230513193638685](https://article.biliimg.com/bfs/article/a93413a9e4a877a380fd213ac8191e002bcfc6ba.png)

```
select distinct var1,var2 from var  

where  var2 is not null
union
select var1 ,cast(sum(var2) as char) from var 

group by var1    having sum(var2) is null
```

## 查询 a，b 表中不相交的数据集

```
第一种方式：select a.id,a.name,b.id ,b.name from a full outer join b on a.id = b.id where a.id
is null or b.id is null;
第二种方式：select b.id ,b.name from a full join b on a.id = b.id where a.id is null union select
a.id,a.name from a full join b on a.id = b.id where b
.id is null
```





## 求出 2019 年第一个季度：2019-01-01 至 2019-03-31 的每个人的利息。

![image-20230513202114498](https://article.biliimg.com/bfs/article/ceb7554e3caa02f9a3efb2c0aa0aace967ac6f58.png)

```
select
id,sum(bal*rate*datediff(if(enddate >"2019-03-31","2019-03-31",enddate),if(startdate<"2019-01- 01","2019-01-01",startdate))) as total _rate_bal from bal_rat_table a where to_date(a.startdate) <= to_date('2019-03-31') and
to_date(a.enddate) >= to_date('2019-01-01') group by id;
```

## 求 app,channel,province 任意组合下的用户数

![image-20230513203133736](https://article.biliimg.com/bfs/article/1c9dfc77584935351c7e4e39dd0132639c83df1f.png)



```
select app,channel,province,count(userid) as total_user_count from temp group by
app,channel,province with cube;
withcube、with rollup、grouping sets((x1,x2),(x1,x2,x3))
```



## 统计连续 5 天登录的用户，login 登录表

![image-20230513203238137](https://article.biliimg.com/bfs/article/45beb1ad6ef840eec5b90a5896d4f22b992dc78e.png)

```
select distinct a.userid from 
(select dt,userid ,row_number() over(partition by userid order by dt ) as rank from temp) a 
join 
(select dt, userid ,row_number() over(partition by userid order by dt ) as rank from temp) b
on a.userid =b.userid and a.rank = b.rank -5 and a.dt = date_add(b.dt, -5）
```





## PV/UV



![image-20230514173544572](https://article.biliimg.com/bfs/article/3ff2bd802bffabf3260f35d1a8194ddd4f186f9d.png)

```
select `date`,count(uid) as pv,count(distinct uid) as uv from temp group by `date`;
```





## 主播收入

![image-20230514173754976](https://article.biliimg.com/bfs/article/505a58e065885deb563922780e41dbd065459bd7.png)

```
select t.uid,sum(t.income_money) as total from user_income t where t.uid not in (select uid
from banned_user) 
```



## 前三大的 ctime 的 group_id 列表

![image-20230514174031183](https://article.biliimg.com/bfs/article/2e77264fce1766f7abbd6486e1086c8ea811b77c.png)

```
select uid,group_id ,ctime from (select uid,group_id,ctime,row_number() over(partition by
uid order by ctime desc ) as rank from my_table) t where t.rank <=3;
```







## 求环比  lag



```
 SELECT product,year_month ,gmv
 lag(giv,1)OVER (PARTITION BY department,product ORDER BY year_month AS   lag_gmv
 ,
cast(gmv as double)/lag(gmv,1)OVER (PARTITION BY department,product ORDER  BY year_month )-1 as growth_rate
 FROM product
```

![image-20230517171807883](https://article.biliimg.com/bfs/article/be931bdf6f45e74ad35461ec97034c21d1c8459a.png)





## 求销量top10%





|                | ![image-20230517172044786](https://article.biliimg.com/bfs/article/bb86410d8893875d746f9e5bc329062d97fdd579.png) |      |
| -------------- | ------------------------------------------------------------ | ---- |
| percent_rank() | ![image-20230517172011039](https://article.biliimg.com/bfs/article/06605c3db652aadd17f539ecb0d0c2ad02f82638.png) |      |
| ntile()        | ![image-20230517172154091](https://article.biliimg.com/bfs/article/b5f14d59cd3c8323621f9899cecc04eace4e2c78.png) |      |
|                |                                                              |      |





## 聚合+窗口

![image-20230517172255025](https://article.biliimg.com/bfs/article/dd26293ed8c44ac0e027a91e5d14dd02bc6b1178.png)





## 连续出现的数字

|      |                                                              |                                                              |
| ---- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 3    | ![image-20230517172537484](https://article.biliimg.com/bfs/article/f3b702ee1de32e801a092a9f7338b3b5e26ae803.png) | ![image-20230517172557055](https://article.biliimg.com/bfs/article/2b28b87b2cf4932e07a065bf6d494480927c33fe.png) |
| 4    | ![image-20230517173127548](https://article.biliimg.com/bfs/article/cad2fb67abdbc06e182e04d756c49bdc2e6fd606.png) | ![image-20230517173142572](https://article.biliimg.com/bfs/article/ca434f16f5751b827ec7977c9208c1b43e38fc75.png) |
|      |                                                              |                                                              |



