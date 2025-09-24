# counter_cacheについて

## ✅ 行ったこと

- いいねボタンのモデル実装時のcounter_cacheについて

## counter_cacheとは
集計用の数値を自動で管理する仕組み

## つぶやきアプリでどういう仕組みか解説

### ①そもそも何を解決するのか
たとえば「ある投稿に何件いいねが付いているか」を数えるとき
```
post.likes.count
```
でもこれだと毎回 SELECT COUNT(*) FROM likes WHERE post_id = ? が実行される。<br>
大量の投稿に対して一覧で出すと、N+1問題になってパフォーマンスが落ちる。

### ②counter_cache の仕組み
belongs_to :post, counter_cache: true を Like に書くと…

- likes テーブルに post_id がある
- posts テーブルに likes_count というカラムを作っておく

この条件がそろったときに Rails が「Like が追加/削除されたら自動的に posts.likes_count を +1 / -1 更新」してくれる。<br>
実際の動き

- Like を作成すると
```
INSERT INTO likes ...
UPDATE posts SET likes_count = likes_count + 1 WHERE id = ?
```

- Like を削除すると
```
DELETE FROM likes ...
UPDATE posts SET likes_count = likes_count - 1 WHERE id = ?
```

### ③使い方

- モデル側
```
# app/models/like.rb
class Like < ApplicationRecord
  belongs_to :post, counter_cache: true
end
```

- DB側<br>
posts テーブルに likes_count カラムを追加して、デフォルトを 0 にしておく：
```
add_column :posts, :likes_count, :integer, default: 0, null: false
```

- ビュー側
```
<%= post.likes_count %>
```
と書けばSQLなしで即座に「いいね数」を表示できる。

### ④注意点

- 既存のデータ
途中からカラムを追加した場合、既存レコードの likes_count はNULLや0のままだから、一度リセットしてあげる必要がある：
```
Post.find_each { |post| Post.reset_counters(post.id, :likes) }
```

- 複数のcounter_cache
もし他の関係（例: コメント数）でも同じことをしたければ、同じように comments_count をカラムに追加すればOK。

- キャッシュとリアルタイムの違い
likes_count は キャッシュだから、実際の件数とズレる可能性はある（手動で直接DBをいじった場合など）。でも通常のアプリの使い方では Rails が責任を持って更新してくれる。