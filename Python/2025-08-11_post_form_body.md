# post_formの処理について

## ✅ 行ったこと

- post_formのbody内でどういう処理が行われているのかについて調べた

## 実際のコード
```
<body>
  {% include "sample_app/partials/headers.html" %}
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
```

### ✅ `{% include "sample_app/partials/headers.html" %}`
- HTMLの共通部分（ナビバーやヘッダー）を別ファイルに切り出して読み込む処理。
- sample_app/templates/sample_app/partials/headers.html にヘッダーがあると想定。

前回やったやつ！

### ✅ `{% if post_id %}` 〜 `{% endif %}`
Djangoのテンプレートタグで、条件分岐をしている
- post_id が 存在する（= 編集）場合
  ```
  <form action="{% url 'sample_app:edit_post' post_id=post_id %}" method="post">
  ```
- post_id が 存在しない（= 新規作成）場合
  ```
  <form action="{% url 'sample_app:create_post' %}" method="post">
  ```

🔸 post_id は Django のビューからテンプレートに渡されてる値で、<br>
その有無によって「編集画面か新規作成画面か」を切り替えてるという仕組み

### ✅ `{% csrf_token %}`
Djangoでフォームを使うときにはCSRF対策が必要<br>
これは「このフォームは信頼できるサイトから送られたものですよ」という証明トークンを自動で挿入する処理

### ✅ `{{ form }}`
Djangoのフォームクラスから生成されたフォーム全体を自動レンダリングしている。<br>
フィールドやラベルなどが自動で出力されるが、Bootstrap風に整えたい場合は個別に指定して書くこともできる（例：`{{ form.name }}`）

## 参考サイト
https://qiita.com/SDTakeuchi/items/6168ea148b27ee6daac7<br>
↑CSRFについて詳しいサイト
