# RailsアプリをGitHubに接続するまで

## ✅ 行ったこと

- Railsで作成した新規アプリに、githubを接続した

## ①GitHubで新しいリポジトリを作成

1. GitHubにログイン<br>
2. 右上の「+」 → 「New repository」<br>
3. 名前を入力（例：my-python-sampleなど）<br>
4. 「Public」にすると公開される<br>
5. README や .gitignore はあとでローカルに作成できる<br>
6. Create repository<br>

## ②ローカルでの操作

1. `git init`でGitの初期化

  ❗️エラー❗️
  ```
  git init
  Reinitialized existing Git repository in /Users/tokunaganaoya/tubuyaki/.git/
  ```

  💡メッセージの意味<br>
  これはすでに .git フォルダ（＝Gitで管理するための情報）がそのディレクトリに存在していたということを意味している<br>
  つまり：
  - 以前に git init したことがある
  - あるいは GitHubから clone してきたプロジェクトだった

  その上でもう一度 git init したから、「再初期化されたよ」と出た<br>

<br>

2. リモートURLを確認（または追加）

  ```
  git remote -v
  ```
  何も表示されなかったので、以下のコマンドで接続
  ```
  git remote add origin git@~~~
  ```
  もう一度確認
  ```
  git remote -v
  origin	git@github.com:teriyakiharumaki/tubukyaki.git (fetch)
  origin	git@github.com:teriyakiharumaki/tubukyaki.git (push)
  ```
  ❓githubのリポジトリ名が「tubukyaki」になってる<br>
  急いでリポジトリを作り直し、ローカルでは以下のコマンドでリモートリポジトリのURLを上書き
  ```
  git remote set-url origin git@~~~
  ```
  もう一度確認
  ```
  git remote -v
  origin	git@github.com:teriyakiharumaki/tubuyaki.git (fetch)
  origin	git@github.com:teriyakiharumaki/tubuyaki.git (push)
  ```
  これでOK<br>

<br>

3. デフォルトで最初のブランチ名が masterなので、mainに変更

  git init すると、デフォルトで最初のブランチ名が masterなので、mainに変更（最近の主流、mainじゃないと警告が出る）
  ```
  git branch -m main
  ```
<br>

4. gitにpush！

このコマンドでpush
```
git push -u origin main
```
❗️エラー❗️
```
error: src refspec main does not match any
error: failed to push some refs to 'github.com:teriyakiharumaki/tubuyaki.git
```
この意味は、「main という名前のブランチにまだ何もコミットしていないから、pushする内容がない」<br>
つまり、コミットを作っていつも通りpushすれば良い
```
git add .
git commit -m "最初のコミット"
git push -u origin main
```
これでOK！