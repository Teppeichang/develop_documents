# WordPressサイトをローカル環境化＆HTTPSで表示させる手順
## 目的

---

既にWeb上に公開されているWordPressサイトの改修にあたって、同じサイトのローカル環境を構築すると同時に、ローカル環境上でHTTPS化させたい

（ざっくりいうとローカル環境で本番環境と同じ状態を再現したい　ということ）

## これからやる手順の要約

---

Webサイトに既に発行されているSSL証明書の情報を自身のPCのApacheの設定ファイルに書き込んで、Webサイトの接続先IPアドレスをローカル環境のIPアドレスに変える。

# DBを用意（使用するDBはMySQL）

---

### 1.DBを新規作成する

```bash
$ mysql -u root
mysql> CREATE DATABASE データベース名 DEFAULT CHARACTER SET utf8mb4;
mysql> CREATE USER ユーザー名 IDENTIFIED BY 'ユーザー名';
mysql> GRANT ALL PRIVILEGES ON * . * TO 'ユーザー名'@'ホスト（localhostとか％）';
```

### 2.リストアする

phpMyAdminを使ってDBファイルをダウンロードし、作成したDBに取り込む

```bash
$ mysql -u root データベース名 < ~/Downloads/mysql.データベース名.dump
```

### 3. 1.でlocalhostにパスワードが設定されなかった場合(MySQL ver5.6.51で確認)

```bash
$ mysql -u root
set PASSWORD FOR 'ユーザー名'@'ホスト（localhostとか％）' = PASSWORD('パスワード');
```

# Apacheの設定

---

### 1. /etc/apache2/httpd-ssl.conf(もしくはhttpd.conf)を編集する

```bash
cd /etc/apache2/
vi httpd-ssl.conf(もしくは vi httpd.conf)
```

以下の行の#を取り除く

```bash
#LoadModule socache_shmcb_module libexec/apache2/mod_socache_shmcb.so
#LoadModule include_module libexec/apache2/mod_include.so
#LoadModule ssl_module libexec/apache2/mod_ssl.so
#LoadModule userdir_module libexec/apache2/mod_userdir.so
#LoadModule php7_module libexec/apache2/libphp7.so
#LoadModule rewrite_module libexec/apache2/mod_rewrite.so
#AddType text/html .shtml
#AddOutputFilter INCLUDES .shtml
#Include /private/etc/apache2/extra/httpd-userdir.conf
#Include /private/etc/apache2/extra/httpd-ssl.conf
```

以下の設定値を追記。任意の場所はこのリポジトリのダウンロード先を指定する

```bash
<VirtualHost *:443>
    ServerName サイトドメイン
    DocumentRoot "/任意の場所/WordPressサイトが保存されているディレクトリ名/"

    SSLEngine on
    SSLCipherSuite ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP:+eNULL
    SSLCertificateFile /etc/apache2/ssl/サイトドメイン.crt
    SSLCertificateKeyFile /etc/apache2/ssl/サイトドメイン.key
    SetEnv ENV_MODE develop

    <Directory "/任意の場所/WordPressサイトが保存されているディレクトリ名">
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

以下の記述を

```bash
<IfModule dir_module>
    DirectoryIndex index.html
</IfModule>
```

以下に変更する

```bash
<IfModule dir_module>
    DirectoryIndex index.php index.html
</IfModule>
```

設定を保存

```bash
:w !sudo tee %
:q!

# 本作業で扱うファイルは書き込み不可・読み込みのみのファイルとなっている。
# :wqを実行すると、権限エラーE45: 'readonly' option is set (add ! to override)となる
# 上記コマンドで、管理者権限(sudo)により書き込みを強制上書き保存(tee)する
```

### 2. /private/etc/apache2/extra/httpd-ssl.conf を編集する

```bash
cd /private/etc/apache2/extra/
vi httpd-ssl.conf
```

以下の記述を

```bash
SSLCertificateFile "/private/etc/apache2/server.crt"
SSLCertificateKeyFile "/private/etc/apache2/server.key"
```

以下に変更

```bash
SSLCertificateFile "/etc/apache2/ssl/サイトドメイン.crt"
SSLCertificateKeyFile "/etc/apache2/ssl/サイトドメイン.key"
```

設定を保存

```bash
:w !sudo tee %
:q!
```

### 3. 自己署名証明書を配置する

.keyファイルと.crtファイルをダウンロードして/etc/apache2/ssl/に配置する。

```bash
# /etc/apache2/ssl ディレクトリが存在しない場合
cd /etc/apache2
sudo mkdir ssl

# ダウンロードした.keyファイルと.crtファイルを/etc/apache2/sslに移動
sudo mv サイトドメイン.key /etc/apache2/ssl
sudo mv サイトドメイン.crt /etc/apache2/ssl
```

### 4. apacheを再起動

```bash
$ sudo apachectl restart
```

# /etc/hostsの編集

```bash
cd /etc/
vi hosts
```

以下を追記する

```bash
127.0.0.1       サイトドメイン
# 指定したサイトの接続先IPアドレスをlocalhost(127.0.0.1)に変更するもの
```

設定を保存する

```bash
:w !sudo tee %
:q!
```

# 表示

Chromeだとセキュリティの警告が表示されてページが表示できないので、Safariで表示する。（警告を無視して表示してOK）

ダウンロードしたDBファイル作成時期が運用中WordPressサイトの更新日より古い場合、その差分のWordPressメディアファイルの画像が読み込めないため表示されない。（開発には影響なし）