# fanqie-publisher-skill

一个开源的番茄小说章节发布 Skill：把本地 Markdown 章节自动填入番茄作者后台，并按发布流程自动点击。

## 功能

- 读取单个 `.md` 文件并解析章节标题/正文
- 自动填写：章节号、章节标题、正文
- 自动发布流程（可视化浏览器）
  - `存草稿`（一次）
  - `下一步`
  - 错别字弹窗 `提交`
  - 风险检测弹窗 `取消`
  - 发布设置选择是否使用 AI（`是/否`）并 `确认发布`
- 首次运行支持扫码登录，后续复用登录态
- 失败自动落盘调试信息（日志、截图、HTML）

## 目录结构

```text
.
├── SKILL.md
├── scripts/
│   └── publish_fanqie.py
└── references/
    └── fanqie_selectors.md
```

## 依赖

- Python 3.11+
- Playwright Chromium

安装：

```bash
pip install playwright loguru
playwright install chromium
```

## 快速开始

### 1) 仅预览解析结果（不打开浏览器）

```bash
python scripts/publish_fanqie.py \
  --md-path /path/to/chapter.md \
  --dry-run true
```

### 2) 自动发布（推荐）

> 默认非无头模式（可视化浏览器），符合人工观察与扫码登录需求。

```bash
python scripts/publish_fanqie.py \
  --md-path /path/to/chapter.md \
  --work-url "https://fanqienovel.com/main/writer/.../publish/...?..." \
  --publish true \
  --headless false \
  --hard-timeout-seconds 420
```

### 3) 只填充内容，手动点击发布

```bash
python scripts/publish_fanqie.py \
  --md-path /path/to/chapter.md \
  --work-url "https://fanqienovel.com/main/writer/.../publish/...?..." \
  --publish false
```

## 标准发布 SOP（脚本内置）

自动发布时使用以下固定流程：

1. 填写章节号 / 标题 / 正文
2. 点击 `存草稿`（一次）
3. **不刷新页面**
4. 点击右上角 `下一步`
5. 若出现错别字弹窗：点击 `提交`
6. 若出现内容风险检测弹窗：点击 `取消`
7. 在发布设置弹窗：选择是否使用 AI（默认 `是`）
8. 点击 `确认发布`
9. 跳转到 `chapter-manage` 或出现成功信号即判定成功

## 常用参数

- `--ai-generated true|false`：发布设置里选择“是否使用AI”（默认 `true`）
- `--profile-dir`：浏览器持久化目录（复用登录态）
- `--editor-wait-seconds`：等待编辑器就绪时长
- `--publish-wait-seconds`：发布阶段等待时长
- `--hard-timeout-seconds`：整次流程硬超时（到时自动结束并关闭浏览器）
- `--trace-log-file`：指定详细日志路径
- `--log-dir`：失败时截图/HTML输出目录

## 调试与日志

建议每次测试都传 `--trace-log-file`，例如：

```bash
python scripts/publish_fanqie.py \
  --md-path /path/to/chapter.md \
  --work-url "https://fanqienovel.com/main/writer/..." \
  --publish true \
  --trace-log-file /tmp/fanqie_logs/trace_run.log
```

发布设置相关日志包含两个阶段：

- `Publish modal visible (phase 1)`：弹窗已可见
- `Publish modal interactive (phase 2)`：弹窗控件可交互，开始执行 AI 选择和确认发布

## 注意事项

- 首次运行需要手动扫码登录番茄作者后台。
- 请使用作者后台合法账号并遵守平台规则。
- 建议使用 `newchapter` 入口 URL；脚本会自动规整常见 `publish/<chapter_id>?enter_from=newchapter` 形式，避免误改旧章节。

## 致谢

本 Skill 源于 `nanobot` 项目内的自动化发布能力抽离与开源。
