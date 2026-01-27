# AIコメント生成コードの修正

## ✅ 行ったこと

- AIによるコメント生成機能のコードを修正の行なった

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

# 修正後の効果

生成にかかる時間

約10秒→ ***約5秒***

# 各部分の解説

## `require "json"`
何をしているのか？<br>
  →Rubyで JSON.parse や JSON.generate を使えるようにしている

## 定数定義（MODEL / KEYS）
```
MODEL = "gpt-4.1".freeze
```
- どのモデルを使うかを1箇所で管理
- 将来モデル変えるときに1行だけ直せばOK

.freeze は：
- 文字列を変更できなくする
- Rubyの定数ではよくある書き方

```
KEYS = %w[
  task_created_comment
  ...
].freeze
```
- AIが返すべきJSONのキー一覧
- DBに保存したいカラム名と一致させている

### `%w`ってなんだ
***「文字列の配列（Array）を簡単に書くための記法」***

普通の配列の書き方
```
keys = ["task_created_comment", "task_updated_comment", "task_deleted_comment"]
```
これを`%w`を使うと
```
keys = %w[task_created_comment task_updated_comment task_deleted_comment]
```
になる

#### メリット
- ダブルクォートがいらない
- スペースや改行で区切れる

### `%w`についてのサイト
https://www.sejuku.net/blog/46939#index_id1

## initialize（APIクライアント準備）
```
def initialize(api_key = ENV["OPENAI_API_KEY"] || Rails.application.credentials.dig(:openai, :api_key))
```
環境変数があればそれを使う、なければ credentials から読む

### 引数に「デフォルト値」を持たせている
Rubyでは、メソッドの引数に「初期値」を設定できる

#### 評価される順番（|| の意味）
```
A || B
```
- A が存在すれば A を使う
- A が nil / false なら B を使う

なので今回：
- ENV["OPENAI_API_KEY"] があればそれを使う
- なければ credentials から読む

### `ENV["OPENAI_API_KEY"]`とは
これは OSの環境変数

Render本番ではほぼ確実に：
- ダッシュボードで APIキーを設定
- コンテナ起動時に ENV として渡される

### `Rails.application.credentials.dig(...)`とは
これは Rails の 暗号化された設定ファイル：
```
openai:
  api_key: sk-xxxx
```
みたいに保存されてる想定

***digの意味***<br>
https://qiita.com/mmaumtjgj/items/3790d9f00780cbe66b62

### `@ai_client = OpenAI::Client.new(access_token: api_key)`

#### ①Rubyとしての動き
1. OpenAI::Client クラスを探す
2. .new でインスタンスを生成
3. initialize(access_token: api_key) が内部で呼ばれる
4. その結果、クライアントオブジェクトが1つ作られる

イメージ：
```
OpenAI::Client クラス
   ↓ new
OpenAI::Clientのインスタンス（メモリ上の実体）
```

```
@ai_client = ...
```
作られたオブジェクトを、このAiCommentGeneratorインスタンスのインスタンス変数`@ai_client`に保存

つまり、このサービスクラスは「OpenAIと通信する道具」を一つ持っている

### `OpenAI::Client`は何？
openai gem が提供しているクラス（ruby-openai gemをインストールしたときにgem側が定義しているクラス）

OpenAIというモジュール内に、Clientというクラスが定義されている
```
module OpenAI
  class Client
  end
end
```
だから使うときは
```
OpenAI::Client
```

役割：<br>
  ✅ APIキーを内部に保持<br>
  ✅ HTTP通信設定を持つ<br>
  ✅ リクエスト送信・レスポンス受信を担当する

***「OpenAI API 専用の通信担当オブジェクト」***

#### ruby-openaiについてのサイト
https://qiita.com/yocaji/items/cbce3cb92c601984314c<br>
https://qiita.com/gakeppuchi/questions/0bc24ceff55ea7620f91(ruby-openaiとopenaiの違い)