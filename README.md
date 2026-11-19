# IBM HR 員工離職預測專案

用 Kaggle 的 IBM HR Analytics 資料集預測員工離職：EDA、Logistic 迴歸模型、SHAP 個人解釋，以及可以直接在瀏覽器操作的互動儀表板。

- **互動儀表板**：`index.html`（GitHub Pages 首頁）
- **分析報告**：[EDA_Report.md](EDA_Report.md)
- **專案企劃書**：[PROPOSAL.md](PROPOSAL.md)

## 資料夾結構
| 路徑 | 內容 |
|---|---|
| `index.html` | 互動儀表板（讀取 `data/` 和 `outputs/dashboard/` 的資料） |
| `data/WA_Fn-UseC_-HR-Employee-Attrition.csv` | 原始資料（IBM 虛構資料，1,470 列 × 35 欄） |
| `scripts/01_eda.py` | EDA：卡方檢定、Mann-Whitney U 檢定、交叉分析、圖表 |
| `scripts/02_model.py` | 基準模型：Logistic / Random Forest / Gradient Boosting |
| `scripts/03_shap.py` | 最終模型（精簡特徵、無敏感特徵）＋ SHAP 解釋＋儀表板資料匯出 |
| `outputs/figs/` | 所有圖表 |
| `outputs/employee_risk_reasons.csv` | 每位員工的風險分數與前 3 大原因 |
| `outputs/dashboard/` | 儀表板用的 JSON 資料 |
| `_config.yml` | GitHub Pages 設定（讓 .md 報告轉成網頁） |

## 放上 GitHub Pages（手動上傳）
1. 在 GitHub 右上角按 **+** → **New repository**，名稱例如 `ibm-hr-attrition-dashboard`，選 **Public**，不要勾選「Add a README」，按 **Create repository**。
2. 在新 repo 頁面點 **uploading an existing file**，把這個資料夾**裡面的所有檔案和資料夾**拖進去（不是拖整個外層資料夾），按 **Commit changes**。
3. 到 repo 的 **Settings** → **Pages**：Source 選 **Deploy from a branch**，Branch 選 **main**、資料夾選 **/(root)**，按 **Save**。
4. 等 1–2 分鐘，網址會是 `https://<你的帳號>.github.io/<repo 名稱>/`。

## 在本機打開儀表板
儀表板需要透過網址讀取資料，直接雙擊 `index.html` 不會顯示資料。在這個資料夾開 PowerShell 執行：

```powershell
python -m http.server 8000
```

再用瀏覽器打開 http://localhost:8000 。

## 重跑分析
需要 Python 3.12（3.10 以上應該也可以）：

```powershell
python -m venv .venv
.\.venv\Scripts\python -m pip install -r requirements.txt
.\.venv\Scripts\python scripts/01_eda.py data/WA_Fn-UseC_-HR-Employee-Attrition.csv outputs
.\.venv\Scripts\python scripts/02_model.py data/WA_Fn-UseC_-HR-Employee-Attrition.csv outputs
.\.venv\Scripts\python scripts/03_shap.py data/WA_Fn-UseC_-HR-Employee-Attrition.csv outputs
```

三支腳本要依序執行，因為 03 會讀取 02 產出的 `outputs/model_results.json`。重跑後 `outputs/dashboard/` 會更新，儀表板也會自動顯示新的結果。

## 資料來源
[IBM HR Analytics Employee Attrition & Performance](https://www.kaggle.com/datasets/pavansubhasht/ibm-hr-analytics-attrition-dataset)（Kaggle，pavansubhasht）。這是 IBM 做的虛構資料，只適合學習和做概念驗證。
