# 忙しい麻酔医 ~重複する期間の抽出(その1)~
## 問題3
### 以下のテーブルに対し、結果のような各proc_idの最大瞬間掛け持ち数を求めよ。

- テーブル定義

```sql
CREATE TABLE Procs(
  proc_id       INTEGER NOT NULL,
  anest_name    CHAR(40) NOT NULL,
  start_time    TIME NOT NULL,
  end_time      TIME NOT NULL
);

INSERT INTO Procs VALUES
(10, "Baker", "08:00:00", "11:00:00"),
(20, "Baker", "09:00:00", "13:00:00"),
(30, "Dow", "09:00:00", "15:30:00"),
(40, "Dow", "08:00:00", "13:30:00"),
(50, "Dow", "10:00:00", "11:30:00"),
(60, "Dow", "12:30:00", "13:30:00"),
(70, "Dow", "13:30:00", "14:30:00"),
(80, "Dow", "18:00:00", "19:00:00");

> select * from procs;
+---------+------------+------------+----------+
| proc_id | anest_name | start_time | end_time |
+---------+------------+------------+----------+
|      10 | Baker      | 08:00:00   | 11:00:00 |
|      20 | Baker      | 09:00:00   | 13:00:00 |
|      30 | Dow        | 09:00:00   | 15:30:00 |
|      40 | Dow        | 08:00:00   | 13:30:00 |
|      50 | Dow        | 10:00:00   | 11:30:00 |
|      60 | Dow        | 12:30:00   | 13:30:00 |
|      70 | Dow        | 13:30:00   | 14:30:00 |
|      80 | Dow        | 18:00:00   | 19:00:00 |
+---------+------------+------------+----------+
```

### 結果


| proc_id | max_inst_count |
|----|----|
|10|2|
|20|2|
|30|3|
|40|3|
|50|3|
|60|3|
|70|2|
|80|1|


### 解答1
