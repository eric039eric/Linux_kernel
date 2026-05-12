# WSL2 Linux Kernel 開發環境設定

**日期**：2026-05-12  
**環境**：Windows 11 + WSL2 Ubuntu（Ubuntu Resolute）  
**核心版本**：v7.1-rc3（`5d6919055dec`）

---

## 安裝必要套件

```bash
sudo apt update && sudo apt full-upgrade -y

sudo apt install -y \
  git git-email build-essential bc bison flex libssl-dev libelf-dev \
  fakeroot dwarves cpio rsync curl wget perl python3 python3-pip \
  patchutils diffstat pahole codespell
```

> 常見錯誤：`build-essentail`（多打了一個 a），正確是 `build-essential`

---

## 設定 Git 基本資訊

```bash
git config --global user.name "Chen-Shi-Hong"
git config --global user.email "eric039eric@gmail.com"
git config --global core.editor "nano"
git config --global sendemail.annotate yes
git config --global format.subjectPrefix "PATCH"
```

---

## 設定 git send-email（Gmail SMTP）

```bash
git config --global sendemail.smtpserver smtp.gmail.com
git config --global sendemail.smtpserverport 587
git config --global sendemail.smtpencryption tls
git config --global sendemail.smtpuser eric039eric@gmail.com
git config --global sendemail.from "Chen-Shi-Hong <eric039eric@gmail.com>"
git config --global sendemail.confirm auto
```

### Gmail App Password 設定方式

1. 前往 [myaccount.google.com](https://myaccount.google.com)
2. 安全性 → 兩步驟驗證（要先開啟）
3. 最下方 → 應用程式密碼
4. 輸入名稱（例如 `git send-email`）→ 產生
5. 複製 16 碼密碼，在 `git send-email` 問密碼時貼上

> ⚠️ 不能用 Gmail 帳號的一般密碼，一定要用 App Password，否則會出現 `5.7.8 BadCredentials` 錯誤

---

## Clone Linux Kernel Source

```bash
mkdir -p ~/src
cd ~/src
git clone https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git
cd linux
```

> 大小約 3.2 GB，需要時間，請耐心等待

---

## 建立 patch 用的 branch

```bash
git checkout -b shihong-first-doc-patch
git branch --set-upstream-to=origin/master shihong-first-doc-patch
```

---

## 確認環境版本

```bash
uname -a          # 確認 WSL2 kernel
git --version     # 應為 2.x 以上
codespell --version
```
