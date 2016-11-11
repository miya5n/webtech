# 3. IDリクワイアド(とりあえずID)

## 3.1. アンチパターン
- すべてのテーブルに「ID」列を用いる
  - 列名はID
  - データ型は32ビットまたは64ビットの整数
  - 一意の値が自動的に生成される

## 3.2. デメリット
### 3.2.1. 冗長なキーが作成されてしまう
- Bugsテーブル

| 列名 | 型 |
|----|----|
| id | SERIAL, PRIMARY KEY |
| bug_id | BIGINT(20), UNIQUE |
| description | VARCHAR(1000) |

- id列とbug_id列が同じ意味合いをもっている

### 3.2.2. 重複行を許可してしまう
- BugsProductsテーブル

| 列名 | 型 |
|----|----|
| id | SERIAL, PRIMARY KEY |
| bug_id | BIGINT(20), FOREIGN KEY |
| product_id | BIGINT(20), FOREIGN KEY |

- 複合キーを利用せずid列を主キーとしてまうと、bug_idとproduct_idの組み合わせが  
常に一意であることを保証しなくなってしまう

### 3.2.3. キーの意味が分かりにくくなる
- SQL例
```SQL
SELECT a.id, b.id
FROM Bugs b
INNER JOIN Account a ON b.assigned_to = a.id
WHERE b.status = 'OPEN'
```

- エイリアスを使用していると、idがAccountテーブルのものか、Bugsテーブルのものか  
見分けがつきにくい

### 3.2.4. USINGを使用できない
- USING使用例
```SQL
SELECT * FROM Bugs INNER JOIN BugsProducts USING (bug_id)
```

- 両テーブルに同じ列名のがある場合、上記のようにUSINGを利用して簡潔クエリが書けるが、  
すべてのテーブルで主キーとしてidという列名にしなければならないとなると、冗長なON構文を  
用いて書かなければならなくなる

## 3.3. 例外
- 一部のORMフレームワークは開発をシンプルにするため、「設定より規約」の原則に従っており、  
すべてのテーブルで主キー「id」を定義する必要がある。そのため、このようなフレームワークでは、  
規約に従ったほうが良い場合があります。

## 3.4. 解決策
### 3.4.1. わかりやすい列名にする
- 主きのー名前は主キーが識別する対象の対象のエンティティを表すものにすべき
  - Bugsテーブルであれば、bug_id
- 外部キーの列名にも同じ命名規則を用いるべき
- 主キー名はスキーマ内で一意となるべき
- 例外として参照する主キーと異なる命名規則の名前を外部キーに与えることが適切な場合もある
- Bugsテーブル

| 列名 | 型 |
|----|----|
| bug_id | BIGINT(20), FOREIGN KEY |
| reported_by | BIGINT(20), FOREIGN KEY(reported_by) REFERENCES Accounts(account_id) |

### 3.4.2. 規則に縛られない
- ORMフレームワークの多くは「id」という名前の疑似キーが使われることを規約にしているが、  
この規約を上書きして、別の名前を宣言する（プログラムにて解決する）
