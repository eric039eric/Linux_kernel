# Linux Kernel 協作流程筆記

> 這份筆記整理 Linux kernel 的完整 patch 提交流程，
> 適合第一次貢獻 kernel 的人對照使用。

---

## 核心觀念

Linux kernel **不用 GitHub Pull Request**，而是以 **mailing list + email patch** 為主流程：

1. 你在本地做修改、commit
2. 用 `git format-patch` 把 commit 轉成 `.patch` 檔
3. 用 `git send-email` 把 patch 寄給 maintainer 和 mailing list
4. Maintainer 或社群會 review，給出意見
5. 如果需要修改，做 v2、v3 再寄
6. Maintainer 決定是否 merge

---

## 標準流程（快速參考）

### Step 1：找問題

```bash
# 用 codespell 掃拼字錯誤
codespell Documentation/process/howto.rst

# 用 checkpatch 確認是否真實問題
./scripts/checkpatch.pl --strict --codespell --file <檔案路徑>
```

### Step 2：做修改

```bash
# 確認上下文
sed -n '<行號前後>p' <檔案路徑>

# 修改
nano +<行號> <檔案路徑>

# 確認改了什麼
git diff -- <檔案路徑>
```

### Step 3：Commit

```bash
git add <檔案路徑>
git diff --cached        # 再次確認 staged 內容
git commit -s            # -s 自動加 Signed-off-by
```

Commit message 格式：
```
子系統: 簡短說明（祈使句，英文，不超過 72 字）

詳細說明這個修改解決什麼問題、為什麼這樣改。

Signed-off-by: 姓名 <email>
```

### Step 4：產生 patch 檔

```bash
mkdir -p ~/patches
git format-patch -1 -o ~/patches --base=auto
```

### Step 5：找 maintainer

```bash
./scripts/get_maintainer.pl ~/patches/0001-*.patch
```

輸出會告訴你要寄給誰（`--to`）和 CC 給誰（`--cc`）。

### Step 6：測試寄信

```bash
# Dry run（不真的寄）
git send-email --dry-run --to <自己的email> ~/patches/0001-*.patch

# 先寄給自己確認格式正確
git send-email --to <自己的email> ~/patches/0001-*.patch
```

### Step 7：正式寄出

```bash
git send-email \
  --to <maintainer> \
  --cc <reviewer> \
  --cc <mailing-list-1> \
  --cc <mailing-list-2> \
  ~/patches/0001-*.patch
```

---

## 重要規則

| 規則 | 說明 |
|------|------|
| 一個 patch 只做一件事 | 不要在同一個 commit 改多個不相關的地方 |
| Commit message 用祈使句 | 例如 `fix`, `add`, `remove`，不要寫「I changed...」 |
| `Signed-off-by` 必須有 | 代表你同意 DCO，用 `git commit -s` 自動加 |
| 先跑 checkpatch | 不要在還有 error 的情況下送出 |
| Documentation 的 SPDX warning 可以忽略 | 那是給 C 原始碼用的，`.rst` 文件不需要 |
| `acknowledgement` 不一定是拼錯 | 英式英文，checkpatch 的 CHECK 不一定都要改 |

---

## 收到 review 後怎麼辦

1. 仔細讀 maintainer 或社群的回信
2. 如果同意修改：在本地改、重新 commit（或 `git commit --amend`）
3. 重新 `git format-patch`，這次在 subject 加上 `v2`：
   ```bash
   git format-patch -1 -o ~/patches -v 2
   # 產生 0001-v2-...
   ```
4. 在 patch 的 cover letter 或 patch 本身說明「v2 的改動」
5. 用 `--in-reply-to` 回覆原本的 thread：
   ```bash
   git send-email --in-reply-to="<原本的Message-ID>" ...
   ```

---

## 常用指令速查

```bash
# 查看目前 branch 狀態
git log --oneline -n 5
git status

# 確認 staged 內容
git diff --cached

# 確認 patch 格式
cat ~/patches/0001-*.patch

# 找 maintainer
./scripts/get_maintainer.pl ~/patches/0001-*.patch

# 掃拼字
codespell <檔案路徑>

# 檢查 patch 風格
./scripts/checkpatch.pl --strict --codespell --file <檔案路徑>
```

---

## 推薦閱讀

- `Documentation/process/howto.rst`：怎麼成為 kernel developer
- `Documentation/process/submitting-patches.rst`：完整 patch 提交指南
- `Documentation/process/coding-style.rst`：kernel 程式碼風格
- [lore.kernel.org](https://lore.kernel.org)：查看 mailing list 歷史
