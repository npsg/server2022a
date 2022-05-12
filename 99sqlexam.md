#SQL練習

##会議室予約システムのテーブル
```SQL
-- 時間帯情報
create table m_time (
  time_cd integer not null
  , time_name varchar(20) not null
  , primary key (time_cd)
) ;

-- 会議室情報
create table m_room (
  room_cd integer not null
  , room_name varchar(20) not null
  , primary key (room_cd)
) ;

-- 予約情報
create table t_reserve (
  reserve_date date not null
  , time_cd integer not null
  , room_cd integer default 1 not null
  , emp_no char(8) not null
  , tel varchar(11) not null
  , purpose varchar(20) not null
  , primary key (reserve_date,time_cd,room_cd)
) ;

-- 従業員情報
create table m_employee (
  emp_no char(8) not null
  , password varbinary(200)
  , department varchar(12)
  , emp_name varchar(20)
  , default_tel varchar(11)
  , primary key (emp_no)
) ;
```

##サンプデータ
```SQL
--従業員情報
INSERT INTO m_employee(emp_no, password, department, emp_name, default_tel)
 VALUES ('10000001', AES_ENCRYPT('10000001', SHA2('tebasaki',512)), '総務部', '総務一雄', '1111')
, ('10000002', AES_ENCRYPT('10000002', SHA2('tebasaki',512)), '総務部', '総務二子', '12345678901')
, ('10000003', AES_ENCRYPT('10000003', SHA2('tebasaki',512)), '開発部', '開発三朗', '99999999999')
, ('10000004', AES_ENCRYPT('10000004', SHA2('tebasaki',512)), '開発部', '開発四朗', '00000000000')
, ('10000005', AES_ENCRYPT('10000005', SHA2('tebasaki',512)), '営業部', '営業五郎', '55555555555');

--会議室情報
INSERT INTO m_room(room_cd, room_name)
 VALUES (1, '3階会議室');

--時間帯情報
INSERT INTO m_time(time_cd, time_name)
 VALUES (1, '午前１（9:00～10:30）')
 ,(2, '午前２（10:30～12:00）')
 ,(3, '午後１（13:00～15:00）')
 ,(4, '午後２（15:00～17:00）');

--予約情報
--2022/05/31
INSERT INTO t_reserve(reserve_date, time_cd, room_cd, emp_no, tel, purpose)
 VALUES ('2022/05/31', 1, 1, '10000001', '1111', 'X社打ち合わせ')
 ,('2022/05/31', 2, 1, '10000001', '1111', 'X社打ち合わせ')
 ,('2022/05/31', 3, 1, '10000002', '12345678901', 'X社打ち合わせ①')
 ,('2022/05/31', 4, 1, '10000002', '12345678901', 'X社打ち合わせ①');
--2022/06/01
INSERT INTO t_reserve(reserve_date, time_cd, room_cd, emp_no, tel, purpose)
 VALUES ('2022/06/01', 1, 1, '10000002', '12345678901', 'スタッフ向け事前研修')
 ,('2022/06/01', 2, 1, '10000002', '12345678901', 'スタッフ向け事前研修')
 ,('2022/06/01', 3, 1, '10000004', '00000000000', 'X社打ち合わせ②')
 ,('2022/06/01', 4, 1, '10000004', '00000000000', 'X社打ち合わせ②');
--2022/06/30
INSERT INTO t_reserve(reserve_date, time_cd, room_cd, emp_no, tel, purpose)
 VALUES ('2022/06/30', 1, 1, '10000004', '09011112222', 'X社打ち合わせ③')
 ,('2022/06/30', 2, 1, '10000004', '09011112222', 'X社打ち合わせ③')
 ,('2022/06/30', 3, 1, '10000001', '1111', 'Y社打ち合わせ')
 ,('2022/06/30', 4, 1, '10000003', '99999999999', '資料作成作業');
--2022/07/01
INSERT INTO t_reserve(reserve_date, time_cd, room_cd, emp_no, tel, purpose)
 VALUES ('2022/07/01', 1, 1, '10000004', '09011112222', 'X社打ち合わせ④')
 ,('2022/07/01', 2, 1, '10000004', '09011112222', 'X社打ち合わせ④')
 ,('2022/07/01', 3, 1, '10000003', '09011112222', 'X社打ち合わせ')
 ,('2022/07/01', 4, 1, '10000003', '99999999999', '資料作成作業');
```
