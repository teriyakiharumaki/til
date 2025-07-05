# バリデーションって何？

## ✅ 行ったこと

- バリデーションについて調べた

## バリデーションとは

- オブジェクトがDBに保存される前に、そのデータが正しいかどうかを検証する仕組みをバリデーションという[1]
- RailsでDBに値を保存する際、無意味なデータや想定外のデータの登録を防ぐ「バリデーション」[2]

## 基本的な書き方、種類[2]

Railsではモデルクラスに、validatesメソッドで指定します。<br>
```
validates :カラム名(シンボルで指定),検証ルール（こちらもシンボルで指定）
```

- 空データを登録できないようにする→presence
```ruby:user.rb
class User < ApplicationRecord
  validates :name, presence: true
end
```

- 文字数の制限を設ける→length
```ruby:user.rb
class User < ApplicationRecord
  validates :name, presence: true, length: { maximum: 50 }
end
```
ここでは名前の長さの上限を50文字にしました。下限や範囲の指定もできます。<br>
```ruby
# 長さの下限を2文字に設定→minimum
validates :name, length: { minimum: 2 }
# 長さの範囲を2-50文字→in .. 
validates :name, length: { in: 2..50 }
# 長さを5文字に指定→is
validates :name, length: { is: 5 }
```

- 数値のみを許可する→numericality、空を許可するallow_blank
ageカラムは数字のみ、空でも構わない例です。
```ruby:user.rb
class User < ApplicationRecord
  validates :name, presence: true, length: { maximum: 50 }
  validates :age, numericality: { only_integer: true }, allow_blank: true
end
```
numericalityは、デフォルトでは小数も許容してしまいます。ageカラムでは整数のみ許可したいので、 only_integerを。<br>
また、numericalityは空を許可しないため、空を許可する場合はallow_blank: trueを追加します。（空を許可せず、数値のみを許可する場合はnumericalityのみ、 presence: trueは無くても良い。）

- 同一データは一つのみ許可する→uniqueness
emailカラムには同じデータを登録できないようにします。<br>
```ruby:user.rb
class User < ApplicationRecord
  validates :name, presence: true, length: { maximum: 50 }
  validates :age, numericality: { only_integer: true }, allow_blank: true
  validates :email, presence: true, uniqueness: true, format: { with: /\A([^@\s]+)@((?:[-a-z0-9]+\.)+[a-z]{2,})\Z/i}
end
```
format:では、文字列が条件に適しているかの検証を行う。

- 特定の値が含まれているか確認する→inclusion
genderカラムに、1~3(1:male, 2:female, 3:other)のいずれかの数字が保存されるとします。
```ruby:user.rb
class User < ApplicationRecord
  validates :name, presence: true, length: { maximum: 50 }
  validates :age, numericality: { only_integer: true }, allow_blank: true
  validates :email, presence: true, uniqueness: true, format: { with: /\A([^@\s]+)@((?:[-a-z0-9]+\.)+[a-z]{2,})\Z/i}
  validates :gender, presence: true, inclusion: { in: 1..3 }
end
```

## 参考サイト
[1](https://qiita.com/h1kita/items/772b81a1cc066e67930e)<br>
[2](https://qiita.com/clbcl226/items/af336ff06bfba4dacfd9)<br>
