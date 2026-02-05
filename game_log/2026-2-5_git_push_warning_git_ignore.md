# git push時の警告

## ✅ 行ったこと

- git pushした際に出た警告について

## git pushした際の警告
```
remote: warning: File docker/db/data/ibdata1 is 76.00 MB;
this is larger than GitHub's recommended maximum file size of 50.00 MB
```
docker/db/data/ibdata1は、MySQLの実データファイルで本来ならpushするべきではない

## ①すでに追跡されているファイルを外す
```
git rm -r --cached docker/db/data
```
git rm catched ~："既にGitの管理下にあるファイルをGitの管理対象から外すコマンドです。つまり、過去にコミットされたファイルをリポジトリから削除し、以降のコミットでは追跡されないようにするために使用されます。"<br>
(https://qiita.com/eric50905/items/25d8571078f25b184328#:~:text=git%20rm%20%2D%2Dcached%E3%82%B3%E3%83%9E%E3%83%B3%E3%83%89%E3%81%AE%E5%A0%B4%E5%90%88,%E3%81%9F%E3%82%81%E3%81%AB%E4%BD%BF%E7%94%A8%E3%81%95%E3%82%8C%E3%81%BE%E3%81%99%E3%80%82)

git -r："-r オプションは「Recursive」の頭文字を取ったものです。 Recursive モードで git rm を実行すると、ターゲット ディレクトリとそのディレクトリのすべてのコンテンツが削除されます。"<br>
(https://www.atlassian.com/ja/git/tutorials/undoing-changes/git-rm)

```
git commit -m "Remove MySQL data directory from git tracking"
git push
```
コミット、プッシュして反映

## ②`.gitignore`の修正
GAME_LOG/game_log/.gitignore
```
~
docker/db/data/         #追記
```
追記をして、追跡しないように設定

**`.gitignore が効く範囲は「その .gitignore が置いてあるディレクトリ以下」だけ`**<br>
(https://zenn.dev/purigen/articles/67bb8de047e1bb)

なので、

**`.gitignoreファイルの場所を移動する`**

```
GAME_LOG/
├─ docker/
│  └─ db/
│     └─ data/   ← ← ← ここを無視したい
└─ game_log/
   ├─ .gitignore ← ← ← ここにある
   └─ resources/
```

移動先は、

```
GAME_LOG/
├─ .gitignore      ← ← ← ここ！！！
├─ docker/
│  └─ db/
│     └─ data/
└─ game_log/
```

## ③`.gitignore`の移動
```
mv game_log/.gitignore .gitignore
git commit -m "gitignoreの移動"
git push
```
