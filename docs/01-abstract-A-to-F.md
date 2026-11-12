# 01. Abstract A–F（重構版）

> 專題：以 TurtleBot3 Burger 為平台之室內導航與任務決策 AI Agent（第 7 組）
> 原始專題 repo：<https://github.com/hank050392/tb3-navigation-agent>
> 狀態：提案階段。下列「預期」皆為待驗證假設，不是實驗結果。

## 重構重點（相對期中版）

| 期中版問題 | 本次重構 |
|---|---|
| A 段以「延遲」開場，與後文主軸（可靠派送與回報）不一致 | 改以「語言合理 ≠ 動作可行」為主軸，A→F 一條線 |
| B 段只列兩個挑戰 | 明確列出三個失效模式：歧義目標、狀態失效、無依據回報 |
| C 段只寫「讓使用者以中文指定地點」 | 點出核心設計原則：**LLM 只做理解與協調；導航、驗證、回報由可稽核工具負責** |
| E 段指標散列 | 對應 RQ1–RQ3，各有主要指標 |
| F 段只有「預期減少」 | 拆成可交付產出 + 學術／工程貢獻 |

---

## A. Attention Getter：為什麼重要

大型語言模型（LLM）讓「用一句話指揮機器人」成為可能，ROS 2 生態也已出現多種語言介面框架 [3][14]。但對移動機器人而言，一句話被「聽懂」不等於能被「安全且正確地執行」：語言上合理的計畫，未必符合機器人當下的能力與狀態 [2]。在實驗室裡，新手使用 TurtleBot3 Burger 時真正卡住的，往往不是路徑規劃本身，而是「要去哪個門口？」「定位還準嗎？」「它說到了，真的到了嗎？」。

## B. But：現有做法的三個失效模式

把語言直接轉成導航目標（text-to-goal），在本專題場景中至少有三種風險：

1. **歧義或不存在的目標被派送**：「去門口」可能對應多個已登錄地點；LLM 也可能產生地圖上不存在的目標。
2. **在狀態失效時仍然出發**：雷射資料過期、TF 中斷、已有任務執行中，仍可能送出新目標。
3. **回報沒有執行依據**：任務結束（取消、逾時、失敗）被誤報為「到達」，或取消請求被接受就宣稱已停止。

Nav2 能解決導航執行 [4]，但不處理中文指令歧義；LLM 能解析語言，但不能把語言合理性當作動作可行性的證據 [2]。

## C. Cure：本專題的解法

提出一個 **證據導向（evidence-grounded）的室內導航任務 Agent**。核心原則是：

> **LLM 負責理解與協調；地點白名單、狀態檢查、Nav2 執行與事件紀錄負責判斷與證據。**

LLM 只能輸出符合 schema 的 intent（navigate／cancel／clarify／reject）與已登錄地點 ID，不能產生座標、輪速或修改參數。Agent 在歧義時先問、在狀態不合格時拒絕派送、只依 Nav2 action 結果回報終態。

## D. Development：方法與架構

八模組架構：① 使用者情境 → ② 中文輸入與結構化解析 → ③ 任務 Agent／Planner → ④ 資料取得（地點表、地圖、定位、雷射、TF、action 回饋）→ ⑤ 分析引擎（資料時效、進度、延遲）→ ⑥ 領域知識與驗證（白名單、車身與地圖規則、逾時、有限重試）→ ⑦ 決策與導航工具（Nav2 派送／取消）→ ⑧ 結果回報與澄清互動。

關鍵機制：

- **Schema + 白名單**：6 個受限工具（`list_landmarks`、`get_robot_status`、`get_path`、`navigate_to_landmark`、`cancel_navigation`、`read_mission_result`）。
- **Clarification Loop**：地點歧義時回傳 `CLARIFY`，不派送。
- **Readiness Gate**：定位、雷射、TF 過期或已有任務時回傳 `NOT_READY`／`BUSY`。
- **Evidence-based Reporting**：終態只能是 `ARRIVED／FAILED／CANCELED／TIMEOUT／NO_PATH／FAULT`，且每筆附 action 結果與 evidence ID。
- **分層安全**：低延遲停止由 Nav2 與底層機制負責，不等待 LLM。

## E. Experiments：評估設計

