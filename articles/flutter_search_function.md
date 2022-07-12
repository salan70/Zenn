---
title: "【Flutter】巷のアプリでよくある検索バーの実装"
emoji: "🎃"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [flutter, hive]
published: false
---
# はじめに

## 検索フォームの実装
### 地味に躓いたところ
・suffixIconを設定すると、TextFieldのsizeが変わる
↓で対応
suffixIconConstraints:
                      BoxConstraints(maxHeight: 24.h, maxWidth: 24.w),

・clear()を使っても、キーボードで確定前の値は保持される

### 甘えた部分
TextFieldの角を丸くしたかったが、角を丸くしつつ周りの線を無効にする方法がわからなかったため断念

## 検索履歴の表示

## 検索結果の表示