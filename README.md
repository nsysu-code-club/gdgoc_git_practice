# Welcome Developers

一個可部署到 GitHub Pages 的互動式成員牆。`members/` 裡的照片會變成有重力、碰撞、彈跳與拖曳效果的圓形球體。

## 功能

- 自動掃描 `members/` 及其子資料夾中的圖片
- 將所有照片裁切成相同大小的圓形
- 圓球總面積最多占容器的 60%，視照片數量及視窗尺寸動態調整
- 模擬重力、碰撞、彈跳、旋轉與摩擦力
- 可用滑鼠或觸控拖曳圓球
- 游標移到圓球上時，照片會變成灰階並顯示不含副檔名的檔名
- 合併到 `main` 後，自動產生照片清單並部署至 GitHub Pages
- Pull Request 會檢查 `members/` 內是否只有合法圖片，通過後自動合併

## 專案結構

```text
.
├── .github/workflows/
│   ├── deploy-pages.yml
│   └── validate-and-automerge.yml
├── members/                 # 成員圖片放這裡
├── generate-photos.js       # 掃描圖片並產生清單
├── photos.js                # 自動產生，請勿手動編輯
├── index.html
├── script.js
└── style.css
```

## 新增成員照片

1. 將圖片放入 `members/`，也可以建立子資料夾。
2. 建立新分支並提交變更。
3. 推送分支並建立 Pull Request。
4. GitHub Actions 會驗證圖片；通過後會啟用自動合併。
5. 合併至 `main` 後，GitHub Pages 會自動重新部署。

允許的格式：

```text
.jpg  .jpeg  .png  .gif  .webp  .avif
```

驗證不只檢查副檔名，也會透過 MIME type（檔案的實際內容類型）確認檔案確實是圖片，因此只把文字檔改名成 `.jpg` 仍會被拒絕。

一般 Pull Request 只允許修改 `members/` 裡的圖片。修改任何 `README.md`（包含 `members/README.md` placeholder）、HTML、CSS、JavaScript 或 workflow 都會讓 `validate-member-images` 失敗；需要維護網站程式時，管理者必須使用 Ruleset bypass 流程。

## 本機預覽

需要先安裝 Node.js。

每次新增或刪除照片後，在專案根目錄執行：

```bash
node generate-photos.js
```

接著可直接開啟 `index.html`，或啟動任一靜態檔案伺服器。本機測試時必須先重新執行產生器，否則 `photos.js` 不會反映最新圖片。

## GitHub Pages 設定

前往儲存庫：

**Settings → Pages → Build and deployment → Source → GitHub Actions**

之後每次有變更進入 `main`，`deploy-pages.yml` 都會：

1. 執行 `generate-photos.js`
2. 產生最新的 `photos.js`
3. 上傳靜態網站
4. 部署至 GitHub Pages

## Ruleset 與自動合併設定

### 1. 啟用一般合併與自動合併

前往 **Settings → General → Pull Requests**，開啟：

- Allow merge commits
- Allow auto-merge

### 2. 允許 Actions 寫入

前往 **Settings → Actions → General → Workflow permissions**，選擇：

- Read and write permissions

### 3. 建立 `main` Ruleset

前往 **Settings → Rules → Rulesets → New branch ruleset**，設定：

- Ruleset name：`Protect main`
- Enforcement status：`Active`
- Target branches：Default branch
- Require a pull request before merging
- Required approvals：`0`（若需要人工審核可自行提高）
- Require status checks to pass：加入 `validate-member-images`
- Block force pushes
- Restrict deletions

第一次設定時，必須先讓 `validate-and-automerge.yml` 在 Pull Request 中執行一次，GitHub 才會在 Ruleset 選單中顯示 `validate-member-images`。

不要把 `enable-auto-merge` 設為必要檢查；必要檢查只需加入 `validate-member-images`。

## 自動化流程

```text
建立 Pull Request
        ↓
驗證 members/ 內的圖片
        ↓
通過後啟用一般 merge commit 自動合併
        ↓
合併至 main
        ↓
主動啟動部署 workflow
        ↓
重新產生 photos.js 並部署 GitHub Pages
```

## 注意事項

- 請勿手動修改 `photos.js`，部署時會被重新產生。
- 圖片檔名會顯示在圓球的 hover 狀態；顯示時會移除最後一段副檔名。
- Pull Request 草稿不會啟用自動合併，改成 Ready for review 後才會執行。
- 從 fork 建立的 Pull Request 可能因 `GITHUB_TOKEN` 權限限制而無法自動合併，但圖片驗證仍可執行。
- 自動合併使用的 `GITHUB_TOKEN` 不會觸發一般 `push` workflow，因此合併 job 會在確認 PR 已合併後，透過 `workflow_dispatch` 主動啟動 Pages 部署。
