# OmniPass 專案紀錄

> 本檔案為開發決策與進度紀錄。正式程式以 repo 根目錄 `index.html` 為準。

最後更新：2026-07-18

---

## Phase 進度

| Phase | 內容 | 狀態 |
|-------|------|------|
| 0 | `.gitignore` 排除 `gemini-*.md` | ✅ 完成 |
| 1 | IndexedDB v2 資料模型 + migration | ✅ 完成 |
| 2 | 掃描前選超商 + 面額；依超商切換條碼格式 | ✅ 完成 |
| 3 | 掃描效能（ROI、降頻、cooldown debug） | ✅ 完成 |
| 4 | default 資料夾 + 獨立封存頁 | ✅ 完成 |
| 5 | 全螢幕：刪除、餘額 Seekbar、確認後 FIFO 歸零 | ✅ 完成 |
| 6 | 虛擬卡條碼高度 Seekbar | ✅ 完成 |
| 7 | 事件分析 + Cloudflare Analytics | ⏸ **暫緩（Padding）** |

---

## 已確認的產品決策

### 掃描流程
- **超商與面額**：每次啟動掃描前先選（適合連續掃同一批）。
- **四超商**：7-ELEVEN、全家、萊爾富、OK（後兩者條碼格式預設 CODE39）。
- **條碼格式**：
  - 全家 → CODE39
  - 7-ELEVEN → CODE128
  - 萊爾富 / OK → CODE39（預留，待實測調整）

### 面額
- 預設：35 / 50 / 100 / 200 元
- 少見：20 / 88 元
- 使用者可自訂新增面額（存入 IndexedDB settings）

### 餘額與 FIFO
- 全螢幕檢視卡片時，以 Seekbar 調整餘額（0 ~ 面額）。
- 須按 **「確認更新餘額」** 才生效。
- 確認後：**同一超商、scanOrder 較早** 的卡片餘額全部設為 0，並移入封存。
- 餘額 = 0 的卡片自動進入 **封存頁**（獨立檢視，非一般資料夾）。

### 資料夾
- 現階段僅 **default** 預設資料夾；新掃描卡片皆歸入此處。
- 封存為獨立頁面，不代表一般資料夾。
- 日後擴充：多分類資料夾，但同一資料夾內必為同一超商。

### 掃描效能 Debug
- 可調整 **掃描成功後 cooldown（ms）**，減輕連續掃描 lag。
- 可調整 **掃描間隔（ms）**，降低每 frame 解碼負擔。
- ROI 裁切：僅掃描畫面中央瞄準框區域。

### Git / 隱私
- `gemini-*.md` 已加入 `.gitignore`，避免 public repo 外洩聊天紀錄。

---

## Phase 7 待辦（暫緩）

> 使用者確認先跳過，待有 Cloudflare site token 或分析需求明確後再實作。

- [ ] 自訂事件分析（掃描成功/失敗、重複、刪除、餘額更新、封存）
- [ ] Cloudflare Web Analytics beacon 嵌入
- [ ] （可選）事件匯出或 debug 面板

---

## 條碼規格備註

公開資料中，7-ELEVEN 的 Code39 規格為 **代收帳單** 用途，非商品卡本身；全家亦無公開商品卡條碼規格。本專案依實體卡實測：

| 超商 | 採用格式 |
|------|----------|
| 全家 | CODE39 |
| 7-ELEVEN | CODE128 |
| 萊爾富 / OK | CODE39（預設） |

若掃描/渲染異常，可於 Debug 區手動覆寫單張卡片格式（待未來需求）。

---

## IndexedDB Schema v2

### `cards`
```
id, pair[], store, denomination, balance, folderId,
archived, scanOrder, barcodeFormat, timestamp
```

### `folders`
```
id ('default'), name, isDefault
```

### `settings`
```
key, value  // scanCooldownMs, scanIntervalMs, customDenominations, ...
```

### Migration
- v1 `{ pair, timestamp }` → v2 補上 store=familymart、denomination=100、balance=100、folderId=default、archived=false、barcodeFormat=CODE39、scanOrder=依載入順序
