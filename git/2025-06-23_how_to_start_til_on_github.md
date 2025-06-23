# GitHub上でTILを始める方法

## ✅ 行ったこと

- GitHubでTIL用の新しいリポジトリを作成
- ローカルにcloneして、エディタで編集できる状態にした

## 🛠 手順

1. GitHubで新しいリポジトリ（例：`til`）を作成（READMEは作らない）<br>
右上の＋ボタンを押して新規作成
[![Image from Gyazo](https://i.gyazo.com/767932eda9ecdc37035f3009c0ab7aa3.png)](https://gyazo.com/767932eda9ecdc37035f3009c0ab7aa3)

2. ローカルにclone  
   ```bash
   cd ~/your/workspace  # 保存したい場所へ移動
   git clone https://github.com/あなたのユーザー名/til.git
   cd til
   ```
  
  cloneするURLは1の真ん中にあるSSHから

3. フォルダ&ファイルの作成
   ```bash
   mkdir git
   ```

  あとはVScodeで開いてファイルを作って編集

4. Gitでcommit & push
   ```bash
   git add .
   git commit -m "TIL: Github上でTILを始める方法（リポジトリ作成＆clone）を学んだ"
   git push origin main
   ```

## ✅ 次回からのTIL更新方法
```bash
cd ~/your/workspace/til

# ファイルを追加・更新したら
git add .
git commit -m "TIL: 今日の学び XYZ を記録"
git push origin main
```

## 🎁 フォルダ構成テンプレート
```
til/
├── rails/
├── ruby/
├── git/
├── javascript/
├── docker/
├── notion/
└── README.md
```
<br>

# おまけ

## TILファイルの作り方
・`.md`ファイルが良い<br>
・おすすめのファイル名「日付_内容の概要.md」<br>

## VScodeでmdファイルのプレビュー
・ファイル名を右クリックして、「プレビューを開く」<br>

## マークダウンの書き方参考サイト
https://qiita.com/Qiita/items/c686397e4a0f4f11683d<br>
https://help.docbase.io/posts/13697#%E3%82%B3%E3%83%BC%E3%83%89<br>
今回学んだのは、コードブロックはバッククオート（`）三つで囲む

## ターミナルのコードをbashと表現する理由
https://qiita.com/kohki_takatama/items/14517d8c38054c1e8033