# Patch #4 — reporting-issues: reroll as v3 on the correct base

**日期：** 2026-05-14  
**類型：** Documentation patch reroll / v3  
**目標檔案：** `Documentation/admin-guide/reporting-issues.rst`  
**Subject：** `[PATCH v3] docs: reporting-issues: replace "these advices" with "all of this advice"`  
**Branch：** `shihong-reporting-issues-v3-clean`

---

## 這次發生了什麼

這次原本我以為要送的是一個 follow-up patch，也就是在前一版
`this advice` 的基礎上，再改成 `all of this advice`。

但 Thorsten 在回覆中指出，前一版 patch 其實還沒有被套用，因此這次不應該送成另一個獨立的 follow-up patch，而是應該回到相同 base，把原 patch 重新整理後以 **v3** 的形式重新寄出。

換句話說，這次最重要的學習不是只有改字句，而是理解：

- 什麼時候該送 follow-up patch
- 什麼時候該做 reroll（v2 / v3 / v4）
- 「same base」在 mailing list workflow 裡的重要性

---

## reviewer 的重點提醒

Thorsten 的意思大致上是：

- 先前討論的是同一個 patch 主題
- 那個 patch 尚未被套用
- 所以這次不應該在 `this advice` 上再額外疊一顆 patch
- 正確方式是回到原始 base，直接把 `these advices` 改成 `all of this advice`
- 然後把這個版本作為 **v3** 重送

這讓我第一次真正理解，patch revision 並不只是把標題改成 v2/v3，而是 **commit 歷史與 diff base 也要正確**。

---

## 這次真正送出的修改

這次在 `Documentation/admin-guide/reporting-issues.rst` 中，直接把兩處：

```diff
- Ignoring these advices will dramatically
+ Ignoring all of this advice will dramatically
```

分別出現在：

- 正文段落一處
- TL;DR 對應段落一處

也就是說，這次最終 v3 的 diff 是從原本錯誤的 `these advices` 直接改到 `all of this advice`，而不是從 `this advice` 再往上疊一層。

---

## 我一開始犯的錯

我一開始是從先前的 follow-up branch 再切出新的 branch，因此 branch history 裡同時包含：

1. `these advices` -> `this advice`
2. `this advice` -> `all of this advice`

這樣的 commit 歷史雖然最終文字看起來正確，但對 mailing list reviewer 來說，它代表的是「另外一顆 follow-up patch」，而不是「同一顆 patch 的 v3」。

這也是這次最關鍵的錯誤點：

- **工作樹內容正確，不代表 patch revision 的歷史正確**
- 寄 v3 時，重要的不只是最後檔案長什麼樣子，還包含這顆 patch 是不是建立在正確 base 上

---

## 正確修正方式

後來我重新從 `master` 切出乾淨 branch：

```bash
git switch master
git switch -c shihong-reporting-issues-v3-clean
```

然後先確認檔案裡還是原本的 `these advices`：

```bash
grep -n "these advices\|this advice\|all of this advice" Documentation/admin-guide/reporting-issues.rst
```

確認 base 沒問題後，才把那兩處一次改成 `all of this advice`，並建立新的單一 commit。

---

## 這次使用的 commit message

```text
docs: reporting-issues: replace "these advices" with "all of this advice"

"Advice" is an uncountable noun, so "these advices" is grammatically
incorrect.

Replace it with "all of this advice" instead, which keeps the sentence
grammatical while also making it clear that it refers to the full set of
recommendations in the paragraph.

Signed-off-by: Chen-Shi-Hong <eric039eric@gmail.com>
```

這個版本比前一次更精準，因為它直接描述：

- 原來的錯誤字串是什麼
- 改成什麼
- 為什麼不是只改成 `advice`，而是改成 `all of this advice`

---

## 這次學到的 Git / mailing list 重點

### 1. `v3` 不是只是改主旨

這次最重要的學習之一，就是 `v2` / `v3` 不只是 subject line 的版本號而已。  
真正的重點是：

- 同一個 patch 主題
- 同一個合理 base
- 同一個 revision 線

如果 base 不對，即使內容看起來對，送出去也可能被視為流程不正確。

### 2. follow-up patch 和 reroll patch 不一樣

這次讓我更清楚分辨：

- **follow-up patch**：前一顆 patch 已經是既定基礎，後續再補送獨立修正
- **reroll patch（v2/v3）**：前一顆 patch 尚未被採用，因此應直接更新原 patch 內容並重送新版

### 3. `git log` 比 `grep` 更能看出 patch 歷史問題

一開始只看檔案內容時，會以為句子已經正確。  
但真正讓我發現問題的是 `git log master..`，因為它直接顯示在 `master` 之上其實疊了兩顆相關 commit。

這讓我學到：

