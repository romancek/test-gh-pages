# Table Display with Jekyll

GitHub Pages を使用した、JSONC 形式のテーブルデータ表示プロジェクトです。

## 概要

このプロジェクトは、Jekyll と Liquid テンプレートを活用して、JSON データから動的にテーブルを生成します。

### 機能

- JSONC 形式のテーブルデータ管理
- item3 列による動的フィルタリング
- note 列のセル結合（スマート結合ロジック）
- レスポンシブなテーブルデザイン

## ディレクトリ構成
GitHub Pagesプロジェクト用の .gitignore と README.md のサンプルを示します。

```text
.gitignore
README.md
docs/
├── _data/
│ └── table_data.json # テーブルデータ
├── _config.yml # Jekyll 設定
├── index.md # ホームページ
├── table_display.md # テーブル表示ページ
└── _posts/ # ブログ投稿（オプション）
```

