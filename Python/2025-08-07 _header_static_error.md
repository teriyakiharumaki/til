# ヘッダー実装時のstaticのエラー

## ✅ 行ったこと

- post_formでヘッダーを実装した際のhead修正時のエラーを解決

## エラー発生時の状況
Templates/post_form.htmlにヘッダーを入れる際に、cssを読み込むためのheadの修正を行った<br>
```
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Post Form</title> #追加
  <link href="https://fonts.googleapis.com/css2?family=Lobster&display=swap" rel="stylesheet"> #追加
  <meta name="viewport" content="width=device-width, initial-scale=1"> #追加
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-9ndCyUaIbzAi2FUVXJi0CjmCapSmO7SnpJef0486qhLnuZ2cdeRhO02iuK6FUUVM" crossorigin="anonymous"> #追加
  <link href="{% static 'css/style.css' %}" rel="stylesheet"> #追加
</head>
<body>
  {% include "sample_app/partials/headers.html" %} #追加
    <h4>Post の編集</h4>
    {% if post_id %}
    <form action="{% url 'sample_app:edit_post' post_id=post_id %}" method="post">
    {% else %}
    <form action="{% url 'sample_app:create_post' %}" method="post">
    {% endif %}
      {% csrf_token %}
      {{ form }}
          <button type="submit">送信</button>
    </form>
    <a href="{% url 'sample_app:read_post' %}">戻る</a>
</body>
</html>

```
Headにそのまま貼り付けて実装した

## エラーの内容
TemplateSyntaxError at /sample_app/post/edit/2/ Invalid block tag on line 9: 'static'. Did you forget to register or load this tag? Request Method: GET Request URL: http://127.0.0.1:8000/sample_app/post/edit/2/ Django Version: 5.2.4 Exception Type: TemplateSyntaxError Exception Value: Invalid block tag on line 9: 'static'. Did you forget to register or load this tag? Exception Location: /Users/tokunaganaoya/.pyenv/versions/3.10.4/lib/python3.10/site-packages/django/template/base.py, line 577, in invalid_block_tag Raised during: sample_app.views.edit_post Python Executable: /Users/tokunaganaoya/.pyenv/versions/3.10.4/bin/python Python Version: 3.10.4 Python Path: ['/Users/tokunaganaoya/my_project', '/Users/tokunaganaoya/.pyenv/versions/3.10.4/lib/python310.zip', '/Users/tokunaganaoya/.pyenv/versions/3.10.4/lib/python3.10', '/Users/tokunaganaoya/.pyenv/versions/3.10.4/lib/python3.10/lib-dynload', '/Users/tokunaganaoya/.pyenv/versions/3.10.4/lib/python3.10/site-packages'] Server time: Mon, 04 Aug 2025 20:02:08 +0900

## 原因
エラーの原因は、テンプレート内で {% static '...' %} を使っているのに、テンプレートの先頭で {% load static %} を書き忘れていることが原因<br>
重要な部分
```
Invalid block tag on line 9: 'static'. Did you forget to register or load this tag?
```
→ Djangoに「{% static %} って何？」と怒られている状態。{% load static %} がないから認識されていない。<br>

## 解決方法
1番上に{% load static %}を追加することで解決