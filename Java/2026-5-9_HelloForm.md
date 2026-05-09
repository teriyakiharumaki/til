# 基本コードの記述

## ✅ 行ったこと

- フォームデータを保持するクラスを作成した

## 参考にしたサイト
https://route-zero.com/recruit/route/922/

## 各コードの記述

### HelloForm.java

```
package com.example.demo.form;
  
  public class HelloForm {
      private String name;
  
      // ゲッターとセッター
      public String getName() {
          return name;
      }
  
      public void setName(String name) {
          this.name = name;
      }
  }
```

このコードでは、**フォームの入力値を入れておく箱**

### ① フィールド（データ本体）
```
private String name;
```
**nameというデータを持つ**

privateって何？ → 外から直接触れないようにする（安全にする）

### ② getter（取り出す）
```
public String getName() {
    return name;
}
```
nameを取得する

#### 使われ方
```
form.getName()
```
入力された名前を取り出す

### ③ setter（セットする）
```
public void setName(String name) {
    this.name = name;
}
```
nameをセットする

#### 使われ方（裏側）
```
HTMLフォーム
↓
Springが自動でsetName()を呼ぶ
```
自分で呼ばなくてOK

### 一番重要なポイント
```
@ModelAttribute HelloForm form
```
これとつながる

### どう動いてるか
```
① フォームで name=tokunaga を送る
↓
② Springが自動で HelloFormを作る
↓
③ setName("tokunaga") を呼ぶ
↓
④ formの中に値が入る
```