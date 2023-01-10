# テーブル名にハイフンが入っているテーブルに対してSQLを実行する時の対応
## 結論

クオートをつけてSQLを実行する。

そしてそもそも論だが、**テーブル名にハイフンはつけないこと。**

```sql
SELECT * FROM 'hoge-test' WHERE name='hogefuga';
```

```sql
SELECT * FROM hoge-test WHERE name='hogefuga';
```

これだとSyntax Error