# PostgreSQL

## SQLとオブジェクト

### データの参照

- 文字の扱い

	- 大文字と小文字は区別しない（内部処理は小文字）

		- SQLキーワード
		- テーブル名やインデックス名、テーブルの列名などのオブジェクト名
		- 関数名
		- データ型の名称

	- テーブル名前区別したい場合、ダブルクォートで囲む

		- 定義後、アクセスする際にも常にダブルクォートが必要

	- 文字や日付情報はシングルクォートで囲む

		- 囲まない場合は、基本的に数値として認識され
		- 文字列中のシングルクォートは二重にしてエスケープ 'abc''123'は「abc'123」として解釈される

	- コメント

		- 「--」（ハイフン二つ）以降は改行までコメント
		- 「/*」から「*/」までは改行を含めてコメント

	- NULLの扱い

		- 「値不定」を表す特殊な値で、空文字列とは区別される

			- 一致、大小比較、文字列連結などの演算の結果はNULLとなる
			- Oracleでは空文字列＝NULL

- SELECT [列名 | テーブル名.列名 | *] [AS 別名]
 FROM テーブル名 [別名] WHERE 検索条件
 [ORDER BY ソート対象 [ASC | DESC]]
 [LIMIT 件数 OFFSET 位置]

	- ORDER BY

		- ソートで行わない場合、SELECTで得られる結果の順序は不定

			- 変換が少ないテーブルは一見「データを挿入した順」に見える場合もあるが、保証なし

				- MVCC

		- ソート対象はSELECTで指定する「列名」や「列数」を指定する

			- 列の別名は使えない

		- 複数指定される場合、左から順

			- 例：ORDER BY 1 DESC, 2 ASC;

		- NULLを含む列のソートはNULLS FIRST／NULLS LASTを活用する

			- PostgreSQLの暗黙では通常の昇順ソートでは、非NULL値の後ろに出力され

	- LIMITとOFFSET

		- 指定のOFFSETから指定の件数まで取得

			- ORDER BYと併用しないと取得レコードの一致性保証はない

				- VACUUMを実行したら変わるなど

		- LIMIT句はPostgreSQLの独自拡張

			- OracleのROWNUMBERのようなものもある
			- SQL標準記法

				- OFFSET count FETCH FIRST|NEXT ROW count ONLY;
				- ORDER BY column_list [ASC |DESC]
OFFSET offset_row_count {ROW | ROWS}
FETCH {FIRST | NEXT} fetch_row_count {ROW | ROWS} ONLY

	- SELECT DISTINCT [ON(重複除去対象の列名)] 列名

		- ONを指定する際に、指定された列名のみ重複除去
指定されない場合、SELECT指定の全て列名を重複除去
		- ORDER BY より優先度が低い

	- ＊ 検索条件 GROUP BY 対象列名 [HAVING 条件]

		- 集約関数を用いた以外、基本的にSELECTで指定できるのはGROUP BYで指定されたものだけ

			- 例外はGROUP BYにユニークキーを指定した場合

		- HAVINGはGROUP BYの結果に対して評価
集約関数や式を使える

	- 検索条件

		- IN(値)とNOT IN(値)

			- 値はカンマ区切りで複数可能

		- 式 演算子 ANY (副問い合わせ)

			- 副問合せの結果が必ず一つの列であること
			- 値を指定する場合は配列形式とする
また、演算子も使える

				- 式と副問い合わせ結果のいずれかとの演算結果が真の場合真となる

		- BETWEEN 値 AND 値

			- 値1＞値2の場合、エラーになりませんが、行の選択にもありません

		- EXISTS(副問合せ)とNOT EXISTS(副問合せ)

	- 結合

		- INNER JOIN

			- 結合対象のテーブルから結合キーの条件に合致する行だけを結び付けて出力

				- SELECT 列名 FROM
 テーブル1 [INNER] JOIN テーブル名2
 ON テーブル名1.列名 = テーブル名2.列名
 WHERE 条件;

					- INNERは省略可能

				- SELECT 列名 FROM
 テーブル1 [INNER] JOIN テーブル名2
 USING(列名,...) WHERE 条件;

					- 結合キーをONで指定

				- SELECT 列名 FROM
 テーブル1 [INNER] JOIN テーブル名2
 WHERE 条件;

					- 結合対象テーブルにある同じ名前の列を全て結合キーとして使用する

		- CROSS JOIN

			- 結合対象のテーブルのすべての行を結び付けて出力

				- SELECT 列名 FROM テーブル名1 CROSS JOIN テーブル名2 WHERE 条件;
				- SELECT 列名 FROM テーブル名1, テーブル名2 WHERE 条件;

		- OUTER JOIN

			- 結合対象のテーブルから結合キーの条件を合致する行に加え、結合キーの条件に合致しなかったいずれか片方のテーブルの行を出力

				- SELECT 列名 FROM テーブル名1 {LEFT | RIGHT | FULL} [OUTER] JOIN テーブル名2 ON テーブル名1.列名 = テーブル名2.列名 WHERE 条件;
				- SELECT 列名 FROM テーブル名1 {LEFT | RIGHT | FULL} [OUTER] JOIN テーブル名2 USING(列名) WHERE 条件;
				- SELECT 列名 FROM テーブル名1 {LEFT | RIGHT | FULL} [OUTER] JOIN テーブル名2 WHERE 条件;

	- 処理順

		- FROM句の結合 → WHERE句の絞り込み → GROUP BY句の集約 → HAVING句の絞り込み → ORDER BY句のソート → OFFSET句のスキップとLIMIT句の件数制限

- SELECT 関数名;
- SELECT 式;
- 副問合せ

	- 副問合せ部分をかっこ（()）でくくる、副問合せをテーブルとして扱う場合、必ずエイリアス（別名）が必要になる

