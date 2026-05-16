# Patch Log: Documentation: hwmon: lm75: document sysfs interface

**日期**：2026-05-16 ~ 2026-05-17  
**作者**：Chen-Shi-Hong <eric039eric@gmail.com>  
**Branch**：`docs-hwmon-lm75-sysfs-doc`  
**目標**：為 `Documentation/hwmon/lm75.rst` 補上 sysfs-Interface 段落，記錄 driver 所支援的 sysfs 屬性。  
**最終版本**：v3（已寄出至 linux-hwmon mailing list）

---

## 背景

lm75 是 Linux kernel 中一個支援多種 I2C 溫度感測器的 hwmon driver，位於 `drivers/hwmon/lm75.c`。  
其 rst 文件（`Documentation/hwmon/lm75.rst`）當時缺少任何關於 sysfs 屬性的說明，只有晶片支援列表和基本介紹。  
這次 patch 的目標是補上一個 `sysfs-Interface` 段落，讓使用者能從文件就知道這顆 driver 會在 sysfs 露出哪些屬性。

---

## 工作流程總覽

```
確認問題
  → 找到 lm75.rst 缺少 sysfs 說明
  → 查閱 lm75.c 確認實際 sysfs 介面
  → 撰寫 patch（v1）
  → 送出 v1，等待 review
  → 收到 sashiko-bot 的意見（temp1_label）
  → 查 lm75.c 驗證 temp1_label 是否確實存在
  → 確認存在且為條件式屬性
  → 補上 temp1_label，更新 commit message，送出 v2
  → 收到 Guenter 回覆，指出缺少 changelog
  → 將 changelog 補到 --- 下方（正確格式）
  → 送出 v3（修正 changelog 位置）
```

---

## 詳細步驟記錄

### 步驟 1：確認 lm75.rst 缺少 sysfs-Interface 段落

```bash
cat Documentation/hwmon/lm75.rst
```

確認原始文件的最後一行是：
```
Both chips are simply not compatible, value encoding differs.
```
完全沒有任何 sysfs 屬性說明。

---

### 步驟 2：查閱 lm75.c 確認 sysfs 介面

```bash
grep -n 'HWMON_T' drivers/hwmon/lm75.c
grep -n 'hwmon_temp' drivers/hwmon/lm75.c
grep -n 'update_interval' drivers/hwmon/lm75.c
```

確認 driver 實際支援的屬性：

| 屬性 | 說明 |
|---|---|
| `temp1_input` | 溫度讀值 |
| `temp1_max` | 最高溫度設定 |
| `temp1_max_hyst` | 最高溫度 hysteresis |
| `temp1_alarm` | 溫度警報（部分晶片支援） |
| `temp1_label` | 溫度通道標籤（有提供 label 時才存在） |
| `update_interval` | 更新間隔（寫入權限依晶片而定） |

---

### 步驟 3：建立開發分支

```bash
git checkout -b docs-hwmon-lm75-sysfs-doc
```

---

### 步驟 4：修改 Documentation/hwmon/lm75.rst

在文件最後加入 sysfs-Interface 段落：

```rst
sysfs-Interface
---------------

================ ============================================
temp1_input      temperature input
temp1_max        maximum temperature
temp1_max_hyst   maximum temperature hysteresis
================ ============================================

If a label is provided for the device, the following attribute is also
available:

================ ============================================
temp1_label      temperature channel label
================ ============================================

If supported by the chip, the following attribute is also available:

================ ============================================
temp1_alarm      temperature alarm
================ ============================================

The standard update_interval attribute is also supported. Its write
permissions depend on the chip.
```

---

### 步驟 5：checkpatch 驗證

```bash
git diff --cached | ./scripts/checkpatch.pl --strict --codespell -
```

結果：
```
total: 0 errors, 0 warnings, 0 checks, 13 lines checked
Your patch has no obvious style problems and is ready for submission.
```

---

### 步驟 6：確認收件人

```bash
./scripts/get_maintainer.pl Documentation/hwmon/lm75.rst
```

