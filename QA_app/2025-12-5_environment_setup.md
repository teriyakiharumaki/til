# 一問一答ができるサイト「QA」を作る

## ✅ 行ったこと

- 環境構築を行なった

## ディレクトリを作成し、移動
```
mkdir qa
cd qa
```

## 以降のセットアップ
TIL/tubuyaki_app/2025-08-16_environment_setup.mdと2025-08-17_setup_error.mdの二つを参考に同様のセットアップにした

## 起きたエラー
2025-08-16_environment_setup.md内で追記済みだが、
```
connection to server on socket "/var/run/postgresql/.s.PGSQL.5432" failed: No such file or directory
```
このエラーは、<br>
config/database.yml の development: に 明示的に host: db と passward を追記して解決していたが、<br>
test: にも追記しなければエラーが出る<br>

## スマートな記述方法
developmentとtestに同じ内容を記述する場合、defaultに一度に記述する方が良い