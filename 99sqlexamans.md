１、従業員番号10000001を指定し、従業員情報を取得する。
```sql
select * from m_employee where emp_no = '10000001';
```

２、従業員番号10000001を指定し、予約情報を取得する（時間帯コード・会議室コードで表示）。
```sql
select * from t_reserve where emp_no = '10000001';
```

３、予約日・時間帯コード・会議室コードを指定し、予約情報を取得する（時間帯コード・会議室コードで表示）。
```sql
select * from t_reserve where reserve_date='2022-05-31' and time_cd=1 and room_cd=1
```

４、予約日・時間帯コードごとの予約数を表示する。
```sql
select reserve_date,time_cd,count(*) from t_reserve
 group by reserve_date,time_cd order by reserve_date,time_cd
 ```

５、本日の日付以降、かつ従業員番号10000001を指定すると、該当する予約情報を取得する（時間帯コード・会議室コードで表示）。
```sql
select * from t_reserve where reserve_date>=CURDATE() and emp_no = '10000001'
```

６、予約日2022-05-31・時間帯コード1・会議室コード1を指定すると、該当する予約情報を取得できる（時間帯コード・会議室コードは名前で表示）。
```sql
select reserve_date, time_name, room_name, emp_no, tel, purpose
 from t_reserve join m_time using(time_cd)
 join m_room using(room_cd) where reserve_date = '2022-05-31' and time_cd=1 and room_cd=1
```

７、1件予約情報を挿入する。ただし、目的は空欄とする。
```sql
insert into t_reserve(reserve_date, time_cd, room_cd, emp_no, tel)
    values('2022-08-01', 1, 1, '10000001', '9999');
```

８、従業員番号10000005とパスワード10000005を指定し、該当する従業員情報を取得する。
```sql
select * from m_employee where emp_no='10000005' and password=AES_ENCRYPT('10000005', SHA2('tebasaki',512));
```

９、従業員番号10000005のパスワードを99999999に更新する。
```sql
update m_employee set password=AES_ENCRYPT('99999999', SHA2('tebasaki',512))
 where emp_no='10000005'
確認用：
select * from m_employee where emp_no='10000005' and password=AES_ENCRYPT('99999999', SHA2('tebasaki',512))
```

10、従業員ごとの予約数を集計する。
```sql
select e.emp_no 従業員番号, e.emp_name 従業員氏名, ifnull(r.cnt,0) 予約数
from m_employee e left outer join (select emp_no, count(*) cnt from t_reserve group by emp_no) r using(emp_no) 
order by emp_no;
```
