# 基本コードの記述

## ✅ 行ったこと

- フォームを処理するコントローラクラスを作成した

## 参考にしたサイト
https://route-zero.com/recruit/route/922/

## 各コードの記述

### HelloController.java

```
package com.example.demo.controller;
  
  import org.springframework.stereotype.Controller;
  import org.springframework.ui.Model;
  import org.springframework.web.bind.annotation.GetMapping;
  import org.springframework.web.bind.annotation.ModelAttribute;
  import org.springframework.web.bind.annotation.PostMapping;
  
  import com.example.demo.form.HelloForm;
  
  // コントローラークラス
  @Controller
  public class HelloController {
  
      // "/"へのGETリクエストを処理するメソッド
      @GetMapping("/")
      public String helloForm(Model model) {
          // 初期値として空のHelloFormオブジェクトをビューに渡す
          model.addAttribute("form", new HelloForm());
          // "hello.html"テンプレートをレンダリング
          return "hello";
      }
  
      // "/send"へのPOSTリクエストを処理するメソッド
      @PostMapping("/send")
      public String submitForm(@ModelAttribute HelloForm form, Model model) {
          // 受け取ったフォームデータをビューに返すためにModelに設定
          model.addAttribute("form", form); // フォームオブジェクトを再度ビューに渡す
          // 入力された名前を利用してメッセージを生成し、Modelに設定
          model.addAttribute("message", "Hello, " + form.getName() + "!");
          // "hello.html"テンプレートを再度レンダリング
          return "hello";
      }
  }
```

ここのクラスは、**URLにアクセスされたときの処理を書く場所**

### ① パッケージ
```
package com.example.demo.controller;
```

「controllerフォルダにある」という宣言

### ② import（使う機能の読み込み）
```
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.*;
```
必要な機能を読み込む

### ③ Controller宣言
```
@Controller
public class HelloController {
```

「これはコントローラーです」とSpringに教えてる

これがないとどうなる？ → URLアクセスしても反応しない

### ④ GET処理（画面表示）
```
@GetMapping("/")
public String helloForm(Model model) {
```
意味 → 「/ にアクセスされたらこのメソッドを実行」

Laravelでいうと
```
Route::get('/', function () {
    return view('hello');
});
```

Modelって何？
```
model.addAttribute("form", new HelloForm());
```
Viewにデータを渡す箱

Laravelでいうと
```
return view('hello', ['form' => new HelloForm()]);
```
になる

```
return "hello";
```

templates/hello.html を表示する

### ⑤ POST処理（フォーム送信）
```
@PostMapping("/send")
public String submitForm(@ModelAttribute HelloForm form, Model model)
```

/send にPOSTされたらここを実行

Laravelでいうと
```
Route::post('/send', [Controller::class, 'store']);
```

### ⑥ フォームの受け取り（ここ重要）
```
@ModelAttribute HelloForm form
```
フォームの値を自動で詰める

### ⑦ データをViewに渡す
```
model.addAttribute("form", form);
```
入力された値をそのまま画面に戻す

### ⑧ メッセージ作成
```
model.addAttribute("message", "Hello, " + form.getName() + "!");
```
入力値を使って文字列を作る

### ⑨ 再表示
```
return "hello";
```
同じ画面をもう一度表示

### 全体の流れ
```
① GET /
↓
画面表示（空フォーム）
↓
② ユーザー入力
↓
③ POST /send
↓
④ Controllerが受け取る
↓
⑤ メッセージ作る
↓
⑥ 画面再表示
```