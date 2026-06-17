# EDIS 簡報 vs 程式碼 完整比對報告

**比對日期：** 2026-06-17
**簡報檔案：** DataCo 物流延遲預測與最佳化調度系統 (EDIS).pptx（共 9 張）
**程式版本：** commit `be645b0` — fix(setup): optimize conda env script and unify dependencies with pydantic-core fallback repair

---

## 一、完全實作且相符

| 簡報聲明 | 對應程式碼 | 狀態 |
|----------|-----------|------|
| XGBoost 延遲預測引擎 | `core/model_pipeline.py` — `XGBOOST_PARAMS` | ✅ 相符 |
| PuLP 整數規劃最佳化（0/1 MILP） | `core/optimizer.py` | ✅ 相符 |
| 地端純本地運算，無外部 AI API 呼叫 | FastAPI + 本地記憶體，所有推理在本機執行 | ✅ 相符 |
| 數值特徵（天數、價格、數量等） | `core/data_pipeline.py` — `NUMERICAL_FEATURES` | ✅ 相符 |
| 類別特徵 One-Hot / Label Encoding | `core/data_pipeline.py` — `ONE_HOT_FEATURES`、`LABEL_ENCODE_FEATURES` | ✅ 相符 |
| 敏感欄位刪除（姓名、地址） | `core/security_utils.py` — `SENSITIVE_COLUMNS`（Fname、Lname、Street、Zipcode、Email、Password） | ✅ 相符 |
| Order Id SHA-256 雜湊 | `core/security_utils.py` — `hash_id_columns()` | ✅ 相符 |
| RBAC：Viewer / Logistics_Manager 兩角色 | `app.py` — `ROLE_VIEWER`、`ROLE_MANAGER` | ✅ 相符 |
| 非授權呼叫 → 403 Forbidden | `app.py` — `require_manager()` 函數 | ✅ 相符 |
| bcrypt 密碼雜湊（含 per-user salt） | `core/auth.py` — `hash_password()` | ✅ 相符 |
| 審計日誌記錄 | `app.py` — SQLite `audit_logs` 資料表，啟動時初始化 | ✅ 相符 |

---

## 二、實作有差異（簡報描述與程式碼不完全一致）

| 簡報聲明 | 實際程式碼 | 差異說明 |
|----------|-----------|---------|
| **雙集分割**（訓練集 + 驗證集） | `core/data_pipeline.py` 比例為 70 / 15 / 15 | 程式碼是**三分割**（train / val / test），並非雙集 |
| **SHA-256 雜湊** | `core/security_utils.py` — `f"{self.hash_salt}:{value}"` | 實際為**加鹽雜湊**（Salt = `"EDIS_2026"`），安全性更高，但與簡報的純 SHA-256 描述不同 |
| **Security API Gateway 動態過濾** | `app.py` — `get_role()` 函數 | 優先讀取 **JWT Bearer Token**（HMAC-SHA256 簽章），次才使用 X-Role Header；簡報未提及 Token 機制 |

---

## 三、程式碼有但簡報完全未提及（功能超出簡報範圍）

| 功能 | 程式碼位置 | 說明 |
|------|-----------|------|
| **全球物流延遲風險熱力圖** | `static/region_map.js` + `/api/regions` | 世界地圖 Leaflet 視覺化，依地區著色顯示延遲風險 |
| **月度診斷 + 事件標記** | `/api/diagnose/monthly` + `/api/diagnose/monthly/flag` | 可標記異常月份，追蹤數據波動原因 |
| **模型重訓機制** | `core/retrainer.py` + `/api/retrain` | 新模型先存暫存目錄，Manager 確認指標後才 adopt 正式取代現有模型 |
| **可解釋性層** | `core/explainer.py` + `/api/explain/{order_id_hash}` | 對單一訂單產生中文決策說明，LLM-ready 架構 |
| **情境分析** | `/api/scenario-analysis` | 多情境參數比較 |
| **Threshold 調校** | `/api/threshold-tuning` | 模型決策閾值調整工具 |
| **CSV 上傳預測** | `/api/upload` | 上傳新訂單 CSV 直接取得延遲預測 |
| **訓練資料累積上傳** | `/api/upload-training` | 累積新訓練資料供重訓使用 |
| **臨時檔案自動清理** | `app.py` — `cleanup_loop()` | 每小時清理超過 24 小時的 session 暫存檔 |

---

## 四、總結

| 類別 | 數量 |
|------|------|
| 完全符合 | 11 項 |
| 描述有差異（實作更嚴謹） | 3 項 |
| 程式有但簡報未提及（額外功能） | 9 項 |

**結論：**
- 簡報的**核心主張**（雙引擎架構、三層隱私防護、RBAC 權限控制）全部有對應實作，內容可信。
- 三項差異均為程式**比簡報描述更嚴謹**（加鹽雜湊、三集分割、JWT Token），不是缺漏。
- 實作功能**遠比簡報豐富**：熱力圖、重訓、可解釋性、月度診斷等 9 項均未出現在簡報中，是可補充的亮點。