- SELECT文 UNION [ALL] SELECT文

	- 和集合を取る、ALLを省略した場合は、重複する行が自動的に除去される
	- 対象となる両方のSELECT文の結果は列数が同じであり、かつ対応する列のデータ型がそれぞれ互換性を持っている必要

- SELECT文A EXCEPT [ALL] SELECT文B

	- 差集合を取る、他の条件はUNIONと同じ

- SELECT文 INTERSECT [ALL] SELECT文

	- 積集合を取る、他の条件はUNIONと同じ

### データの変更

- INSERT INTO テーブル名 [(列名)] VALUES (挿入データ);

	- 一度INSERTで複数行を挿入することが可能
→ VALUES (1), (2)
	- SELECTの結果をINSERTを利用するも可能

		- INSERT INTO テーブル名 [(列名)] SELECT文;

- UPDATE テーブル名 SET 列名 = 更新データ WHERE 条件;

	- SET col1=X, col2=Y 或は SET(col1, col2) = (X, Y)
	- WHERE条件を書かないと、全件更新になってしまうことを注意

- DELETE FROM テーブル名 WHERE 条件;

	- UPDATEと同様に、WHERE条件を書かないと全件削除されてしまうこと

- TRUNCATE テーブル名;

	- 対象のテーブルを空にする

### データ型

- 数値型

	- smallint

		- 2byte

			- -32768から+32767

	- integer/int

		- 4byte

			- -2147483648から+2147483647

				- 10位

	- bigint

		- 8byte

			- -9223372036854775808から+9223372036854775807

				- 19位

	- numeric/decimal

		- 可変長

			- 小数点より上は131072桁まで、小数点より下は16383桁まで

	- real/float

		- 4byte

			- 6桁の精度

	- double precision

		- 8byte

			- 15桁の精度

- 文字列

	- char(n)/character(n)

		- 空白で埋められた固定長（n文字数）の文字列

			- nを省略した場合、1桁になる
			- nに満たない文字列を格納した場合は、文字の右側が半角の空白文字で埋められる
			- nより大きい文字数を格納しようとするとエラーになる

				- 例外：はみ出した文字のすべてが半角の空白文字だった場合は、nまで切り詰められて格納

	- varchar(n)/character varying(n)

		- 文字列の上限をした可変長の文字列

			- nを省略した場合、上限なしになる
			- それ以外はchar型と同様

	- text

		- 可変長の文字列

			- varcharのnを省略した場合と同様の振る舞いになる

				- 実際には、2019_Version11では、1GBの文字格納領域が上限となっている

	- name

		- 63byteまでのオブジェクト名称（システム内部でのみ使用）

	- bytea

		- 可変長のバイナリ列

			- 標準SQLではBLOBという型が使われる

- 日付／時刻

	- date

		- 4byte

			- 日付（時刻なし）

				- 2019-11-15

	- time [without time zone]

		- 8byte

			- 時刻（時間帯なし）

				- 8:00:00.000000

	- timestamp [without time zone]

		- 8byte

			- 日付と時刻（時間帯なし）

				- 2019-11-15 8:00:00.000000

	- time with time zone

		- 12byte

			- 時刻（時間帯あり）

				- 8:00:00.000000+09

	- timestamp with time zone

		- 8byte

			- 日付と時刻（時間帯あり）

				- 2019-11-15 8:00:00.000000+09

	- interval [fields]

		- 16byte

			- 時間間隔

				- ‘163 days 18:53:00.21035’

					- 時間感覚を表現するやや特殊なデータ型、[interval ‘数値 単位’]の形で記述される。単位は、year、month、day、hour、minuteなどが使用できる。計算式で多く用いられる

						- 例：SELECT current_date - interval ‘1 day’;

- 通貨データ型

	- 	money

		- 8byte

			- 金額（精度はlc_monetaryパラメー タで決定）

- 論理値

	- boolean

		- 1byte

			- 真や偽、もしくは不定（NULL）の状態を保持

				- 真の表記文字（小文字と大文字混在も可）

					- ‘t’
					- ‘true’
					- ‘y’
					- ‘yes’
					- ‘on’
					- ‘1’
					- TRUE（シングルクォートで囲まない）

				- 偽の表記文字（小文字と大文字混在も可）

					- ‘f’
					- ‘false’
					- ‘n’
					- ‘no’
					- ‘off’
					- ‘0’
					- FALSE（シングルクォートで囲まない）

				- 不定

					- NULL

- 連番

	- smallserial

		- 2byte

			- 1から32767

	- serial

		- 4byte

			- 1から2147483647

	- bigserial

		- 8byte

			- 1から9223372036854775807

- OID

	- PostgreSQL独自のデータ型、主にオブジェクト（テーブルやインデックス、関数など）の識別用

		- 4byteの数値で、データベースクラスタ全体一意の値
		- 隠し列であるため、SELECTの列を指定しない限りは参照することはできない
		- テーブルやインデックスでは、regclassへキャストするとoidが取る

			- 例：oid = ‘pg_type’::regclass;

- 配列

	- 配列はすべてのデータ型に使用でき

		- c1 データ型[][]

			- 多次元もサポートする、また、次元数を指定しても実際にはその次元数を超えるの配列にすることもできる

	- 挿入

		- 一次元

			- INSERT INTO array_table VALUES (‘{1,2,3}’)

		- 二次元

			- INSERT INTO array_table VALUES (‘{{1,2,3},{2,3,4}}’)

	- 検索

		- SELECT c1[1:2] FROM array_table;

			- 配列の1から2番目の要素を取得

		- SELECT * FROM array_table WHERE c1[2] = ‘VUT’;

			- 配列の2番目の要素が[VUT]であるものを検索

		- SELECT * FROM array_table WHERE ‘ABC’ = ANY(c1);

			- 配列の要素が１つでも特定の条件を満たせるばよいの場合、ANYを利用

	- 更新

		- 特定の要素番号の指定や、関数の使用する

			- UPDATE array_table SET c1[4] = ‘JKL’ WHERE ‘ABC’ = ANY(c1);

