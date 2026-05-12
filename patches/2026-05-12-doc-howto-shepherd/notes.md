# Patch #1：修正 howto.rst 中 Shepherd 拼字錯誤

**日期**：2026-05-12  
**子系統**：`Documentation/process/`  
**檔案**：`Documentation/process/howto.rst`  
**Commit**：`bfcbd2ed4ce9`  
**Message-ID**：`<20260512000946.3234-1-eric039eric@gmail.com>`  
**狀態**：✅ 已正式寄出到 maintainer 與 mailing list

---

## 問題描述

用 `codespell` 掃描 `Documentation/process/howto.rst`，發現第 619 行：

```
David A. Wheeler, Junio Hamano, Michael Kerrisk, and Alex Shepard for
```

`Shepard` 應為 `Shepherd`（人名拼寫錯誤）。

用 `checkpatch.pl` 確認後，排除 SPDX warning（Documentation 檔案本來就不需要 SPDX），
只剩這一個 CHECK，確認是真實問題。

---

## 修改內容

```diff
-David A. Wheeler, Junio Hamano, Michael Kerrisk, and Alex Shepard for
+David A. Wheeler, Junio Hamano, Michael Kerrisk, and Alex Shepherd for
```

只改一個字，範圍最小，符合 kernel patch 的「一次只做一件事」原則。

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

確認是致謝名單中的人名拼寫錯誤，不是程式邏輯，可以安全修改。

### 3. 用 nano 直接修改

```bash
nano +619 Documentation/process/howto.rst
```

把 `Shepard` 改成 `Shepherd`，存檔離開。

### 4. 確認修改正確

```bash
git diff -- Documentation/process/howto.rst
# 確認只改了一行，內容正確

codespell Documentation/process/howto.rst
# 再掃一次，確認 Shepard 警告消失
```

### 5. Commit

```bash
git add Documentation/process/howto.rst
git status           # 確認 staged 正確
git diff --cached    # 再次確認 diff
git commit -s
```

Commit message（含 `Signed-off-by` 自動加入）：

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

## 遇到的問題與解法

| 問題 | 原因 | 解法 |
|------|------|------|
| `git send-email` 密碼錯誤（5.7.8 BadCredentials） | 用了 Gmail 帳號密碼，不是 App Password | 到 Google 帳號設定產生 App Password，用那組密碼 |
| `git format-patch --base=auto` 失敗 | branch 沒有設定 upstream | 先執行 `git branch --set-upstream-to=origin/master` |
| `git log --online` 指令錯誤 | 打錯，`--online` 不存在 | 正確是 `--oneline` |
| 第一次 `apt install` 失敗 | `build-essentail` 打錯字 | 正確是 `build-essential` |

---

## 這次學到的觀念

- **kernel 協作不用 GitHub PR**，而是用 `git send-email` 寄 patch 給 maintainer 和 mailing list
- **`Signed-off-by` 是必要的**，用 `git commit -s` 自動加入，代表你同意 DCO（Developer Certificate of Origin）
- **patch 要小且單一目的**，maintainer 看一眼就能判斷對不對
- **`checkpatch.pl` 是寄出前的自我檢查工具**，要先過它才送
- **`get_maintainer.pl` 幫你找對人**，不要自己猜要寄給誰
- **SPDX warning 在 Documentation 不用管**，那是 C 原始碼才需要的標頭
- **App Password ≠ 帳號密碼**，Gmail 帳號若有兩步驟驗證，`git send-email` 要用 App Password

---

## 下一步

- [ ] 等 maintainer 回信（如果有 review 意見，準備做 v2）
- [ ] 繼續找 `Documentation/` 或 `drivers/iio/` 的下一個小 patch
- [ ] 學習如何回覆 review email 並送出 `[PATCH v2]`
