# scaffoldを使ってCRUD機能を実装

## ✅ 行ったこと

- scaffoldを利用してCRUD機能の作成を行なった

## コンテナ内で以下を実行
```
rails g scaffold Question content:text answer:text category:string is_starred:boolean
rails db:migrate
```

## 各コードの解説
```
rails g scaffold Question content:text answer:text category:string is_starred:boolean
```
これは、Questionモデルを作り、<br>
content / answer / category / is_starred の4つのカラムを持つテーブルを作る<br>

| カラム名 | 型 | 説明 |
| :-- | :-: | --: |
| content | text | 質問文 |
| answer | text | 回答 |
| category | string | カテゴリ |
| is_starred | boolean | お気に入りマーク |

今回のモデル
```
app/models/question.rb
```

### さらに、以下のものが生成される

- マイグレーションファイル
```
db/migrate/2025xxxx_create_questions.rb
```
テーブルの定義をするファイル<br>
scaffold の文字列から content, answer, category, is_starred がここに入る

- コントローラ
```
app/controllers/questions_controller.rb
```
CRUD の処理（一覧・作成・更新・削除）をまとめて自動生成する<br>
中に入るメソッド(index, show, new, create, edit, update, destroy)

- ビュー
```
app/views/questions/index.html.erb
app/views/questions/show.html.erb
app/views/questions/new.html.erb
app/views/questions/edit.html.erb
app/views/questions/_form.html.erb
app/views/questions/_questions.html.erb
```
質問一覧・詳細・新規作成・編集などの画面<br>
HTMLが全部自動生成される

- ルーティング
config/routes.rb に自動で追加
```
resources :questions
```
ルーティング一覧
```
GET /questions
GET /questions/new
POST /questions
GET /questions/:id
GET /questions/:id/edit
PATCH /questions/:id
DELETE /questions/:id
```

- ヘルパー
```
app/helpers/questions_helper.rb
```
ビュー用のヘルパーメソッドを記述する場所<br>
始めは何も記述されていない

- jbuilder（JSONテンプレート）
```
app/views/questions/index.json.jbuilder
app/views/questions/show.json.jbuilder
```
API で JSON を返す場合に使うテンプレート<br>
API不要なら使用しない

- テストファイル
```
test/controllers/questions_controller_test.rb
test/models/question_test.rb
test/system/questions_test.rb
```
Rails 7 の場合は Minitest が自動で生成される

## 参考になるサイト
https://zenn.dev/goldsaya/articles/02360f3fcb89e2