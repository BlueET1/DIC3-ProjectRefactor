# Proposal

## Why

把中文指令直接轉成導航目標，會在 TurtleBot3 Burger 上造成三種失效：派送歧義或不存在的地點、在定位／感測資料失效時仍出發、以及在取消或逾時後誤報「到達」。原始專題目前只有離線 mock 原型（見 `source/DESIGN.md` §3），需要先把預期行為寫成可驗證的規格，才能在接入 LLM 與 ROS 2／Nav2 時有明確的驗收標準。

## What Changes

- 新增中文任務解析：LLM 只輸出 `navigate`／`cancel`／`clarify`／`reject` 四種 intent 與已登錄地點 ID，其餘一律拒絕。
- 新增地點歧義澄清：多個候選地點或未知地點時回傳 `CLARIFY` 或 `REJECTED`，不派送。
- 新增派送前狀態檢查：定位、雷射、TF 時效與任務互斥檢查，不合格回傳 `NOT_READY`／`BUSY`。
- 新增受限導航派送：只能透過 6 個白名單工具呼叫 Nav2；地點 ID 由地點表轉為座標；失敗後最多重派一次；取消優先。
- 新增證據導向回報：終態只依 Nav2 action 結果與觀測決定，每筆回報附 evidence ID，並寫入事件紀錄供評估使用。

## Capabilities

### New Capabilities
- `task-intent-parsing`: 將中文指令轉為受 schema 與白名單約束的任務 intent，含歧義澄清與拒絕。
- `robot-readiness`: 派送前檢查機器人狀態（定位、感測、TF 時效與任務互斥）。
- `navigation-dispatch`: 經驗證的地點目標派送、路徑可行性檢查、有限重試與取消。
- `mission-reporting`: 以導航 action 結果與事件紀錄為依據的終態回報與紀錄格式。

### Modified Capabilities
（無；本 repo 尚無既有 spec。）

## Impact

- **程式**（原始專題 repo）：新增 `src/intent_parser.py`、`src/mission_manager.py`、`src/nav2_adapter.py`；`src/agent_demo.py` 的 `validate_request` 與 JSON 輸出升級為正式 gateway 與 JSONL logger。
- **介面**：6 個工具 `list_landmarks`、`get_robot_status`、`get_path`、`navigate_to_landmark`、`cancel_navigation`、`read_mission_result`（`source/INTERFACES.md`）。
- **相依**：ROS 2 + Nav2（單一固定發行版，待與實機核對）、可輸出結構化 JSON 的既有 LLM（供應者與 model ID 待固定）。
- **資料**：`config/landmarks.json` 需改為實機地圖上重新登錄的座標；新增 60 筆中文指令標註集與 JSONL 事件紀錄。
- **不影響**：Nav2 規劃／控制演算法、footprint 與安全參數（皆維持固定，不由 Agent 修改）。
