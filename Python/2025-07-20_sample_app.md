# サンプルアプリ作成①

## ✅ 行ったこと

- アプリケーションの作成

## 作成
アプリケーションの作成
```
python manage.py startapp sample_app
``` 
コマンドを実行すると以下のような階層構造になる
```
my_project/
    manage.py
    my_project/
        __init__.py
        asgi.py
        settings.py
        urls.py
        wsgi.py
    sample_app/
        __init__.py
        admin.py
        apps.py
        migrations
        models.py
        tests.py
        views.py
```
Djangoではプロジェクトディレクトリ my_project/ の下に、<br>
プロジェクト設定 my_project/my_project/ とアプリケーションディレクトリ my_project/sample_app/ が紐づくという形になる

## アプリケーションのプロジェクトへの登録
アプリケーションを作成したら、プロジェクトに登録しましょう。<br>
これをしないとプロジェクトからアプリケーションを認識出来ません。<br>
my_project/sample_app/apps.py を開いてみると、SampleAppConfig というクラスが定義されています。
```python:my_project/sample_app/apps.py
from django.apps import AppConfig


class SampleAppConfig(AppConfig):
    default_auto_field = 'django.db.models.BigAutoField'
    name = 'sample_app'

```
これを my_project/my_project/settings.py の INSTALLED_APPS に *'sample_app.apps.SampleAppConfig',*という文字列で追記します。

```python:my_project/my_project/settings.py
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'sample_app.apps.SampleAppConfig', # Add
]
```
アプリケーション作成完了！

## 参考サイト
[1]()<br>