# 5. EAV(エンティティ・アトリビュート・バリュー)

## 5.１. アンチパターン
- 汎用的な属性テーブルを使用する

- Issuesテーブル

| 列名 | 型 |
|----|----|
| issue_id | BIGINT(20) |

- IssueAttributesテーブル

| 列名 | 型 |
|----|----|
| issue_id | BIGINT(20) |
| attr_name | VARCHAR(100) |
| attr_value | VARCHAR(100) |


| issue_id | attr_name | attr_value |
|----|----|----|
| 1 | product_id | 1 |
| 1 | date_reported | 2009-06-01 |
| 1 | status | NEW |
| 1 | version | 1.3 |


### 5.2. デメリット
- 必須属性を設定できない
- SQLのデータ型を利用できない
- 参照整合性を保証できない
- SQLにて行を再構築しなければならない

## 5.3. 例外
- ありません。なるべく使用しないほうがよい
- もしくは別の非リレーショナルなKVSを使うなどをお勧めする

## 5.4. 解決策
### 5.4.1. シングルテーブル継承
- すべてのタイプの属性を個別に格納して、関連するすべてのサブタイプを一つのテーブルに格納する
- 一つの属性列を、その行がどのサブタイプであるかを定義するために用いる
- 以下の例ではサブタイプ識別に使う属性はissue_typeとする
- Issuesテーブル

| 列名 | 型 | 備考|
|----|----|----|
| issue_id | BIGINT(20) | |
| reported_by | BIGINT(20) | |
| product_id | BIGINT(100) | |
| priority | VARCHAR(20) | |
| issue_type | VARCHAR(20) | 'BUG'または'FEATURE'が格納される |
| security | VARCHAR(20) | 'BUG'のときのみ使う属性 |
| sponsor | VARCHAR(20) | 'FEATURE'のときのみ使う属性 |

- この場合、securityもしくはsponsorに対してNULL許容しなくてはならない
- サブタイプの数とサブタイプ固有の属性数が少なく、単一のテーブルに対するデータアクセスを  
使う場合は適切

### 5.4.2. 具象テーブル継承
- サブタイプごとにテーブルを作成する
- Bugsテーブル

| 列名 | 型 | 備考|
|----|----|----|
| issue_id | BIGINT(20) | |
| reported_by | BIGINT(20) | |
| product_id | BIGINT(100) | |
| priority | VARCHAR(20) | |
| security | VARCHAR(20) | 'BUG'のときのみ使う属性 |

- Featuresテーブル

| 列名 | 型 | 備考|
|----|----|----|
| issue_id | BIGINT(20) | |
| reported_by | BIGINT(20) | |
| product_id | BIGINT(100) | |
| priority | VARCHAR(20) | |
| sponsor | VARCHAR(20) | 'FEATURE'のときのみ使う属性 |

- 共通属性とサブタイプ属性の見分けがつきにくい
- すべてのテーブルを跨いだ検索を実行する頻度が低い場合に適切

### 5.4.3. クラステーブル継承
- テーブルをオブジェクト指向のクラスであるかのように見なして、継承を模倣する
  - すべてのサブタイプに共通する属性を含む基底型のテーブルを一つ作る
  - サブタイプごとに一つずつ追加のテーブルを作成し、基底型テーブルに対する外部キーの役割を  
  もつ主キーを設定する

- Issuesテーブル

| 列名 | 型 | 備考|
|----|----|----|
| issue_id | BIGINT(20) | |
| reported_by | BIGINT(20) | |
| product_id | BIGINT(100) | |
| priority | VARCHAR(20) | |


- Bugsテーブル

| 列名 | 型 | 備考|
|----|----|----|
| issue_id | BIGINT(20) | 外部キー設定 |
| security | VARCHAR(20) | 'BUG'のときのみ使う属性 |

- Featuresテーブル

| 列名 | 型 | 備考|
|----|----|----|
| issue_id | BIGINT(20) | 外部キー設定 |
| sponsor | VARCHAR(20) | 'FEATURE'のときのみ使う属性 |

- サブタイプをすべてJOINするクエリは、ビュー定義の候補にもなります
- すべてのサブタイプに共通する列を参照するクエリが頻繁に実行されるときに最適

### 5.4.4. 半構造化データ(シリアライズLOB)
- サブタイプの数が多い場合や、頻繁に新しい属性を追加しなければならない場合は、  
XMLやJSONなどの形式で属性名と値をともに格納することもできます。
- Issuesテーブル

| 列名 | 型 | 備考|
|----|----|----|
| issue_id | BIGINT(20) | |
| reported_by | BIGINT(20) | |
| product_id | BIGINT(100) | |
| priority | VARCHAR(20) | |
| issue_type | VARCHAR(20) | 'BUG'または'FEATURE'が格納される |
| attributes | TEXT | その他の動的属性が格納される |

- 拡張性が極めて高い
- テーブルの各行にはまったく異なる属性群を格納できる
- 特定の属性にアクセスする手段がほとんどない
- アプリケーションコードに頼らざるをえない
- サブタイプの数を制限できない場合たや、新しい属性を随時定義できる柔軟性が必要な場合に適切
