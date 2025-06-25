# Active Recordって何？

## ✅ 行ったこと

- Active Recordについて調べた

## Active Recordとは

- Active Recordとは、MVCで言うところのM、つまりモデルに相当するものであり、ビジネスデータとビジネスロジックを表すシステムの階層です。Active Recordは、データベースに恒久的に保存される必要のあるビジネスオブジェクトの作成と利用を円滑に行なえるようにします。Active Recordは、ORM (オブジェクト/リレーショナルマッピング) システムに記述されている「Active Recordパターン」を実装したものであり、このパターンと同じ名前が付けられています。
（Railsガイドから）

## ORMとは
- オブジェクト/リレーショナルマッピング(O/RマッピングやORMと略されることもあります)とは、アプリケーションが持つリッチなオブジェクトをリレーショナルデータベース(RDBMS)のテーブルに接続することです。ORMを用いると、SQL文を直接書く代りにわずかなアクセスコードを書くだけで、アプリケーションにおけるオブジェクトの属性やリレーションシップをデータベースに保存することもデータベースから読み出すこともできるようになります。
(Railsガイドから)

- Object/Relational Mappingの略。（O/Rマッピングとも呼ばれる）<br>
DBをSQL文で直接操作するのではなく、Modelを介して操作する、この技術をORMと呼ぶ。<br>
ORMの技術を使うことで、DBとModelが接続できる。このDBとの接続処理をRailsではActive Recordと呼ぶ。

- ORMとは、Object Relational Mappingの略です。<br>
これは、オブジェクト指向プログラミング言語（たとえばRubyやJavaなど）とリレーショナルデータベース（たとえばMySQLやPostgreSQLなど）の間をつなぐ仕組みを指します。<br>
通常、リレーショナルデータベースの操作にはSQLが必須ですが、開発者はSQLではなく、自分が書いているプログラムのクラスやメソッドを通じてデータベースにアクセスできるようになります。<br>
たとえば、データを取得するときも、プログラムコードの形式で「User.find(1)」のような書き方をすると、裏側で「SELECT * FROM users WHERE id=1」が実行されるイメージです。<br>
このように、ORMを使うことで、データベースとやり取りする部分がプログラミング言語のオブジェクトとして扱いやすくなるという利点があります。<br>
ORMは多数のフレームワークやライブラリで採用されており、 Railsで言えばActive Record、PHPのLaravelで言えばEloquentなどが挙げられます。<br>
ORMを利用するとSQLを書く頻度が減るので、コード量やメンテナンス工数の削減が期待できます。<br>


## つまり、どういうものか
- Active Recordとは、Railsのモデルに相当するもので、Rubyのオブジェクトとデータベースのリレーショナルテーブルを紐付け、オブジェクト指向の方法でデータベース操作を行う仕組み
- ActiveRecordを使うことで、SQLを直接書かずにデータベースとやり取りできるため、開発者は直感的にデータ操作が可能になる

## 参考サイト
https://railsguides.jp/v5.2/active_record_basics.html
https://www.highengenius.com/career/ruby/technical-interview.html
https://qiita.com/shizen-shin/items/d3c3a41ff9b46ebd1436#active-record%E3%81%A8%E3%81%AF


https://qiita.com/masakichi_eng/items/6a9faa60d5cdb5449f61