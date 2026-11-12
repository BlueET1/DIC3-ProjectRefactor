# Spec Delta

## Purpose

讓每一次任務的結束狀態都能回溯到導航 action 結果與觀測資料，避免把取消、逾時或失敗誤報為到達，並提供評估所需的事件紀錄。

## ADDED Requirements

### Requirement: Closed set of terminal states
每個任務 SHALL 以下列其中一個狀態結束：`ARRIVED`、`FAILED`、`CANCELED`、`TIMEOUT`、`NO_PATH`、`NOT_READY`、`BUSY`、`CLARIFY`、`REJECTED`、`FAULT`。`CANCELED`、`TIMEOUT`、`FAILED` MUST NOT 被回報為 `ARRIVED`。

#### Scenario: 任務逾時
- **WHEN** 任務超過 deadline 仍未收到 SUCCEEDED
- **THEN** 系統取消目標並回傳 `TIMEOUT`，不回傳 `ARRIVED`

### Requirement: Evidence-based arrival
系統 SHALL 只在 Nav2 action 結果為 SUCCEEDED 且觀測到的終點姿態落在預先固定的容許誤差內時回傳 `ARRIVED`。

#### Scenario: action 成功且終點正確
- **WHEN** action 回報 SUCCEEDED 且最終姿態在容許誤差內
- **THEN** 系統回傳 `ARRIVED`，附 action 結果與 final pose

#### Scenario: action 成功但姿態不符
- **WHEN** action 回報 SUCCEEDED，但觀測姿態超出容許誤差
- **THEN** 系統不回傳 `ARRIVED`，並在 evidence 中標記姿態不一致

### Requirement: Structured report fields
每筆回報 SHALL 包含 `mission_id`、`status`、`goal_sent`、`timestamp`、`evidence_id`；派送過的任務另含 `action_state` 與 `final_pose`；取消類回報含 `stop_confirmed`。

#### Scenario: 拒絕請求的回報
- **WHEN** 請求因地點不存在被拒絕
- **THEN** 回報含 `status: REJECTED`、`goal_sent: false`、`timestamp` 與 `evidence_id`

### Requirement: Event log for evaluation
系統 SHALL 以 JSONL 記錄每次工具呼叫：工具名稱、經驗證的參數、時間、狀態摘要與結果。紀錄 MUST NOT 包含 API 金鑰，也 MUST NOT 依賴模型的內部思考過程。

#### Scenario: 完整任務的紀錄
- **WHEN** 一個導航任務從解析到 ARRIVED 結束
- **THEN** 事件紀錄中可依 `mission_id` 取出所有工具呼叫與時間戳，用來計算解析延遲、派送延遲與回報延遲
