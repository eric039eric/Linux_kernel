# Patch #1：修正 howto.rst 中 Shepherd 拼字錯誤

**日期**：2026-05-12  
**子系統**：`Documentation/process/`  
**檔案**：`Documentation/process/howto.rst`  
**Commit**：`bfcbd2ed4ce9`  
**Message-ID**：`<20260512000946.3234-1-eric039eric@gmail.com>`  
**最終狀態**：❌ Patch 不成立 — maintainer 指出 Shepard 可能是正確人名，已回信撤回

---

## 問題描述

用 `codespell` 掃描 `Documentation/process/howto.rst`，發現第 619 行：

```
David A. Wheeler, Junio Hamano, Michael Kerrisk, and Alex Shepard for
```

`Shepard` 看起來像是 `Shepherd` 的拼字錯誤。

用 `checkpatch.pl` 確認後，排除 SPDX warning（Documentation 檔案本來就不需要 SPDX），
只剩這一個 CHECK，判斷是真實問題後送出。

---

## 修改內容

```diff
-David A. Wheeler, Junio Hamano, Michael Kerrisk, and Alex Shepard for
+David A. Wheeler, Junio Hamano, Michael Kerrisk, and Alex Shepherd for
```

---

## 完整操作流程

### 1. 用工具找問題

```bash
codespell Documentation/process/howto.rst
# Documentation/process/howto.rst:619: Shepard ==> Shepherd

./scripts/checkpatch.pl --strict --codespell --file Documentation/process/howto.rst
# CHECK: 'Shepard' may be misspelled - perhaps 'Shepherd'?
```

### 2. 確認上下文

```bash
sed -n '610,625p' Documentation/process/howto.rst
```

確認是致謝名單中的人名拼寫，判斷可以修改。

### 3. 用 nano 直接修改

```bash
nano +619 Documentation/process/howto.rst
```

### 4. 確認修改正確

```bash
git diff -- Documentation/process/howto.rst
codespell Documentation/process/howto.rst
# 警告消失
```

### 5. Commit

```bash
git add Documentation/process/howto.rst
git commit -s
```

Commit message：

```
docs: fix spelling of Shepherd in howto

Correct the spelling of "Shepherd" in the acknowledgements section of
Documentation/process/howto.rst.

Signed-off-by: Chen-Shi-Hong <eric039eric@gmail.com>
```

### 6. 產生 patch 檔

```bash
mkdir -p ~/patches
git branch --set-upstream-to=origin/master shihong-first-doc-patch
git format-patch -1 -o ~/patches --base=auto
# /home/eric0/patches/0001-docs-fix-spelling-of-Shepherd-in-howto.patch
```

### 7. 找 maintainer

```bash
./scripts/get_maintainer.pl ~/patches/0001-*.patch
```

輸出：

```
Jonathan Corbet <corbet@lwn.net>          (maintainer:DOCUMENTATION PROCESS)
Shuah Khan <skhan@linuxfoundation.org>    (reviewer:DOCUMENTATION PROCESS)
workflows@vger.kernel.org                (open list:DOCUMENTATION PROCESS)
linux-doc@vger.kernel.org                (open list:DOCUMENTATION)
linux-kernel@vger.kernel.org             (open list)
```

### 8. 設定 git send-email（Gmail SMTP）

```bash
git config --global sendemail.smtpserver smtp.gmail.com
git config --global sendemail.smtpserverport 587
git config --global sendemail.smtpencryption tls
git config --global sendemail.smtpuser eric039eric@gmail.com
git config --global sendemail.from "Chen-Shi-Hong <eric039eric@gmail.com>"
git config --global sendemail.confirm auto
```

> 注意：Gmail 需要使用「App Password」而非帳號密碼。
> 路徑：Google 帳號 → 安全性 → 兩步驟驗證 → 應用程式密碼

### 9. 先寄給自己測試

```bash
git send-email --dry-run --to eric039eric@gmail.com ~/patches/0001-*.patch
# Dry-OK. Result: OK

git send-email --to eric039eric@gmail.com ~/patches/0001-*.patch
# Result: 250（成功）
```

### 10. 正式寄出

```bash
git send-email \
  --to corbet@lwn.net \
  --cc skhan@linuxfoundation.org \
  --cc workflows@vger.kernel.org \
  --cc linux-doc@vger.kernel.org \
  --cc linux-kernel@vger.kernel.org \
  ~/patches/0001-*.patch

# Result: 250（成功）
```

---

## Maintainer Review（2026-05-12）

寄出約 23 分鐘後，收到 Randy Dunlap（`rdunlap@infradead.org`）回信。  
信件同時寄給 corbet、skhan、workflows、linux-doc、linux-kernel。

