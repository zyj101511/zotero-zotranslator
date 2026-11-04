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

1. 下载 [`zotranslator-1.0.0.xpi`](./zotranslator-1.0.0.xpi)。
2. 在 Zotero 10 中打开“工具 → 插件”，点击右上角齿轮，选择“从文件安装插件”，选中 `.xpi`。
3. 打开 PDF，选中单词或句子测试。使用 DeepLX 前，先按下节启动服务；只用 Cambridge 查词则不需要启动 DeepLX。

更新插件时安装新的 `.xpi`。本项目目前不提供自动更新；安装后可在插件列表确认版本号。

## 部署并启动 DeepLX-Serverless

DeepLX-Serverless 是独立项目，zotranslator 和下文的 Manggo 插件可以共用同一个服务。先安装 [Git](https://git-scm.com/downloads) 和 **Node.js 22 或更新版本**，再按系统运行：

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

返回结果包含 `data` 译文字段即表示服务可用。需要开机后自动运行时，可参考 [Manggo DeepLX 项目的 Windows 任务计划程序、Linux systemd 和 macOS launchd 配置](https://github.com/zyj101511/manggo-deepl#开机后台启动-deeplx)。如果把服务部署到另一台设备，请将插件里的 `localhost` 改为那台设备可访问的地址；远程部署建议使用 HTTPS。接口与其他部署方式见 [DeepLX-Serverless 项目说明](https://github.com/guobao2333/DeepLX-Serverless)。

## 在 Manggo 中使用 DeepLX

同一套 DeepLX-Serverless 服务也可以供 [Manggo 的 DeepLX 翻译插件](https://github.com/zyj101511/manggo-deepl) 使用。它支持自动识别源语言，也可以在 Manggo 中手动指定源语言和目标语言。

1. 下载 Manggo 项目中的 [`manggo-deeplx.mplugin`](https://github.com/zyj101511/manggo-deepl/blob/main/manggo-deeplx.mplugin)。
2. 在 Manggo 的“插件管理”中安装该文件。
3. 到“翻译服务”设置中添加“DeepLX 翻译”。服务运行在本机时，地址保持默认的 `http://localhost:6119` 即可；部署在其他设备时，填写该设备的可访问地址。
4. 翻译时可选择“自动识别”源语言，或手动选择源语言和目标语言。使用时需保持 DeepLX 服务运行。

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

**为什么发音按钮不可用？**  只有上游词典返回了相应地区的发音地址，插件才显示按钮。音频预加载完成前按钮不可点击；下载失败时按钮会显示“重试”，悬停可查看错误。播放使用内存中的音频，不会保存音频文件；播放结束后释放临时播放地址，查询另一段文字时清除已下载的缓存。

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
