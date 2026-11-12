# Spec Delta

## Purpose

將使用者的中文室內移動指令轉為受 schema 與地點白名單約束的結構化任務，讓後續派送只會收到合法、無歧義、已登錄的目標。

## ADDED Requirements

### Requirement: Schema-constrained intent output
系統 SHALL 只接受 `navigate`、`cancel`、`clarify`、`reject` 四種 intent。`navigate` MUST 只含 `intent` 與 `landmark_id` 兩欄；`cancel` MUST 只含 `intent`；`clarify` MUST 含非空 `question`；`reject` MUST 含非空 `reason`。

#### Scenario: 合法導航請求
- **WHEN** 解析結果為 `{"intent": "navigate", "landmark_id": "desk"}` 且 `desk` 存在於地點表
- **THEN** 系統將請求交給狀態檢查，不回傳錯誤

#### Scenario: 未定義的 intent
- **WHEN** 解析結果的 intent 為 `teleport`
- **THEN** 系統回傳 `REJECTED`，且不送出任何導航目標

#### Scenario: 額外欄位
- **WHEN** `navigate` 請求額外包含 `x`、`y` 座標欄位
- **THEN** 系統回傳 `REJECTED`，且不使用該座標

### Requirement: Landmark allowlist
系統 MUST 只派送存在於目前地點表（registry）中的 `landmark_id`。模型產生的座標、輪速或任意 pose MUST NOT 被當作導航目標。

#### Scenario: 不存在的地點
- **WHEN** 使用者說「去三樓的咖啡廳」，而地點表沒有對應 ID
- **THEN** 系統回傳 `REJECTED` 並說明地點未登錄，不派送

### Requirement: Ambiguity clarification
當一句指令對應到兩個以上已登錄地點，系統 SHALL 回傳 `CLARIFY` 並列出候選地點，MUST NOT 自行選擇其中一個派送。

#### Scenario: 兩個門口
- **WHEN** 使用者說「去門口」，地點表中有 `door_a`（A 門口）與 `door_b`（B 門口）
- **THEN** 系統回傳 `CLARIFY`，問題中包含「A 門口」與「B 門口」，且 `goal_sent` 為 false

#### Scenario: 使用者回答澄清
- **WHEN** 系統已詢問候選地點，使用者回答「A 門口」
- **THEN** 系統以 `door_a` 產生 `navigate` 請求並進入狀態檢查

### Requirement: Parser failure handling
當 LLM 服務逾時、失敗或輸出不符合 schema，系統 SHALL 回傳明確錯誤或澄清，MUST NOT 以猜測的目標派送。

#### Scenario: LLM 逾時
- **WHEN** 解析呼叫超過設定的 timeout
- **THEN** 系統回傳錯誤狀態並請使用者重述，不送出導航目標
