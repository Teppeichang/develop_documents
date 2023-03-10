## SQLエラー原因・解決策
主に実務でSQLを書いた際にハマったエラーの記録

---

### #1060 **Duplicate column name 'id’**

**【原因】**

テーブル結合時（JOIN）のPK(主キー)重複。

結合するテーブル同士に id というカラムがあり、それらがPKだった。

**【解決策】**

テーブル結合におけるSELECTで * を使用しない。*をテーブル結合時に使用すると、全てのテーブルのすべてのカラムを参照してしまうからである。

---

### #1406 **Data too Long for Column**

**【原因】**

カラムに設定された文字数（桁数）制限のオーバー

**【解決策】**

文字数（桁数）の制限を見直す、もしくはINSERTしようとしているデータが正しい形式かチェックする。

自分のケースでは、連想配列からデータを取り出してINSERTする際、誤って連想配列ごとINSERTしていたため発生した。

---

### #1062 Duplicate entry  ‘○○’ for key ‘PRIMARY’

**【原因】**

データをINSERTしようとしたところ、主キーとなっているカラムの中で値の重複が発生

**【解決策】**

重複しない値をINSERTするか、そもそも重複の可能性がある値を取り扱うカラムをPKにしない。主キー2つ設定でもいける。

---

### #1052 Column ‘○○’ in field list is ambiguous

**【原因】**

テーブルをJOIN（INNER JOIN）するとき、SELECT文で「どのテーブルのどのカラム」という指定ができていないから。

テーブル間で同じ名前のカラムがあると発生しがち

**【解決策】**

SELECT table_name.column_name と記述する

---

### #1136 Column count doesn’t match value count at row

**【原因】**

INSERTする値とカラムの数が合っていない

**【解決策】**

INSERTする値とカラムの数を合わせる