# go-vanity

Go Vanity URL 服務，用於為 Asgard AI 平台的 Go 套件提供自定義的 import 路徑。

## 專案目的

這個專案提供 Go vanity URL 服務，讓我們可以使用自定義的 import 路徑（如 `go.asgard-ai.com/asgard-sdk-go`）來導入 Go 套件，而不是直接使用 GitHub URL。

## 工作原理

當用戶執行 `go get go.asgard-ai.com/asgard-sdk-go` 時：

1. Go 工具會向 `https://go.asgard-ai.com/asgard-sdk-go?go-get=1` 發送 HTTP 請求
2. 我們的 HTML 頁面返回包含 `go-import` meta 標籤，指示實際的 Git 倉庫位置
3. Go 工具從指定的 GitHub 倉庫克隆代碼

## 專案結構

```
.
├── HOWTO.md
└── asgard-sdk-go/
    └── index.html      # asgard-sdk-go 套件的 vanity URL 頁面
```

每個子目錄對應一個 Go 套件，包含一個 `index.html` 文件。

## 部署方式

本專案使用 **GitHub Pages** 托管靜態頁面。

### 設定步驟

1. 在 GitHub 倉庫設定中啟用 GitHub Pages
2. 選擇 Source 為 `main` 分支
3. 設定自定義域名為 `go.asgard-ai.com`
4. 在 DNS 設定中添加 CNAME 記錄指向 `<username>.github.io`

### 域名設定

在 DNS 服務商（如 Cloudflare）中添加：

```
CNAME  go  asgard-ai-platform.github.io.
```

## 如何添加新的 Vanity URL

### 步驟 1：創建目錄和 HTML 文件

為新套件創建一個目錄，例如 `my-package/`：

```bash
mkdir my-package
```

### 步驟 2：創建 index.html

在目錄中創建 `index.html` 文件：

```html
<!DOCTYPE html>
<html>

<head>
    <meta http-equiv="refresh" content="0; url=https://github.com/asgard-ai-platform/my-package">
    
    <meta name="go-import"
        content="go.asgard-ai.com/my-package git https://github.com/asgard-ai-platform/my-package">

    <meta name="go-source"
        content="go.asgard-ai.com/my-package https://github.com/asgard-ai-platform/my-package https://github.com/asgard-ai-platform/my-package/tree/main{/dir} https://github.com/asgard-ai-platform/my-package/blob/main{/dir}/{file}#L{line}">
</head>

<body>
    Redirecting to <a href="https://github.com/asgard-ai-platform/my-package">GitHub</a>...
</body>

</html>
```

### 步驟 3：提交並推送

```bash
git add my-package/
git commit -m "Add vanity URL for my-package"
git push origin main
```

### 步驟 4：等待部署

GitHub Pages 會自動部署更新（通常需要幾分鐘）。

## HTML 模板說明

每個 `index.html` 文件包含以下關鍵元素：

### 1. 自動重定向（給瀏覽器用戶）

```html
<meta http-equiv="refresh" content="0; url=https://github.com/asgard-ai-platform/PACKAGE-NAME">
```

當用戶在瀏覽器中訪問時，會自動重定向到 GitHub 倉庫。

### 2. go-import meta 標籤（給 go get 使用）

```html
<meta name="go-import" 
    content="go.asgard-ai.com/PACKAGE-NAME git https://github.com/asgard-ai-platform/PACKAGE-NAME">
```

格式：`[import-prefix] [vcs] [repo-url]`

### 3. go-source meta 標籤（給 godoc.org 使用）

```html
<meta name="go-source"
    content="go.asgard-ai.com/PACKAGE-NAME https://github.com/asgard-ai-platform/PACKAGE-NAME https://github.com/asgard-ai-platform/PACKAGE-NAME/tree/main{/dir} https://github.com/asgard-ai-platform/PACKAGE-NAME/blob/main{/dir}/{file}#L{line}">
```

格式：`[import-prefix] [home-url] [directory-template] [file-template]`

## 測試

### 測試 go get

```bash
go get go.asgard-ai.com/asgard-sdk-go
```

### 測試瀏覽器重定向

在瀏覽器中訪問：
```
https://go.asgard-ai.com/asgard-sdk-go
```

應該會自動重定向到 GitHub 倉庫。

### 測試 meta 標籤

```bash
curl -L https://go.asgard-ai.com/asgard-sdk-go?go-get=1
```

應該看到包含正確 meta 標籤的 HTML 響應。

## 維護指南

### 更新套件 URL

如果 GitHub 倉庫名稱或位置改變：

1. 編輯對應的 `index.html` 文件
2. 更新所有 URL 引用
3. 提交並推送變更

### 刪除套件

1. 刪除對應的目錄
2. 提交並推送變更

### 注意事項

- 確保 `go-import` meta 標籤的 import-prefix 與實際路徑匹配
- 保持 VCS 類型為 `git`
- 如果倉庫使用 `master` 而非 `main` 分支，需要更新 `go-source` 中的分支名稱
- GitHub Pages 部署可能需要幾分鐘，請耐心等待

## 現有套件

| 套件名稱 | Vanity URL | GitHub 倉庫 |
|---------|-----------|------------|
| asgard-sdk-go | go.asgard-ai.com/asgard-sdk-go | https://github.com/asgard-ai-platform/asgard-sdk-go |

## 參考資料

- [Go Modules: v2 and Beyond - Custom Import Paths](https://go.dev/blog/module-compatibility)
- [Remote import paths](https://go.dev/cmd/go/#hdr-Remote_import_paths)
- [GitHub Pages Documentation](https://docs.github.com/en/pages)