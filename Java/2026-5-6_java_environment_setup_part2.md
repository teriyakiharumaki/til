# 基本コードの記述

## ✅ 行ったこと

- spring bootのプロジェクト内のコードの記述について

## 参考にしたサイト
https://route-zero.com/recruit/route/922/

## 各コードの記述

### DemoApplication.java

エントリポイントとなるクラスで、自動生成されたものを使用

```
package com.example.demo;
  
  import org.springframework.boot.SpringApplication;
  import org.springframework.boot.autoconfigure.SpringBootApplication;
  
  @SpringBootApplication
  public class DemoApplication {
      public static void main(String[] args) {
          SpringApplication.run(DemoApplication.class, args);
      }
  }
```

### ① パッケージ
```
package com.example.demo;
```

ファイルの住所みたいなもの

### ② import
```
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
```

必要な機能を読み込んでる（Dependenciesとつながる）

### ③ 大事な部分
```
@SpringBootApplication
```

Spring Bootを起動するためのアノテーション

中身はこれのセット
```
@Configuration
@EnableAutoConfiguration
@ComponentScan
```
ざっくり意味

アノテーション	役割

Configuration	設定クラス

EnableAutoConfiguration	自動設定

ComponentScan	クラスを探す

***Controllerとかを自動で見つけてくれる***

だから`@Controller`が動く

### ④ クラス

```
public class DemoApplication {
```
Javaはすべてクラスで書く

### ⑤ mainメソッド
```
public static void main(String[] args)
```

プログラムの開始地点（ここから全部始まる）

### ⑥ 実際の起動処理
```
SpringApplication.run(DemoApplication.class, args);
```

これがやってること
```
① Spring Boot起動
② 設定読み込み
③ Controller探す
④ サーバー（Tomcat）起動
```

## エントリーポイントとは？

> プログラムの実行を開始する場所のことで、一番最初に呼ばれるメソッドになります。<br>
> メソッドは、メソッドからメソッドを呼び、さらにその次のメソッドを呼ぶ、という流れで実行されていきます。<br>
> その中で一番最初に呼ばれるメソッドをエントリーポイント（開始点）と言います。<br>
> Javaではmainメソッドがプログラム全体のエントリーポイントに設定されます（C、C++、Objective-C等の多言語でもmainメソッドがエントリーポイントとして使われます）。<br>
> なのでJavaでの エントリーポイント = mainメソッドと覚えておけば良さそうです。
> 参考サイト（https://qiita.com/mojirico/items/b1c6f1816d4027254ea5）

いままでにエントリーポイントを意識していtことがなかったので、調べてみたところ

> 多くのプログラムは複数の関数やメソッドを持っているため、起動するためには、プログラムの開始位置を示す必要があります。<br>
> このプログラムの開始位置は、プログラムによって規定されるのではなく、言語仕様によって規定されています。<br>
> 開始位置のルールは、言語によっても異なり、大きく２種類に分けられます。<br>
> - コードの先頭から始まる言語
> - 特定の仕様の関数やメソッドから始まる言語<br>
> 
> Python・Ruby・PHPが前者で、Javaをはじめ、C・Goなどが後者にあたります。<br>
> 参考サイト（https://zenn.dev/wakinoza/articles/e8d547b4d00e56）

このよう違いがあった