# AIコメント生成コードの修正についての続き

## ✅ 行ったこと

- AIによるコメント生成機能のコード修正の解説

## 修正したコード
ai_comment_generator.rb
```
require "json"

class AiCommentGenerator
  MODEL = "gpt-4.1".freeze

  KEYS = %w[
    task_created_comment
    task_updated_comment
    task_deleted_comment
    task_completed_comment
    sub_task_completed_comment
    task_incomplete_comment
  ].freeze

  def initialize(api_key = ENV["OPENAI_API_KEY"] || Rails.application.credentials.dig(:openai, :api_key))
    @ai_client = OpenAI::Client.new(access_token: api_key)
  end

  def generate_and_save_all(oshi_profile)
    ai_comment = oshi_profile.build_ai_comment

    attrs = generate_comments_hash(oshi_profile)
    ai_comment.assign_attributes(attrs)
    ai_comment.save!
  end

  private

  def generate_comments_hash(oshi_profile)
    profile_info = <<~TEXT
      名前: #{oshi_profile.name}
      性格: #{oshi_profile.personality}
      年齢: #{oshi_profile.generation}
      性別: #{oshi_profile.gender}
      一人称: #{oshi_profile.pronoun}
    TEXT

    prompt = <<~PROMPT
      あなたは次のプロフィールの「推し本人」として話します。
      読者は「ユーザー本人」です。必ずユーザーに向けて話しかけてください。
    
      出力は **必ずJSONのみ**（前後に文章を付けない）で返してください。
      キーは次の6つを必ず含めてください:
      #{KEYS.join(", ")}
    
      各値（セリフ）のルール:
      - 50文字以内、1文（長くても2文まで）
      - 一人称はプロフィールの一人称を必ず使う
      - 文末は自然な口調（性格に合わせる）
      - ユーザーに向けた呼びかけ・共感・励ましを必ず含める
      - 状況説明は禁止（例:「タスクを作成して〜」みたいな説明文NG）
      - 「頑張る」という単語はできるだけ使わず、別表現に言い換える
      - セリフ内で「ユーザー」という単語は絶対に使わない
      - 呼びかけは「あなた」「君」など自然な表現にする
      - 絵文字は禁止
    PROMPT

    response = @ai_client.chat(
      parameters: {
        model: MODEL,
        temperature: 0.5,
        messages: [
          { role: "system", content: prompt },
          { role: "user", content: profile_info }
        ]
      }
    )

    text = response.dig("choices", 0, "message", "content").to_s
    payload = JSON.parse(text)

    attrs = payload.slice(*KEYS)
    KEYS.each { |k| attrs[k] = "AIコメントの生成に失敗しました。" if attrs[k].blank? }

    attrs
  rescue JSON::ParserError
    KEYS.index_with { "AIコメントの生成に失敗しました。" }
  end
end
```

# 各部分の解説

## `def generate_and_save_all(oshi_profile)`
このメソッドの役割： ***推しプロフィールを受け取って、AIコメントを生成し、DBに保存する」***

外から見た時は、
```
AiCommentGenerator.new.generate_and_save_all(oshi_profile)
```
これで完結する設計

***

### ①引数oshi_profileとはなにか
```
def generate_and_save_all(oshi_profile)
```
ここで受け取っているものは、OshiProfileモデルのインスタンス（DB上の「推しプロフィール1件」）
```
oshi_profile = OshiProfile.find(1)
```
このメソッドは、「どの推しのコメントを作るのか」を外から注入してもらう設計

***

### ②`oshi_profile.build_ai_comment`
**何をしているのか？**

OshiProfile モデルに：
```
has_one :ai_comment
```
が定義されている、その場合、Railsは自動で：
```
build_ai_comment
```
というメソッドを生やす

**主な動き**
- ai_comments テーブルの新しいレコードを作る
- oshi_profile_id を自動でセットする
- まだDBには保存しない（newと同じ）

イメージ：
```
ai_comment = AiComment.new(oshi_profile_id: oshi_profile.id)
```
とほぼ同じ意味

**build_ai_comment を使うメリット**
- 外部キーを自分でセットしなくていい
- 関連が保証される
- コードが読みやすい
- 関連設計が崩れにくい

***

### ③`attrs = generate_comments_hash(oshi_profile)`
**何をしているのか？**
- AIに問い合わせる
- JSONを受け取る
- RubyのHashに変換
- 必要なキーだけ抜き出す
- エラー対策する

全部まとめた「重たい処理」を別メソッドに切り出す

**なぜ分けている？**<br>
もしここに全部書くと、generate_and_save_allが巨大メソッドになる<br>
→**「生成責務」と「保存責務」を分離**

***

### ④`ai_comment.assign_attributes(attrs)`
**何をしているのか？**
attrsはこういうhash：
```
{
  "task_created_comment" => "...",
  "task_updated_comment" => "...",
  ...
}
```
これを
```
ai_comment.task_created_comment = "..."
ai_comment.task_updated_comment = "..."
...
```
を一括でやるメソッド

#### `attr`ってなに
```
attrs = generate_comments_hash(oshi_profile)
```
この attrs は、ただの RubyのHash（ハッシュ） で、中身は
```
{
  "task_created_comment" => "いいスタートだね！私がそばにいるよ",
  "task_updated_comment" => "少しずつ形になってきたね",
  "task_deleted_comment" => "切り替えも大事だよ",
  "task_completed_comment" => "やったね、本当にえらいよ",
  "sub_task_completed_comment" => "一歩ずつ進めてるの、ちゃんと見てるよ",
  "task_incomplete_comment" => "また挑戦すればいいさ"
}
```
**「カラム名 → そのカラムに入れたい値」のセット**

**attrs は attributes（属性） の略で、Rails界隈ではよく使う名前**

#### `じゃあ「attribute（属性）」って何？`
Railsのモデルでいう attribute とは、**DBテーブルのカラム1つ1つのこと**

task_created_comment、task_updated_commentなどが全て**モデルの属性（attributes）**

#### `assign_attributes(attrs) って何？`
これは、**attrs に書いてある内容をai_comment の各カラムに一気にセットする**

もし assign_attributes を使わなかったら、
```
ai_comment.task_created_comment = attrs["task_created_comment"]
ai_comment.task_updated_comment = attrs["task_updated_comment"]
ai_comment.task_deleted_comment = attrs["task_deleted_comment"]
ai_comment.task_completed_comment = attrs["task_completed_comment"]
ai_comment.sub_task_completed_comment = attrs["sub_task_completed_comment"]
ai_comment.task_incomplete_comment = attrs["task_incomplete_comment"]
```
って6行書く必要がある、これを1行でできるようにしている

***

### ⑤`ai_comment.save!`
バリデーション実行し、問題なければDBにINSERT（失敗したら例外を投げる）

**なぜsave!なのか**<br>
save だと：
  - false が返るだけ
  - 失敗に気づきにくい

save! は：
  - 失敗した瞬間に例外
  - バグにすぐ気づける
  - ログに残る

## まとめ：1行ずつわかりやすく書くと
```
def generate_and_save_all(oshi_profile)
  # 推しプロフィールに紐づくAIコメントを新規作成する
  ai_comment = oshi_profile.build_ai_comment

  # AIからコメントをまとめて生成する
  attrs = generate_comments_hash(oshi_profile)

  # 生成された内容をモデルにセットする
  ai_comment.assign_attributes(attrs)

  # DBに保存する（失敗したら例外）
  ai_comment.save!
end
```