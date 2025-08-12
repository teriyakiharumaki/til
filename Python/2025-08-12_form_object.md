# body内の`{{ form }}`いついて

## ✅ 行ったこと

- `{{ form }}`について調べた

## `{{ form }}`はなぜ二重カッコなのか
### 変数の値を表示するための構文
{{ 変数名 }} → ビュー（views.py）から渡された変数の中身を表示するための構文<br>
`{{ form }}` は「ビューから渡されたformオブジェクトを、HTMLとして出力して表示してね」という意味。

## どこからformは渡されているのか
理解のために、create_postの処理を振り返っていく
```
def create_post(request):
    """
    新たなデータを作成する
    """
    # オブジェクトを新規作成する
    post = Post()

    # ページロード時
    if request.method == 'GET':
        # 新規作成オブジェクトにより form を作成
        form = PostForm(instance=post)

        # ページロード時は form を Template に渡す
        return render(request,
                      'sample_app/post_form.html',  # 呼び出す Template
                      {'form': form})  # Template に渡すデータ

    # 実行ボタン押下時
    if request.method == 'POST':
        # POST されたデータにより form を作成
        form = PostForm(request.POST, instance=post)

        # 入力されたデータのバリデーション
        if form.is_valid():
            # チェック結果に問題なければデータを作成する
            post = form.save(commit=False)
            post.save()

        return redirect('sample_app:read_post')
```

まずは、
```
post = Post()
```
👉 Postモデルの空のインスタンスを作成する。つまり、まだ何もデータが入っていない「空の投稿」オブジェクト。<br>

```
if request.method == 'GET':
```
👉 ページが最初に表示されたとき（＝ブラウザで新規作成フォームを開いた時）の処理<br>

```
form = PostForm(instance=post)
```
👉 PostForm というフォームを作るよ。<br>
ここで instance=post とすることで、空のPostインスタンスを使ってフォームを作る＝空のフォームが表示される。<br>

```
# ページロード時は form を Template に渡す
        return render(request,
                      'sample_app/post_form.html',  # 呼び出す Template
                      {'form': form})  # Template に渡すデータ
```
これのそれぞれの意味は、
- render(request, テンプレートパス, コンテキスト)：

  これは Django の関数で、「テンプレートにデータを渡してHTMLを生成し、クライアントに返す」ための処理です。

- request：

  これは HTTPリクエストオブジェクト。今アクセスしてきているユーザーの情報や、GET/POSTなどのメソッドを含んでいます。

- 'sample_app/post_form.html'：

  これは 使いたいテンプレートファイルのパス。<br>
  templates/sample_app/post_form.html が読み込まれて、HTMLが生成されます。

- `{'form': form}`；

  これは テンプレートに渡すデータ（コンテキスト）<br>
  辞書形式で、キー 'form' をテンプレートに渡しています。<br>
  つまり、テンプレート内で {{ form }} というふうに書くと、この form が表示されるようになります。

### つまり、ページロード時は、空っぽのフォームを作って、templateに送り、それを表示させることで、フォーム入力前に空っぽのフォームができる

## 実行ボタン押下時について
```
if request.method == 'POST':
```
これは、ユーザーが「送信ボタンを押した時（フォームが送信された時）」を判定する条件文です。
- GET → 通常のページ表示
- POST → ユーザーがフォームを送信してきたとき
なので、この中に「送信後の処理」を書くのが基本<br>

```
 form = PostForm(request.POST, instance=post)
```
これは、「送られてきたデータ（request.POST）を使ってフォームを生成」しています。
- request.POST：フォームに入力されたデータ
- instance=post：既存のpostオブジェクトにデータを流し込む（新規作成時は空のPost()、編集時は既存のPost）
➡️ これで、フォームとデータのひも付けができます。<br>

```
if form.is_valid():
```
ここで、フォームの*バリデーション（入力チェック）を行います。
例えば：
- 空欄禁止の項目が空になっていないか？
- 文字数制限に引っかかっていないか？
- フォーマットに問題がないか？

などを Django が自動でチェックしてくれます。

```
post = form.save(commit=False)
```
ここで、フォームの内容を使って post オブジェクトを作成しますが、まだデータベースには保存していません。<br>
commit=False にすることで、「保存する前に、何か追加の処理をしたい」ときに使います。<br>
※ 今回の例ではそのまま保存するだけですが、今後カスタマイズのために便利です。<br>

```
post.save()
```
ここで、実際に データベースに保存されます<br>
この1行で、DBに新しいデータが登録されるか、既存のデータが更新されます<br>

```
return redirect('sample_app:read_post')
```
保存が終わったら、ユーザーを一覧ページにリダイレクト（移動）させます。
- 'sample_app:read_post' は URLの名前（urls.pyで定義している名前付きルート）
- redirect() は ページ遷移をさせる Django の関数

## PostForm()とは
PostForm は views.py 内で定義されている<br>
view.py内で上の方の
```
from django.forms import ModelForm
```
で、Djangoの ModelForm を使ってフォームを定義する準備をしている、ということ<br>
さらに、下の方に
```
class PostForm(ModelForm):
    """
    フォーム定義
    """
    class Meta:
        model = Post
        # fields は models.py で定義している変数名
        fields = ('name', 'micropost')
```
これがあり、ここでは<br>
class PostForm(ModelForm):	ModelForm を継承して、新しいフォームクラス PostForm を定義しています。<br>
class Meta:	ModelForm にとって必要な「設定（メタデータ）」を書く内部クラスです。<br>
model = Post	このフォームは、Post モデルに基づいて作られます。つまり保存先も Post テーブルです。<br>
fields = ('name', 'micropost')	フォームとして入力を受け付けるフィールドを指定しています。今回は name と micropost の2つだけ。<br>

### PostForm(request.POST, instance=post)について
PostForm(...)：PostForm フォームを作成します（クラスを呼び出してインスタンス化）<br>
request.POST：ユーザーがフォームに入力して送信したデータが入っています（`<input>` や `<textarea>` の値）<br>
instance=post：このフォームは、既存の post オブジェクトを編集するものだと明示します。つまり「更新フォーム」になります<br>

🤔 なぜ instance=post が必要なのか<br>
✔ ありの場合：
```
form = PostForm(request.POST, instance=post)
```
→ フォームの送信内容で、既存の post オブジェクトの内容を上書きします。<br>

✖ なしの場合：
```
form = PostForm(request.POST)
```
→ 入力内容をもとに、新しい Post を作成しようとする動きになる<br>
つまり、PostForm(request.POST, instance=post) は、「ユーザーが入力したフォームの内容（request.POST）を指定された post オブジェクトに上書きするつもりで使うフォーム」