- `grep` 只能確認目前文字
- `git show` 能看單顆 commit 的 diff
- `git log master..` 才能幫我判斷目前 branch 相對 base 到底多了哪些 commit

### 4. Author 和 Signed-off-by 要一致

我這次 commit 完後發現：

- `Author` 顯示成 `eric039eric`
- `Signed-off-by` 也顯示成 `eric039eric`

但正式送 kernel patch 時，我希望使用的是：

`Chen-Shi-Hong <eric039eric@gmail.com>`

所以我先改 repo-local Git identity：

```bash
git config user.name "Chen-Shi-Hong"
git config user.email "eric039eric@gmail.com"
```

再用下面這個指令重寫最後一顆 commit：

```bash
git commit --amend --reset-author -s
```

這樣就能同時修正：

- Author
- Signed-off-by

而且不用重新做整顆 patch。

### 5. `format-patch -v3` 的用途

我用：

```bash
git format-patch -v3 -1 HEAD
```

來產生 patch 檔。  
這個 `-v3` 會把 reroll count 加進主旨中，讓寄出去的 patch subject 變成：

```text
[PATCH v3] docs: reporting-issues: replace "these advices" with "all of this advice"
```

這正是 reviewer 希望看到的 revision 形式。

### 6. checkpatch warning 也可能是 false positive

這次 `checkpatch.pl` 仍然出現：

```text
WARNING: 'advices' may be misspelled - perhaps 'advice'?
```

但這次 warning 出現在：

- subject line
- commit message 內文

因為我是在描述原本的錯誤字串 `"these advices"`，所以這個 warning 並不是 diff 本身有問題，而是 checkpatch 在掃描文字時把它當成一般拼字錯誤。

這讓我知道：

- checkpatch 很重要
- 但 warning 不能機械式地全盤接受
- 還要判斷它是不是來自 commit message 說明文字的 false positive

### 7. `get_maintainer.pl` 再次確認收件人

這次我也再次用：

```bash
./scripts/get_maintainer.pl v3-0001-docs-reporting-issues-replace-these-advices-with-.patch
```

確認收件人，結果和前一次相同，包含：

- Thorsten Leemhuis
- Jonathan Corbet
- Shuah Khan
- linux-doc
- linux-kernel

這讓我更熟悉 patch 送出前應該再驗證一次 maintainer / mailing list，而不是完全依賴記憶。

### 8. 用 `Message-ID` 把 v3 掛回原討論串

這次還學到另一個重要技巧：  
當新版 patch 要掛回原本 thread 時，不能只靠主旨相同，應該使用原始 patch 郵件的 `Message-ID` 搭配：

```bash
git send-email --in-reply-to="<message-id>" ...
```

這樣新的 `[PATCH v3]` 才會正確接在原本 thread 下面，方便 maintainer 和 reviewer 追蹤整個討論過程。

---

## 這次實際送出的流程

```bash
git switch master
git switch -c shihong-reporting-issues-v3-clean

grep -n "these advices\|this advice\|all of this advice" Documentation/admin-guide/reporting-issues.rst

nano Documentation/admin-guide/reporting-issues.rst
git diff -- Documentation/admin-guide/reporting-issues.rst

git add Documentation/admin-guide/reporting-issues.rst
git commit -s

git config user.name "Chen-Shi-Hong"
git config user.email "eric039eric@gmail.com"
git commit --amend --reset-author -s

git format-patch -v3 -1 HEAD

./scripts/checkpatch.pl --strict --codespell v3-0001-docs-reporting-issues-replace-these-advices-with-.patch
./scripts/get_maintainer.pl v3-0001-docs-reporting-issues-replace-these-advices-with-.patch

git send-email \
  --annotate \
  --in-reply-to="<20260512150431.894-1-eric039eric@gmail.com>" \
  --to="linux@leemhuis.info" \
  --cc="corbet@lwn.net" \
  --cc="skhan@linuxfoundation.org" \
  --cc="linux-doc@vger.kernel.org" \
  --cc="linux-kernel@vger.kernel.org" \
  v3-0001-docs-reporting-issues-replace-these-advices-with-.patch
```

---

## 送出結果

這次 `git send-email` 最後成功收到：

```text
250
```

代表 SMTP 伺服器已接受郵件，patch 已成功送出。

---

## 這次對我最重要的收穫

這次最大的收穫，不只是又成功送出一封 patch，而是第一次真正理解 Linux kernel mailing list workflow 裡面「revision」的概念：

- 不是每次有新想法都送一顆 follow-up patch
- 有時候 reviewer 要的是回到原 base 重做成 v2 / v3
- diff 的內容、commit 歷史、thread 關聯方式，其實都同樣重要

這讓我比前一次更接近真正理解 upstream patch submission 的工作方式，而不只是照步驟操作。