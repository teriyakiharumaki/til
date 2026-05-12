# HTMLテンプレート(hello.html)の作成

## ✅ 行ったこと

- HTMLテンプレート(hello.html)のコードについて解説

## 参考にしたサイト
https://route-zero.com/recruit/route/922/

## hello.html
src/main/resources/templates/hello.html
```
<!DOCTYPE html>
  <html lang="ja">
    <head>
      <meta charset="UTF-8" />
      <meta name="viewport" content="width=device-width, initial-scale=1.0" />
      <title>Hello Page</title>
    </head>
    <body>
      <h1>フォーム入力</h1>
      <form action="/send" method="post">
        <label for="name">名前:</label>
        <input type="text" id="name" name="name" th:value="${form.name}" />
        <button type="submit">送信</button>
      </form>
      <h2>メッセージ:</h2>
      <p th:text="${message}"></p>
    </body>
  </html>
```

>このHTMLテンプレートは、ユーザーがフォームに入力した名前をサーバーに送信し、受け取ったメッセージを表示します。<br>
>Thymeleafの th:value や th:textを使って、動的な内容をページに反映しています。<br>
>(https://route-zero.com/recruit/route/922/)

ここは、ユーザーに表示する画面の部分

## 各コードについて

### ① HTML宣言
```
<!DOCTYPE html>
<html lang="ja">
```
いつものHTML

### ② head
```
<meta charset="UTF-8" />
```
日本語を扱えるようにする

```
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
```
スマホ対応デザイン

### ③ formタグ
```
<form action="/send" method="post">
```
post通信で「どこに送るか」を決めてる

```
action="/send"
```
/send に送信する

#### 対応するController
```
@PostMapping("/send")
```
ここに飛ぶ

```
method="post"
```
POST通信を使う

### ④入力欄
```
<input type="text" id="name" name="name"
```
name="name" が重要

この name が
```
private String name;
```
と対応してる。

つまり
```
HTMLのname
↓
HelloForm.name
```
に自動で入る

### ⑤Thymeleaf部分
```
th:value="${form.name}"
```

**form.name の値を表示する**

>プロフィール編集の初期表示、確認画面、エラーで戻ったときの再表示など、「いま持っている値を見せたい」場面で頻繁に使います。
>参考サイト（https://star-school.jp/thymeleaf/167）

### ⑥ボタン
```
<button type="submit">送信</button>
```
フォーム送信

### ⑦メッセージ表示
```
<p th:text="${message}"></p>
```

Controllerから渡されたmessageを表示

>HTMLタグの中に「サーバーから受け取った値」を表示するための属性です。
>例えば、`<span>`や`<div>`に設定することで、画面に出したい文字を動的に差し替えることができます。
>参考サイト（https://star-school.jp/thymeleaf/165）

#### Controller側
```
model.addAttribute("message", "Hello, " + form.getName() + "!");
```

#### HTML側
```
th:text="${message}"
```
実際の表示は、
```
Hello, teriyaki!
```
