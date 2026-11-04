# zotranslator

一个用于 Zotero 10 的 PDF 划词翻译插件。选中英文单词或短语时优先查询 Cambridge 词典，句子交给 DeepLX 翻译；也可以在阅读器右侧手动查词或翻译。默认目标语言是简体中文。

> **非官方项目。**本插件与 Zotero、Cambridge Dictionary、Cambridge University Press & Assessment、DeepL 官方均无隶属或授权关系。词典数据来自第三方 Cambridge 接口；DeepLX 使用非官方翻译接口，并非 DeepL 官方 API。插件包不包含 DeepLX 服务。

## 功能

- PDF 划词后默认自动查询，结果显示在 Zotero 原生选词弹窗；可在设置中关闭自动翻译，改为点击“翻译”按钮。
- 英文单词或短语显示中文释义、词性、英文定义、双语例句、词形、标签及英美音标。词典提供发音地址时显示“美音”和“英音”按钮。
- 词典结果出现后并行预加载可用的英美音频。同一段文字可重复播放而不重新下载；查询另一段文字时清除旧音频缓存。
- 阅读器右侧提供输入框和“查词”“翻译”两个按钮，分别直接调用 Cambridge 与 DeepLX。
- 服务模式可设为两者都用、仅 Cambridge 或仅 DeepLX。默认模式下，词典未找到词条或请求失败时回退 DeepLX。

词典接口返回的字段因词条而异。插件目前**不提供同义词、反义词，也不支持中文词头反查英文**。扫描版 PDF 若没有可选中的文字层，无法直接划词查询。

## 安装

1. 下载 [`zotranslator-x.x.x.xpi`](./zotranslator-1.0.0.xpi)。
2. 在 Zotero 10 中打开“工具 → 插件”，点击右上角齿轮，选择“从文件安装插件”，选中 `.xpi`。
3. 打开 PDF，选中单词或句子测试。使用 DeepLX 前，先按下节启动服务；只用 Cambridge 查词则不需要启动 DeepLX。

更新插件时安装新的 `.xpi`。本项目目前不提供自动更新；安装后可在插件列表确认版本号。

## 部署并启动 DeepLX-Serverless

