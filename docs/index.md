---
layout: default
title: Home
---

# Welcome to Table Display Sample

このサイトは Jekyll と Liquid を使用して、JSONC 形式のテーブルデータを動的に表示する例です。

## Features

- JSONC 形式のデータ管理
- item3 列によるフィルタリング機能
- note 列のセル結合対応

## Pages

- [Table Display]({{ site.baseurl }}/table_display.html) - テーブルデータの表示ページ

## 使用方法

1. `_data/table_data.json` にテーブルデータを JSON 形式で記述
2. `table_display.md` でテンプレートを定義
3. Jekyll でビルド・デプロイ

詳しくは [テーブル表示ページ]({{ site.baseurl }}/table_display.html) をご覧ください。
