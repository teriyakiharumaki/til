# LLMの環境構築

## ✅ 行ったこと

- LLMをpythonで動かすための環境構築
- 参考文献8章の理解、実装

## 開発環境
LLM：Gemini<br>
開発環境：Google Colaboratory

## 学んだこと
- GeminiのAPIキーの取得方法
- GeminiのAPIは制限はあるが、無料で使える
- シークレットキー（Gemini APIキー）を、Google Colaboratoryで使えるようにする方法（シークレットの登録方法）
- Google Colaboratoryでモデルを実行、テキスト生成のプロセスの確認できた
- GenerationConfigの設定内で、
  - max_output_tokens = ~~~ で生成する文章の長さを調整
  - temperture = ~~~ (0 ~ 1の間)で、出力のランダム性を制御できる

## 8.3.2.2の#7のコードの解説について
処理がわからないので一文ずつ考えた

### def generate_content(model, prompt):
関数を定義している<br>

引数：
- model:使いたいAIモデル（#5でmodel内にどのGeminiモデルを入れるかは実装済み）
- prompt:ユーザーの入力（AIに渡す文章や質問）

### response = model.generate_content(prompt, generation_config=config)
- AIに実際にプロンプトを送信して、返答を受け取る場所
- `model.generate_content()` は Gemini のAPIメソッドで、プロンプトを送って 生成結果（＝AIの返事） を response として返す
- `generation_config=config` では、生成の設定（「文章の長さ」や「ランダム性{温度}」）を渡している

### return response.text
APIから返ってきた response はオブジェクトで、テキスト部分を取り出すために.text を使う

### わからなかった点
関数として定義する「generate_content(model, prompt)」と、 <br>
この関数の定義内の「generate_content(prompt, generation_config=config)」 がごっちゃになってたのが原因

## 参考文献
末次拓斗『誰でもわかる大規模言語モデル入門』