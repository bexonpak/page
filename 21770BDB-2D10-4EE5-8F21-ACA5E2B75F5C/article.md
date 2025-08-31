使用 GPG 簽名 git commits

## 目的

因為 git 本身允許修改 commit，本文章指導如何在 macOS 作業系統中使用 GPG 對 git 提交進行數字簽名。透過這種方式，提交在 GitHub 或其它 git 程式碼管理系統上將被標記為已驗證，從而增加提交的可信度。🎉

## 配置步驟

### 安裝 GPG

透過 Homebrew 安裝 GPG：

```ssh
brew install gpg
```

安裝 GPG 工具，之後可以使用它來生成和管理金鑰對。

檢查現有的 GPG 金鑰

在生成新的 GPG 金鑰對之前，首先檢查是否已經存在 GPG 金鑰。執行以下命令：

```ssh
gpg --list-secret-keys --keyid-format LONG
如果命令沒有返回任何金鑰，說明你還沒有配置 GPG 金鑰。
% gpg --list-secret-keys --keyid-format LONG
gpg: directory '/Users/UserName/.gnupg' created
gpg: /Users/UserName/.gnupg/trustdb.gpg: trustdb created
```

如果返回了金鑰資訊，則可以跳過生成 GPG 金鑰對的步驟。

```ssh
% gpg --list-secret-keys --keyid-format LONG
[keyboxd]
---------
sec   ed25519/E7175B0CAF8C4FE0 2024-09-16 [SC]
      *****************
uid                 [ultimate] UserName <email@exmple.com>
ssb   cv25519/88CAD4AB29BCB4B5 2024-09-16 [E]
```

後面也可以透過這條命令檢視已有的金鑰

### 生成 GPG 金鑰對

生成新金鑰對：

```ssh
gpg --full-generate-key --expert
# 使用 --expert 選項可以支援 ECC 
# 加密演算法。如果選擇 RSA 演算法，則無需使用 --expert 選項
```

* 選擇金鑰型別：選擇 (9) ECC (sign and encrypt) *default*
* 選擇演算法：選擇 (1) Curve 25519
* 設定金鑰過期時間：0 為永不過期
* 確認資訊 Key is valid for? (0)  Is this correct? (y/N)
* 設定使用者 ID：確保郵箱地址與commit郵箱一致
* 設定密碼


### 匯出公鑰

獲取金鑰列表：

```ssh
gpg --list-secret-keys --keyid-format LONG
```

示例輸出：

```ssh
% gpg --list-secret-keys --keyid-format LONG
gpg: checking the trustdb
gpg: marginals needed: 3  completes needed: 1  trust model: pgp
gpg: depth: 0  valid:   1  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 1u
[keyboxd]
---------
sec   ed25519/E7175B0CAF8C4FE0 2024-09-16 [SC]
      *****************
uid                 [ultimate] UserName <email@exmple.com>
ssb   cv25519/88CAD4AB29BCB4B5 2024-09-16 [E]
```

記下 sec ed25519/ 後面的 GPG Key ID，例如 E7175B0CAF8C4FE0

匯出公鑰：

```ssh
gpg --armor --export E7175B0CAF8C4FE0
```

將匯出的公鑰複製並貼上到 GitHub 的 GPG 金鑰新增頁面上。

```ssh
-----BEGIN PGP PUBLIC KEY BLOCK-----
PUBLIC KEY
-----END PGP PUBLIC KEY BLOCK-----
```

### 配置 Git

設定 gpg-agent 環境變數：

```ssh
echo 'export GPG_TTY=$(tty)' >> ~/.zshrc
source ~/.zshrc
```

這裡只寫了Oh my zsh的，其它工具自行改一下

配置 Git 使用 GPG 簽名：

當前目錄

```ssh
git config user.signingkey E7175B0CAF8C4FE0
git config commit.gpgsign true
```

全域性

```ssh
git config --global user.signingkey E7175B0CAF8C4FE0
git config --global commit.gpgsign true
```

注意替換為實際的 GPG Key ID。

對於不需要 GPG 簽名的 Git 倉庫，進入倉庫目錄並執行：

```ssh
git config commit.gpgsign false
```

祝程式設計愉悅！