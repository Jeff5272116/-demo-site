# 師傅請款與專案成本管理系統 — 前端 Demo

依《系統規格書 v0.1》製作的**純前端示範**，用固定示範資料操作，**未串接資料庫／後端**。
依客戶的兩種角色拆成兩個獨立 HTML，雙擊即可在瀏覽器開啟，無需安裝環境、不需網路。

## 檔案
| 檔案 | 角色 | 內容 |
|------|------|------|
| `使用者.html` | 師傅（worker） | 請款提交（自動計價）、我的請款紀錄；右上可**切換師傅**示範資料隔離 |
| `管理員.html` | 最高管理者（admin） | 審核（單筆／批次）、全部請款、專案／單價、使用者管理、報表儀表板、CSV 匯出 |
| `師傅請款與專案成本管理系統_系統規格書_v0.1.docx` | — | 原始規格書（唯讀，請勿修改） |

> 兩個檔各自獨立、資料不互通（純前端、無資料庫）。各自將「自己這半的後台資料庫寫入邏輯」呈現清楚。

## 重點：後台資料庫寫入邏輯（本案難點）
每個檔下方都有深色「📦 後台資料庫（示意）」面板，把後端會發生的事攤開：
- **兩張對應規格的資料表**：`billing_records`（5.4）、`status_history`（5.5）
- **每次操作顯示寫入流程**：
  - 提交：取得 `unit_price_snapshot` → 計算 `amount` → 組合 `unique_key` → 檢查唯一索引（防重複）→ `INSERT billing_records`（只新增不覆蓋）→ `INSERT status_history`
  - 審核：驗證 role → 狀態機檢查 → `UPDATE billing_records SET status` → `INSERT status_history`

對應規格 4.2（計價／快照）、4.3（唯一鍵防重複、INSERT 不 UPDATE）、4.4（狀態機）、5.5（歷史可追溯）、7.2（並發與完整性）。

## 名詞對照（畫面用詞 ＝ 規格第 5 章欄位）
資料結構與面板欄位皆對齊規格書，方便對照講解：
- `users`（5.1）：`id`、`name`、`account`、`role`（worker/admin/third_party）、`status`（啟用/停用）
- `projects`（5.2）：`id`、`name`、`code`、`status`（進行中/已結案）
- `unit_prices`（5.3）：`id`、`project_id`、`item_name`、`spec_type`（制式/客變）、`default_qty`、`unit_price`
- `billing_records`（5.4）：`worker_id`、`project_id`、`floor`、`item_name`、`spec_type`、`quantity`、`unit_price_snapshot`、`amount`、`unique_key`、`status`（pending/approved/rejected）、`reviewed_by`、`reviewed_at`、`reject_reason`、`submitted_at`
- `status_history`（5.5）：`record_id`、`from_status`、`to_status`、`actor_id`、`note`、`created_at`

> `worker_id`／`project_id` 為外鍵，對應 `users`／`projects` 表的 `id`（在管理員端「使用者管理」「專案／單價」分頁可看到 id ↔ 名稱對應）。

## Demo 操作建議
1. 開 `使用者.html`：
   - 選案名→樓層→項目→數量→提交，看下方「寫入流程」與兩張表如何 `INSERT`。
   - 再送一筆**相同**的（同案名/樓層/項目）→ 觸發「疑似重複」（`unique_key` 已存在）。
   - 右上切換不同師傅 → 到「我的請款紀錄」確認只看得到本人資料（資料隔離）。
   - 在「我的請款紀錄」用**期間起／迄**篩選 → 薪資總額依指定期間重算（規格 4.1）。範例資料橫跨 5 月～6 月可看出差異。
2. 開 `管理員.html`（內含範例請款；建案範例 A~E）：
   - 「請款審核」勾選 → 批次批准／退回，看 `status` 如何 `UPDATE`、`status_history` 如何留痕。
   - 疑似重複那筆會標紅，批准時要求二次確認。
   - 「報表儀表板」含關鍵指標與**各專案／樓層 責任人與金額**明細（規格 4.8），並可**勾選建案後匯出**對帳檔。
   - 其餘分頁示範專案／單價（制式/客變）、使用者與 CSV 匯出。

## 對應規格條目
| 規格 | 功能 |
|------|------|
| 3 | 角色與權限（拆成兩個 HTML）、資料隔離（切換師傅） |
| 4.1 / 4.2 | 請款提交、自動計價、單價快照、制式預填數量 |
| 4.3 | 防重複（`unique_key = project_id｜floor｜item_name｜日期`）、只新增不覆蓋 |
| 4.4 / 4.5 | 狀態機、單筆／批次審核、退回原因 |
| 4.6 | 專案 CRUD、單價維護 |
| 4.7 | 對帳 CSV 匯出 |
| 4.8 | 報表儀表板 |
| 5.4 / 5.5 | `billing_records`、`status_history` 資料表呈現 |

## 注意
- 資料為寫死的示範值，**重新整理頁面會還原**（這是 demo，正式版才接後端／DB）。
- 登入畫面依主管指示不呈現（改以兩個 HTML 區分角色）。
- 永豐銀行匯出格式待確認，demo 先以一般 CSV 範例取代。
- 兩個檔資料不互通；正式版會由共用的後端＋資料庫串起師傅端與管理員端。
