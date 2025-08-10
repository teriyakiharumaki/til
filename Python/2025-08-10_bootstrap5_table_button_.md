# bootstrap5のtable、buttonクラスについて

## ✅ 行ったこと

- post_list内の表のデザインをした

## 実装

```
<body>
  {% include "sample_app/partials/headers.html" %}
  <h1>Post List</h1>
  <table class="table"> #class="table"追加
    <thead>
      <tr>
        <th scope="col">ID</th> #scope="col"追加
        <th scope="col">User Name</th> #scope="col"追加
        <th scope="col">Post</th> #scope="col"追加
      </tr>
    </thead>
    <tbody>
      {% for post in posts %}
      <tr>
        <th scope="row">{{ post.id }}</th> #scope="row"追加
        <td>{{ post.name }}</td>
        <td>{{ post.micropost }}</td>
        <td>
          <a class="btn btn-outline-success" href="{% url 'sample_app:edit_post' post_id=post.id %}" role="button">修正</a> #class="btn btn-outline-success", role="button"追加
          <a class="btn btn-outline-danger" href="{% url 'sample_app:delete_post' post_id=post.id %}" role="button">削除</a> #class="btn btn-outline-danger", role="button"追加
        </td>
      </tr>
      {% endfor %}
    </tbody>
  </table>
  <a class="btn btn-outline-primary" href="{% url 'sample_app:create_post' %}" role="button">作成</a> #追加
  <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js" integrity="sha384-geWF76RCwLtnZ8qwWowPQNguL3RmwHVBC9FhGdlKrxdiJJigb/j/68SIy3Te4Bkz" crossorigin="anonymous"></script>
</body>

```

## table[1]
表の実装で`<table>`に`class="table"`を追加する<br>
→Bootstrapの用意している「テーブル用の装飾スタイル」を適用するという意味<br>

`scope="col"`と`scope="row"`<br>
→行（行末）方向に対する見出しの`<th>`タグには scope="row" を入れ、列（下）方向に対する見出しの`<th>`タグには scope="col" を入れる[2]

## button[3]
ボタンは、基本的に以下のように実装
```
<button type="button" class="btn">Base class</button>
```

`<a>`タグでは、
```
class="btn btn-outline-success", role="button"
```
のように、`role="button`を与えて、デザインをする<br>
`<input>`タグも同様に実装

## 参考サイト
[1](https://getbootstrap.jp/docs/5.3/content/tables/#%E6%A6%82%E8%A6%81)<br>
[2](https://bootstrap-guide.com/content/tables)<br>
[3](https://getbootstrap.jp/docs/5.3/components/buttons/)