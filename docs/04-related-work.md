# 04. Related Work（重構版）

## 重構說明

| 期中版 | 重構後 |
|---|---|
| 依「AI 技術 3 篇／領域知識 3 篇」分類，是作業規範的分類，不是研究主題的分類 | 改依**研究主題**分成四條脈絡，每條結尾寫「與本專題的差異」 |
| 澄清與安全只在設計中出現，文獻沒有支撐 | 新增 KnowNo [13]（何時該問）與 RoboGuard [15]（LLM 機器人安全護欄） |
| 只有 1 篇 ROS 2 語言介面文獻 [3] | 新增 ROS-LLM [14] 與 LM-Nav [16]，補齊「語言 → 機器人行為」光譜 |
| 研究缺口只有一句 | 用比較表呈現：現有工作各自做到哪些，本專題補哪一塊 |

保留原有 [1]–[12] 編號，新增 [13]–[16]。原有 8 欄文獻摘要表仍保留在原始 repo 的 `PROPOSAL.md` §3.5–3.6。

---

## 2. Related Work

### 2.1 LLM 作為機器人的規劃者與工具使用者

ReAct 讓語言模型交替產生推理與工具行動，並根據環境觀測更新下一步 [1]。SayCan 指出 LLM 的計畫必須受機器人實際技能的可行性約束，以價值函數為語言計畫打分 [2]。ROS-LLM 將這個思路帶入 ROS 生態：使用者以對話描述任務，LLM 產生序列、行為樹或狀態機，並以人類與環境回饋做反思修正 [14]。Pollini 等人則在 ROS 2 與 Gazebo 中整合 ChatGPT，聚焦語言指令處理的延遲 [3]。

**與本專題的差異**：上述工作重視讓 LLM 產生**更豐富的行為**；本專題刻意反其道而行，把 LLM 的輸出縮到四種 intent 與已登錄地點 ID，換取可驗證性。我們沿用 [1] 的觀測回饋迴圈、[2] 的可行性約束概念，以及 [3] 的延遲量測方式。

### 2.2 語言驅動的移動機器人導航

LM-Nav 串接 GPT-3（抽取地標）、CLIP（地標落地）與 ViNG（視覺導航），不需語言標註資料即可在戶外完成長距離導航 [16]。這類工作證明語言可以直接驅動導航，但目標是**開放詞彙與探索**，地標的落地依賴視覺相似度。

**與本專題的差異**：本專題處理的是封閉、已登錄的室內地點；地點落地由人工維護的地點表完成，而非模型推論。這讓「派送了不該派送的目標」可以被精確定義為錯誤，並成為主要評估指標。

### 2.3 不確定性、澄清與安全護欄

KnowNo 以 conformal prediction 量化 LLM 規劃器的不確定性：當候選動作不只一個時，機器人主動請求人類協助，並在使用者設定的成功率下盡量減少求助次數 [13]。RoboGuard 則針對 LLM 機器人的物理風險提出兩階段護欄，用可信的 LLM 推理落地安全規格，再以控制合成過濾不安全計畫，並在 Clearpath Jackal 上測試對越獄攻擊的穩健性 [15]。

**與本專題的差異**：本專題採用較簡單、可重現的機制——以地點表比對判斷歧義（而非機率校準），以白名單與固定門檻判斷可否派送（而非 LLM 護欄）。我們把 [13]、[15] 視為更通用的上限參考：若規則式方法已能在實驗室場景達到低不當派送率，就不需要引入更複雜的推論。越權／矛盾輸入被列入 60 筆測試指令之一類，正是受 [15] 啟發。

### 2.4 模組化導航、建圖與標準化評測

Nav2 以行為樹協調定位、規劃、控制與恢復，並在校園長距離實驗中展示 ROS 2 導航系統的可用性 [4]；SLAM Toolbox 提供建圖、定位與地圖維護 [5]。BARN Challenge 以狹窄環境比較多種導航方法，顯示受限空間中的可靠導航仍有挑戰 [6][12]；NavBench 則提出跨平台的 RL 導航評測 [7]。官方文件 [8]–[11] 提供 Burger 硬體、footprint 調校、Simple Commander API 與 Collision Monitor 的工程依據，不視為獨立實驗證據。

**與本專題的差異**：本專題不比較導航演算法，而是**把 Nav2 固定為共同底層**，借用 [6][7][12] 的「固定場景、重複測試、保留失敗」原則，單獨量測任務層的增益。

### 2.5 研究缺口

