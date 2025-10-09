# Strong Parameters

## ✅ 行ったこと

- Strong Parametersについて調べた

## Strong Parameterとは[1]
Rails には、 **ストロングパラメーター（Strong Parameter）** という、ユーザーから送信されるデータを制限する機能があります。
この仕組みを使うことで、安全では無いデータの登録・更新を防いでくれます。

## ストロングパラメータの書き方[1]

### 基本構文
**user_controller.rb(一部)**
```ruby:user_controllers.rb
private
def user_params
  params.require(:キー(モデル名)).permit(:カラム名１,：カラム名２,・・・).merge(カラム名: 入力データ)
end
```

### 具体例
**user_controller.rb（具体例）**
```ruby:user_controller.rb
class UsersController < ApplicationController
  # (中略)

  private 

    def user_params
      params.require(:user).permit(:name, :email, :password,:password_confirmation)
    end
end
```
メソッド名は **[モデル名_params]** とするのが一般的

## requireメソッドとparmidメソッド[1]

① require: **受け取る値のキーを設定** <br>
② permit: **許可するカラムを設定**

具体例の場合はパラメーターの **:userというキーの中の:name, :email,:password,:password_confirmationだけを許可する設定** <br>
👉つまり、パラメーターとしての **:name, :email, :password,:password_confirmation4つだけデータの作成・更新を許可する** といった内容

ストロングパラメーターを使わずに、そのままユーザーから送られてきた値を使ってデータを作ったり、更新しようとすると、 

**ユーザーに書き換えられたくない値まで書き換えられてしまう** 

という問題が起きてしまいます。

## strong parameterを設定する理由[2]
### 以下別サイトの例を用いる（元サイト参照）

createアクションの引数に下記のようにparamsとするとエラーが出て保存できません。
```
Hoge.create(params)
```

これはもし不正なパラメーターが入っていた場合、それまで取り出してデータベースに保存されるのを防ぐためです。<br>
どのような仕組みでエラーが出るのでしょうか？それを確認してみましょう。<br>

binding.pryで処理を止めてparamsでパラメーターを取得すると下記のようなコードになっています。
```
[1] pry(#<UsersController>)> params
=> <ActionController::Parameters 
{"utf8"=>"✓", 
"authenticity_token"=>"hogehoge==",
 "name"=>"プログラマン2号",
 "job_id"=>2,
 "controller"=>"users",
 "action"=>"create"} permitted: false>
```

最後の行を見るとpermitted: falseというコードがあります。<br>
この状態でcreateアクションの引数にparamsを指定するとエラーが出てしまうというわけです。<br>

では次にストロングパラメータ内に書いたpermitメソッドを使った返り値を確認してみましょう。
```
pry(#<UsersController>)> params.permit(:name,:job_id)
Unpermitted parameters: :utf8, :authenticity_token
=> <ActionController::Parameters
 {"name"=>"プログラマン2号", "job_id"=>2} permitted: true>
```
最後の行が **permitted: true** になっているのが確認できますね。<br>
そしてpermitで指定していないparamsで取得した他のパラメータは **Unpermitted parameters** となっています。<br>
このようにストロングパラメータを定義するとエラーなく保存ができるというわけです。<br>

ストロングパラメータはclass内で **privateメソッド** を使い定義します。

### privateメソッド（補足）
クラスを定義するときにprivateメソッドを呼び出すとそれ以降に定義されるインスタンスメソッドはそのクラス中でしか呼び出されなくなります。<br>
privateメソッドを使っておけばクラス外部で呼び出されるとエラーが起きるようなメソッドが使われるのを防ぐことができたり、コードの可読性が上がるなどのメリットがあります。<br>

ストロングパラメータはクラス内部でしか使わないので、そういうメソッドはprivate以下に定義しておきましょう。

### データの書き換えについての具体的な例について

https://pikawaka.com/rails/strong_parameter#データ書き換えの危険性を確認しよう<br>

主に上のページを参考に見る<br>
検証からパラメータを書き換えることで好きなカラムを編集することが可能になってしまう

## 参考サイト
[1](https://qiita.com/Hashimoto-Noriaki/items/736b734b250f059dd34f)
[2](https://pikawaka.com/rails/strong_parameter)