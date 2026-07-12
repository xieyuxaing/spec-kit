# CLI 参考

> 本文是 [英文原文](overview.md) 的中文翻译。命令、选项名称和文件路径保持原样。

Specify CLI（`specify`）管理规格驱动开发的完整生命周期，从项目初始化一直到工作流自动化。

## 核心命令

核心命令用于创建和管理 Spec Kit 项目。你可以用它们初始化包含必要目录结构、模板和脚本的新项目，确认系统是否已安装所需工具，以及查看版本和系统信息。

[核心命令参考 →](core.zh-CN.md)

## Integrations

Integration 用于把 Spec Kit 连接到 AI 编程 Agent。每个 Integration 都会为特定 Agent 设置合适的命令文件、上下文规则和目录结构。一个项目同一时间只能激活一个 Integration，但你可以随时切换。

[Integrations 参考（英文）→](integrations.md)

## Extensions

Extension 为 Spec Kit 增加新能力，例如领域专用命令、外部工具集成和质量门禁。Extension 通过 Catalog 发现，可以独立安装、更新、启用、禁用或移除。一个项目中可以同时存在多个 Extension。

[Extensions 参考（英文）→](extensions.md)

## Presets

Preset 用于定制 Spec Kit 的工作方式。它可以覆盖命令文件、模板文件和脚本文件，而无需修改工具本身。Preset 可以用来执行组织标准、适配团队方法论，或者本地化整个使用体验。多个 Preset 可以按优先级叠加，从而形成分层定制。

[Presets 参考（英文）→](presets.md)

## Workflows

Workflow 把多步骤规格驱动开发过程自动化为可重复执行的序列。它可以串联命令、提示词、Shell 步骤和人工检查点，并支持条件逻辑、循环、扇出/扇入，以及从中断点准确暂停和恢复。

[Workflows 参考 →](workflows.zh-CN.md)

## Bundles

Bundle 把现有 Extension、Preset、Workflow 和 Step 组合成一个可安装、带版本的单元。Bundle 不增加新行为，而是整理出一组基础能力，例如一个团队或角色需要的完整工具栈，并通过各组件自己的安装机制一次性安装，同时支持版本固定、冲突检查和来源追踪，以便安全更新和移除。

[Bundles 参考（英文）→](bundles.md)
