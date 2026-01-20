---
name: create-directory
description: 協助建立目錄和資料夾結構。當使用者需要創建新目錄、設置專案結構或組織檔案系統時使用此 skill。
---

# Create Directory Skill

此 skill 提供建立目錄的最佳實踐和指引，確保目錄結構清晰、有組織且符合專案需求。

## 何時使用此 skill

- 當使用者要求建立新目錄或資料夾時
- 當需要設置專案的目錄結構時
- 當需要組織現有檔案到新的目錄結構時
- 當需要創建多層次的目錄架構時

## 建立目錄的步驟

### 1. 確認需求

在建立目錄前，先確認：
- 目錄的用途和目的
- 是否需要多層次結構
- 是否需要同時建立多個目錄
- 目標位置（絕對路徑或相對路徑）

### 2. 選擇適當的命令

根據作業系統使用正確的命令：

#### Windows (PowerShell)
```powershell
# 建立單一目錄
New-Item -ItemType Directory -Path "目錄名稱"

# 建立多層次目錄（自動建立父目錄）
New-Item -ItemType Directory -Path "父目錄\子目錄\孫目錄" -Force

# 建立多個目錄
New-Item -ItemType Directory -Path "目錄1", "目錄2", "目錄3"
```

#### Linux/Mac (Bash)
```bash
# 建立單一目錄
mkdir 目錄名稱

# 建立多層次目錄
mkdir -p 父目錄/子目錄/孫目錄

# 建立多個目錄
mkdir 目錄1 目錄2 目錄3
```

### 3. 驗證結果

建立目錄後，應該：
- 確認目錄已成功建立
- 檢查目錄權限是否正確
- 列出目錄內容以驗證結構

## 常見目錄結構模式

### 專案基本結構
```
project/
├── src/           # 原始碼
├── tests/         # 測試檔案
├── docs/          # 文檔
├── config/        # 配置檔案
└── build/         # 建置輸出
```

### Web 應用程式結構
```
webapp/
├── public/        # 靜態資源
├── src/
│   ├── components/
│   ├── pages/
│   ├── utils/
│   └── styles/
└── tests/
```

### 資料處理專案
```
data-project/
├── data/
│   ├── raw/       # 原始資料
│   ├── processed/ # 處理後資料
│   └── output/    # 輸出結果
├── scripts/       # 處理腳本
└── notebooks/     # Jupyter notebooks
```

## 最佳實踐

1. **使用有意義的名稱**
   - 使用清晰、描述性的目錄名稱
   - 避免使用空格（使用 `-` 或 `_` 代替）
   - 使用小寫字母以提高跨平台相容性

2. **保持結構簡單**
   - 避免過深的巢狀結構（建議不超過 3-4 層）
   - 將相關檔案組織在一起
   - 使用標準的目錄命名慣例

3. **考慮未來擴展**
   - 預留空間給未來可能的新功能
   - 使用模組化的目錄結構
   - 保持彈性以適應變化

4. **添加說明文件**
   - 在重要目錄中添加 README.md
   - 說明目錄的用途和內容
   - 提供使用範例

## 錯誤處理

### 目錄已存在
- 使用 `-Force` (PowerShell) 或檢查目錄是否存在
- 詢問使用者是否要覆寫或使用現有目錄

### 權限不足
- 確認使用者有足夠的權限
- 考慮使用管理員權限或更改目標位置

### 路徑無效
- 驗證路徑格式是否正確
- 確認父目錄存在（或使用 `-Force`/`-p` 自動建立）

## 範例場景

### 場景 1：建立新專案結構
```powershell
# 建立完整的專案目錄結構
New-Item -ItemType Directory -Path "my-project\src", "my-project\tests", "my-project\docs" -Force
```

### 場景 2：組織現有檔案
```powershell
# 先建立分類目錄
New-Item -ItemType Directory -Path "images", "documents", "videos" -Force
# 然後移動檔案到對應目錄
```

### 場景 3：建立深層結構
```powershell
# 一次建立多層目錄
New-Item -ItemType Directory -Path "project\src\components\ui\buttons" -Force
```

## 後續步驟

建立目錄後，通常需要：
1. 添加 README.md 或說明文件
2. 設置 .gitignore（如果使用版本控制）
3. 建立初始檔案或模板
4. 配置相關工具和設定
