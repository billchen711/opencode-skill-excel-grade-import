# 環境安裝指南

本文檔說明如何從零開始安裝 Demo 所需的所有軟體。

---

## macOS 安裝

### 1. 安裝 Homebrew（如果還沒裝）

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

### 2. 安裝 VS Code

```bash
brew install --cask visual-studio-code
```

或從 https://code.visualstudio.com/ 下載安裝。

### 3. 安裝 Git

```bash
brew install git
```

確認安裝：
```bash
git --version
```

### 4. 安裝 Node.js

```bash
brew install node
```

確認安裝：
```bash
node --version
npm --version
```

### 5. 安裝 .NET SDK

```bash
brew install dotnet-sdk
```

確認安裝：
```bash
dotnet --version
```

### 6. 安裝 OpenCode 或 MimoCode

若使用 OpenCode，可先用 npm 安裝：

```bash
npm i -g opencode-ai
opencode --version
```

若使用 MimoCode，請參考 MimoCode 官方文件安裝。

---

## Windows 安裝

### 1. 安裝 VS Code

從 https://code.visualstudio.com/ 下載安裝。

或使用 winget：
```powershell
winget install Microsoft.VisualStudioCode
```

### 2. 安裝 Git

從 https://git-scm.com/ 下載安裝。

或使用 winget：
```powershell
winget install Git.Git
```

確認安裝：
```powershell
git --version
```

### 3. 安裝 Node.js

從 https://nodejs.org/ 下載 LTS 版本安裝。

或使用 winget：
```powershell
winget install OpenJS.NodeJS.LTS
```

確認安裝：
```powershell
node --version
npm --version
```

### 4. 安裝 .NET SDK

```powershell
winget install Microsoft.DotNet.SDK.10
```

確認安裝：
```powershell
dotnet --version
```

### 5. PowerShell 執行政策問題

如果安裝或執行 OpenCode 時出現「無法載入檔案，因為這個系統上已停用指令碼執行」錯誤：

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

### 6. 安裝 OpenCode 或 MimoCode

若使用 OpenCode，可先用 npm 安裝：

```powershell
npm i -g opencode-ai
opencode --version
```

若使用 MimoCode，請參考 MimoCode 官方文件安裝。

---

## Linux (Ubuntu/Debian) 安裝

### 1. 安裝 VS Code

```bash
sudo snap install code --classic
```

### 2. 安裝 Git

```bash
sudo apt install git
git --version
```

### 3. 安裝 Node.js

```bash
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt install -y nodejs
```

### 4. 安裝 .NET SDK

```bash
sudo apt install dotnet-sdk-10.0
```

### 5. 安裝 OpenCode 或 MimoCode

若使用 OpenCode，可先用 npm 安裝：

```bash
npm i -g opencode-ai
opencode --version
```

若使用 MimoCode，請參考 MimoCode 官方文件安裝。

---

## 驗證安裝

安裝完成後，請執行以下指令確認所有工具都已就緒：

```bash
git --version
code --version
node --version
npm --version
dotnet --version
opencode --version
```

如果所有指令都有回應版本號碼，表示安裝成功。

---

## 常見問題

### Q：dotnet 指令找不到？

**macOS：** 確認 Homebrew 安裝的 .NET SDK 路徑有在 PATH 中。
```bash
echo $PATH | grep dotnet
```

**Windows：** 重新開啟 PowerShell 或重新啟動電腦。

### Q：npm 指令找不到？

確認 Node.js 已安裝：
```bash
node --version
```

如果 Node.js 有安裝但 npm 沒有，重新安裝 Node.js。

### Q：PowerShell 執行政策錯誤？

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

### Q：.NET SDK 版本不對？

確認安裝的是 .NET 10 或 .NET 8：
```bash
dotnet --list-sdks
```
