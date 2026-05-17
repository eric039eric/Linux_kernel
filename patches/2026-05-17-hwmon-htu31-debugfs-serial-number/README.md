# Patch 記錄：docs: hwmon: htu31: document debugfs serial_number

## 基本資訊

| 項目 | 內容 |
|---|---|
| 日期 | 2026-05-17 |
| Patch 標題 | `[PATCH] docs: hwmon: htu31: document debugfs serial_number` |
| 子系統 | hwmon（Hardware Monitoring）|
| 類型 | Documentation fix |
| 作者 | Chen-Shi-Hong <eric039eric@gmail.com> |
| Maintainer | Guenter Roeck <linux@roeck-us.net> |
| 結果 | **Applied** ✅ |

---

## 背景：為什麼會找到這個問題

在閱讀 `Documentation/hwmon/htu31.rst` 文件的過程中，發現文件只描述了 sysfs 介面的屬性，
但 `drivers/hwmon/htu31.c` 裡實際上還透過 `debugfs` 暴露了一個 `serial_number` 欄位，
用來讀取感測器的出廠序號，文件卻完全沒有提到這個介面。

這是一個典型的「文件與實作不一致」的問題：
- driver 有做 debugfs entry
- 文件只記錄了 sysfs
- 使用者或開發者無從得知 debugfs serial_number 的存在

---

## 分析過程

1. 閱讀 `Documentation/hwmon/htu31.rst`，理解現有文件的結構。
2. 對照 `drivers/hwmon/htu31.c`，確認 driver 有沒有暴露比文件更多的介面。
3. 發現 driver 用 `debugfs_create_blob()` 建立了 `serial_number` entry。
4. 確認 debugfs 不是 sysfs，兩者掛載路徑與用途不同：
   - sysfs：`/sys/class/hwmon/hwmon*/`
   - debugfs：`/sys/kernel/debug/htu31-*/`
5. 判斷這是文件缺漏，而不是 driver bug，因此應補充至文件而非修改 code。

---

## Patch 內容說明

在 `Documentation/hwmon/htu31.rst` 的適當位置，補充一個 debugfs section，說明：

- `serial_number` 這個 debugfs 屬性的存在
- 它的格式（8-byte binary blob，來自感測器的製造序號）
- 它與 sysfs interface 的區別
- 掛載路徑說明（`/sys/kernel/debug/htu31-*/serial_number`）

這樣做的目的是讓使用者或下游開發者在查閱文件時，能完整知道這個 driver 提供了哪些介面，
不只是 sysfs，也包含 debugfs。

---

## 送出流程

1. 本地修改 `Documentation/hwmon/htu31.rst`
2. 執行 `scripts/checkpatch.pl` 確認格式無問題
3. 執行 `scripts/get_maintainer.pl` 確認收件人：
   - Guenter Roeck（hwmon maintainer）
   - linux-hwmon@vger.kernel.org
   - linux-doc@vger.kernel.org
   - linux-kernel@vger.kernel.org
4. 用 `git format-patch` 產生 patch 檔
5. 用 `git send-email` 寄出

---

## Maintainer 回覆

寄出後約隔天，Guenter Roeck 回覆：

```
Applied.

Thanks,
Guenter
```

這代表 patch 已被 hwmon maintainer 接受進 hwmon tree，
後續會隨 hwmon subsystem 的 pull request 整合進 Linus 的主線。

---

## 這次學到什麼

- `debugfs` 和 `sysfs` 是不同的介面，不能混為一談。
- driver 可能同時暴露 sysfs 和 debugfs，文件要兩個都涵蓋。
- 「文件只記錄了部分介面」這種缺漏，對 hwmon 來說是很可以貢獻的切入點。
- maintainer 對 documentation patch 的接受流程通常很快，沒有複雜的技術討論。
- 小而明確、可驗證的 patch，比大而模糊的修改更容易被 applied。

---

## 與上游的連結

- [lore.kernel.org patch thread](https://lore.kernel.org/linux-hwmon/?q=htu31+debugfs+serial_number)
- Maintainer：Guenter Roeck <linux@roeck-us.net>
- 所屬子系統：`Documentation/hwmon/`
