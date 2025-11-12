It might be the quickest cross-platform codesign alternative for iOS 12+, supporting macOS, Linux, Windows, and more features.
If this tool helps you, please don't forget to <font color=#FF0000 size=5>🌟**star**🌟</font> [ME](https://github.com/zhlynn).

## Compile 

### macOS:

```bash
brew install pkg-config openssl minizip
git clone https://github.com/zhlynn/zsign.git
cd zsign/build/macos
make clean && make
```

Install `ideviceinstaller` for test:
```bash
brew install ideviceinstaller
```

### Linux:

#### Ubuntu 22.04 / Debian 12 / Mint 21:

```bash
sudo apt-get install -y git g++ pkg-config libssl-dev libminizip-dev
git clone https://github.com/Scott96003/zsign
cd zsign/build/linux
make clean && make
```

Install `ideviceinstaller` for test:
```bash
sudo apt-get install -y ideviceinstaller
```

#### RHEL / CentOS / Alma / Rocky / Other clones:

You must install `epel-release` first, eg:

RHEL / CentOS / Alma / Rocky 8:
```bash
sudo rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
```

RHEL / CentOS / Alma / Rocky 9:
```bash
sudo rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
```

Then, install the dependencies and compile:
```bash
sudo yum install -y git gcc-c++ pkg-config openssl-devel minizip1.2-devel
git clone https://github.com/zhlynn/zsign.git
cd zsign/build/linux
make clean && make
```

### Windows:

Use `Visual Studio 2022` to open `build/windows/vs2022/zsign.sln`, then compile it on Windows 10/11.
  
## Usage:

```bash
Usage: zsign [-options] [-k privkey.pem] [-m dev.prov] [-o output.ipa] file|folder
options:
-k, --pkey              Path to private key or p12 file. (PEM or DER format)
-m, --prov              Path to mobile provisioning profile.
-c, --cert              Path to certificate file. (PEM or DER format)
-a, --adhoc             Perform ad-hoc signature only.
-d, --debug             Generate debug output files. (.zsign_debug folder)
-f, --force             Force sign without cache when signing folder.
-o, --output            Path to output ipa file.
-p, --password          Password for private key or p12 file.
-b, --bundle_id         New bundle id to change.
-n, --bundle_name       New bundle name to change.
-r, --bundle_version    New bundle version to change.
-e, --entitlements      New entitlements to change.
-z, --zip_level         Compressed level when output the ipa file. (0-9)
-l, --dylib             Path to inject dylib file. Use -l multiple time to inject multiple dylib files at once.
-w, --weak              Inject dylib as LC_LOAD_WEAK_DYLIB.
-i, --install           Install ipa file using ideviceinstaller command for test.
-t, --temp_folder       Path to temporary folder for intermediate files.
-2, --sha256_only       Serialize a single code directory that uses SHA256.
-C, --check             Check if the file is signed.
-q, --quiet             Quiet operation.
-v, --version           Shows version.
-h, --help              Shows help (this message).
```

1. Show mach-o and codesignature segment info.
```bash
./zsign demo.app/demo
```

2. Sign ipa with private key and mobileprovisioning file.
```bash
./zsign -k privkey.pem -m dev.prov -o output.ipa -z 9 demo.ipa
```

3. Sign folder with p12 and mobileprovisioning file (using cache).
```bash
./zsign -k dev.p12 -p 123 -m dev.prov -o output.ipa demo.app
```

4. Sign folder with p12 and mobileprovisioning file (without cache).
```bash
./zsign -f -k dev.p12 -p 123 -m dev.prov -o output.ipa demo.app
```

5. Sign ipa with ad-hoc.
```bash
./zsign -a -o output.ipa demo.ipa
```

6. Inject dylib into ipa and re-sign.
```bash
./zsign -k dev.p12 -p 123 -m dev.prov -l demo.dylib -o output.ipa demo.ipa
```

7. Change bundle id and bundle name
```bash
./zsign -k dev.p12 -p 123 -m dev.prov -b 'com.new.bundle.id' -n 'NewName' -o output.ipa demo.ipa
```

8. Inject dylib(LC_LOAD_DYLIB) into mach-o file.
```bash
./zsign -a -l "@executable_path/demo1.dylib" -l "@executable_path/demo2.dylib" demo.app/execute
```

9. Inject dylib(LC_LOAD_WEAK_DYLIB) into mach-o file.
```bash
./zsign -w -l "@executable_path/demo.dylib" demo.app/execute
```

## How to sign quickly?

