# Blog 寫作指南

## 建立文章

```bash
hugo new content post/my-article/index.md
hugo server -D
```

文章與圖片放在同一個 page bundle：

```text
content/post/my-article/
├── index.md
├── cover.jpg
└── diagram.png
```

完成後將 front matter 的 `draft` 改成 `false`。

## Front matter 建議

- `title`：清楚描述文章解決的問題。
- `description`：一到兩句摘要，供搜尋及社群分享使用。
- `slug`：發布後盡量不要修改，避免網址失效。
- `categories`：每篇一個主要分類。
- `tags`：二到五個具體主題。
- `image`：同目錄的封面檔名，例如 `cover.jpg`。
- `lastmod`：內容有實質更新時再改。
- `math: true`：需要數學公式時啟用。

## 內容結構

建議依文章類型調整，不必每篇硬套：

1. 問題與背景
2. 環境、版本與前置條件
3. 實作或解法
4. 驗證結果
5. 限制、踩坑與替代方案
6. 總結及參考資料

使用 `<!--more-->` 明確切分首頁摘要，避免首頁顯示過長內容。

## 發布前

```bash
hugo --minify --gc
```