- NULL

	- 不定を表すデータ

		- 通常の演算子で比較などができない

			- IS NULL
			- IS NOT NULL

		- NULLを含む列のソートはNULLS FIRST／NULLS LASTを活用する

			- PostgreSQLの暗黙では通常の昇順ソートでは、非NULL値の後ろに出力され

- キャスト

	- あるデータ型を別のデータ型に変換する処理

		- データ::変換後のデータ型
		- CAST(データ AS 変換後のデータ型)

### テーブルの定義

- 作成と削除

	- CREATE TABLE テーブル名 (列名 データ型 [デフォルト値][, ...]);

		- デフォルト値には、固定値の他、関数なども利用できる、指定しないと暗黙的にNULLが指定される

	- ALTER TABLEより追加や削除可能

		- 既存のデータが制約に合致しない場合は挿入・更新時にエラーとなる

	- DROP TABLE テーブル名;

- 制約

	- 主キー（プライマリーキー）制約

		- 重複値、NULLの格納を禁止する。１つのテーブルに１つだけ作成可

			- CREATE TABLE テーブル名 (列名 データ型 PRIMARY KEY);
			- CREATE TABLE テーブル名 (列定義..., PRIMARY KEY(列名[, 列名...]));
			- ALTER TABLE テーブル名 ADD PRIMARY KEY(列名);
			- ALTER TABLE テーブル名 ADD CONSTRAINT 主キー名 PRIMARY KEY(列名);
			- ALTER TABLE テーブル名 DROP CONSTRAINT 主キー名;

	- ユニーク制約

		- 重複値の格納を禁止する

			- CREATE TABLE テーブル名 (列名 データ型 UNIQUE);
			- CREATE TABLE テーブル名 (列定義..., UNIQUE(列名[, 列名...]));
			- ALTER TABLE テーブル名 ADD UNIQUE(列名);
			- ALTER TABLE テーブル名 ADD CONSTRAINT ユニークキー名 UNIQUE(列名);
			- ALTER TABLE テーブル名 DROP CONTRAINT ユニークキー名;

	- NOT NULL制約

		- CREATE TABLE テーブル名 (列名 データ型 NOT NULL);
		- ALTER TABLE テーブル名 ALTER COLUMN 列名 SET NOT NULL;
		- ALTER TABLE テーブル名 ALTER COLUMN 列名 SET NULL;

			- ALTER TABLE テーブル名 ALTER COLUMN 列名 SET NOT NULL;

				- スタイルがあるSET NULLはエラー出ている、DROP NOT NULLはいける

	- 外部キー制約

		- 参照先の列にない値の格納を禁止する、参照整合性制約にも呼ばれる

			- CREATE TABLE テーブル名(列名 データ型 REFERENCES 参照先テーブル名(列名));
			- CREATE TABLE テーブル名(列名 データ型, FOREIGN KEY (列名) REFERENCES 参照先テーブル名(列名));
			- ALTER TABLE テーブル名 ADD CONTRAINT 外部キー名 FOREIGN KEY(列名) REFERENCES 参照先テーブル名(列名);
			- ALTER TABLE テーブル名 DROP CONSTRAINT 外部キー名;

	- チェック制約

		- ユーザが定義した値以外の格納を禁止する

			- CREATE TABLE テーブル名(列名 データ型 CHECK(条件式));
			- ALTER TABLE テーブル名 ADD CONSTRAINT チェック制約名 CHECK(条件式);
			- ALTER TABLE テーブル名 DROP CONSTRAINT チェック制約名;

	- ドメイン制約

		- ユーザが定義した値以外の格納を禁止する、チェック制約を伴った独特のデータ型と考える、管理上は楽になれる

			- CREATE DOMAIN ドメイン名 AS データ型 [NULL | NOT NULL | CHECK(条件式)];
			- CREATE DOMAIN ドメイン名 AS データ型 [NOT NULL | NULL] [CHECK(チェック基準)];

				- チェック基準では、列値を「VALUE」というキーワードで参照可能

					- 例：NOT NULL CHECK(VALUE = upper(VALUE));

			- DROP DOMAIN ドメイン名 [CASCADE];

				- 利用中がある場合、削除はエラーになる。CASCADEにして削除できるが、ドメイン制約を使っている列がすべてのテーブルから削除されてしまうので注意

- 列のデフォルト値

	- データ挿入時に値が省略された列に使用する値

		- CREATE TABLE テーブル名 ( 列名 データ型 DEFAULT デフォルト値, ...);
		- ALTER TABLE テーブル名 ALTER COLUMN 列名 SET DEFAULT デフォルト値;
		- ALTER TABLE テーブル名 ALTER COLUMN 列名 DROP DEFAULT;

- 定義の変更

	- テーブル名の変更

		- ALTER TABLE テーブル名 RENAME TO 新しいテーブル名;

	- テーブルの所有者変更

		- ALTER TABLE テーブル名 OWNER TO 新しい所有者となるユーザ名;

	- 列名の変更

		- ALTER TABLE テーブル名 RANAME COLUMN 列名 TO 新しい列名;

	- 列の追加

		- ALTER TABLE テーブル名 ADD COLUMN 追加する列名 データ型 制約;

	- 列の削除

		- ALTER TABLE テーブル名 DROP COLUMN 削除する列名;

	- 列のデータ型変更

		- 互換性があるデータ型の変更が必要

			- ALTER TABLE テーブル名 ALTER COLUMN 列名 SET DATA TYPE 新しいデータ型 [USING 式];

				- USING句は新しいデータ型にキャストできない場合に必要

### パーティション

