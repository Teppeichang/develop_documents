# Google Cloud SQLのDB文字コードをutf8mb4に設定したが、絵文字を保存できない

## 実行環境

Google Cloud SQL(MySQL ver5.7)

## 結論

Google Cloud SQLのコンソールですべき「データベースフラグ」設定が漏れていた。

## 経緯

実務において運用している広告レポートサービスで、以下のSQLエラーが発生。

```sql
1366 Incorrect string value: '\xF0\x9F\x92\xA2" ...' for column
```

調べてみると、「絵文字は保存できません」という意味のエラーとのこと。

## 対応

DBの文字コードを utf8mb4 に変更すれば絵文字のレコードも保存できるようになるのだが・・・

DB作成時に文字コードは utf8mb4に設定している。

「utf8mb4 絵文字 保存できない」という感じで調べてみると・・・

どうやら文字コードはDB本体だけでなくサーバーとクライアント側の通信に対しても設定されているようなので、DBのサーバーとクライアント側の文字コードを確認してみる。

```sql
mysql> show variables like '%char%';

+--------------------------+------------------------------
| Variable_name            | Value                       |
------------------------+--------------------------------+
| character_set_client     | utf8                        | クライアント
| character_set_connection | utf8                        | クライアント
| character_set_database   | utf8mb4                     | サーバ
| character_set_filesystem | binary                      |
| character_set_results    | utf8                        | サーバ
| character_set_server     | utf8                        | サーバ
| character_set_system     | utf8                        |
| character_sets_dir       | /usr/local/mysql/charsets/  |
+--------------------------+-----------------------------+
```

それぞれの項目の意味は下記の通り。

![MySQL_Variables](img/mysql_variables.png)

DBサーバーの文字コードだけがutf8mb4になっている状態なので、filesystemとdir以外の項目をすべてutf8mb4に修正する。

```sql
SET character_set_client=utf8mb4;
SET character_set_results=utf8mb4;
SET character_set_connection=utf8mb4;
SET character_set_server=utf8mb4;
```

修正後の文字コード↓

```sql
mysql> show variables like '%char%';

+--------------------------+------------------------------
| Variable_name            | Value                       |
------------------------+--------------------------------+
| character_set_client     | utf8mb4                     | クライアント
| character_set_connection | utf8mb4                     | クライアント
| character_set_database   | utf8mb4                     | サーバ
| character_set_filesystem | binary                      |
| character_set_results    | utf8mb4                     | サーバ
| character_set_server     | utf8mb4                     | サーバ
| character_set_system     | utf8mb4                     |
| character_sets_dir       | /usr/local/mysql/charsets/  |
+--------------------------+-----------------------------+
```

再度絵文字の保存を試みてみるが、冒頭と同じエラーとなる。

Google Cloud SQL側の設定なのでは？と思い「Google Cloud SQL utf8mb4 emoji」で調べてみると・・・

Google Cloud SQLのコンソールに「データベースフラグ」の設定があり、そこで「character_set_server」をutf8mb4に設定しなければならないことが判明。

公式ドキュメントの手順に沿ってデータベースフラグの設定を行なって、絵文字が保存できるようになった。

## 参照記事

データベースフラグの設定(GCP公式)

[https://cloud.google.com/sql/docs/mysql/flags](https://cloud.google.com/sql/docs/mysql/flags)

同じ問題に直面した方の対応(GCPコミュニティ)

[https://medium.com/google-cloud/enable-full-unicode-in-mysql-on-google-cloud-aaa2635486d6](https://medium.com/google-cloud/enable-full-unicode-in-mysql-on-google-cloud-aaa2635486d6)

データベースの文字コード確認方法

[https://qiita.com/yukiyoshimura/items/d44a98021608c8f8a52a](https://qiita.com/yukiyoshimura/items/d44a98021608c8f8a52a)

データベースの文字コード変更方法

[https://qiita.com/katsuyan/items/0c7ed34d9f8235b8363a](https://qiita.com/katsuyan/items/0c7ed34d9f8235b8363a)