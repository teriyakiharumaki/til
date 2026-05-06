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