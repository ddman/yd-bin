# yd-bin

`yd-bin` 是個人的 CLI 工具箱，旨在為跨工作環境（如換新電腦、離職移交、切換開發環境）時，提供一套能夠快速同步、隨拿即用的高效開發工具鏈。

---

## 💡 創作背景與目的

在日常開發中，常需要重複性的工具自動化（例如自動撰寫 Git Commit Message、迅速在 Terminal 詢問 AI 等）。

原本採用 **Gemini CLI** (後改名為 **Antigravity CLI**)，但由於完整的 CLI 工具包含了較多封裝層與處理機制，在單純執行特定小型任務時速度不夠迅速。

因此重新設計架構：
1. **`genai`**：採用 Python 搭配官方 `google-genai` SDK，透過 `uv` 的單檔腳本技術打造極輕量、極速回應的 CLI 介面。
2. **`gcommit`**：維持 Bash/Zsh 腳本的簡潔與靈活度，將 diff 內容與提示詞經由 Pipe 傳送給 `genai` 處理，實現流暢的 Git 自動化提交體驗。

---

## 🛠️ 工具列表

| 工具名稱 | 說明 | 依賴/語言 |
| :--- | :--- | :--- |
| **`genai`** | 通用型 Gemini AI 命令行工具，支援 Prompt 傳入與 Pipe 管道流輸入。 | Python 3 + `uv` (`google-genai`) |
| **`gcommit`** | 根據 Git 暫存區內容 (`git diff --cached`) 自動生成符合 Conventional Commits 規範的 Commit Message，並支援終端機預覽與即時編輯。 | Zsh + `genai` |

---

## 📋 前置需求 (Prerequisites)

在使用 `yd-bin` 前，請確保您的開發環境已安裝以下基礎工具：

1. **Zsh Shell**（macOS 預設，Linux 可自行安裝）
2. **Git**
3. **Python 3.8+**
4. **`uv`（Python 快速包管理器與腳本執行器）**
   > 💡 `genai` 使用 PEP 723 (Inline Script Metadata)，需要 `uv` 來自動管理與執行依賴，無需手動建立 `venv` 或執行 `pip install`。

### 📦 `uv` 安裝方式（若尚未安裝）

如果您尚未安裝 `uv`，請根據您的系統執行以下指令安裝：

- **macOS / Linux**:
  ```bash
  curl -LsSf https://astral.sh/uv/install.sh | sh
  ```
- **Homebrew (macOS)**:
  ```bash
  brew install uv
  ```

安裝完成後，建議執行 `uv --version` 確認是否安裝成功。

---

## 🚀 設定指南 (Setup)

### 1. Clone 本專案

將專案下載至您的個人目錄（例如 `~/projects/yd-bin`）：

```bash
git clone https://github.com/your-username/yd-bin.git ~/projects/yd-bin
```

### 2. 設定環境變數

編輯您的 Shell 設定檔（例如 `~/.zshrc` 或 `~/.bashrc`）：

```bash
# 1. 設定 Gemini API Key (可至 Google AI Studio 免費申請)
export GEMINI_API_KEY="your_gemini_api_key_here"

# 2. 將 yd-bin/bin 目錄加入系統 PATH
export PATH="$HOME/projects/yd-bin/bin:$PATH"
```

### 3. 載入最新設定與賦予執行權限

```bash
# 重新載入 shell 設定
source ~/.zshrc

# 確保 bin 目錄下的指令具有執行權限
chmod +x ~/projects/yd-bin/bin/*
```

---

## 📖 使用說明 (Usage)

### 1. `genai` — 終端機 Gemini AI 工具

您可以直接傳入問題，或是將文字檔/指令輸出透過 Pipe 傳給 `genai`。

* **單純提問**：
  ```bash
  genai "請用簡單的語言解釋什麼是 RESTful API"
  ```

* **使用 Pipe 串接**：
  ```bash
  cat error.log | genai "請幫我分析這段錯誤 Log 的原因與解法"
  ```
  ```bash
  git diff | genai "請幫我摘要這些變更重點"
  ```

---

### 2. `gcommit` — 自動化 Git Commit 工具

在 Git 儲存庫中變更程式碼後，使用 `gcommit` 即可快速生成 Commit Message。

1. **暫存程式碼變更**：
   ```bash
   git add .
   ```

2. **執行 `gcommit`**：
   ```bash
   gcommit
   ```

3. **互動流程**：
   - 選擇 Commit 類型（`feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`, `auto` 等，預設為 `auto`）。
   - 系統將會調用 `genai` 分析 `git diff --cached`，生成繁體中文 Commit Message 並顯示預覽。
   - **編輯與提交**：
     - 若滿意生成內容：直接按 **Enter** 即可完成 `git commit`。
     - 若欲修改內容：直接在 Console 編輯顯示的文字後按下 **Enter** 提交。

---

## ⚙️ 技術架構細節

* **極速執行**：`genai` 預設採用 `gemini-3.5-flash-lite` 模型，專為高頻率、低延遲的 CLI 操作設計。
* **零污染環境依賴**：透過 `uv run --script` 宣告獨立依賴（`google-genai`），在首次執行時會自動下載相應 Package 於快取區，不影響全域 Python 環境。
* **互動編輯**：`gcommit` 利用 Zsh 內建的 `vared` 指令，允許使用者在終端機內進行多行文字微調與修正。
