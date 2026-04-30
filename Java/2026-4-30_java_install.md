# javaのインストール

## ✅ 行ったこと

- javaのインストールを行った

## 参考にしたサイト
https://route-zero.com/recruit/route/922/

## Java Development Kit (JDK)でインストール

以下のサイトで選ぶ<br>
https://www.oracle.com/java/technologies/downloads/#jdk21-mac

[![Image from Gyazo](https://i.gyazo.com/610e3274bd162ae97b2de8a0aabe4278.png)](https://gyazo.com/610e3274bd162ae97b2de8a0aabe4278)

## 選ぶ基準

### ①バージョン

- JDK 24 → 最新（でも安定性はこれから）
- JDK 21 → 長期サポート（LTS）で一番無難

なので、 JDK 21（LTS）を選択

### ②自分のMacの種類

Intel Mac の場合　→　macOS x64

Apple Silicon（M1 / M2 / M3）の場合　→　macOS Arm64

### ③どの形式？

3種類くらいある

- .dmg → インストーラー（簡単）
- .pkg → これでもOK（似てる）
- .tar.gz → 手動設定

.dmg を選ぶ

## 選んだもの

**JDK 21（LTS）→ macOS Arm64→ .dmg**

## インストール後の確認

```
MacBook-Pro ~ % java -version 
java version "21.0.11" 2026-04-21 LTS 
Java(TM) SE Runtime Environment (build 21.0.11+9-LTS-211) 
Java HotSpot(TM) 64-Bit Server VM (build 21.0.11+9-LTS-211, mixed mode, sharing)
```

- 21 → ちゃんと JDK 21
- LTS → 長期サポート版（安定）
- エラーなし