- テーブルをある範囲に区切ったものをパーティションという。また、そのようにテーブルを区切ることをパーティショニングと言う。

	- 親テーブルをパーティションテーブルに分割する形で作成、分割することになる。

		- メリット

			- 巨大なテーブルを小さなテーブルに物理的に分割できるので、削除やメンテナンスの粒度を細かくできる
			- キャッシュヒットの向上や集計処理ときのアクセス対象テーブルの絞り込みができる

		- CREATE TABLE 親テーブル名(列名 データ型) PARTITION BY RANGE(列名);

			- その他分割方法は、RANGE/LIST/HASHといった数種類の分割方法がサポートする

		- CREATE TABLE パーティションテーブル名(列名 データ型) PARTITION OF 親テーブル名 FOR VALUES 条件;
		- DROP TABLE パーティションテーブル名;

			- この時、親テーブルのデータも一緒に削除される

		- ALTER TABLE 親テーブル名 ATTACH PARTITION パーティションテーブル名 FOR VALUES 条件;

			- 親テーブルとは独立したテーブルとして作成しておき、後から親テーブルへ追加することができる

		- ALTER TALBE 親テーブル名 DETACH PARTITION パーティションテーブル名;

			- パーティションテーブルを切り離して、親テーブルとは別テーブルとして独立させることもできる

### シーケンス

- 自動で連番を払い出してくれるオブジェクトです。データベースでは、ユニークなIDなどを払い出す際に多用される

	- CREATE SEQUENCE シーケンス名 [オプション];

		- オプション

			- INCREMENT増加値

				- シーケンスの増加値を指定する。デフォルトは1。マイナス値の指定可

			- START値

				- 開始値、デフォルトは1。マイナス値の指定可

			- [NO]CYCLE

				- シーケンスが最大値まで達した場合の振る舞いを指定する。CYCLEを指定すると開始値に戻る。NO CYCLEはデフォルトの設定でおり、最大値をお超えた値を払いでそうとするとエラーになる

	- 現在値の確認

		- SELECT currval(‘シーケンス名’);

			- currval()関数は、そのセッション内でnextval()関数を実施してからでないとエラーになる

	- 新しい値の取り出し

		- SELECT nextval(‘シーケンス名’);

	- 特定の値に設定

		- SELECT setval(‘シーケンス名’, 設定値);

			- シーケンスが一度払い出した値は、トランザクション内でROLLBACKやABORTを行っても元に戻りません、戻りたい場合、setval()を使ってもどしましょ

	- DROP SEQUENCE シーケンス名;

- データ型:bigint、8byteの整数範囲

### ビュー

- あるSELECT文の結果を定義したものです。

	- メリット

		- 複雑／長大なSQLをビューにすることで可視性を高めることができ、またクライアントからDBサーバへのネットワークの流量を減らすこともできる。
		- ユーザやアプリケーションごとに見せたい検索範囲を限定するなども有効

	- CREATE VIEW ビュー名[(列名)] AS SELECT文;

		- 列名を省略すると、SELECT文で出力される列名がそのままビューでも使われる

	- DROP VIEW ビュー名;

### インデックス／索引

- 注意点

	- 小さいなテーブルにはあんまり効果が出ない、運用上のデータ量に対して考慮する
	- インデックスを使うとアクセスはランダムアクセスになる（B＋ツリーなど）
	- 更新する際に、テーブルとインデックスの２つを更新する必要になるため、更新処理性能が低下することもある

- 種類

	- B-Tree

		- 一般的に用いられるインデックス。スカラー値（単一の値）に対する一致や不等号の検索に用いる

	- GiST

		- 主に空間情報の検索に用いられるインデックス。二次元以上の空間に対する一致や包括の検索に用いる

	- GIN

		- 主に全文検索に用いられる転置インデックス。ある要素がどの列にあるかを検索するのに用いる

	- ハッシュ

		- 値の一致検索のみをサポートするハッシュインデックス

	- SP-GiST
	- BRIN など

- インデックスの作成／削除

	- CREATE INDEX インデックス名 ON テーブル名 [USING インデックス種別] (列名);

		- 種別のデフォルトはB-Tree、B-TreeやGiST、GINの場合列名にはカンマ区切りで複数を列挙することができる

	- ALTER TABLE DROP INDEX インデックス名;

- マルチカラムインデックス

	- 異なるデータ型の列を含めることもできる

- 関数インデックス／式インデックス

	- 関数や式の結果を検索条件としている場合、これらをインデックス化することで検索性能を向上させることが可能

		- 実行するたびに出力が異なるような関数の結果はインデックスとして用いることができない（now()、random()関数など）
		- 式は二重のカッコ(())で包む必要があるを注意

			- 例：CREATE INDEX インデックス名 ON テーブル名 ((列名1 || ‘-’ || 列名2))

- 部分インデックス

	- テーブルの特定の範囲のデータのみにインデックスを作成する場合に使用する

		- 例：あるテーブルのある列の9割近くが「X」という値で占められており、残り1割に「別の値」がいれっているの場合は有効

			- 例：CREATE INDEX インデックス名 ON テーブル名(列名) WHERE 列名 < 100;

### トリガ

- 特定のテーブルに対して挿入／更新／削除が行われた場合に、規定のユーザ定義関数（プロシージャ）を発動させる機能です。

	- トリガーの作成

		- CREATE TRIGGER トリガー名 {BEFORE | AFTER}{UPDATE | INSERT | DELETE | TRUNCATE} ON テーブル名 [FOR [EACH] {ROW | STATEMENT}] EXECUTE PROCEDURE 関数名;

			- 更新種別は、ORで繋げて複数指定することが可能
			- UPDATEを指定した場合に限り、「UPDATE OF 列名」の書式で、特定の列が更新された時にトリガーが発動させることが可能

				- 特定の列が更新されたかどうかは、NEWとOLDというキーワードを使って判別する

					- OLDは更新前の値