確認收件人：
- **TO**：`linux@roeck-us.net`（hwmon maintainer Guenter Roeck）
- **CC**：`corbet@lwn.net`, `skhan@linuxfoundation.org`, `linux-hwmon@vger.kernel.org`, `linux-doc@vger.kernel.org`, `linux-kernel@vger.kernel.org`

---

### 步驟 7：建立 commit（v1 版本）

commit message：

```
Documentation: hwmon: lm75: document sysfs interface

Document the sysfs attributes supported by the lm75 driver.

The driver exposes temp1_input, temp1_max, temp1_max_hyst, and the
standard update_interval attribute. Some chips also expose temp1_alarm.

Add a sysfs-Interface section to Documentation/hwmon/lm75.rst to
describe the supported attributes and clarify that temp1_alarm and
the write permissions of update_interval depend on the chip.

Signed-off-by: Chen-Shi-Hong <eric039eric@gmail.com>
```

---

### 步驟 8：產生 v1 patch 並送出

```bash
git format-patch -1
git send-email --dry-run \
  --to linux@roeck-us.net \
  --cc corbet@lwn.net \
  --cc skhan@linuxfoundation.org \
  --cc linux-hwmon@vger.kernel.org \
  --cc linux-doc@vger.kernel.org \
  --cc linux-kernel@vger.kernel.org \
  0001-Documentation-hwmon-lm75-document-sysfs-interface.patch
```

**v1 Message-ID**：`<20260516160823.1461-1-eric039eric@gmail.com>`

dry-run 正常後正式送出。

---

### 步驟 9：收到 sashiko-bot 的回覆

bot 提示：`temp1_label` 屬性未被文件記錄。

進行驗證：

```bash
grep -n 'temp_label\|HWMON_T_LABEL\|hwmon_temp_label' drivers/hwmon/lm75.c
grep -n 'label' drivers/hwmon/lm75.c
```

確認 `lm75.c` 中存在：
- `HWMON_T_LABEL`
- `hwmon_temp_label`
- `device_property_read_string(dev, "label", &data->label)`
- 條件式：`return config_data->label ? 0444 : 0`

結論：`temp1_label` 確實存在，但只在裝置有 label 屬性時才會出現，是**條件式屬性**。

---

### 步驟 10：更新文件並送出 v2

補上 `temp1_label` 的條件式說明，更新 commit message 加入 `temp1_label` 相關敘述：

```bash
git commit --amend
git format-patch -1 -v2
git send-email --dry-run \
  --in-reply-to="<20260516160823.1461-1-eric039eric@gmail.com>" \
  --to linux@roeck-us.net \
  --cc corbet@lwn.net \
  --cc skhan@linuxfoundation.org \
  --cc linux-hwmon@vger.kernel.org \
  --cc linux-doc@vger.kernel.org \
  --cc linux-kernel@vger.kernel.org \
  v2-0001-Documentation-hwmon-lm75-document-sysfs-interface.patch
```

**v2 Message-ID**：`<20260516163825.1767-1-eric039eric@gmail.com>`

dry-run 確認 `[PATCH v2]` 主旨、`In-Reply-To` 指向 v1、`Result: OK` 後正式送出。

---

### 步驟 11：收到 Guenter 的回覆

回覆內容：

```
Change log is missing.

Yes, I do see Sashiko's reply, but that is not an excuse for neglecting
to provide a change log.
```

問題：v2 的 commit message 雖然有補到 `temp1_label`，但沒有提供 **revision changelog**（說明相比前版改了什麼）。

---

### 步驟 12：修正 changelog 位置並送出 v3

kernel patch 的 revision changelog 應放在 `Signed-off-by` 之後、`---` 之下，而不是 commit message body 裡。

操作流程：

```bash
# 1. amend 把 commit message 改回乾淨版（不含 changelog）
git commit --amend

# 2. 重新產生 v3 patch
rm v3-0001-Documentation-hwmon-lm75-document-sysfs-interface.patch
git format-patch -1 -v3

# 3. 手動在 patch 檔的 --- 下方加入 changelog
nano v3-0001-Documentation-hwmon-lm75-document-sysfs-interface.patch
```

在 `---` 下方加入：

```
Changes in v2:
- Document temp1_label as conditionally available when a device label is
  provided.

Changes in v3:
- Add changelog requested during review.
```

