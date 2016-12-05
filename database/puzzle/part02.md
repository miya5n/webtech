# 欠勤 ~条件付きのUPDATE/DELETE~
## 問題2
### 社員の欠勤状況を記録するためのテーブルに対し、以下のビジネスルールを実現するSQL文を書け
1. 罰点を40ポイント貯めるともれなくクビになる
  - Personnelテーブルから該当する社員のレコードを削除する
2. 2日以上連続して休んだ場合は「長期病欠」扱いとなり、2日目以降の欠勤には罰点が付かないし、
欠勤の総日数にもカウントされない

```sql
CREATE TABLE Personnel (
  emp_id  INTEGER  NOT NULL PRIMARY KEY,
  name    CHAR(20) NOT NULL
);

CREATE TABLE ExcuseList (
  reason_code  CHAR(40)  NOT NULL,
  INDEX (reason_code)
);

INSERT INTO Personnel VALUES (1, "A"),(2, "B"),(3, "C"),(4, "D"),(5, "E"),(6, "F");
INSERT INTO ExcuseList VALUES ("xxx"), ("yyy"), ("zzz");

-- oracle,postgresql
CREATE TABLE Absenteeism (
  emp_id          INTEGER  NOT NULL REFERENCES Personnel (emp_id),
  absent_date     DATE     NOT NULL,
  reason_code     CHAR(40) NOT NULL REFERENCES ExcuseList (reason_code),
  severity_points INTEGER  NOT NULL
                  CHECK(severity_points BETWEEN 1 AND 4),
  PRIMARY KEY (emp_id, absent_date)
);

-- mysql
CREATE TABLE Absenteeism (
  emp_id          INTEGER  NOT NULL,
  absent_date     DATE     NOT NULL,
  reason_code     CHAR(40) NOT NULL,
  severity_points INTEGER  NOT NULL,
  PRIMARY KEY (emp_id, absent_date),
  CONSTRAINT `emp_id_fk` FOREIGN KEY (emp_id) REFERENCES Personnel(emp_id),
  CONSTRAINT `reason_code_fk` FOREIGN KEY (reason_code) REFERENCES ExcuseList(reason_code)
) ENGINE = InnoDB DEFAULT CHARSET=utf8;
DELIMITER $$

CREATE TRIGGER trigger_absenteeism BEFORE INSERT ON Absenteeism FOR EACH ROW
BEGIN
    IF
        new.severity_points BETWEEN 1 AND 4
    THEN
        SET @severity_points = new.severity_points;
    ELSE
        signal sqlstate '45000' set message_text = 'bad severity_points';
    END IF;
END$$

DELIMITER ;

INSERT INTO Absenteeism VALUES
(1, "2015-02-02", "xxx", 5),
(1, "2015-10-02", "yyy", 5),
(3, "2015-01-05", "xxx", 5),
(3, "2015-02-14", "xxx", 5),
(3, "2015-05-20", "yyy", 5),
(4, "2015-04-02", "zzz", 5),
(4, "2015-04-03", "zzz", 0),
(4, "2015-04-04", "zzz", 0),
(4, "2015-04-05", "zzz", 0),
(4, "2015-04-06", "zzz", 0),
(4, "2015-04-07", "zzz", 0),
(5, "2015-06-01", "xxx", 5),
(5, "2015-06-03", "xxx", 5),
(5, "2015-08-19", "yyy", 5),
(5, "2015-08-25", "xxx", 5),
(5, "2015-09-16", "yyy", 5),
(5, "2015-09-19", "xxx", 5),
(5, "2015-12-14", "zzz", 5),
(5, "2015-12-15", "zzz", 0),
(5, "2015-12-16", "zzz", 0),
(6, "2015-01-19", "xxx", 5),
(6, "2015-03-09", "xxx", 5),
(6, "2015-04-11", "xxx", 5),
(6, "2015-04-13", "xxx", 5),
(6, "2015-04-22", "xxx", 5),
(6, "2015-06-05", "xxx", 5),
(6, "2015-11-02", "xxx", 5),
(6, "2015-11-29", "xxx", 5),
(6, "2015-12-15", "xxx", 5);

- oracle用
INSERT INTO Absenteeism VALUES
(1, to_date("2015-02-02","yyyy/mm/dd"), "xxx", 5),
(1, to_date("2015-10-02","yyyy/mm/dd"), "yyy", 5),
(3, to_date("2015-01-05","yyyy/mm/dd"), "xxx", 5),
(3, to_date("2015-02-14","yyyy/mm/dd"), "xxx", 5),
(3, to_date("2015-05-20","yyyy/mm/dd"), "yyy", 5),
(4, to_date("2015-04-02","yyyy/mm/dd"), "zzz", 5),
(4, to_date("2015-04-03","yyyy/mm/dd"), "zzz", 0),
(4, to_date("2015-04-04","yyyy/mm/dd"), "zzz", 0),
(4, to_date("2015-04-05","yyyy/mm/dd"), "zzz", 0),
(4, to_date("2015-04-06","yyyy/mm/dd"), "zzz", 0),
(4, to_date("2015-04-07","yyyy/mm/dd"), "zzz", 0),
(5, to_date("2015-06-01","yyyy/mm/dd"), "xxx", 5),
(5, to_date("2015-06-03","yyyy/mm/dd"), "xxx", 5),
(5, to_date("2015-08-19","yyyy/mm/dd"), "yyy", 5),
(5, to_date("2015-08-25","yyyy/mm/dd"), "xxx", 5),
(5, to_date("2015-09-16","yyyy/mm/dd"), "yyy", 5),
(5, to_date("2015-09-19","yyyy/mm/dd"), "xxx", 5),
(5, to_date("2015-12-14","yyyy/mm/dd"), "zzz", 5),
(5, to_date("2015-12-15","yyyy/mm/dd"), "zzz", 0),
(5, to_date("2015-12-16","yyyy/mm/dd"), "zzz", 0),
(6, to_date("2015-01-19","yyyy/mm/dd"), "xxx", 5),
(6, to_date("2015-03-09","yyyy/mm/dd"), "xxx", 5),
(6, to_date("2015-04-11","yyyy/mm/dd"), "xxx", 5),
(6, to_date("2015-04-13","yyyy/mm/dd"), "xxx", 5),
(6, to_date("2015-04-22","yyyy/mm/dd"), "xxx", 5),
(6, to_date("2015-06-05","yyyy/mm/dd"), "xxx", 5),
(6, to_date("2015-11-02","yyyy/mm/dd"), "xxx", 5),
(6, to_date("2015-11-29","yyyy/mm/dd"), "xxx", 5),
(6, to_date("2015-12-15","yyyy/mm/dd"), "xxx", 5);
```

```
Absenteeism:欠勤テーブル    emp_id:社員ID         absent_date:欠勤日
reason_code:理由コード      severity_points:罰点  Personnel:社員テーブル
ExcuseList :言い訳テーブル
```

## 解答1