NEWは更新後の値

				- INSERTはNEWのみ使える、DELETEはOLDのみ使える

			- 例：CREATE TRIGGER trg_1 AFTER UPDATE OR INSERT ON table FOR EACH ROW WHEN(OLD.c1 IS DISTINCT FROM NEW.c1)EXECUTE PROCEDURE trg_f();

	- 発動タイミング

		- 行ごと、SQLごとのいずれかを指定する

			- トリガーの発動を、処理対象の行ごとに行うか、或いはSQLごとに行うかを指定できる

				- FOR EACH ROW

					- 各行に対して一回ずつトリガー関数を実行

				- FOR EACH STATEMENT

					- 元SQLに対して一回だけトリガー関数を実行

		- UPDATE／INSERT／DELETE／TRUNCATEの種別を指定
		- 更新対象の列を指定する

			- 特定の列の更新をトリガを発動させる、といった指定が可能です。特定の列を対象にすることも可能

		- 実際に更新されたどうかを指定する

			- 実際の値が更新されたらトリガーが発動させる、特定の列を対象にすることも可能

		- 更新の前後を指定する

			- 更新処理の前後のどちらでトリガーを発動させるかを指定できる、更新処理の前でトリガを発動させる場合、更新結果を参照することはできない

		- BEFORE STATEMENTトリガー

➔ BEFORE ROWトリガー

➔ 元SQLの行処理（INSERT、UPDATE、DELETE）

➔ AFTER ROWトリガー

AFTER STATEMENTトリガー

	- トリガーの削除

		- DROP TRIGGER トリガー名 ON テーブル名;

### ルール

- ある条件のSQLを発行された場合、それを別のSQLで置き換えたり、別の処理を付け加えたりしたい場合利用

	- CREATE RULE ルール名 AS ON イベント TO テーブル名 DO [INSTEAD | ALSO] SQL文;

		- イベント名

			- テーブル名に対して実施されるINSERT/SELECT/UPDATE/DELETEのいずれかを指定する
			- INSERT/UPDATEの際に、NEWキーワードが使える

	- DROP RULE ルール名 ON テーブル名;
	- PostgreSQLではビューを実現するために使っている
➔
ビューに対するSELECTクエリをビュー定義のSELECTクエリに置換
	- パーティションテーブルやビューへの直感的な更新を可能に

### スキーマ

- 名前空間とも呼ばれ、テーブルや関数などのオブジェクトが含まれる。PostgreSQLでは、デフォルトでpublicスキーマが作成され

	- スキーマの作成

		- CREATE SCHEMA スキーマ名 [AUTHORIZATION データベースユーザ名];

			- AHTHORIZATIONが省略する場合は、発行したデータベースユーザ名を所有者としてスキーマが作成する

		- CREATE SCHEMA AUTHORIZATION データベースユーザ名;

			- スキーマ名が省略する場合は、データベースユーザ名がスキーマ名になる

	- 特定のスキーマにテーブルなどの作成

		- スキーマ名.テーブル名
		- 異なるスキーマ間で同じ名前のテーブルや関数を作成できる

			- スキーマ検索パスより、優先的に使用する

				- postgresql.confのsearch_pathパラメータ

					- デフォルト「$user,public」

				- SETを使ってsearch_pathパラメータを動的に変更

					- SET search_path TO スキーマ名;

				- 標準的に使われる関数はすべてpublicスキーマにある

### 関数とプロシージャ

- ストアドプロシージャと呼ばれる

	- 関数（FUNCTION）

		- 戻り値定義できる
		- SELECTで呼び出す
		- 定義内トランザクションのCOMMITやROLLBACKができない

	- プロシージャ

		- 戻り値定義できない
		- CALLで呼び出す
		- 定義内にトランザクションのCOMMITやROLLBACKができる

	- 関数の実装

		- SQL、PL/pgSQL、C言語はインストール時点で利用可能
		- PL/Perl、PL/Pythonなどは追加機能で利用可能（パッケージインスト ールだと最初から利用可能）

			- PerlやTcl、Pythonなどが利用可能だが、利用の前に createlangコマンドで手続き言語をデータベースに登録する必要がある

- 定義

	- CREATE [OR REPLACE] FUNCTION 関数名(引数) RETURNS [SETOF] 戻り値 AS $$関数の記述
$$ LANGUAGE 使用言語;

		- 例

			- CREATE FUNCTION cat_func(int, text) RETURNS text AS $$
SELECT ($1 || $2)::text /* コメント */
$$ LANGUAGE SQL;

	- CREATE [OR REPLACE] PROCEDURE プロシージャ名(引数) AS $$関数の記述 
$$ LANGUAGE 使用言語;

		- 例

			- CREATE FUNCTION insert_func(int, text) RETURNS SETOF tbl AS $$
SELECT * FROM tbl WHERE col1=$1 AND col2=$2
$$ LANGUAGE SQL;

- 構造

	- CREATE ... 
$$
DECLARE
  変数の宣言
BEGIN
  関数の処理記述
END
$$
LANGUAGE plgpsql;

		- 変数の宣言

			- 変数名 [CONSTANT] データ型 [NOT NULL] [{DEFAULT | :=} 値など];

		- 関数の処理記述

			- ループではLOOP、EXIT、CONTINUE、WHILE、FORを使える、
条件分岐はIF、CASE文をそれぞれ使える。

		- コメント

			- 「−−コメント」（1行）或いは「/* コメント */」

### テーブルスペース

- 物理的にはテーブルスペース（表領域）と呼ばれる場所に格納される

	- PostgreSQLでは、ユーザが新たにテーブルスペースを用意してそこへテーブルを作成することができる

		- 性能の良いストレージを利用したり、データを複数のテーブルスペースに分散して格納することもできる

	- CREATE TABLESPACE テーブルスペース名 LOCATION ‘パス名‘;

		- PostgreSQLでは、default_tablespaceパラメータでデフォルトのテーブルスペースを変更できる

			- SET default_tablespace = テーブルスペース名;

		- テーブルスペース指定してテーブル作成もできる

			- CREATE TABLE テーブル名(列名) TABLESPACE テーブルスペース名;

	- DROP TABLESPACE テーブルスペース名;

### マテリアライズドビュー（通称マテビュー）

