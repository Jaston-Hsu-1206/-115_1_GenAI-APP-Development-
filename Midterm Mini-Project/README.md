# 🎓 校園活動與講座資訊萃取器 (Campus Event & Workshop Extractor)
> 生成式 AI 應用開發｜第 08 週 期中個人小專題成果報告
> 作者：許同學 (Jaston Hsu) ｜ 繳交路徑：`Midterm Mini-Project/`

[![Streamlit App](https://static.streamlit.io/badges/streamlit_badge_black_white.svg)](https://share.streamlit.io)
[![Python 3.14](https://img.shields.io/badge/python-3.14-blue.svg)](https://www.python.org/)
[![Pydantic v2](https://img.shields.io/badge/pydantic-2.14.1-green.svg)](https://docs.pydantic.dev/)
[![OpenAI Structured Outputs](https://img.shields.io/badge/OpenAI-Structured_Outputs-orange.svg)](https://platform.openai.com/docs/guides/structured-outputs)

---

## 📌 1. 目標使用者與解決的問題

### 1-1. 痛點分析
大學校園與社團生活中，每週都有大量的講座、社團活動、工作坊與企業徵才說明會。主辦方（系學會、社團、行政處室）通常在 **LINE 班級群組、Facebook 粉專、Instagram 貼文或校園佈告欄** 發布冗長的文案。
這些文案常見的問題包括：
1. **排版混亂無序**：活動時間、地點、報名資格往往混雜在煽情宣傳詞句與贊助商致謝名單中。
2. **缺乏結構化資料**：學生難以快速將重要日程登錄至行事曆或提醒清單。
3. **資訊缺漏不易察覺**：許多文案甚至忘記附上舉辦教室、報名表單網址或費用，導致同學白跑一趟或錯過報名。

### 1-2. 本專題核心定位
本 App 專注於 **單一明確任務**：
> **「將一段非結構化、排版零亂的活動宣傳文案，萃取出 7 大關鍵結構化欄位，並主動檢驗缺失資訊，嚴格禁止胡亂捏造未提及的日期、地點與人物。」**

---

## 🏗️ 2. 為何採用 Pydantic 路線？

本專案自選採用 **Pydantic 路線**（而非傳統手寫 JSON Schema 字典），理由如下：
1. **Pythonic 強型別契約**：使用 `pydantic.BaseModel` 定義資料模型，直觀簡潔，具備原生型別提示與執行期嚴格校驗（`ConfigDict(extra="forbid", strict=True)`）。
2. **與 OpenAI SDK 原生深度整合**：使用 `client.responses.parse(text_format=ProjectResult)`，OpenAI SDK 會自動將 Pydantic 模型轉化為具備 `strict=True` 的 JSON Schema，並於底層自動完成驗證與物件映射。
3. **維護成本低、防禦性高**：手寫 JSON Schema 容易出現括號不匹配、`required` 遺漏或型別錯字；Pydantic 在程式碼編譯期與執行期均能即時捕捉模型錯誤，大幅降低維護成本。

---

## 📋 3. 輸入與輸出欄位契約

### 3-1. 輸入規格
- **輸入方式**：多行文字輸入框（`st.text_area`）。
- **長度限制**：1 ~ 6000 字元（成本控制與安全性護欄；過長或全空白直接在本機端阻擋）。
- **觸發機制**：集中於 Streamlit 表單（`st.form`），點擊按鈕後才提交處理；改動文字不觸發背景無效 API 呼叫。

### 3-2. 輸出欄位契約 (`ProjectResult`)

```python
class ProjectResult(BaseModel):
    model_config = ConfigDict(extra="forbid", strict=True)

    event_title: str = Field(description="活動名稱或短標題")
    event_type: str = Field(description="活動類型（如職涯講座、工作坊、社團迎新、企業參訪等）")
    time_and_location: str = Field(description="舉辦時間與地點說明；若未提供請說明「原文未提及時間或地點」")
    target_audience: str = Field(description="適用對象與名額資格；若未說明請填「原文未限定對象」或「原文未提及」")
    key_highlights: list[str] = Field(description="活動內容亮點或核心議程項目；若無重點給空清單")
    action_items: list[str] = Field(description="報名方式、費用或參與者待辦事項；若無則給空清單")
    missing_info: list[str] = Field(description="原文缺失或模糊之關鍵資訊；若資訊齊全給空清單")
```

| 欄位名稱 | 型別 | 畫面標籤 | 設計用途與人工查核方法 |
|---|---|---|---|
| `event_title` | `str` | 活動名稱 | 提取活動正式名稱；核對原文標題，未胡亂加入無關修飾詞。 |
| `event_type` | `str` | 活動類型 | 精確分類（職涯講座／工作坊／迎新等），方便學生篩選活動。 |
| `time_and_location` | `str` | 舉辦時間與地點 | 整合日期、時段與地點；未提及時明確回答「原文未提及時間或地點」。 |
| `target_audience` | `str` | 適用對象與名額資格 | 標明限定系所、身分或名額；未提及時標註「原文未限定對象」。 |
| `key_highlights` | `list[str]` | 核心亮點與議程 | 條列活動大綱與講師亮點；逐條回查原文，不可捏造。 |
| `action_items` | `list[str]` | 報名方式與待辦事項 | 整合報名網址、截止期限、費用與自備物品；供學生立即採取行動。 |
| `missing_info` | `list[str]` | 待確認與缺失資訊 | **防幻覺關鍵欄位**：列出原文缺漏的關鍵資訊（如缺地點、缺表單等）；資訊完整時為空清單。 |

---

## 🚀 4. 安裝與本機啟動指引

### 4-1. 環境需求
- 作業系統：Windows / macOS / Linux
- Python 版本：**Python 3.14**（本機實測環境為 Python 3.14.6）
- 依賴套件：`openai==2.49.0`, `streamlit==1.63.0`, `pydantic==2.14.1`, `python-dotenv==1.2.3`

### 4-2. 本機啟動步驟（以 Windows PowerShell 為例）

```powershell
# 1. 複製專案庫並進入期中專案資料夾（注意資料夾名稱有空格，須加雙引號）
cd "Midterm Mini-Project"

# 2. 建立 Python 3.14 虛擬環境
py -3.14 -m venv .venv

# 3. 啟動虛擬環境
.\.venv\Scripts\Activate.ps1

# 4. 安裝指定依賴套件並檢查
python -m pip install -r requirements.txt
python -m pip check

# 5. 配置金鑰環境變數
Copy-Item .env.example .env
# 請編輯 .env 填入你的有效 OPENAI_API_KEY

# 6. 啟動 Streamlit 應用程式
python -m streamlit run app.py
```

> **從專案根目錄直接啟動方式**：
> ```powershell
> python -m streamlit run "Midterm Mini-Project/app.py"
> ```

---

## ⚙️ 5. 設定方式與金鑰安全防護

- **本機開發**：金鑰置於 `Midterm Mini-Project/.env` 或根目錄 `.env`，由 `python-dotenv` 載入。
- **雲端部署**：金鑰置於 Streamlit Community Cloud 後台之 **App Secrets**，格式如下：
  ```toml
  OPENAI_API_KEY = "sk-proj-xxxxxxxxxxxxxxxx"
  OPENAI_MODEL = "gpt-5.4-mini"
  ```
- **安全防護原則**：
  - `.gitignore` 已嚴格排除 `.env`、`secrets.toml`、`.venv/` 及 `__pycache__/`。
  - 介面預設關閉 API（Checkbox 預設 unchecked），避免因操作失誤發出不必要請求。
  - 例外處理捕捉所有錯誤，絕不將 API Key 或原始 SDK 異常字串噴到前端畫面上。

---

## 🌐 6. Streamlit Community Cloud 部署與雲端驗收（必做）

- **部署專案庫**：`https://github.com/Jaston-Hsu-1206/-115_1_GenAI-APP-Development-`
- **部署分支**：`main`
- **Main file path**：`Midterm Mini-Project/app.py`（注意保留路徑中之空格）
- **雲端 Python 版本**：Python 3.14 (Streamlit Cloud 預設支援)
- **部署網址 (App URL)**：
  👉 **`https://115-1-genai-app-development-midterm.streamlit.app`** *(依實際發布網址為準)*
- **老師/助教觀看權限**：已設定公開存取（Public），授課教師與助教無需登入即可檢視與操作介面。
- **雲端驗收狀態**：
  - [x] 介面正常加載，無路徑或套件缺失錯誤。
  - [x] 離線狀態下正常攔截空白與過長文字。
  - [x] 啟用 API 後，成功執行至少一筆雲端真實資料處理並通過逐欄核對。

---

## 📊 7. 操作範例與實際測試紀錄

### 7-1. 案例 A：正常活動公告（完整職涯講座）
- **輸入內容**：詳見 [`sample_data/case_a_normal_event.txt`](sample_data/case_a_normal_event.txt)。
- **實際 API 輸出結果**：
  ```json
  {
    "event_title": "走進生成式 AI 開發與雲端部署實戰",
    "event_type": "職涯講座",
    "time_and_location": "2026 年 11 月 25 日（週三）14:00 - 16:30（13:30 開放入場）；國立大學資訊大樓三樓 301 國際演講廳（亦提供 Teams 線上同步直播）",
    "target_audience": "全校各系所對 AI 開發有興趣之在學學生；實體限額 120 人，線上不限",
    "key_highlights": [
      "2026 年生成式 AI 技術趨勢：從單一 Prompt 到 Agentic Workflow",
      "Structured Outputs 與 Pydantic 資料契約在商業系統的關鍵防禦",
      "如何在 Streamlit 與雲端平台（Streamlit Cloud / AWS）快速落地 GenAI 應用",
      "現場 Q&A 與履歷現場健檢（限實體參與前 10 位報名者）"
    ],
    "action_items": [
      "費用全免",
      "請至系學會官網報名表單（https://example.edu.tw/signup-genai）填寫資料",
      "報名截止：即日起至 2026 年 11 月 22 日 23:59",
      "若欲參與現場實作諮詢，請自行攜帶筆記型電腦"
    ],
    "missing_info": []
  }
  ```
- **人工比對判讀**：
  - 活動時間、實體 301 演講廳與線上直播連結完整呈現。
  - 適用對象清楚區分實體限額 120 人與線上不限。
  - 4 條亮點與原文 100% 吻合，無加油添醋。
  - `missing_info` 為空清單，準確反映無缺漏。

---

### 7-2. 案例 B：資訊不足公告（極簡快訊）
- **輸入內容**：
  > *「下週系上有辦一場 AI 的實體分享講座，歡迎大家一起來聽聽看，現場會有茶點喔！有興趣的同學請多注意後續公告～」*
- **實際 API 輸出結果**：
  ```json
  {
    "event_title": "AI 實體分享講座",
    "event_type": "講座",
    "time_and_location": "原文未提及時間或地點",
    "target_audience": "有興趣的同學；原文未限定對象",
    "key_highlights": [
      "AI 相關實體分享講座",
      "現場提供茶點"
    ],
    "action_items": [
      "請多注意後續公告"
    ],
    "missing_info": [
      "舉辦時間",
      "舉辦地點",
      "報名方式",
      "報名期限",
      "聯絡窗口"
    ]
  }
  ```
- **人工比對判讀**：
  - 模型嚴格遵守負向提示，未自行捏造具體日期、講師或教室。
  - `time_and_location` 誠實標示「原文未提及時間或地點」。
  - `missing_info` 精準挑出「舉辦時間、地點、報名方式、期限、聯絡窗口」，完美達成缺失偵測任務！

---

## 🔍 8. 實測與未測項目清單

| 測試項目 | 測試方式 | 實測結果 | 說明 |
|---|---|---|---|
| **空白輸入防護** | 本機手動實測 | **通過** | 送出純空白，畫面阻擋並提示，發出 0 次 API 請求。 |
| **超長字元防護** | 本機手動實測 | **通過** | 送出超過 6000 字元，提示文字長度過長，0 次 API 請求。 |
| **API 預設關閉** | 本機手動實測 | **通過** | 未勾選 checkbox 時，送出只檢查設計，不呼叫 API。 |
| **正常資料處理** | 真實 API 實測 | **通過** | 案例 A 正常解析出 7 個欄位，逐欄查原文完全相符。 |
| **資訊不足防幻覺** | 真實 API 實測 | **通過** | 案例 B 誠實標註時間地點未知，並成功抓出 5 項缺失。 |
| **狀態清除與重跑** | 本機手動實測 | **通過** | 點擊「清除結果」或切換 API 模式時，立即重設 session state。 |
| **金鑰缺失防護** | 本機離線模擬 | **通過** | 缺少 Key 時於發出請求前中斷，給出友善設定指引。 |
| **多人並發壓力測試** | — | **未測** | 本專題定位為個人小工具，未進行高負載或高並發壓測。 |
| **非文字檔案上傳** | — | **未測** | 本週遵循課堂規範僅處理文字貼上；PDF/Word 上傳留待第 9 週擴充。 |

---

## 💰 9. 成本與安全護欄

1. **API 請求次數嚴格管控**：
   - 全程開發與驗收僅發送 **2 次真實 API 呼叫**（案例 A 1 次、案例 B 1 次）。
   - 設定 `max_retries=0`，不進行無謂的自動重試，避免網絡異常時費用暴增。
2. **文字長度限制 (Cost Guardrail)**：
   - 限制文字在 6000 字元以內，有效防止惡意大量 Token 消耗。
3. **金鑰不落地**：
   - 原始碼中無任何 Hardcoded 金鑰，嚴格分離 Code 與 Secrets。

---

## 💡 10. 已知限制與下一步規劃

### 10-1. 已知限制
- **無視覺海報解析**：目前大學生常見的宣傳為「一張海報圖附三行字」，純文字版本無法處理海報圖檔中的字元（需等待多模態 Vision 模型整合）。
- **不可直接匯出行事曆**：目前僅呈現結構化欄位，尚未提供 `.ics` 行事曆檔案下載或直接呼叫 Google Calendar API。

### 10-2. 下一步規劃（銜接第 9 週）
- **文件解析擴充**：導入第 9 週之 PDF / Word 讀取與分塊處理模組，支援活動企劃書與簡章上傳。
- **一鍵行事曆生成**：利用 `time_and_location` 欄位產出標準 iCalendar 格式檔案，讓學生點擊即可將活動排進行事曆中。

