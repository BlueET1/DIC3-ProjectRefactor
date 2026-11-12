# Spec Delta

## Purpose

透過受限的白名單工具，把已驗證的地點目標交給 Nav2 執行，並定義路徑檢查、有限重試、取消與底層安全的邊界。

## ADDED Requirements

### Requirement: Whitelisted tool gateway
Agent SHALL 只能呼叫 `list_landmarks`、`get_robot_status`、`get_path`、`navigate_to_landmark`、`cancel_navigation`、`read_mission_result` 六個工具。未知工具、shell 指令或修改地圖、footprint、安全參數的請求 MUST 被拒絕。

#### Scenario: 呼叫未知工具
- **WHEN** 模型要求呼叫 `set_inflation_radius`
- **THEN** gateway 拒絕該呼叫並記錄事件，Nav2 參數不變

### Requirement: Registry-based goal resolution
地點 ID SHALL 由 gateway 依地點表轉為 map frame 中的 pose（x、y、yaw）。目標 MUST 使用地點表指定的 `frame_id` 與 `map_id`。

#### Scenario: 解析地點座標
- **WHEN** 派送 `desk`
- **THEN** 送給 Nav2 的目標座標與地點表中 `desk` 的登錄值一致，frame 為地點表指定的 map

### Requirement: Path feasibility before dispatch
派送前系統 SHALL 以有限 timeout 呼叫 `get_path`。沒有有效路徑時 MUST 回傳 `NO_PATH`，MUST NOT 派送或宣稱到達。

#### Scenario: 路徑被擋
- **WHEN** `get_path` 回傳 `valid: false`
- **THEN** 系統回傳 `NO_PATH`，`goal_sent` 為 false

### Requirement: Bounded retry
Nav2 自身的恢復行為結束並回報 FAILED 後，Agent SHALL 在重試額度內且狀態與路徑重新檢查皆合格時，最多重新派送一次。

#### Scenario: 第一次失敗後重試
- **WHEN** 第一次導航回報 FAILED，重新檢查狀態合格且 `get_path` 有效
- **THEN** 系統重新派送一次

#### Scenario: 第二次仍失敗
- **WHEN** 重試後的導航仍回報 FAILED
- **THEN** 系統回傳 `FAILED` 並附兩次的 action 結果，不再重試

### Requirement: Cancellation priority
取消請求 SHALL 優先於任何新任務。系統 MUST 觀察 action 終止結果與速度／里程計以確認停止；無法確認時 MUST 回報 `FAULT` 並將 `stop_confirmed` 設為 false。

#### Scenario: 取消成功
- **WHEN** 任務進行中使用者說「停下來」，且 action 回報已取消並觀測到速度為零
- **THEN** 系統回傳 `CANCELED`，`stop_confirmed` 為 true

#### Scenario: 無法確認停止
- **WHEN** 取消請求被接受，但通訊中斷無法讀到終止結果
- **THEN** 系統回傳 `FAULT`，`stop_confirmed` 為 false，並提示現場操作員處理

### Requirement: LLM-independent stopping
低延遲停止與避障 MUST 由 Nav2 與底層機制負責，MUST NOT 等待 LLM 回應。

#### Scenario: 途中出現障礙物
- **WHEN** 行駛中雷射偵測到新障礙物而 LLM 服務無回應
- **THEN** 底層照常減速或停止，Agent 之後依 action 結果回報