- 通常のビューとはことなり、実体を持つビューのこと

	- 即時性を求めないようなデータで、結果取得に時間がかかるビューが必要となる場合、効果が発揮する
	- 実体が持っているので、インデックス作成もできる

- CREATE MATERIALIZED VIEW マテリアライズドビュー名 AS SELECT文;
- REFRESH MATERIALIZED VIEW マテリアライズドビュー名;

	- マテリアライズドビューに格納されているデータは、常に最新というわけではないため、必要に応じて随時データを更新する必要がある

- DROP MATERIALIZED VIEW マテリアライズドビュー名;

## トランザクション

### 概念

- 複数のSQL処理をBEGINやCOMMITなどのコマンドで囲み、グループ化することでのもの

	- ACID特性

		- Atomicity（原子性）

			- トランザクションは実行が完了するか、或いは全く実行されないかのどちらかとなる

		- Consistency（整合性）

			- トランザクションの開始と終了時には、データベースは整合性を保った状態となっている

		- Isolation（分離性）

			- トランザクションは別のトランザクションによる処理の影響を受けない

		- Durability（持続性）

			- トランザクションにより変更されたデータは確実に保持される

	- 制御コマンド

		- BEGIN;
または
START TRANSACTION;

（SQLなどによる処理）

COMMIT;
あるいは
ROLLBACK;
もしくは
ABORT;

			- COMMIT,END,END TRANSACTIONは同義です

				- ENDを用いた方法はPostgreSQL独自の構文

			- ROLLBACKとABORTは同義です
			- PostgreSQLでは、トランザクション中にSQLなどがエラーとなったり、セッションの切断が発生するなどした場合、その時点でトランザクションはABORT扱いとなる
			- SQLでエラーでトランザクション全体がアボート、それまでの全ての変更はロールバックされる、明示的にABORT（ROLLBACK）するまで、全てのSQLがエラーとなる SQLの文法エラーやテーブル名の間違いなどでも同様

				- エラー後に処理継続できる他のRDBMS（Oracleなど）から移行した 場合は要注意！

			- 一部DDLがトランザクション内で実行可能

				- PostgreSQLでは、CREATE TABLEやDROP TABLEなどのDDLもトラ ンザクション内で実行可能

					- ロールバックするとDDL実行前の状態に戻る
					- TRUNCATEもロールバック可能
					- ロールバックできないDDL（DROP DATABASEなど）もあるので注意

						- 更新したシーケンスもロールバックできない

			- PostgreSQLの操作で多用するpsqlでは、デフォルトでautocommitが有効になっている

				- SQL実行前にBEGINが暗黙的に発行され、SQLがエラーなく終了すれば自動でSQLの実行結果がコミットされる
				- ❗️ただ、BEGINが明示的に発行された場合は、COMMITが発行されるまでトランザクションがコミットされることがありません

					- 重要データ操作する際に必ず事前にBEGINを入れましょ

	- トランザクション内のSETの有効性もトランザクションの内のみ、トランザクション外でのSETの有効性は接続セッション

### SAVEPOINT

- トランザクション中には部分的なロールバックをしたい場合に利用され、任意の名称のSAVEPOINTをトランザクション中に作成し、必要があればその時点までロールバックする

	- SAVEPOINT セーブポイント名;
	- ROLLBACK TO 戻る時点のセーブポイント名;
	- RELEASE SAVEPOINT セーブポイント名;

		- トランザクション内で同じセーブポイント名を設定した場合、一時的に古い（先に設定した）方を上書きする。その後、新しい方のSAVEPOINTをRELEASEすると、再び古いSAVEPOINTを利用できるようになる

### 分離性

- DBMSでは分離性のレベルを設定できるようになっているが多い

	- 分離レベルの影響

		- ダーティリード（Dirty Read）

			- 自分のトランザクションから、別のトランザクションのみコミットな状態の挿入／更新／削除結果が見えてしまうこと

		- ファジーリード（または反復不能読み取り）（Fazzy Read）

			- 自身のトランザクションから、別のトランザクションのコミットされた更新／削除結果が見えてしまうこと

		- ファントムリード（Phantom Read）

			- 自身のトランザクションから、別のトランザクションのコミットされた挿入結果が見えてしまうこと

		- 直列化異常

			- 複数のトランザクションのコミット結果が、トランザクションを一つずつ重ねならないように実行した結果と比較して、どのような順序を仮定しても目的と違う結果になり、一貫性のない状態になってしまうこと

	- 分離レベル

		- Read uncommited

			- dirtyRead,FuzzingRead,PhantomRead,直列化異常が起こり得る

				- PostgreSQLでも指定できますが、振る舞いとしてはRead commitedと同じものになっている

		- Read commited

			- ファージーリード、ファントムリード、直列化異常が起こり得る

		- Repeatable read

			- ファントムリード、直列化異常が起こり得る

		- serializable

			- 上記四つの異常のいずれも起こらない

				- ただし、複数トランザクションを同じ項目を更新しようとする場合、整合性に矛盾を生じる可能性がある、回避するために、片方（後発）のトランザクションをエラーにし自動的にROLLBACKされ

	- 分離レベルの設定

		- SET default_transaction_isolation TO|= '分離レベル';

			- セッションのデフォルト分離レベル設定

		- SET TRANSACTION ISOLATION LEVEL 分離レベル;

			- トランザクション開始直後に分離レベルを明示

		- BEGIN|START TRANSACTION ISOLATION LEVEL 分離レベル;

			- トランザクション開始時に分離レベルを明示

	- 分離レベルの確認

		- SHOW default_transaction_isolation;
		- SHOW transaction_isolation;

### ロック

- 行ロック

	- 排他ロック

		- 同時に他の排他や共有ロックを同時に取得できない

			- UPDATEやDELETEでは自動的に排他ロックを取る

	- 共有ロック

		- 同時に他の排他ロックは取得できない

	- 書式

		- SELECT 列名 FROM テーブル名 WHERE 条件 [FOR UPDATE | SHARE];

			- SELECTデフォルトではロックを掛けない
			- SELECT対象のすべて行に対してロックを取る

		- ロックされた対象に対してロックを取得する際に、デフォルトはWAIT