DeepLX-Serverless 是独立项目。先安装 [Git](https://git-scm.com/downloads) 和 **Node.js 22 或更新版本**，再按系统运行：

**Windows（PowerShell）：**

```powershell
node --version
git clone https://github.com/guobao2333/DeepLX-Serverless.git
cd DeepLX-Serverless
npm.cmd install
npm.cmd run start
```

**Linux / macOS（终端）：**

```bash
node --version
git clone https://github.com/guobao2333/DeepLX-Serverless.git
cd DeepLX-Serverless
npm install
npm run start
```

如果已经下载了项目源码，直接进入现有目录运行安装和启动命令即可。首次安装或更新依赖时才需要执行 `npm install`；Windows 使用 `npm.cmd` 可避开 PowerShell 对 `npm.ps1` 的执行限制。服务默认监听 `http://localhost:6119`，手动启动时需要保持终端开启。

**验证服务：**保持启动服务的终端开启，再打开一个终端发送测试请求。PowerShell：

```powershell
$body = @{ text = '你好，世界！'; source_lang = 'zh'; target_lang = 'en' } | ConvertTo-Json
Invoke-RestMethod -Uri 'http://localhost:6119/translate' -Method Post -ContentType 'application/json' -Body $body
```

Linux / macOS：

```bash
curl -X POST http://localhost:6119/translate -H 'Content-Type: application/json' -d '{"text":"你好，世界！","source_lang":"zh","target_lang":"en"}'
```

返回结果包含 `data` 译文字段即表示服务可用。如果把服务部署到另一台设备，请将插件里的 `localhost` 改为那台设备可访问的地址；远程部署建议使用 HTTPS。接口与其他部署方式见 [DeepLX-Serverless 项目说明](https://github.com/guobao2333/DeepLX-Serverless)。

## 开机静默启动 DeepLX

先确认手动启动和测试都成功，再关闭手动运行的服务，按系统选择一种方法。以下配置直接运行 `node src/server.js`，与项目的 `npm run start` [启动脚本](https://github.com/guobao2333/DeepLX-Serverless/blob/main/package.json)相同；启动后无需保持终端窗口开启。

### Windows：任务计划程序

1. 在开始菜单打开**任务计划程序**，选择“创建任务”。在“常规”中命名为 `DeepLX`，选择“**不管用户是否登录都要运行**”。
2. 在“触发器”中新建“**启动时**”。
3. 在“操作”中新建“启动程序”，按实际安装位置填写：

   | 项目 | 示例值 |
   | --- | --- |
   | 程序或脚本 | `C:\Program Files\nodejs\node.exe` |
   | 添加参数 | `src\server.js` |
   | 起始于 | DeepLX-Serverless 所在目录 |

4. 在“设置”中取消“如果任务运行时间超过 3 天则停止”；笔记本用户还应检查“条件”中是否限制为接通电源才运行。保存时 Windows 可能要求输入账户密码。
5. 右键新任务选择“运行”，用上面的测试请求确认成功。要停止自动启动，可在任务计划程序中禁用该任务。

可在 PowerShell 运行 `(Get-Command node).Source` 查找实际的 `node.exe` 路径。“起始于”应填写 **DeepLX-Serverless 目录的完整路径**。Windows 任务计划程序支持[启动时触发](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/schtasks-create)和[设置工作目录](https://learn.microsoft.com/en-us/windows/win32/taskschd/execaction-workingdirectory)。

### Linux：systemd

适用于使用 systemd 的发行版。先用 `whoami`、`pwd` 和 `command -v node` 查出用户名、DeepLX 项目绝对路径和 Node 可执行文件路径。创建服务文件：

```bash
sudo nano /etc/systemd/system/deeplx.service
```

填入以下内容，并把 `YOUR_USER`、项目目录和 `/usr/bin/node` 改成实际值：

```ini
[Unit]
Description=DeepLX-Serverless
Wants=network-online.target
After=network-online.target

[Service]
Type=simple
User=YOUR_USER
WorkingDirectory=/home/YOUR_USER/DeepLX-Serverless
ExecStart=/usr/bin/node /home/YOUR_USER/DeepLX-Serverless/src/server.js
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

保存后启用、立即启动并检查状态：

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now deeplx
sudo systemctl status deeplx
```

查看报错用 `journalctl -u deeplx -e`；停止并取消自启动用 `sudo systemctl disable --now deeplx`。如果 Node 通过 nvm 安装，`ExecStart` 必须填写 `command -v node` 显示的完整路径。[systemd 的工作目录说明](https://www.freedesktop.org/software/systemd/man/latest/systemd.exec.html)。

### macOS：launchd

此方法在**用户登录后**自动后台启动。先在 DeepLX 项目目录运行 `pwd` 和 `command -v node`，记下两个绝对路径。创建配置文件：

```bash
mkdir -p ~/Library/LaunchAgents
nano ~/Library/LaunchAgents/com.zyj101511.deeplx.plist
```

粘贴以下内容，并替换 `YOUR_USER` 和 `/opt/homebrew/bin/node`；Intel Mac 的 Node 位置也可能是 `/usr/local/bin/node`：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0"><dict>
  <key>Label</key><string>com.zyj101511.deeplx</string>
  <key>ProgramArguments</key><array>
    <string>/opt/homebrew/bin/node</string>
    <string>/Users/YOUR_USER/DeepLX-Serverless/src/server.js</string>
  </array>
  <key>WorkingDirectory</key><string>/Users/YOUR_USER/DeepLX-Serverless</string>
  <key>RunAtLoad</key><true/>
  <key>KeepAlive</key><true/>
  <key>StandardOutPath</key><string>/tmp/deeplx.out.log</string>
  <key>StandardErrorPath</key><string>/tmp/deeplx.err.log</string>
</dict></plist>
```

保存后加载；以后每次登录会自动启动：

```bash
launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.zyj101511.deeplx.plist
launchctl print gui/$(id -u)/com.zyj101511.deeplx
```

停止并取消当前登录会话中的启动：`launchctl bootout gui/$(id -u) ~/Library/LaunchAgents/com.zyj101511.deeplx.plist`；如需永久取消，再删除该 plist 文件。报错可看 `/tmp/deeplx.err.log`。[Apple 的 LaunchAgents 说明](https://developer.apple.com/library/archive/documentation/MacOSX/Conceptual/BPSystemStartup/Chapters/CreatingLaunchdJobs.html)。

## 设置

在 Zotero 的“设置 → zotranslator”中调整：

| 设置 | 默认值 | 作用 |
| --- | --- | --- |
| 服务模式 | Cambridge + DeepLX | 短词先查词典，句子直接翻译；也可只用其中一项 |
| PDF 选词后自动翻译 | 开启 | 关闭后在选词弹窗中手动点击“翻译” |
| Cambridge 接口根地址 | `https://dictionary-api.eliaschen.dev` | 可改为自行部署的兼容词典接口 |
| DeepLX 接口根地址 | `http://localhost:6119` | 可改为自行部署的 DeepLX 服务地址 |
| 显示词性、英文定义、例句、例句中文翻译 | 开启 | 控制词典义项的展示内容 |
| 每个义项最多例句数 | 2 | 范围 0–10 |
| 显示词形变化、词性标签、义项分隔线 | 开启 | 控制词典附加信息 |

接口地址以浅色默认值作为提示；输入新地址后会保存，留空则使用默认地址。侧栏的“查词”和“翻译”按钮是手动指定服务，不受自动服务模式限制。

## 常见问题

**为什么短词最后显示 DeepLX 译文？**  默认模式会先查询 Cambridge。词典未找到词条、请求失败或等待超过 30 秒时，才会回退 DeepLX。选中的英文不超过五个词且没有句末标点时，插件才把它当作词或短语。

**为什么“翻译”一直显示请求失败？**  确认 DeepLX-Serverless 已启动，设置中的地址可访问。`localhost` 指运行 Zotero 的这台电脑；服务若部署在另一台设备，需要填写那台设备的可访问地址。

**需要 Cambridge 或 DeepL 官方 API Key 吗？**  不需要。词典使用第三方接口；DeepLX 需要自行运行或部署。

## 数据来源与声明

- 词典数据通过 [chenelias/cambridge-dictionary-api](https://github.com/chenelias/cambridge-dictionary-api) 获取。其代码许可证不代表 Cambridge 词典内容可以自由复制或再分发；使用者应遵守相应服务条款。
- 文本翻译由 [DeepLX-Serverless](https://github.com/guobao2333/DeepLX-Serverless) 提供。非官方接口可能随上游变化而失效。
- 查询文字会发送到所选词典或 DeepLX 服务；DeepLX 服务还会向其翻译上游发送文字。插件不要求或收集官方 API Key。

## 作者

[zyj101511](https://github.com/zyj101511)

## 支持项目

如果这个插件对你有帮助，欢迎到[项目主页](https://github.com/zyj101511/zotranslater)点一个 Star ⭐

![star](assets/star.webp)
