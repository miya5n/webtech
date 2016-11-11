# 会計年度テーブル ~範囲外の日付を入力しないための制約~
## 問題1
- 以下のテーブルに対し、正しい情報だけを持つように制約を設定しろ
- ただし会計年度を10月1日～9月30日とする
```sql
CREATE TABLE FiscalYearTable1(
  fiscal_year   INTEGER,
  start_date    DATE,
  end_date      DATE
);
```

- FiscalYearTable1:会計年度手テーブル
- fiscal_year:fiscal_year:会計年度
- start_date:年度開始日
- end_date:年度終了日


 上記のCREATE文の場合、以下のINSERT文がすべて「OK」となる
```sql
-- OKになる
INSERT INTO FiscalYearTable1 VALUES (1995, '1994/10/01 00:00:00', '1995/09/30 00:00:00');
-- NGになるはずのデータ
INSERT INTO FiscalYearTable1 VALUES (1996, '1995-10-01', '1995-08-30');
-- OKになる
INSERT INTO FiscalYearTable1 VALUES (1997, '1996-10-01', '1997-09-30');
-- OKになる
INSERT INTO FiscalYearTable1 VALUES (1998, '1997-10-01', '1998-09-30');

-- 以下、oracle用
-- OKになる
INSERT INTO FiscalYearTable1 VALUES (1995, to_date('1994-10-01','yyyy/mm/dd'), to_date('1995-09-30','yyyy/mm/dd'));
-- NGになるはずのデータ
INSERT INTO FiscalYearTable1 VALUES (1996, to_date('1995-10-01','yyyy/mm/dd'), to_date('1995-08-30','yyyy/mm/dd'));
-- OKになる
INSERT INTO FiscalYearTable1 VALUES (1997, to_date('1996-10-01','yyyy/mm/dd'), to_date('1997-09-30','yyyy/mm/dd'));
-- OKになる
INSERT INTO FiscalYearTable1 VALUES (1998, to_date('1997-10-01','yyyy/mm/dd'), to_date('1998-09-30','yyyy/mm/dd'));

```

## 解答1(oracle, postgresql)

```sql
CREATE TABLE FiscalYearTable1(
  fiscal_year   INTEGER  NOT NULL PRIMARY KEY,
  start_date    DATE     NOT NULL,
  CONSTRAINT valid_start_date
    CHECK ((EXTRACT(YEAR FROM start_date) = fiscal_year-1)
            AND (EXTRACT(MONTH FROM start_date)=10)
            AND (EXTRACT(DAY FROM start_date)=01)),
  end_date      DATE     NOT NULL,
  CONSTRAINT valid_end_date
    CHECK ((EXTRACT(YEAR FROM end_date) = fiscal_year)
            AND (EXTRACT(MONTH FROM end_date)=09)
            AND (EXTRACT(DAY FROM end_date)=30))
);
```

### 結果(oracle)

```sql
SQL> select * from FiscalYearTable1;
FISCAL_YEAR START_DAT END_DATE
----------- --------- ---------
       1995 01-OCT-94 30-SEP-95
       1997 01-OCT-96 30-SEP-97
       1998 01-OCT-97 30-SEP-98

SQL> INSERT INTO FiscalYearTable1 VALUES (1996, to_date('1995-10-01','yyyy/mm/dd'), to_date('1995-08-30','yyyy/mm/dd'));
INSERT INTO FiscalYearTable1 VALUES (1996, to_date('1995-10-01','yyyy/mm/dd'), to_date('1995-08-30','yyyy/mm/dd'))
*
ERROR at line 1:
ORA-02290: check constraint (SYSTEM.VALID_END_DATE) violated
```

### 結果(postgresql)

```sql
> select *  from FiscalYearTable1;
  1995 | 1994-10-01 | 1995-09-30
  1997 | 1996-10-01 | 1997-09-30
  1998 | 1997-10-01 | 1998-09-30

> INSERT INTO FiscalYearTable1 VALUES (1997, '1996-10-01', '1995-09-30');
  ERROR:  new row for relation "fiscalyeartable1" violates check constraint "valid_end_date"
  DETAIL:  Failing row contains (1997, 1996-10-01, 1995-09-30)
```

### mysqlはcheck制約の機能がないので、2行目がエラーとならずINSERTされる

```sql
> select * from FiscalYearTable1\G
  *************************** 1. row ***************************
  fiscal_year: 1994
  start_date: 1994-10-01
  end_date: 1995-09-30
  *************************** 2. row ***************************
  fiscal_year: 1995
  start_date: 1995-10-01
  end_date: 1995-08-30
  *************************** 3. row ***************************
  fiscal_year: 1996
  start_date: 1996-10-01
  end_date: 1995-09-30
  *************************** 4. row ***************************
  fiscal_year: 1997
  start_date: 1997-10-01
  end_date: 1995-09-30
4 rows in set (0.00 sec)
```

## 解答2(mysql)
### TRIGGERを使ってみた

```sql
CREATE TABLE FiscalYearTable1(
  fiscal_year   INTEGER  NOT NULL PRIMARY KEY,
  start_date    DATE     NOT NULL,
  end_date      DATE     NOT NULL
);

DELIMITER $$

CREATE TRIGGER trigger_fiscalYearTable1 BEFORE INSERT ON FiscalYearTable1 FOR EACH ROW
BEGIN
    IF
        EXTRACT(YEAR FROM new.start_date)=new.fiscal_year-1 AND EXTRACT(MONTH FROM new.start_date)=10 AND EXTRACT(DAY FROM new.start_date)=01 AND
        EXTRACT(YEAR FROM new.end_date)=new.fiscal_year AND EXTRACT(MONTH FROM new.end_date)=09 AND EXTRACT(DAY FROM new.end_date)=30
    THEN
        SET @start_date = new.start_date;
        SET @end_date = new.end_date;
    ELSE
        signal sqlstate '45000' set message_text = 'bad date';
    END IF;
END$$

DELIMITER ;
```

### 結果

```sql
> select * from fiscalyeartable1;
+-------------+------------+------------+
| fiscal_year | start_date | end_date   |
+-------------+------------+------------+
|        1995 | 1994-10-01 | 1995-09-30 |
|        1997 | 1996-10-01 | 1997-09-30 |
|        1998 | 1997-10-01 | 1998-09-30 |
+-------------+------------+------------+
3 rows in set (0.00 sec)

> INSERT INTO FiscalYearTable1 VALUES (1996, '1995-10-01', '1995-08-30');
ERROR 1644 (45000): bad start_date
```

とりあえず1問目完了。