- テーブルロック

	- テーブル単位での排他／共有ロックを明示的に取得する場合に使用する。主に一時的に特定のテーブルへの検索や更新処理をブロックしたい場合に使われる

		- LOCK TABLE テーブル名 [IN ロックモード];

			- ロックモード

				- EXECLUSIVE MODE

					- ロック対象てのテーブルへのSELECTのみを許し。INSERT／UPDATE／DELETEなどの処理の他、VACUUMやANALYZEなどのメンテナンス処理もブロックする

				- ACCESS EXCLUSIVE MODE

					- ロック対象のテーブルへのすべての処理をブロックする、ロックモードを省略した場合はこのACCESS EXCLUSIVE MODEでロックを取る
					- ALTER TABLEや一部のメンテナンスコマンドでは、暗黙的なテーブルの排他ロックが取得される

- ロックの有効期間

	- トランザクション中に掛かったロックはこのトランザクションの終了まで有効（COMMITやROLLBACK）

- デッドロック

	- 複数の処理がお互いの処理の終了（ロックの解放）を待つでおり、処理を進めることができない状態になってしまったこと

		- PostgreSQLでは、SQLの実行時点でデッドロックを検知したトランザクションをABORTし、デッドロック状態から自動的に回復する

## 組込み関数と演算子

### 集約関数

- 単純に検索条件に使えないです。使いたい場合は、副問合せを使うか、HAVINGを利用する
- count(列名)

	- 列名を指定した場合、NULLであるレコードは無視され
	- 「*」（アスタリスク）を指定する場合は条件に合致する全レコード数を取得

- sum(列名),avg(列名)

	- 引数には、int型やbigint型など、数値のデータ型しか指定できない
	- NULL値である場合、その行を無視される

- min(列名),max(列名)

	- 引数には数値型、日付型や文字列型を受け付ける

		- 注意：文字列の数学意味とエンコード値の順位ずれの可能性がある←Localeより

### 比較演算子

- ＜、＞、＜＝、＞＝、＝、＜＞または！＝

### 算術関数

- abs(x)

	- 絶対値

- div(y, x)

	- 商

		- 入力値がfloat型やnumeric型のような小数点をサポートしているデータ型であっても、整数値を返す

			- 注意：演算子「/」では、入力値のデータ型によって戻り値のデータ型が変わる

- mod(y, x)

	- 余

		- 二つ引数の内、精度の高いデータ型に揃えられてから処理する、そのデータ型で結果が返される

- sqrt(x)

	- 平方根

- power(x, y)

	- y乗

- ceil(x)

	- 整数まで切り上げ

- floor(x)

	- 整数まで切り捨て

- round(x)／round(x, 小数点を四捨五入する位)

	- 四捨五入

- trunc(x)／trunc(x, 小数点を切り捨てる位)

	- 小数点を切り捨て

- log(x)

	- 常用対数（10のなん乗はx）

- pi()

	- 円周率

- random()

	- 0.0 <= x < 1.0の範囲の乱数

		- 例：(random() * 100)::int % 100

	- 引数のデータ型と同じデータ型が返される

### 算術演算子

- +

	- 和

- -

	- 差

- *

	- 積

- /

	- 商（整数の割り算では余りを切り捨て）

- %

	- 剰余（余り）

- ^

	- 累乗

		- 2^3=8

- |/

	- 平方根

		- |/2=1.4142135623731

- ||/

	- 立方根

		- ||/2=1.25992104989487

- !

	- 階乗

		- 4!=24

- !!

	- 階乗（前置演算子）

		- !!4=24

- @

	- 絶対値

		- @‘-2.5’=2.5

- 演算子の優先順位

	- 累乗（^）
	- 乗算（*）、除算（/）、剰余（%）
	- 加算（+）、減算（-）
	- その他の算術演算値
	- <,>,=,<=,>=,<>

### ビット演算子

- AND、OR、XOR、NOTはそれぞれ「&」、「|」、「#」、「~
- ビットシフトは左シフトが「<<」、右シフトが「>>」

### 文字列演算子と述語

- ||

	- 文字列を連携する

- LIKE

	- 文字列に対する前方／後方／中間一致検索を行い

		- 中間一致のは「%ABC%」のイメージ

	- SELECT 列名 FROM テーブル名 WHERE 列名 LIKE ‘条件’;

		- 条件

			- 文字列
			- 「%」（パーセント記号）

				- 任意の文字列

			- 「_」アンダースコア

				- 任意の1文字

			- \

				- 直後のメタ文字をエスケープ

	- ILIKE

		- パターンが文字列全体にマッチしたら真（大文字小文字区別なし）

			- PostgreSQL独自の拡張である

- SIMILAR TO

	- 正規表現を使ったパターンマッチング、パターンが文字列全体にマッチしたら真
	- 文字列 SIMILAR TO ‘条件’

		- メタ文字

			- %
			- _
			- |

				- 二者択一

			- *

				- 直前の項目の0回以上の繰り返し

			- +

				- 直前の項目の1回以上の繰り返し

			- ?

				- 直前の項目の0回もしくは1回の繰り返し

			- {m}

				- 直前の項目のm回の繰り返し

			- {m,}

				- 直前の項目のm回以上の繰り返し

			- {m,n}

				- 直前の項目のm回以上かつn回以下の繰り返し

			- ()

				- ()内を１つの論理項目として指定

			- []

				- []内のいずれか１つの文字に一致

			- ^$など一般式
			- 非拡張正規表現、%と_は例外

- ~

	- POSIX正規表現：パターンが文字列の一部にマッチを行う演算子

		- 文字列 ~ 正規表現

			- パターンが文字列の一部にマッチしたら真
