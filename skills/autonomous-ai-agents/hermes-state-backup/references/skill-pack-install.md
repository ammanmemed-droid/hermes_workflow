# 安装第三方技能包（Superpowers / OpenSpec）

从 hermes-workflow-assets 合并而来。两类经常一起出现的工作：把第三方技能包（Superpowers / OpenSpec / 其他 market 包）装进 Hermes 和 Codex，然后同步 `~/hermes_workflow` 备份仓库并 push GitHub。

## 何时使用

- 用户要求"安装 Superpowers / OpenSpec / 某技能"。
- 用户要求"更新工作流备份"或"都同步备份并上传 github"。
- 新技能安装完成后需要同时做备份同步（本用户惯例：装完即同步）。

## 一、安装第三方技能包

### Superpowers（从 Codex 插件缓存安装）

本机已存在 Codex 插件缓存（`~/.codex/plugins/cache/<marketplace>/<plugin>/<version>/`），直接复制即可，无需联网：

```bash
sp=~/.codex/plugins/cache/superpowers-marketplace/superpowers/6.2.0
dst=$LOCALAPPDATA/hermes/skills/superpowers
cp -R "$sp/skills/." "$dst/" && cp -R "$sp/assets" "$dst/"
```

要点：
- **保持目录结构**：技能间有相对引用（`../requesting-code-review/code-reviewer.md`），必须连 `assets/`、`scripts/`、`references/` 一起复制，不能只拷 SKILL.md。
- **重名去重**：与 Hermes 现有分类重名的技能（如 test-driven-development、systematic-debugging、requesting-code-review）会共存不覆盖，Hermes 索引保留先注册的版本；安装后确认 `skills_list(category=...)` 数量符合预期。
- **必改 using-superpowers 的 description**：原版描述是 "Use when starting any conversation"，会诱导 Hermes 每次对话自动加载，违背"按需加载"铁律。安装后必须 patch 为"用户明确请求或任务匹配时加载"。
- **插件级 marketplace 配置**：Codex 侧启用插件需在 `~/.codex/config.toml` 写 `[plugins."superpowers@superpowers-marketplace"] enabled = true` + `[marketplaces.superpowers-marketplace]`。

### OpenSpec（CLI + 项目初始化）

```bash
npm install -g @fission-ai/openspec@latest   # 当前 1.8.0
openspec init --tools hermes --no-animation   # 在项目目录执行
```

产物：
- `openspec/config.yaml`（默认是英文注释模板，需按用户要求汉化 context）
- `openspec/changes/`、`openspec/specs/`
- `.hermes/skills/` 下 6 个官方技能（openspec-propose / apply-change / explore / update-change / sync-specs / archive-change）

要点：
- **汉化配置**：`openspec/config.yaml` 保持 `schema: spec-driven`，在 `context` 写"语言：中文（简体），所有产出物必须用简体中文撰写；技术术语、代码、命令、文件路径保留英文"，并把 BPD 分级约束（L1 不用 OpenSpec、L3 不自动等于 OpenSpec）写进去。
- **官方技能要复制到全局技能库**：init 只生成项目级 `.hermes/skills/`，Hermes 默认不加载；把 6 个技能复制到 `$LOCALAPPDATA/hermes/skills/openspec/` 才全局可用。
- 用户贴的 `/opsx:*` 命令（onboard/explore/propose/new/continue/ff/apply/update/verify/sync/archive/bulk-archive）是 agent 端 slash 命令；CLI 等价命令是 `openspec new change <name>`、`openspec status --change <name> --json`、`openspec validate <name>`、`openspec archive <name>` 等。
- 安装适配：在 OpenSpec 技能正文里写明"不自动触发"，触发条件限定为用户明确要求正式规格/契约/迁移追踪。

## 二、安装后 Pitfalls（血泪教训）

1. **rm -rf 备份仓库的 `.git`（真实事故）**：同步脚本里 `find "$repo" -type d -name .git -prune -exec rm -rf {} +` 会把**备份仓库自身的 `.git`** 也删掉（仓库根目录就是 `$repo/.git`）。恢复方法：
   ```bash
   cd "$repo" && git init -b main
   git remote add origin <url>
   git fetch origin
   git reset --mixed origin/main    # 保留工作区文件，对齐远程基线
   git add -A && git commit -m ... && git push origin main
   ```
   清理内嵌 .git 只该针对复制进来的插件/技能目录，绝不从仓库根 find。

2. **`git push` 的 `-c` 参数位置**：`git push -c http.lowSpeedLimit=0 ...` 会报错（打印帮助）；必须 `git -c http.lowSpeedLimit=0 -c http.lowSpeedTime=999 push origin main`（`-c` 在子命令前）。

3. **Codex exec 在 Windows 用 MSYS 路径报错**：`codex exec -C /e/... ` 报 `os error 3 系统找不到指定的路径`；必须用原生 Windows 路径 `-C 'E:\\...'`，`-o` 输出文件同理。

4. **Codex 读 SKILL.md 中文乱码**：Codex exec 经 PowerShell `Get-Content` 读 UTF-8 技能时控制台显示乱码（`鍚庣` 之类），但**路由结论仍然正确**，属显示层问题，不要误判安装失败。

5. **openspec init 后项目新增文件**：`openspec/` 和 `.hermes/` 会出现在项目 git status，属预期产物；不自动 commit（用户未说"提交"不动）。

## 三、验证

- `skills_list(category=...)` 确认技能被识别、数量正确。
- `skill_view(name=...)` 抽查入口技能（如 `openspec:openspec`）可加载。
- Codex 侧只读冒烟：`codex exec -C 'E:\\...' --sandbox read-only --ephemeral -o <out> '只做流程路由...'`，验证路由结论（L1/L2-Lite/L3）符合 BPD。
- 备份仓库 `git status --porcelain` 应干净（push 后）。
