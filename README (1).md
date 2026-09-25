# TradingAgent-Taiwan-version：支援台股的多代理人 LLM 交易分析框架

> 本專案 fork 並修改自 [TauricResearch/TradingAgents](https://github.com/TauricResearch/TradingAgents)（Apache-2.0 授權），新增**台股（上市 / 上櫃）支援**與**回測模式**。非原創框架，核心多代理人架構、研究方法與原始程式碼版權皆屬原作者所有，本 fork 僅新增台股在地化功能，詳見下方「這個 fork 改了什麼」。

[![License](https://img.shields.io/badge/license-Apache--2.0-blue)](LICENSE)
[![Based on](https://img.shields.io/badge/based%20on-TauricResearch%2FTradingAgents-black?logo=github)](https://github.com/TauricResearch/TradingAgents)

> ⚠️ 下方「TradingAgents Framework」以後的內容為原專案的英文說明，保留供對照參考；圖片連結請改回指向原專案（`https://github.com/TauricResearch/TradingAgents/raw/main/assets/...`），除非你已經把 `assets/`、`tradingagents/`、`cli/` 等資料夾一併複製進本 repo，否則這些圖會是破圖。

---

## 這個 fork 改了什麼

原專案已經支援中國 A 股 / 港股等市場，這個版本延續同樣的架構模式，把台股（上市 `.TW`、上櫃 `.TWO`）也接進去，並補上回測功能，方便驗證分析報告的實際準確度。

### 1. 台股資料源判斷與接入

台股在 Yahoo Finance 的代號格式是 `2330.TW`（上市）或 `6505.TWO`（上櫃）。修改的檔案與資料流：

```
interface.py           ← 統一入口，加入台股（.TW / .TWO）判斷邏輯
      ↓
y_finance.py            ← 股價、技術指標
alpha_vantage_*.py      ← 基本面、新聞資料
yfinance_news.py        ← 新聞資料
      ↓
agents/*.py             ← 各分析師 Prompt，加入中文語境與台股背景描述
```

- `interface.py`：新增市場判斷，辨識代號結尾是否為 `.TW` / `.TWO`，自動導向對應的資料處理流程。
- `y_finance.py`：沿用 Yahoo Finance 取得台股股價與技術指標。
- `alpha_vantage_*.py`、`yfinance_news.py`：取得基本面與新聞資訊（部分資料源在台股的覆蓋率有限，屬已知限制）。
- `agents/*.py`：各分析師（基本面 / 情緒 / 新聞 / 技術）Prompt 補上中文與台股市場的上下文，讓 LLM 輸出的分析更貼近台股語境。

介面操作方式與原版**完全相同**，使用者不需要額外學習新的指令，只要輸入台股代號即可。

### 2. 回測模式

新增回測模式，可選擇回測的起訖日期，检验策略在歷史區間內的報酬表現，用於評估分析結果的實際準確度（而不只是看單次報告）。

### Demo

- 美股（原版）：`NVDA`、`AMZN`、`GOOG`
- 台股（本 fork 新增）：`2330.TW`（台積電）、`2885.TW`（元大金）

---

## 專案簡介（原版）

TradingAgents 是一個模擬真實交易公司運作方式的多代理人交易框架。透過部署多個專責的 LLM 代理人：

- **分析團隊（Analyst Team）**：基本面分析師、情緒分析師、新聞分析師、技術分析師
- **研究團隊（Researcher Team）**：多方 / 空方研究員互相辯論，權衡潛在收益與風險
- **交易員代理（Trader Agent）**：整合前述報告，做出交易時機與部位大小的判斷
- **風險管理團隊（Risk Management Team）**：持續評估市場波動性、流動性等風險因子，並將建議交給投資組合經理人做最終決策

> ⚠️ 本框架僅供研究用途。實際交易表現會受所選 LLM、模型溫度、交易期間、資料品質等多重非決定性因素影響，**不構成任何財務、投資或交易建議**。

---

## 安裝

```bash
git clone https://github.com/<your-username>/TradingAgents-TW.git
cd TradingAgents-TW
```

建立虛擬環境：

```bash
conda create -n tradingagents-tw python=3.12
conda activate tradingagents-tw
```

安裝套件與相依項目：

```bash
pip install .
```

### 所需 API Key

| 用途 | 環境變數 | 說明 |
|---|---|---|
| 金融數據 | `FINNHUB_API_KEY` | Finnhub（免費額度即可使用） |
| LLM（擇一或多個） | `OPENAI_API_KEY` | OpenAI，建議預先儲值約 USD $30 供測試 |
| | `GOOGLE_API_KEY` | Google Gemini |
| | `ANTHROPIC_API_KEY` | Anthropic Claude |
| | `DEEPSEEK_API_KEY` | DeepSeek |

也可以直接複製 `.env.example` 為 `.env` 並填入金鑰：

```bash
cp .env.example .env
```

其餘 Provider（xAI、Qwen、GLM、MiniMax、OpenRouter、Ollama、Azure、Bedrock 等）設定方式與原專案相同，詳見原始 README 或 `tradingagents/default_config.py`。

---

## 新手安裝教學（含 Windows / Mac 逐步操作）

> 本教學參考並改寫自 YouTube 教學影片：[https://www.youtube.com/watch?v=0vcxNINMOBQ](https://www.youtube.com/watch?v=0vcxNINMOBQ)。原影片示範的是官方原版 TradingAgents 的安裝流程，這裡把 repo 網址換成本專案（台股版），並補充了台股代號範例；其餘操作步驟大致相同，適合完全沒用過 Git / Conda 的新手照著做。

### 第一次使用

1. 先下載安裝好以下工具：[Python](https://www.python.org/)、[Git](https://git-scm.com/)、[Anaconda](https://www.anaconda.com/)、[VS Code](https://code.visualstudio.com/)。
2. 在桌面新增一個空資料夾，準備放置專案。
3. 打開 **Anaconda Prompt**（終端機）。
4. 切換到剛剛建立的資料夾：

   ```bash
   cd 你的資料夾路徑
   ```

5. Clone 本專案（台股版，取代原影片中的官方 repo 網址）：

   ```bash
   git clone https://github.com/Anouo1023/TradingAgent-Taiwan-version.git
   cd TradingAgent-Taiwan-version
   ```

6. 建立並啟用虛擬環境：

   ```bash
   conda create -n tradingagents python=3.13
   conda activate tradingagents
   ```

7. 安裝套件相依項目：

   ```bash
   pip install -r requirements.txt
   ```

8. 申請必要的 API Key：
   - 到 [Finnhub](https://finnhub.io/) 註冊，取得 `FINNHUB_API_KEY`（金融數據，免費額度即可）。
   - 到 [OpenAI](https://platform.openai.com/) 註冊並儲值（建議先儲值約 USD $5～$30 供測試）取得 `OPENAI_API_KEY`；也可以改用 Google Gemini 或 Anthropic Claude 的 API Key。

9. 設定環境變數（把下面的 `YOUR_FINNHUB_API_KEY`、`YOUR_OPENAI_API_KEY` 換成你剛剛申請到的金鑰）：

   **macOS / Linux**
   ```bash
   export FINNHUB_API_KEY=YOUR_FINNHUB_API_KEY
   export OPENAI_API_KEY=YOUR_OPENAI_API_KEY
   ```

   **Windows（Anaconda Prompt）**
   ```bash
   set FINNHUB_API_KEY=YOUR_FINNHUB_API_KEY
   set OPENAI_API_KEY=YOUR_OPENAI_API_KEY
   ```

10. （Windows 常見的中文編碼問題）用 VS Code 打開專案，用左側搜尋功能找 `with open`，把檔案輸出相關的幾處 `open(...)` 補上 `encoding='utf-8'`，例如：

    ```python
    with open('output.txt', 'w', encoding='utf-8') as f:
    ```

    Windows 也建議額外設定：

    ```bash
    set PYTHONUTF8=1
    ```

11. 啟動程式：

    ```bash
    python -m cli.main
    ```

    看到互動式選單出現就代表安裝成功。

12. 依照畫面提示輸入：
    - 股票代碼（美股如 `NVDA`；**台股請輸入 `2330.TW`（上市）或 `6505.TWO`（上櫃）格式**）
    - 分析日期
    - 想使用的分析師與 LLM 模型

13. 等待執行完成，即可看到完整的投資分析報告。

### 第二次之後使用

不需要重新 clone 或安裝套件，每次只要：

```bash
cd 你的資料夾路徑/TradingAgent-Taiwan-version
conda activate tradingagents
```

接著重新設定 API Key（同步驟 9，終端機重開後環境變數會重置）：

**macOS / Linux**
```bash
export FINNHUB_API_KEY=YOUR_FINNHUB_API_KEY
export OPENAI_API_KEY=YOUR_OPENAI_API_KEY
```

**Windows**
```bash
set FINNHUB_API_KEY=YOUR_FINNHUB_API_KEY
set OPENAI_API_KEY=YOUR_OPENAI_API_KEY
set PYTHONUTF8=1
```

最後啟動：

```bash
python -m cli.main
```

> 💡 小提醒：把 API Key 直接寫在終端機指令中，重開機或關掉終端機後就會消失，需要重新輸入。若不想每次都手動設定，可以改用前面「安裝」章節提到的 `.env` 檔案方式（`cp .env.example .env` 後填入金鑰），程式會自動讀取，不需要每次手動 `export` / `set`。

---

## 使用方式

### CLI

```bash
tradingagents          # 已安裝套件時可直接呼叫
python -m cli.main     # 或直接從原始碼執行
```

啟動後選擇股票代號、分析日期、LLM Provider、研究深度等參數即可。

**支援的代號格式**

- 美股：`NVDA`、`AAPL`
- 台股上市：`2330.TW`（台積電）
- 台股上櫃：`6505.TWO`
- 中國 A 股 / 港股等：沿用原專案支援

### 回測模式

在 CLI 或設定中選擇回測模式，指定回測的開始與結束日期，即可檢視策略在該區間的模擬報酬，用來檢驗分析報告的實際準確度。

### Python 套件用法

```python
from tradingagents.graph.trading_graph import TradingAgentsGraph
from tradingagents.default_config import DEFAULT_CONFIG

ta = TradingAgentsGraph(debug=True, config=DEFAULT_CONFIG.copy())

# 以台股代號進行分析
_, decision = ta.propagate("2330.TW", "2026-01-15")
print(decision)
```

其餘設定方式（切換 LLM Provider、調整辯論輪數等）與原專案一致，請參考 `tradingagents/default_config.py`。

---

## 已知限制

- 部分基本面 / 新聞資料源（如 Alpha Vantage）對台股的覆蓋率不如美股完整，分析品質可能受限於資料可得性。
- 本框架具有 LLM 驅動的非決定性特性，同一標的、同一日期的兩次執行結果可能不完全相同，屬預期行為而非錯誤，詳見原專案 README 的 Reproducibility 章節。
- 回測結果不保證reproduce特定數字，僅供研究參考，不構成投資建議。

---

## 致謝與授權

本專案基於 [TauricResearch/TradingAgents](https://github.com/TauricResearch/TradingAgents)（Apache-2.0 授權）修改而成，核心多代理人架構、研究方法與原始程式碼版權皆屬原作者所有。本 fork 僅新增台股資料接入與回測功能，並依照 Apache-2.0 授權條款釋出。

若你也在研究中使用到本專案，請一併引用原始論文：

```bibtex
@misc{xiao2025tradingagentsmultiagentsllmfinancial,
      title={TradingAgents: Multi-Agents LLM Financial Trading Framework},
      author={Yijia Xiao and Edward Sun and Di Luo and Wei Wang},
      year={2025},
      eprint={2412.20138},
      archivePrefix={arXiv},
      primaryClass={q-fin.TR},
      url={https://arxiv.org/abs/2412.20138},
}
```

- 原專案：https://github.com/TauricResearch/TradingAgents
- 原始論文：https://arxiv.org/abs/2412.20138
- License：[Apache-2.0](LICENSE)（沿用原專案授權條款）
