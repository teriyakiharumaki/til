# headerタグがbodyタグにあってもいいのか

## ✅ 行ったこと

- ヘッダーを実装した際に、bodyタグ内に実装したが、bodyタグ外の上がいいのかと思ったので調べた

## 結論：`<body>`内に`<header>`を置くのは「正しいHTML構造」

📘 理由<br>
🔹 HTML5の仕様に沿っている<br>
`<header>` は `<body>` の子要素として定義されており、文書全体の見出しやナビゲーション、ロゴなどを含めるためのセクショニング要素
```
<!DOCTYPE html>
<html>
<head> ... </head>
<body>
  <header> ← ✅ 正しい位置
    <nav> ... </nav>
  </header>

  <main> ... </main>

  <footer> ... </footer>
</body>
</html>
```

## 参考サイト
https://web-de-asobo.net/2021/09/10/header-footer-main-tag/