```bash
# 4. dry-run 確認
git send-email --dry-run \
  --in-reply-to="<20260516160823.1461-1-eric039eric@gmail.com>" \
  --to linux@roeck-us.net \
  --cc corbet@lwn.net \
  --cc skhan@linuxfoundation.org \
  --cc linux-hwmon@vger.kernel.org \
  --cc linux-doc@vger.kernel.org \
  --cc linux-kernel@vger.kernel.org \
  v3-0001-Documentation-hwmon-lm75-document-sysfs-interface.patch
```

dry-run 確認：
- `Subject: [PATCH v3] Documentation: hwmon: lm75: document sysfs interface`
- `In-Reply-To: <20260516160823.1461-1-eric039eric@gmail.com>`
- `Result: OK`

**正式送出 v3**。  
**v3 Message-ID**：`<20260516165449.1887-1-eric039eric@gmail.com>`（接回原 thread）

---

## 最終 patch 內容（v3 diff）

```diff
diff --git a/Documentation/hwmon/lm75.rst b/Documentation/hwmon/lm75.rst
index 4269da04508e..fa8ddcaa0c2b 100644
--- a/Documentation/hwmon/lm75.rst
+++ b/Documentation/hwmon/lm75.rst
@@ -181,3 +181,28 @@ is supported by this driver, other specific enhancements are not.
 
  The LM77 is not supported, contrary to what we pretended for a long time.
  Both chips are simply not compatible, value encoding differs.
+
+sysfs-Interface
+---------------
+
+================ ============================================
+temp1_input      temperature input
+temp1_max        maximum temperature
+temp1_max_hyst   maximum temperature hysteresis
+================ ============================================
+
+If a label is provided for the device, the following attribute is also
+available:
+
+================ ============================================
+temp1_label      temperature channel label
+================ ============================================
+
+If supported by the chip, the following attribute is also available:
+
+================ ============================================
+temp1_alarm      temperature alarm
+================ ============================================
+
+The standard update_interval attribute is also supported. Its write
+permissions depend on the chip.
```

---

## 這次學到的事情

### 技術層面

- 怎麼查 kernel driver 的實際 sysfs 介面（grep HWMON_T_*、hwmon_temp_*）
- rst 文件的表格語法（`=== ===` 格式）
- `checkpatch.pl --strict --codespell` 的用途與意義
- `get_maintainer.pl` 怎麼找正確收件人

### patch 流程層面

- `git format-patch -1`：產生 patch 檔
- `git format-patch -1 -vN`：產生 vN 版本的 patch（reroll count）
- `git send-email --dry-run`：模擬寄信但不真的送出
- `--in-reply-to="<Message-ID>"`：讓新版本 patch 接回原 thread
- `git commit --amend`：修改已有的 commit 內容或 message
- 版本號（v1/v2/v3）是根據「寄出次數」計算，不是本地修改次數

### review 流程層面

- sashiko-bot：自動化 review bot，會提示文件可能遺漏的屬性
- revision changelog 應放在 `---` 下方，不是 commit message body
- v2 之後每版都需要 `Changes in vN:` 說明相比前版改了什麼
- `In-Reply-To` 讓 reviewer 能在同一條 thread 看到所有版本的演進

---

## mailing list 討論串

| 版本 | Message-ID | 說明 |
|---|---|---|
| v1 | `<20260516160823.1461-1-eric039eric@gmail.com>` | 初版，缺少 `temp1_label` |
| v2 | `<20260516163825.1767-1-eric039eric@gmail.com>` | 補上 `temp1_label`，缺少 changelog |
| v3 | `<20260516165449.1887-1-eric039eric@gmail.com>` | 補上 changelog，格式修正 |

---

## 結論

這次 patch 雖然只是一個文件新增（`Documentation/hwmon/lm75.rst` 加一個段落），  
但走完了完整的 upstream patch submission 流程：  
從找問題 → 驗證 → 寫 patch → 檢查格式 → 送出 → 收 review → 修版本 → 再送出。  

這個流程本身，就是 Linux kernel contribution 最核心的工作循環。
