---
title: 員工離職預測專案企劃書
---

# 員工離職預測專案企劃書

[← 回到互動儀表板](./) ・ [分析報告](EDA_Report.html) ・ [下載 PDF](docs/員工離職預測專案企劃書.pdf) ・ [下載 Word](docs/員工離職預測專案企劃書.docx)

## 1. 主題與方向

本專案用機器學習預測「哪位員工可能離職、為什麼」，讓 HR 在員工提離職前，把有限的留才資源集中在最可能離開、而且留得住的人身上。

**要解決的問題**：員工離職的成本很高。業界常用的估法是，替換一位員工約需年薪的 50%（招募、培訓、產能損失）。但 HR 通常要等到員工遞辭呈才知道，來不及介入；就算想主動關懷，也不知道該先找誰、該談什麼。

**專案目標**

- **預測**：替每位員工算出離職風險機率，分成高、中、低三級。
- **解釋**：用 SHAP 說明每個人被列為高風險的原因，並區分 HR 能不能介入。
- **行動**：把分析結果做成互動儀表板，讓 HR 和主管可以查名單、看原因、試算介入方案的效益。

**成功指標**：高風險名單能涵蓋 70% 以上的實際離職者；模型 ROC-AUC ≥ 0.80；同時排除性別、年齡、婚姻等敏感特徵，確保公平。

## 2. 資料集選擇

使用 Kaggle 上的 [IBM HR Analytics Employee Attrition & Performance](https://www.kaggle.com/datasets/pavansubhasht/ibm-hr-analytics-attrition-dataset)（上傳者 pavansubhasht）。這份資料是 IBM 資料科學家做的虛構資料，共 1,470 位員工、35 個欄位，沒有缺失值。目標變數 `Attrition` 中有 237 人離職，佔 16.1%，屬於類別不平衡。

| 類別 | 主要欄位 |
| --- | --- |
| 個人背景 | Age、Gender、MaritalStatus、Education、EducationField、DistanceFromHome |
| 職務 | Department、JobRole、JobLevel、OverTime、BusinessTravel |
| 薪酬與激勵 | MonthlyIncome、PercentSalaryHike、StockOptionLevel |
| 年資 | TotalWorkingYears、YearsAtCompany、YearsInCurrentRole、YearsSinceLastPromotion、NumCompaniesWorked |
| 滿意度問卷（1–4 分） | JobSatisfaction、EnvironmentSatisfaction、WorkLifeBalance、JobInvolvement、RelationshipSatisfaction |
| 目標變數 | Attrition（Yes / No） |

限制：這是虛構的單一時間點資料，適合驗證方法、做概念驗證；要實際導入，需要公司自己的每月 HR 資料。

## 3. 專案流程規劃

**(1) EDA 探索性資料分析**

- 資料品質：缺失值、重複列、常數欄位、類別不平衡。
- 單變量分析：類別變數做卡方檢定（以 Cramér's V 排序），數值變數做 Mann-Whitney U 檢定，比較離職與留任者的差異。
- 交叉分析：加班 × 職級、年資帶、年齡帶、收入四分位，找出高風險族群。
- 初步發現：加班者離職率 30.5%（沒加班 10.4%）；年資 0–1 年 34.9%；Sales Representative 39.8%；績效與調薪幅度與離職無顯著關係。

**(2) 資料前處理與特徵工程**

- 刪除常數欄位（EmployeeCount、Over18、StandardHours）與 ID。
- 檢查共線性（VIF），移除高度重疊的欄位，例如 JobLevel 與 MonthlyIncome 相關係數 0.95。
- 移除敏感特徵（性別、年齡、婚姻）；認股等級轉成「有／沒有」二元變數。
- 類別變數 One-Hot 編碼、數值變數標準化；用 class_weight 處理類別不平衡。

**(3) 模型選擇**（5-fold 分層交叉驗證）

| 模型 | 角色 | ROC-AUC |
| --- | --- | --- |
| Logistic Regression | 主模型：可解釋性高 | 0.821 |
| Random Forest | 非線性對照 | 0.815 |
| Gradient Boosting | 非線性對照 | 0.799 |

評估指標用 ROC-AUC、PR-AUC、Recall 與 F1，不用 Accuracy，因為類別不平衡會讓 Accuracy 失真。解釋模型用 SHAP。

**(4) 預期輸出**

- 每位員工的離職風險機率與分級（高／中／低）。
- 每位員工前 3 個「HR 可介入」的風險原因與 2 個保護因子。
- 全公司的風險驅動因子排序，以及政策建議。

## 4. 預期成果與 Demo

最終成果是一個互動式「員工離職風險儀表板」，搭配一份分析報告與可重複執行的 Python 程式。

| 成果 | 內容 | 使用者 |
| --- | --- | --- |
| 離職熱點分析 | 各族群離職率圖表，可切換職務、加班、年資等維度 | HR 主管、管理層 |
| 風險名單 | 依風險排序的員工清單，可依部門、風險等級篩選 | HRBP、部門主管 |
| SHAP 個人解釋 | 點選員工後顯示風險原因瀑布圖 | HRBP |
| What-if 模擬器 | 調整加班、出差、認股、滿意度等，即時看風險變化 | HRBP、主管 |
| ROI 試算 | 輸入替換成本、介入成本與成功率，算出淨效益 | 管理層 |

**商業價值（示意）**：以替換成本為年薪 50% 計算，這份資料中 237 位離職者的年度離職成本約 680 萬。若只對 472 位高風險員工介入（每人成本 1,000），並留住其中 20% 的實際離職者，淨效益約 41 萬，ROI 約 87%；對全員介入則成本高於效益。數字為假設，實際應以試點驗證。

**後續擴充**：存活分析預測「何時」離職、員工分群找出離職型態、以 A/B 試點驗證介入措施的因果效果。