- **資料**：60 筆繁體中文指令（明確、歧義、未知、取消、複合、越權／矛盾各 10 筆），依語意模板分組切成 20 開發／40 鎖定測試。
- **場景**：空曠、窄通道、新增障礙物、狀態異常／不可達四類；每類 2 組起終點 × 5 次 = 每方法 40 次（模擬）；實機 12 配對案例 × 3 方法。
- **比較**：B0 固定地點選單、B1 直接語言轉目標、B2 完整 Agent；消融 A 去澄清、消融 B 去狀態檢查。Nav2 版本、地圖、參數全部固定。
- **研究問題與主要指標**：
  - RQ1 澄清是否降低歧義／未知目標的**不當派送率**？
  - RQ2 狀態檢查是否降低故障情境下的**不當派送率**？
  - RQ3 依事件回報是否提高**回報正確率**？代價是多少**延遲（p50/p95）**？
- 比例報 Wilson 95% 信賴區間；失敗與逾時全部保留。

## F. Findings：預期產出與貢獻

**可交付產出**

1. 一套經 schema 與白名單驗證的中文導航任務介面。
2. 整合定位、感測、TF 時效檢查的任務管理器。
3. 以 Nav2 action 結果為依據的事件紀錄與回報格式。
4. 60 筆標註中文指令集與可重現的評估腳本。

**預期貢獻（待驗證）**

- 證明在**固定 Nav2 底層**下，任務層 Agent 的增益可以被單獨量測，與導航演算法本身的表現分開。
- 預期 B2 相較 B1 降低不當派送、提高回報正確率，但互動延遲可能增加；所有效益與代價都將如實報告。
- 本專題定位為教學／實驗室場景的系統整合與評估，不宣稱首次提出 LLM 導航。

---

## 約 300 字濃縮摘要（中文）

LLM 讓使用者能以自然語言指揮移動機器人，但語言上合理的指令未必能被安全且正確地執行。將文字直接轉成導航目標，可能派送歧義或不存在的地點、在定位與感測資料失效時仍然出發，並在取消或逾時後誤報到達。本專題以 TurtleBot3 Burger 與 ROS 2／Nav2 為平台，提出證據導向的室內導航任務 Agent：LLM 只輸出符合 schema 的 intent 與已登錄地點 ID，並透過六個白名單工具完成地點查詢、狀態檢查、Nav2 派送與結果讀取；歧義時主動澄清，狀態不合格時拒絕派送，終態只依 action 結果與事件紀錄回報。評估將以 60 筆中文指令與四類導航場景，在固定 Nav2 設定下比較固定選單、直接語言轉目標與完整 Agent，並以兩項消融檢驗澄清與狀態檢查的貢獻，量測不當派送率、任務完成率、回報正確率與延遲。

## IEEE Transactions 風格（英文）

**Title** — EvidenceNav: An Evidence-Grounded LLM Task Agent for Reliable Indoor Navigation on TurtleBot3

**Abstract** — Large language models (LLMs) enable natural-language interfaces for mobile robots, yet a linguistically plausible command is not necessarily an executable one. Directly translating text into navigation goals can dispatch ambiguous or non-existent destinations, start missions while localization or sensor data are stale, and report arrival after a mission was actually canceled or timed out. This paper presents an evidence-grounded task agent for indoor navigation on a TurtleBot3 Burger running ROS 2 and Nav2. The LLM is restricted to producing schema-constrained intents and registered landmark identifiers, and interacts with the robot only through six whitelisted tools for landmark lookup, readiness checking, path feasibility, goal dispatch, cancellation, and result retrieval. The agent asks for clarification when a destination is ambiguous, refuses to dispatch when readiness checks fail, and reports terminal states solely from navigation action results and logged events, while low-latency stopping remains independent of the LLM. The planned evaluation fixes the Nav2 configuration and compares a fixed landmark menu, a direct language-to-goal baseline, and the full agent on 60 annotated Chinese commands and four simulated scenario types, complemented by paired real-robot trials and two ablations. Metrics include inappropriate-dispatch rate, mission completion, report correctness, and latency. No experimental results are reported at the proposal stage.

**Index Terms** — Large language models, robot navigation, ROS 2, Nav2, tool-using agents, human–robot interaction, clarification, safety-aware systems.

> 英文標題中的 "EvidenceNav" 為本次重構建議的簡稱，組內確認後可替換。
