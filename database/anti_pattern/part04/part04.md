# 4. キーレスエントリ(外部キー嫌い)

## 4.１. アンチパターン
- 外部キー制約を使用しない
  - 使用しない方が、設計がシンプルになり、柔軟性が高まり、実行速度が速くなるという固定観念から
  - そこには代償があり、代償は別の形で支払わなければならない
    - 参照整合性を保証するための実装を書く責任が生じる

### 4.1.2. 外部キーを使用しない理由
- データの更新が参照整合性規約と衝突する
- データベース設計の柔軟性が極めて高いので、参照整合性制約をサポートできない
- データベースが外部キーのために作成するインデックスがパフォーマンスに影響する
- 外部キーをサポートしていないデータベース製品を利用している
- 「キャッチ=22」的なジレンマに遭遇する
```SQL
-- 外部キー制約があると、以下２クエリーはエラーとなる
UPDATE BugsStatus SET status = 'INVALID' WHERE status = 'BOGUS';　-- 子で使用されているので、変更できない
UPDATE Bugs SET status = 'INVALID' WHERE status = 'BOGUS'; -- 親にないステータスは設定できない
```

## 4.2. 例外
- 外部キー制約をサポートしていないデータベース製品を使わざるをえないとき

## 4.3. 解決策
### 4.3.1. 外部キーを宣言する
- データベースに登録される時点で参照整合性を強制する

### 4.3.2. サポート機能を利用する
- 親の行の更新や削除を行うと、その行を参照しているあらゆる子の行もデータベースが適切に処理してくれる
- CASCADE更新
  - 親のデータが更新されると子のデータも更新される
- RESTRICT
  - これを宣言すると子が参照している親データが削除できなくなる

```SQL
CREATE TABLE Bugs (
  -- ... 他の列 ...
  reported_by   BIGINT(20) NOT NULL,
  status        VARCHAR(10) NOT NULL DEFAULT 'NEW',
  FOREIGN KEY (reported_by) REFERENCES Accounts (account_id)
    ON UPDATE CASCADE
    ON DELETE RESTRICT,
  FOREIGN KEY (status) REFERENCES BugsStatus (status)
    ON UPDATE CASCADE
    ON DELETE SET DEFAULT  
)
```

## 4.4. オーバーヘッドに関して
- 多少のオーバーヘッドはあるが、効率的になる面もある
  - 挿入、更新、削除の前に、チェックのためのSELECTクエリを実行する必要がない
  - 複数テーブルの変更を防ぐためにテーブルをロックする必要がなう
  - 他の方法のように孤児ができてしまうことがないので、データ品質管理用スクリプトを定期的に実行しなくてもよい