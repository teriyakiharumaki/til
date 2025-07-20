# Djangoのインストール

## ✅ 行ったこと

- Djangoのインストール
- プロジェクトの作成、設定
- データベースのマイグレート
- super userの作成
- 開発用サーバーの起動

## インストール方法[1]
djangoをインストールします。<br>
Linux系OSやMacならターミナルから、Windowsならコマンドプロンプトから以下のコマンドを実行します。
```
pip install django
```

## 実際に実行してみた
```
MacBook-Pro ~ % pip install django  
Collecting django
  Downloading django-5.2.4-py3-none-any.whl (8.3 MB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 8.3/8.3 MB 10.0 MB/s eta 0:00:00
Collecting sqlparse>=0.3.1
  Downloading sqlparse-0.5.3-py3-none-any.whl (44 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 44.4/44.4 KB 3.6 MB/s eta 0:00:00
Collecting asgiref>=3.8.1
  Downloading asgiref-3.9.1-py3-none-any.whl (23 kB)
Collecting typing_extensions>=4
  Downloading typing_extensions-4.14.1-py3-none-any.whl (43 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 43.9/43.9 KB 4.7 MB/s eta 0:00:00
Installing collected packages: typing_extensions, sqlparse, asgiref, django
Successfully installed asgiref-3.9.1 django-5.2.4 sqlparse-0.5.3 typing_extensions-4.14.1
WARNING: You are using pip version 22.0.4; however, version 25.1.1 is available.
You should consider upgrading via the '/Users/tokunaganaoya/.pyenv/versions/3.10.4/bin/python3.10 -m pip install --upgrade pip' command.
```
*警告は、は単なる「pipが古いよ」というお知らせだけで、動作には全く支障ありません。（GPT）

## プロジェクトの作成[1]
作成コマンド
```
$ django-admin startproject my_project
```
コマンドの意味(GPT)

- django-admin：Django に最初から付属している 管理用コマンドラインツール
- startproject：新しいプロジェクトを作るためのサブコマンド
- my_project：作りたいプロジェクトの名前（任意）

作成されるファイル群
```
my_project/
    manage.py
    my_project/
        __init__.py
        asgi.py
        settings.py
        urls.py
        wsgi.py
```
一層目のmy_projectディレクトリで各種設定を行う
```
$ cd my_project
```

## プロジェクト設定
Djangoの各種設定は my_project/my_project/settings.py で定義されています。<br>

データベースの設定などは特にこだわりがなければ変えなくてOKです。<br>
標準で良い感じの設定になっています。<br>

言語とタイムゾーンだけ日本にしておきましょう。

```python:my_project/my_project/settings.py
# LANGUAGE_CODE = 'en-us'
LANGUAGE_CODE = 'ja'

# TIME_ZONE = 'UTC'
TIME_ZONE = 'Asia/Tokyo'
```

## データベースのマイグレート
```
python manage.py migrate
```
このコマンドでマイグレーションする<br>
コマンドを実行することで、 my_project/db.sqlite3 というファイルが作成される
```
MacBook-Pro my_project % python manage.py migrate
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, sessions
Running migrations:
  Applying contenttypes.0001_initial... OK
  Applying auth.0001_initial... OK
  Applying admin.0001_initial... OK
  Applying admin.0002_logentry_remove_auto_add... OK
  Applying admin.0003_logentry_add_action_flag_choices... OK
  Applying contenttypes.0002_remove_content_type_name... OK
  Applying auth.0002_alter_permission_name_max_length... OK
  Applying auth.0003_alter_user_email_max_length... OK
  Applying auth.0004_alter_user_username_opts... OK
  Applying auth.0005_alter_user_last_login_null... OK
  Applying auth.0006_require_contenttypes_0002... OK
  Applying auth.0007_alter_validators_add_error_messages... OK
  Applying auth.0008_alter_user_username_max_length... OK
  Applying auth.0009_alter_user_last_name_max_length... OK
  Applying auth.0010_alter_group_name_max_length... OK
  Applying auth.0011_update_proxy_permissions... OK
  Applying auth.0012_alter_user_first_name_max_length... OK
  Applying sessions.0001_initial... OK
```

## super userの作成
super user（作成するアプリの管理者）も作成
```
$ python manage.py createsuperuser
Username (leave blank to use 'hoge'): admin
Email address: admin@example.com
Password: a~
Password (again): a~
Superuser created successfully.
```

## 開発用サーバーの起動
次のサーバーで起動
```
python manage.py runserver
```
ブラウザで http://127.0.0.1:8000/ にアクセスして確認<br>

control + c で開発者サーバーを終了

## 参考サイト
[1](https://qiita.com/pythonista/items/19613663ef7bb3c57d4f)