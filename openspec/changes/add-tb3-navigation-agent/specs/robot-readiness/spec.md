# Spec Delta

## Purpose

在派送任何導航目標之前，確認機器人的定位、感測、座標轉換與任務狀態都是新鮮且合格的，避免在資料失效或任務衝突時出發。

## ADDED Requirements

### Requirement: Sensor and localization freshness gate
系統 SHALL 在派送前讀取 `scan_age`、`pose_age`、`tf_ok`。任一資料超過設定的時效門檻或 TF 不可用時，系統 MUST 回傳 `NOT_READY` 並附上不合格項目，MUST NOT 派送。

#### Scenario: 雷射資料過期
- **WHEN** `scan_age` 超過時效門檻
- **THEN** 系統回傳 `NOT_READY`，evidence 中標記 scan 過期，`goal_sent` 為 false

#### Scenario: TF 中斷
- **WHEN** `tf_ok` 為 false
- **THEN** 系統回傳 `NOT_READY`，且不呼叫 `navigate_to_landmark`

#### Scenario: 狀態全部合格
- **WHEN** scan、pose 皆在時效內且 `tf_ok` 為 true、無進行中任務
- **THEN** 系統進入路徑可行性檢查

### Requirement: Single active mission
系統 SHALL 同一時間只允許一個移動任務。已有任務進行中時，新的移動請求 MUST 回傳 `BUSY`；取消請求不受此限制。

#### Scenario: 執行中收到新目標
- **WHEN** `task_active` 為 true 且收到新的 `navigate` 請求
- **THEN** 系統回傳 `BUSY`，原任務不受影響

### Requirement: Time source separation
資料時效 SHALL 依 ROS 時間計算，控制流程 deadline SHALL 使用單調計時器；模擬（`use_sim_time`）與實機設定 MUST 分開，模擬暫停 MUST NOT 被視為新鮮的實機資料。

#### Scenario: 模擬暫停
- **WHEN** Gazebo 暫停使 ROS 時間停止
- **THEN** 系統不會因時間未前進而把舊資料判定為新鮮資料
