# ExcelExport

将策划用 Excel 数据表导出为游戏项目可用的配置 / 数据文件（**JSON / Lua / XML**）。

纯 Go 单文件可执行程序，启动本地 Web 界面，无需安装 Excel、无需终端操作。

---

## 下载

| 平台 | 文件 |
|------|------|
| Windows (64 位) | [excelexport-v1.0.2-windows-amd64.zip](v1.0.2/excelexport-v1.0.2-windows-amd64.zip) |
| Linux (64 位) | [excelexport-v1.0.2-linux-amd64.zip](v1.0.2/excelexport-v1.0.2-linux-amd64.zip) |
| macOS (Intel) | [excelexport-v1.0.2-darwin-amd64.zip](v1.0.2/excelexport-v1.0.2-darwin-amd64.zip) |
| macOS (Apple Silicon) | [excelexport-v1.0.2-darwin-arm64.zip](v1.0.2/excelexport-v1.0.2-darwin-arm64.zip) |

---

## 更新日志

### v1.0.2（当前版本）
- **单实例**：再次双击 EXE 会先按 PID + 映像名校验结束已运行实例，再接管 17890 端口，实现「重启」而非多开（仅 Windows 生效）。
- **显式退出**：界面右上「退出」按钮调用 `POST /api/quit` 立即关闭后台进程并释放 exe 文件锁，便于替换升级；不再随关闭浏览器标签页自动退出。
- **无控制台窗口**：Windows 发行版用 `-H windowsgui` 编译，双击不再弹黑色命令行窗口；打开文件 / 选路径时也不闪黑窗。
- **粘贴导出**：任务表每行新增「粘贴导出」按钮，可直接把浏览器中打开的在线文档选区（含 4 行表头）粘贴导出；CSV 行仍决定输出路径 / 文件名 / Sheet，不支持 `r:Sheet` 跨表引用（显式报错）。
- **长路径可见**：路径输入框与任务表的 Excel 名 / Sheet 单元格、CSV 编辑器的长 server/client 路径，悬浮即显示完整内容（原生 tooltip）。
- 文档（`AGENTS.md` / `help.html`）同步上述变更。
- 修复：跨平台编译问题——`SysProcAttr.CreationFlags` 为 Windows 专属字段，已抽为 build tag 辅助函数（`cli_windows.go` / `cli_posix.go` / `sysproc_windows.go` / `sysproc_posix.go`），确保 linux / macOS 可正常交叉编译。

### v1.0.1
- **DSL 语法重构**：类型标注去除 `@` 前缀，改为裸写（`k` / `i` / `f` / `s` / `b` / `uniq` / `a` / `r:Sheet`）。
- 字段归属端完全由第 1 行 `cs` 决定，移除 `@i^s` / `@s^c` 类型级双端标记与 `@@` 死代码。
- 单行常量表的转置标记由 `@transpose` 改为在 A1 单元格写 `/`。
- 引擎、解析器、生成器（JSON / Lua / XML）同步新 DSL 语义；转置检测与跨表引用解析相应调整。
- 依赖调整：`golang.org/x/net`、`golang.org/x/text` 升为直接依赖。
- 文档同步（`AGENTS.md`、各 `DSL_SPEC*.md`、`PARSER_DESIGN.md`、README）。
- UI：帮助页重写、样式增强、文案微调；移除前端轮询改为按需刷新、导出按钮等宽。
- 新增 `workspace` 解码回归测试（GBK / UTF-8）。

> 旧版 v1.0.0 / v1.0.1 仍可在 [`v1.0.0/`](v1.0.0/) / [`v1.0.1/`](v1.0.1/) 获取，但建议升级到 v1.0.2。

---

## 使用方法

1. 下载对应平台的 zip，**解压**得到 `excelexport`（或 `excelexport.exe`）。
2. 双击运行（Windows）或在终端执行 `./excelexport`（Linux / macOS）。
   - 程序启动本地服务（默认 `http://127.0.0.1:17890`）并自动打开浏览器。
   - 不想自动开浏览器：`./excelexport --no-browser`，再手动访问上面的地址。
3. 在界面里配置你的项目：指定 Excel 数据表与 `_export.csv`（任务定义），点「导出全部」即可生成 JSON / Lua / XML。
   - 首次使用建议点界面上的「生成初始案例」，会生成一份示例数据 + 配置，便于快速上手。

---

## 注意事项

- **Windows**：程序未签名，首次运行可能被 SmartScreen 拦截。点击「更多信息」→「仍要运行」即可。
- **macOS**：若提示“无法打开”，在终端执行 `xattr -d com.apple.quarantine ./excelexport` 后再运行；或在「系统设置 → 隐私与安全性」中点「仍要打开」。
- **Linux**：直接 `./excelexport`（如无执行权限先 `chmod +x excelexport`）。
- 无需安装 Go，无需安装 Excel；数据表仍是 `.xlsx`，由程序内嵌的 excelize 读取。

---

## 说明

- 本仓库仅用于**分发编译好的发行包**与说明文档。
- 完整**源代码**在私有仓库：<https://github.com/yabhit/excelexport>
- 当前版本：**v1.0.2**
- 许可证：待定（源代码见上方私有仓库）

---

## 数据表格式（简要）

每个 Excel 中的 sheet 按行组织：

| 行 | 含义 | 示例 |
|----|------|------|
| 1 | `cs` 标记 | `cs` / `s` / `c`（两端 / 仅服务端 / 仅客户端） |
| 2 | 类型标注 | `i` / `f` / `s` / `b`（整数 / 浮点 / 字符串 / 布尔），裸写、不带 `@` |
| 3 | 字段名 | `id` / `name` / `hp` |
| 4 | 字段说明 | `编号` / `名称` / `血量` |
| 5+ | 数据行 | `1001` / `slime` / `100` |

完整 DSL 与双文件模型（`_export.csv` + 个人 `local.json`）详见源代码仓库的 `AGENTS.md` 与 `excelexport/README.md`。
