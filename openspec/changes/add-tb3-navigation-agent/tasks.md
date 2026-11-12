# Tasks

## 1. 平台與導航基準（B0）

- [ ] 1.1 核對實機與電腦端 ROS 2／Nav2／Gazebo 版本並寫入 README；驗證：兩端 `ros2 --version` 與 Nav2 套件版本一致並記錄
- [ ] 1.2 以 SLAM Toolbox 建立實驗室地圖，在地圖上重新登錄 `landmarks.json`（移除 `demo_only`）；驗證：每個地點在 RViz 中目視落在可通行區
- [ ] 1.3 建立固定地點選單 B0，在 Gazebo 對每個地點派送一次；驗證：action 回報 SUCCEEDED 並記錄 Nav2 參數 hash

## 2. 工具 gateway 與 mock adapter

- [ ] 2.1 將 `validate_request` 升級為 gateway，實作 6 個白名單工具的 schema 驗證；驗證：未知工具、額外欄位、未登錄地點的單元測試全部回傳 REJECTED
- [ ] 2.2 定義 `nav2_adapter` 介面並實作 mock 後端；驗證：原有 8 個 mock 情境測試全部通過
- [ ] 2.3 實作 registry 地點轉 pose；驗證：單元測試確認座標與 frame 與 `landmarks.json` 一致

## 3. 狀態檢查（robot-readiness）

- [ ] 3.1 實作 `get_robot_status`（scan_age、pose_age、tf_ok、task_active）；驗證：在 Gazebo 停止 `/scan` 發布後回傳 NOT_READY
- [ ] 3.2 實作單一任務互斥；驗證：任務進行中送新目標回傳 BUSY 的測試通過
- [ ] 3.3 分離 ROS 時間與單調計時器；驗證：Gazebo 暫停時舊資料不被判為新鮮的測試通過

## 4. 中文解析與澄清（task-intent-parsing）

- [ ] 4.1 固定 LLM model ID、temperature、prompt hash，實作 `intent_parser`；驗證：20 筆開發指令全部輸出合法 schema
- [ ] 4.2 實作歧義偵測與 CLARIFY 回合；驗證：「去門口」測試回傳包含 A、B 兩門口的問題且 goal_sent 為 false
- [ ] 4.3 實作 LLM 逾時與格式錯誤處理；驗證：mock 逾時測試回傳錯誤且未呼叫 navigate

## 5. 派送、重試與取消（navigation-dispatch）

- [ ] 5.1 實作 `get_path` 可行性檢查；驗證：Gazebo 中以箱子封路時回傳 NO_PATH
- [ ] 5.2 實作有限重試（最多一次）；驗證：mock 連續兩次 FAILED 只派送兩次並回傳 FAILED
- [ ] 5.3 實作取消與停止確認；驗證：Gazebo 取消後 action 已取消且速度為零時 stop_confirmed 為 true，斷線時回傳 FAULT

## 6. 回報與事件紀錄（mission-reporting）

- [ ] 6.1 實作 ARRIVED 判定（action SUCCEEDED + 終點姿態容許誤差）；驗證：mock 姿態超出容許誤差時不回傳 ARRIVED
- [ ] 6.2 實作 JSONL logger；驗證：一個完整任務可依 mission_id 取出所有工具呼叫，且檔案中無 API 金鑰

## 7. 整合評估

- [ ] 7.1 標註 60 筆中文指令（兩人獨立標註），鎖定 40 筆測試集與 commit hash；驗證：標註一致性與 hash 記錄於 `data/`
- [ ] 7.2 在 Gazebo 執行 B0／B1／B2 各 40 次與消融 A、B；驗證：`results.csv` 每筆有 run_id 且保留所有失敗
- [ ] 7.3 實機 12 配對案例 × 3 方法；驗證：每次有事件紀錄與影片，安全中止另列
- [ ] 7.4 計算不當派送率、任務完成率、回報正確率（Wilson 95% CI）與延遲 p50/p95；驗證：報表可由腳本從原始紀錄重新產生

## Workflow follow-up

- 期末實作完成並通過 7.x 驗證後，執行 `openspec archive add-tb3-navigation-agent` 將 specs 合併到 `openspec/specs/`。
- Archive 後以 `openspec validate --specs --strict` 確認主規格完整。