| 研究 | 自然語言輸入 | 澄清／求助 | 派送前狀態檢查 | 依執行證據回報 | 固定導航底層下的任務層比較 | 平台 |
|---|:-:|:-:|:-:|:-:|:-:|---|
| ReAct [1] | ✓ | — | — | 部分（觀測回饋） | — | 文字／網頁環境 |
| SayCan [2] | ✓ | — | 技能可行性 | — | — | 移動操作機器人 |
| Pollini et al. [3] | ✓ | — | — | — | — | Gazebo 模擬 |
| ROS-LLM [14] | ✓ | 人類回饋修正 | — | 環境回饋反思 | — | 機械手臂等 |
| LM-Nav [16] | ✓ | — | — | — | — | 戶外移動機器人 |
| KnowNo [13] | ✓ | ✓（conformal） | — | — | — | 模擬與實機操作 |
| RoboGuard [15] | ✓ | — | 安全規格過濾 | — | — | Jackal |
| Nav2 [4] / BARN [6][12] | — | — | 部分（底層） | action 結果 | — | 多平台 |
| **本專題** | ✓（中文） | ✓（地點表比對） | ✓（定位／雷射／TF／互斥） | ✓（action + 觀測姿態） | ✓（B0／B1／B2 + 消融） | Burger 模擬 + 實機 |

> 上表為本組依各文獻主要貢獻所做的歸納，「—」表示**非該論文的主要焦點**，不代表該系統完全沒有此能力。

**缺口陳述**：現有研究分別處理了語言規劃、語言導航、澄清與安全，但缺少一個在**固定 Nav2 底層**下，把「澄清、派送前狀態檢查、證據回報」三者對任務可靠性的增益**分開量測**的實驗。對中文、實驗室規模、低成本差速小車的場景，這些增益也尚未被報告。本專題以 B0／B1／B2 比較與兩項消融回答 RQ1–RQ3。

---

## References

[1] S. Yao *et al.*, "ReAct: Synergizing Reasoning and Acting in Language Models," ICLR 2023. arXiv:2210.03629.

[2] M. Ahn *et al.*, "Do As I Can, Not As I Say: Grounding Language in Robotic Affordances," CoRL 2022. arXiv:2204.01691.

[3] D. Pollini, B. V. Guterres, R. S. Guerra, and R. B. Grando, "Reducing Latency in LLM-Based Natural Language Commands Processing for Robot Navigation," INDIN 2025. arXiv:2506.00075.

[4] S. Macenski, F. Martín, R. White, and J. G. Clavero, "The Marathon 2: A Navigation System," IROS 2020. arXiv:2003.00368.

[5] S. Macenski and I. Jambrecic, "SLAM Toolbox: SLAM for the dynamic world," *JOSS*, 6(61), 2783, 2021. doi:10.21105/joss.02783.

[6] X. Xiao *et al.*, "Autonomous Ground Navigation in Highly Constrained Spaces: Lessons Learned from the BARN Challenge at ICRA 2022," *IEEE RAM*, 2022. arXiv:2208.10473.

[7] M. El-Hariry *et al.*, "NavBench: A Unified Robotics Benchmark for Reinforcement Learning-Based Autonomous Navigation," 2025. arXiv:2505.14526.

[8] ROBOTIS, "TurtleBot3 Features and Navigation," e-Manual. https://emanual.robotis.com/docs/en/platform/turtlebot3/features/

[9] Nav2, "Tuning Guide." https://docs.nav2.org/jazzy/configuration_and_development/tuning_guide/

[10] Nav2, "Simple Commander API." https://docs.nav2.org/jazzy/configuration_and_development/simple_commander_api/simple_commander_api/

[11] Nav2, "Collision Monitor." https://github.com/ros-navigation/navigation2/blob/main/nav2_collision_monitor/README.md

[12] X. Xiao and BARN organizers, "The BARN Challenge 2026." https://people.cs.gmu.edu/~xiao/Research/BARN_Challenge/BARN_Challenge26.html

[13] A. Z. Ren *et al.*, "Robots That Ask For Help: Uncertainty Alignment for Large Language Model Planners," CoRL 2023, PMLR 229, pp. 661–682. arXiv:2307.01928. **（新增）**

[14] C. E. Mower *et al.*, "ROS-LLM: A ROS framework for embodied AI with task feedback and structured reasoning," 2024. arXiv:2406.19741. **（新增）**

[15] Z. Ravichandran, A. Robey, V. Kumar, G. J. Pappas, and H. Hassani, "Safety Guardrails for LLM-Enabled Robots," 2025. arXiv:2503.07885. **（新增）**

[16] D. Shah, B. Osiński, B. Ichter, and S. Levine, "LM-Nav: Robotic Navigation with Large Pre-Trained Models of Language, Vision, and Action," CoRL 2022, PMLR 205. arXiv:2207.04429. **（新增）**

網路來源查閱日期：[1]–[12] 為 2026-10-30（沿用期中版）；[13]–[16] 為 2026-11-12。
