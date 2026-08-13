# 安装/恢复配套技能：Superpowers / OpenSpec（已验证 2026-08）

安装 ≠ 自动使用：两者都保持"按需加载"，不因安装而扩大任何任务的流程深度（遵守 BPD 上限）。

## Superpowers（来自 Codex 插件缓存）

本机 Codex 通过 marketplace 启用 superpowers（`~/.codex/config.toml` 里 `[plugins."superpowers@superpowers-marketplace"] enabled = true`）。插件包缓存在：

```text
~/.codex/plugins/cache/superpowers-marketplace/superpowers/<ver>/   (6.6M，含 skills/ + assets/ + hooks/ + tests/)
```

安装到 Hermes：

```bash
sp=~/.codex/plugins/cache/superpowers-marketplace/superpowers/<ver>
dst="$LOCALAPPDATA/hermes/skills/superpowers"
rm -rf "$dst" && mkdir -p "$dst"
cp -R "$sp/skills/." "$dst/"
cp -R "$sp/assets" "$dst/"   # 技能内相对引用 ../assets 需要
```

要点：

1. **必须改写 `using-superpowers` 的 frontmatter description**。原版是 "Use when starting any conversation... MUST invoke"，会诱导 Hermes 每次对话都加载它，破坏"按需加载"原则。改成："Use when the user explicitly requests Superpowers workflow guidance, or when a task matches a Superpowers process skill... Load on demand only - do NOT auto-load for every conversation."
2. **同名技能自动去重，保留 Hermes 现有版本**：`test-driven-development`、`systematic-debugging`、`requesting-code-review` 与 `skills/software-development/` 下已有技能重名；skills_list 只显示 Hermes 现有版本，superpowers 副本仍在磁盘但不进索引——不用删，也不会覆盖。
3. 插件包里的 `hooks/`、`tests/`、`scripts/`、`docs/` 不是 Hermes 技能，不要复制进技能库；只取 `skills/` + `assets/`。
4. 备份时 `hermes_workflow/codex/plugins/superpowers-marketplace/` 保存完整插件包用于换机恢复。

## OpenSpec（npm CLI + 官方 Hermes 技能）

```bash
npm install -g @fission-ai/openspec@latest   # 已验证 1.8.0
openspec --version
```

项目初始化（生成 `openspec/` + `.hermes/skills/`）：

```bash
cd <project>
openspec init --tools hermes --no-animation
# 产物：openspec/{config.yaml,changes/,specs/} + .hermes/skills/{openspec-propose,openspec-apply-change,
#       openspec-explore,openspec-update-change,openspec-sync-specs,openspec-archive-change}
```

把官方技能复制到 Hermes 全局库（任何项目可用）：

```bash
mkdir -p "$LOCALAPPDATA/hermes/skills/openspec"
cp -R <project>/.hermes/skills/. "$LOCALAPPDATA/hermes/skills/openspec/"
```

汉化产出物：`openspec/config.yaml` 顶部加：

```yaml
context: |
  语言：中文（简体）
  所有产出物必须用简体中文撰写。
  技术术语、代码、命令、文件路径保留英文。
```

## 验证安装

- `skills_list` / `skill_view(name)` 能看到新分类与技能（superpowers 14 个目录、openspec 7 个 SKILL.md）。
- 用 `codex exec --sandbox read-only --ephemeral -m <model> -o <out.txt> "只做流程路由，不修改文件：..."` 实测 Codex 能读到技能并按 BPD 分级。
  - ⚠️ Windows 下 `-C` 必须用原生路径：`-C 'E:\path\to\repo'`（MSYS 的 `/e/...` 会报 `os error 3 系统找不到指定的路径`）。
  - `-o <file>` 把末条消息写入文件，避免解析 stdout 噪音。
- 检查技能目录不含凭据：`grep -rlE '(sk-[A-Za-z0-9]{20,}|AKIA[0-9A-Z]{16}|-----BEGIN (RSA|OPENSSH|EC) PRIVATE KEY)' <dir>`。
