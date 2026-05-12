# Patch #3 — Follow-up: reporting-issues: clarify advice wording

**日期：** 2026-05-12  
**類型：** Documentation follow-up patch  
**目標檔案：** `Documentation/admin-guide/reporting-issues.rst`  
**Subject：** `[PATCH] docs: reporting-issues: clarify advice wording`  
**Branch：** `shihong-reporting-issues-followup`

---

## 這個 patch 做了什麼

這是針對 Patch #2（修正 `advices` → `advice`）的後續 follow-up patch。

Patch #2 雖然修正了不可數名詞問題，但 `this advice` 的措辭可能讓人誤解為只指前面某一條建議，語意範圍太窄。這個 follow-up patch 將其改為 `all of this advice`，明確表示涵蓋整段落的所有建議。

### 修改內容

檔案中有兩處需修改（同一段內容的正文版與 TL;DR 版各一處）：

```diff
-   ideally use a 'vanilla' build. Ignoring this advice will dramatically
+   ideally use a 'vanilla' build. Ignoring all of this advice will dramatically
```

```diff
-    ideally use a 'vanilla' built. Ignoring this advice will dramatically
+    ideally use a 'vanilla' built. Ignoring all of this advice will dramatically
```

---

## 這次學到的事情

### 1. follow-up patch 的定義

- follow-up patch 是在前一個 patch 送出並收到 review 討論後，再補送的後續小修補。
- 它和 v2 不同：v2 是原 patch 尚未合併時的改版；follow-up patch 則是在前一個 patch 討論基礎上，針對相關點再補一刀。
- 送 follow-up patch 不代表前一個 patch 有根本性問題，有時候只是 wording 細節需要再精確一次。

### 2. `advice` 是不可數名詞

- 英文 `advice` 本身是不可數名詞，沒有複數形 `advices`。
- 但 `this advice` 有時會被理解成只指某一條建議，若想強調「以上所有建議」，用 `all of this advice` 更準確。
- 這種語意細節是 reviewer 才會特別指出的，自己讀文件不一定會注意到。

### 3. checkpatch 的 false positive

`checkpatch.pl` 跑完後出現 warning：

```
WARNING: 'advices' may be misspelled - perhaps 'advice'?
```

這是因為 commit message 內文裡故意提到舊的錯誤字 `these advices` 作為說明背景，checkpatch 把它當成一般拼字掃描，所以誤報。這種情況在文件類 patch 的 commit message 裡很常見，不需要因為這個 warning 改掉 commit message，可以直接送出。

### 4. 送出流程（和上次完全一樣）

```bash
# 建立新 branch
git switch -c shihong-reporting-issues-followup

# 確認要修改的行數
grep -n "Ignoring" Documentation/admin-guide/reporting-issues.rst

# 編輯檔案
nano Documentation/admin-guide/reporting-issues.rst

# 確認 diff
git diff -- Documentation/admin-guide/reporting-issues.rst

# stage + commit（-s 自動加 Signed-off-by）
git add Documentation/admin-guide/reporting-issues.rst
git commit -s

# 用 patch 檔跑 checkpatch（不是用 HEAD~1）
./scripts/checkpatch.pl --strict --codespell 0001-xxx.patch

# 找收件人
./scripts/get_maintainer.pl Documentation/admin-guide/reporting-issues.rst

# 產生 patch
git format-patch -1

# 送出
git send-email \
  --to="linux@leemhuis.info" \
  --cc="corbet@lwn.net" \
  --cc="skhan@linuxfoundation.org" \
  --cc="linux-doc@vger.kernel.org" \
  --cc="linux-kernel@vger.kernel.org" \
  0001-docs-reporting-issues-clarify-advice-wording.patch
```

### 5. `git send-email` 成功的訊號

送出成功時，log 最後一行會顯示：

```
Result: 250
```

SMTP 250 代表伺服器已接受郵件，patch 送達。

---

## Commit Message

```
docs: reporting-issues: clarify advice wording

A previous change replaced "these advices" with "this advice", but that
wording can be read too narrowly and may seem to refer only to a single
recommendation.

Use "all of this advice" instead to make it clearer that the sentence
refers to the broader set of recommendations in the paragraph.

Signed-off-by: Chen-Shi-Hong <eric039eric@gmail.com>
```

---

## 送出的收件人

| 角色 | 信箱 |
|---|---|
| Maintainer (reporting-issues) | linux@leemhuis.info |
| Maintainer (Documentation) | corbet@lwn.net |
| Reviewer | skhan@linuxfoundation.org |
| Mailing list | linux-doc@vger.kernel.org |
| Mailing list | linux-kernel@vger.kernel.org |

---

## 狀態

- [x] 修改完成
- [x] checkpatch 通過（warning 為 false positive，可忽略）
- [x] `git format-patch` 產生 patch 檔
- [x] `git send-email` 送出，Result: 250
- [ ] 等待 maintainer review