**Randy 的問題：**

> How do you know that the last name is misspelled?  
> I found email from "Alex Shepard" on lore.kernel.org/all/ but none from Alex Shepherd.  
> Names can often be spelled many ways.

他在 [lore.kernel.org](https://lore.kernel.org/all/1137702713.3205.43.camel@athena.sea.amer.gettywan.com/) 找到了 `Alex Shepard` 本人寄過信的紀錄，也就是說 `Shepard` 本來就是這個人的真實姓氏，不是拼字錯誤。

**我的回信：**

```
Hi Randy,

Thanks for pointing that out.

I based the change mainly on the codespell/checkpatch suggestion and
did not verify the person's actual last name before sending the patch.

Given your note and the lore reference for Alex Shepard, this change
was not sufficiently justified. Please ignore this patch.

Thanks,
Chen-Shi-Hong
```

---

## 遇到的問題與解法

| 問題 | 原因 | 解法 |
|------|------|------|
| `git send-email` 密碼錯誤（5.7.8 BadCredentials） | 用了 Gmail 帳號密碼，不是 App Password | 到 Google 帳號設定產生 App Password |
| `git format-patch --base=auto` 失敗 | branch 沒有設定 upstream | 先執行 `git branch --set-upstream-to=origin/master` |
| `git log --online` 指令錯誤 | 打錯，`--online` 不存在 | 正確是 `--oneline` |
| 第一次 `apt install` 失敗 | `build-essentail` 打錯字 | 正確是 `build-essential` |
| **Patch 不成立** | **只靠 codespell 判斷，沒驗證人名來源** | **見下方 Lesson Learned** |

---

## 這次學到的觀念

### 流程類

- **kernel 協作不用 GitHub PR**，而是用 `git send-email` 寄 patch 給 maintainer 和 mailing list
- **`Signed-off-by` 是必要的**，用 `git commit -s` 自動加入，代表你同意 DCO（Developer Certificate of Origin）
- **patch 要小且單一目的**，maintainer 看一眼就能判斷對不對
- **`checkpatch.pl` 是寄出前的自我檢查工具**，要先過它才送
- **`get_maintainer.pl` 幫你找對人**，不要自己猜要寄給誰
- **App Password ≠ 帳號密碼**，Gmail 若有兩步驟驗證，`git send-email` 要用 App Password

### ⚠️ Lesson Learned：人名不能只靠工具判斷

這是這次最重要的教訓：

> **`codespell` 和 `checkpatch.pl` 是提示工具，不是最終裁判。**

`codespell` 把 `Shepard` 標記為疑似拼錯，但這只是「字典比對」的結果，
工具沒有能力判斷它是不是某個人的真實姓氏。

**正確流程是**：
1. 工具找到候選 → 先不要立刻改
2. 確認這個詞是什麼性質：是一般英文單字？還是人名、專有名詞、縮寫？
3. 如果是人名 → **去 lore.kernel.org 或 Git log 查這個人有沒有在 kernel 社群活動過**
4. 有找到本人名字拼法的紀錄 → 以該紀錄為準，不要改
5. 確定是真的拼錯 → 才能送 patch

**可安全修改的 typo 類型：**
- 明確的英文常用字拼錯（如 `recieve` → `receive`）
- 技術術語錯誤（如 `interupt` → `interrupt`）
- 文法錯誤、句子結構明確有問題

**不能只靠工具判斷的類型：**
- 人名（姓、名都可能有多種拼法）
- 地名
- 縮寫
- 引用自外部文件的專有名詞

---

## 最終結果與心得

這個 patch 最終不成立，但這次的整體經驗非常有價值：

- ✅ 完整走通了 kernel patch 的提交流程（codespell → checkpatch → commit → format-patch → send-email）
- ✅ 真的收到 maintainer 的 review 回信，不是被靜音忽略
- ✅ 理解了如何用專業方式回覆 review，承認問題並請對方忽略
- ✅ 學到了最重要的 lesson：**工具是輔助，人名類 typo 一定要先驗證來源**

第一個 patch 雖然不成立，但整個流程都是真實的。  
這比「送出後沒人理」更有收穫。

---

## 下一步

- [x] 寄出 patch v1（2026-05-12）
- [x] 收到 Randy Dunlap 的 review，指出人名問題
- [x] 回信承認錯誤，請對方忽略此 patch
- [ ] 找下一個 Documentation patch 候選，這次先用 `lore.kernel.org` 驗證人名
- [ ] 學習 `drivers/iio/` 的 cleanup 類型 patch
- [ ] 學習如何回覆 review email 並送出 `[PATCH v2]`
