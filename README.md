# Linux Kernel Contribution Log

這個 repository 用來持續記錄 Linux kernel 貢獻過程，不只是放 patch，而是把每一次的學習、問題、決策與回顧都保存下來，讓之後的第二個、第三個 patch 可以越走越順。[cite:148][cite:47]

這份紀錄特別適合用在「AI 當教練 + 自己動手驗證」的學習方式，因為每次遇到環境設定、checkpatch、maintainer、send-email 或 review 問題時，都能留下可以複用的經驗。[cite:149][cite:148]

## Repository 目的

- 記錄每一次 Linux kernel patch 的完整流程。
- 建立可重複使用的投稿 SOP。
- 保存每次 review 與修正版的學習點。
- 把零散經驗整理成自己的長期知識庫。

## 建議目錄結構

```text
linux-kernel-contribution-log/
├─ README.md
├─ 00_profile/
│  └─ learning-goals.md
├─ 01_environment/
│  └─ wsl-kernel-dev-setup.md
├─ 02_patches/
│  ├─ 2026-05-12-first-doc-patch/
│  │  ├─ note.md
│  │  ├─ patch.md
│  │  ├─ review-log.md
│  │  └─ assets/
├─ 03_templates/
│  ├─ patch-note-template.md
│  ├─ review-reply-template.md
│  └─ postmortem-template.md
└─ 04_lessons/
   └─ kernel-contribution-playbook.md
```

## 第一個 patch 摘要

- 類型：文件 typo 修正。[cite:148]
- 檔案：`Documentation/process/howto.rst`。[cite:148]
- 修改內容：`Alex Shepard` 改成 `Alex Shepherd`。[cite:148]
- branch：`shihong-first-doc-patch`。[cite:148]
- 核心流程：`codespell` → `checkpatch.pl` → `git add` → `git commit -s` → `git format-patch` → `scripts/get_maintainer.pl` → `git send-email`。[cite:148]

## 建議使用方式

每完成一次 patch，就在 `02_patches/日期-主題/` 建一個資料夾，至少留下四份紀錄：

- `note.md`：這次做了什麼、為什麼這樣做。
- `patch.md`：patch 內容摘要、命令、寄送對象。
- `review-log.md`：收到的 review 與修正計畫。
- `assets/`：截圖、終端機輸出、補充資料。

## 長期維護建議

- 每次只記錄一個 patch 主題，避免混在一起。
- 每次都寫「學到什麼」與「下次會怎麼更快」。
- 建立自己的 commit message 模板與 send-email 模板。
- 第二個 patch 開始就要保留 review 回信與 v2 變更摘要。

