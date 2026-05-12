# Linux Kernel Contribution Log

> Chen-Shi-Hong 的 Linux kernel 貢獻學習紀錄
> 從第一個 Documentation patch 開始，逐步累積。

---

## 關於這個 repo

這個 repo 不是 Linux kernel 的 fork，
而是我個人學習「如何對 Linux kernel 做貢獻」的過程紀錄。

每一次送出 patch，就新增一個資料夾記錄：
- 這次改了什麼
- 遇到什麼問題、怎麼解決
- patch 的完整內容
- 學到了什麼

---

## 目錄結構

```
Linux_kernel/
├── README.md                          ← 這個檔案
├── setup/
│   └── wsl2-env-setup.md             ← WSL2 開發環境設定紀錄
├── learning/
│   └── kernel-workflow.md            ← Linux kernel 協作流程筆記
└── patches/
    └── 2026-05-12-doc-howto-shepherd/
        └── notes.md                  ← 第一個 patch 完整學習筆記
```

---

## Patch 紀錄

| # | 日期 | 子系統 | 說明 | 狀態 |
|---|------|--------|------|------|
| 1 | 2026-05-12 | `Documentation` | 修正 `howto.rst` 中 Shepherd 拼字錯誤 | ✅ 已送出 |

---

## 環境

- OS：Windows 11 + WSL2 Ubuntu
- Kernel source：`git.kernel.org/torvalds/linux.git`（v7.1-rc3）
- Branch：`shihong-first-doc-patch`
- 寄送工具：`git send-email` + Gmail SMTP + App Password

---

## 聯絡

- GitHub：[eric039eric](https://github.com/eric039eric)
- Email：eric039eric@gmail.com