、大文字小文字区別する

		- 文字列 ~* 正規表現

			- パターンが文字列の一部にマッチしたら真
、大文字小文字は区別しない

		- 文字列 !~ 正規表現

			- パターンが文字列の一部にマッチしなかったら真
、一致しないか、大文字小文字は区別する

		- 文字列 !~* 正規表現

			- パターンが文字列の一部にマッチしなかったら真
、一致しないか、大文字小文字は区別しない

### 文字列関数

- length(文字列)/char_length(文字列)

	- 文字数を取得

- octet_length(文字列)

	- DB使っているエンコードよりバイト数が違う可能です

- substring(文字列 [from 開始位置] [for 文字列])

	- 指定位置から指定文字数（デフォルトは残り全部）分の部分文字列を返す
	- 開始位置は文字数ベースで先頭文字が「1」

- upper(文字列)
- lower(文字列)
- repeat(文字列, repeatcount)
- replace(文字列, 置換前文字列, 置換後文字列)
- lpad(文字列, 文字数, [文字])
- rpad(文字列, 文字数, [文字])
- trim([leading | trailing | both][文字] from 文字列)

	- 文字設定しないと❗️半角空白削除になる
	- それぞれltrim(),rtrim(),btrim()の関数も存在

- ltrim()
- rtrim()
- btrim()
- coalesce(arg1, ...)

	- 任意数のパラメータを受け、先頭から第一番目NULLではないパラメータを返却

### 日付／時刻の関数と演算子

- 現在の日付／時刻を取得する関数

	- トランザクション開始時の取得関数

		- now()

			- タイムスタンプを取得

		- current_timestamp

			- タイムスタンプを取得

		- localtimestamp

			- タイムスタンプを取得

		- localtime

			- 時刻を取得

		- current_time

			- 時刻を取得

		- current_date

			- 日付を取得

		- BEGINの実行から、COMMITもしくはABORT/ROLLBACKの実行までの間、右の関数は常に同じ日付／時刻を取得する

	- SQL開始時の取得関数

		- statement_timestamp()

			- タイムスタンプを取得

				- 一つのSQL内で複数回実行しても、そのSQLでは常に同じ日付／時刻を取得することになる

					- 同じトランザクションで複数のSQLでそれぞれ取得すると違う値を取れる

	- 関数実行された時点の取得

		- clock_timestamp()

- age(timestamp値, timestamp値)

	- 「年 years 月 mons 日 days」という形で、日付の差分を返す

- extract(field from timestamp値|interval値)

	- extract(フィールド指定 FROM 日付・時刻/時間間隔)

		- 日付・時刻/時間間隔から指定したフィールド部分を取得する
		- フィールド指定はsecond/month/isodow/timezone/centuryなど
		- フィールド指定は文字列ではないのでクォートは不要

- data_part(‘field’, timestamp値|interval値)

	- date_part(フィールド指定文字列, 日付・時刻/時間間隔）

		- extract()と同様だが、フィールド指定が文字列型

- data_trunc(field, timestamp値)

	- 指定のフィールド精度までの情報を抽出

- 日付／時刻演算子

	- +

		- data ‘2019-07-01’ + integer ‘7’

			- data ‘2019-07-08’

		- data ‘2019-07-01’ + interval ‘1 hour’

			- timestamp ‘2019-07-01 01:00:00’

		- data ‘2019-07-01’ + time ‘03:00’

			- timestamp ‘2019-07-01 03:00:00’

		- interval ‘1 day’ + interval ‘1 hour’

			- interval ‘1 day 01:00:00’

		- timestamp ‘2019-07-01 01:00’ + interval ‘23 hours’

			- timestamp ‘2019-07-02 00:00:00’

	- -

		- date ‘2019-11-19’ - date ‘2019-11-15’

			- integer ‘2’

		- date ‘2019-11-01’ - integer ‘7’

			- date ‘2019-10-25’

		- date ‘2019-11-01’ - interval ‘1 hour’

			- timestamp ‘2019-10-31 23:00:00’

		- time ‘05:00’ - time ‘03:00’

			- interval ‘02:00:00’

		- timestamp ‘2019-11-01 23:00’ - interval ‘23 hours’

			- timestamp ‘2019-11-01 00:00:00’

		- timestamp ‘2019-11-01 03:00’ - timestamp ‘2019-10-30 00:00’

			- interval ‘2 days 03:00:00’

	- *

		- 900 * interval ‘1 second’

			- interval ‘00:15:00’

		- 21 * interval ‘1 day’

			- interval ‘21 days’

	- /

		- interval ‘1 hour / double precision ‘1.5’

			- interval ‘00:40:00’

- データ型書式設定関数／データ型変換

	- to_char(データ, パターン)

		- 日付／時刻型のパターン

			- YYYY
			- MM
			- DD
			- HH

				- デフォルトではHH12

			- HH12
			- HH24
			- MI

				- 分（00〜59）

			- SS
			- MS

				- ミリ秒（000〜999）

			- US

				- マイクロ秒（000000〜999999）

			- AM,am,PM,pm
			- MON

				- 大文字での短縮形の月名

			- DY

				- 大文字での短縮形の曜日名

		- 数値のパターン

			- 9

				- 指定された桁での値

					- 桁数不満の場合、数値の先頭に半角空白が埋め込まれる

			- 0

				- 前にゼロが付いた値

			- .（ピリオド）

				- 小数点

			- .（カンマ）

				- 千単位で区切り符号

	- to_date(文字列, パターン)/to_timestamp(文字列, パターン)/to_timestamp(数値)

		- パターンはto_char()の日付／時刻パターンと同じ

	- to_number(文字列, パターン)

		- パターンはto_char()の数値パターンと同じ

- 部分フィールド取得

	- フィールド：year,month,day,hour,min,sec

## 日付／時刻のデータ型と単純な文字列（’7‘など）は演算できない

*XMind: ZEN - Trial Version*