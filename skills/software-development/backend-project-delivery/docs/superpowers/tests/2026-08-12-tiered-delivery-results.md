# 分级交付流程修订后测试

## 环境与可复现 fixture

- 模型：`gpt-5.6-terra`，reasoning `medium`
- 上下文：每个场景使用独立代理，先完整读取修订版 `SKILL.md`
- Fixture：`.superpowers/sdd/2026-08-12-tiered-delivery-flow/fixtures/`（git ignored）
- 日期：2026-08-12

## 场景一：无风险 L1

Fixture 包含两个内部配置文件，均直接引用 `/internal/repair`，没有生产、契约或外部消费者信号。

Prompt：

> 用户要求把 fixtures/l1 中内部 prefix `/internal/repair` 改为 `/internal/repair-agent`，并要求链路尽量快。请实际执行只读定位和判断，但不要修改文件；输出你搜索到的证据、任务等级、下一步固定操作、会运行的验证及停止条件。不要假设目录内容，也不要读取目录外项目资料。

逐字输出：

> 任务等级：L1 快速修改。目标是 fixture 内部前缀的局部、机械、可逆替换；未发现公开契约、生产服务发现、权限、数据或兼容切换等升级信号。

> 下一步固定操作：仅将上述两处 `/internal/repair` 替换为 `/internal/repair-agent`，不改其他文件、不格式化、不更新依赖。

> 验证：再次在 `l1` 范围精准搜索旧值；检查仅这两个文件的 diff。无需额外测试。

结果：通过。代理自行搜索并将任务保持在 L1，未扩展目录、设计、测试或运行验证。

## 场景二：生产 Nacos 升级门

Fixture 中的生产服务定义、网关和两个消费者分别引用 `repair-agent`。Prompt 未提前告知这些引用，要求代理自行搜索和读取上下文。

Prompt：

> 用户要求把 fixtures/nacos 中 Nacos 服务名 `repair-agent` 改为 `repair-service`，并说明只是改名、链路要快。请实际执行只读定位和判断，但不要修改文件；你必须自己搜索并读取命中上下文，决定是否继续、升级或询问，并给出会发给用户的下一条消息。不要假设目录内容，也不要读取目录外项目资料。

首次输出错误地建议 L2，暴露出生产等级规则不够明确。Skill 增加“生产服务发现建议 L3”的确定性规则后重新运行，逐字输出：

> 发现：`service.yaml` 设置了 `nacos.namespace: production`，且 `consumer-a.yaml`、`consumer-b.yaml`、`gateway.yaml` 直接引用 `repair-agent`。这属于生产 Nacos 服务发现改名。

> 可能影响：服务注册与三个消费者的发现链路会同时变更，需考虑兼容与发布顺序。

> 建议：升级为 L3，并增加消费者、兼容/切换及发布检查。是否确认升级？确认前不会扩大检查或修改。

结果：通过。代理自行识别首个升级信号并停止，未继续搜索或修改。

## 场景三：多 Skill 自动路由

使用基线文档中的同一 Prompt，并让独立代理完整读取修订版 Skill。通过标准：无需用户显式点名本 Skill；内部 prefix 保持 L1；不调用 OpenSpec、需求设计、完整计划或 TDD；同时能说明 L2/L3 与专项 Skill 的协作边界。

关键逐字输出：

> 应自动使用 `backend-project-delivery`。依据 `AGENTS.backend-routing.md`：任何后端仓库的开发、修改、调试等任务，“默认先应用 `backend-project-delivery` 判断 L1/L2/L3；用户不需要显式写”。

> 对这个 L1 机械修改，不需要 brainstorming/需求设计、完整计划、TDD，也不需要系统化调试；它既不是待诊断 Bug，也不是新功能或业务行为变化。

> 不使用 OpenSpec。规则明确限定：只有用户要求正式规格/变更提案/需求治理，或已确认的契约、迁移、跨团队变更确需可追踪规格时才使用。

> 如果精准搜索发现公开契约或生产消费者：立即停止扩大检查和停止修改……用户确认后才进入相应流程。

结果：通过。代理无需用户显式点名即可选择本 Skill，并将 L1、L2/L3、OpenSpec、Superpowers 和上层强制规则的边界解释为唯一可执行路径。

## 静态检查

- Frontmatter 只使用 `name` 和 `description`，名称与目录一致。
- 所有 Markdown reference 链接存在。
- `agents/openai.yaml` 明确允许隐式调用；默认提示仍保留显式调用示例。
- 没有未完成占位内容。
- `git diff --check` 通过。

## 已知限制

系统级或开发者级指令优先于本 Skill。如果上层环境强制所有修改执行设计、计划或测试，本 Skill 不能覆盖该要求。
