# 欠勤 ~条件付きのUPDATE/DELETE~
## 問題2
### 社員の欠勤状況を記録するためのテーブルに対し、以下のビジネスルールを実現するSQL文を書け
- 罰点を40ポイント貯めるともれなく首になる
  - Personnelテーブルから該当する社員のレコードを削除する
- 2日以上連続して休んだ場合は「長期病欠」扱いとなり、2日目以降の欠勤には罰点が付かないし、
欠勤の総日数にもカウントされない

```sql
CREATE TABLE Personnel (
  emp_id  INTEGER  NOT NULL PRIMARY KEY,
  name    CHAR(20) NOT NULL
);

CREATE TABLE ExcuseList (
  reason_code  CHAR(40)  NOT NULL
);

CREATE TABLE Absenteeism (
  emp_id          INTEGER  NOT NULL REFERENCES Personnel (emp_id),
  absent_date     DATE     NOT NULL,
  reason_code     CHAR(40) NOT NULL REFERENCES ExcuseList (reason_code),
  severity_points INTEGER  NOT NULL
                  CHECK(severity_points BETWEEN 1 AND 4),
  PRIMARY KEY (emp_id, absent_date)
);
```

```
Absenteeism:欠勤テーブル    emp_id:社員ID         absent_date:欠勤日
reason_code:理由コード      severity_points:罰点  Personnel:社員テーブル
ExcuseList :言い訳テーブル
```