First, unzip the IPA file, then use zsign to sign the folder containing assets. During the initial signing, zsign will perform a complete signature and cache the signing information into a .zsign_cache directory in the current path. When re-signing the folder with different assets, zsign will utilize the cached data to significantly speed up the process—making signing extremely fast! Give it a try!


您在安裝 **zsign** 時遇到的這個錯誤，是一個非常常見的 **函式庫依賴性 (Library Dependency)** 問題，特別是在 macOS 或 Linux 系統上進行編譯時。

-----

## 🛠️ 錯誤分析與解決方案

### 🔑 核心問題

核心問題是您的編譯器找不到 **OpenSSL** 函式庫，因為 `zsign` 需要用到它來進行加密和簽名操作。

錯誤訊息清晰地指出了問題：

1.  `No package 'openssl' found`：`pkg-config` 工具找不到 OpenSSL 函式庫的配置資訊（`.pc` 檔案）。
2.  `fatal error: 'openssl/pem.h' file not found`：編譯器在編譯 `openssl.cpp` 時，找不到標頭檔 `openssl/pem.h`。

### 🍎 macOS 解決步驟 (使用 Homebrew)

由於您使用的是 macOS，最簡單且標準的解決方法是使用 **Homebrew**（macOS 上的套件管理器）來安裝 OpenSSL，並確保您的系統知道它在哪裡。

#### 步驟一：安裝或更新 OpenSSL

請確保您已經安裝了 Homebrew，然後執行以下命令：

```bash
brew install openssl
```

#### 步驟二：設定環境變數

即使安裝了 OpenSSL，Homebrew 為了避免與系統內建的 OpenSSL 衝突，通常會將其安裝在一個特定路徑下，但這個路徑預設並不會自動加入到 `PKG_CONFIG_PATH` 和編譯器的搜尋路徑中。

您需要設定兩個環境變數：

1.  **`PKG_CONFIG_PATH`**：讓 `pkg-config` 找到 OpenSSL 的配置資訊。
2.  **`CPATH` / `CPLUS_INCLUDE_PATH`** 和 **`LIBRARY_PATH`**：告訴編譯器頭文件和函式庫的位置。

請根據您使用的 Shell 執行以下指令 (假設您使用 **Homebrew 安裝的 openssl@3**)：

```bash
# 找出 Homebrew 安裝 openssl 的路徑 (例如 /opt/homebrew/opt/openssl@3)
OPENSSL_PATH=$(brew --prefix openssl@3)

# 導出環境變數
export PKG_CONFIG_PATH="$OPENSSL_PATH/lib/pkgconfig:$PKG_CONFIG_PATH"
export CPLUS_INCLUDE_PATH="$OPENSSL_PATH/include:$CPLUS_INCLUDE_PATH"
export LIBRARY_PATH="$OPENSSL_PATH/lib:$LIBRARY_PATH"
```

> **💡 注意：**
>
>   * 如果您使用的是舊版 Homebrew 且安裝的是 `openssl` 而非 `openssl@3`，請將指令中的 `openssl@3` 改為 `openssl`。
>   * 這些變數**只在當前的終端機視窗中有效**。

#### 步驟三：重新編譯 zsign

在設定好環境變數的**同一個終端機視窗**中，回到 `zsign` 的源碼目錄，然後重新執行編譯命令（通常是 `make`）：

```bash
# 確保您在 zsign 專案根目錄下
make clean
make
```

### 永久解決方案 (設定 Shell 配置)

如果您不希望每次打開終端機都要重新輸入環境變數，您可以將步驟二中的 `export` 命令添加到您的 Shell 配置文件中：

  * 如果您使用 **zsh**，請編輯 `~/.zshrc`。
  * 如果您使用 **bash**，請編輯 `~/.bash_profile` 或 `~/.bashrc`。

在檔案末尾添加：

```bash
# Zsign/OpenSSL 依賴設定
export OPENSSL_PATH=$(brew --prefix openssl@3)
export PKG_CONFIG_PATH="$OPENSSL_PATH/lib/pkgconfig:$PKG_CONFIG_PATH"
export CPLUS_INCLUDE_PATH="$OPENSSL_PATH/include:$CPLUS_INCLUDE_PATH"
export LIBRARY_PATH="$OPENSSL_PATH/lib:$LIBRARY_PATH"
```

然後執行 `source ~/.zshrc` (或對應的檔案) 讓設定立即生效。

## License

zsign is licensed under the terms of MIT License. See the [LICENSE](LICENSE) file.
