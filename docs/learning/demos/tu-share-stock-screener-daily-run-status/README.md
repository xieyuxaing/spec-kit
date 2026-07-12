# Daily Run Status Spec Workflow Demo

> 学习对象：`/home/xieyx/projects/tu-share-stock-screener`
>
> 真实提交：`9cc1b86 Add daily run status dashboard`
>
> 学习目标：用一个已经完成的真实功能，反向理解 Spec Kit 的
> spec -> plan -> tasks -> implementation -> verification 工作流。

## 这个 demo 是什么

这是一个学习用 worked example。它把 `tu-share-stock-screener`
项目中已经完成的 `daily-run-status` 功能，重新整理成一套
Spec Kit 风格的学习材料。

它不是官方示例，不是主项目的事实源，也不是让你直接照抄的模板。
它的价值在于：你学习 Spec Kit 之后，可以拿这个案例检查自己是否真的理解了
“一个需求应该怎样被拆成 spec、plan、tasks、测试、实现和文档影响”。

## 推荐阅读顺序

先按顺序读一遍，不要一开始就跳到答案：

1. [00-project-context.md](00-project-context.md)
2. [01-feature-request.md](01-feature-request.md)
3. [02-spec.md](02-spec.md)
4. [03-plan.md](03-plan.md)
5. [04-tasks.md](04-tasks.md)
6. [05-implementation-notes.md](05-implementation-notes.md)
7. [06-verification.md](06-verification.md)
8. [07-doc-impact.md](07-doc-impact.md)
9. [answer-key.md](answer-key.md)

如果你刚开始学 Spec Kit，先读 `00` 到 `04`。如果你已经学到
`/speckit.implement` 和验证阶段，再读 `05` 到 `07`。
`answer-key.md` 留到最后，用来对答案。

## 文件职责

| 文件 | 作用 | 对应 Spec Kit 阶段 |
| --- | --- | --- |
| `00-project-context.md` | 说明业务背景、现有项目记忆、约束和事实源 | constitution / memory |
| `01-feature-request.md` | 把自然语言需求整理成可规格化的问题 | input / clarify |
| `02-spec.md` | 写出用户故事、需求、边界、实体和成功标准 | specify |
| `03-plan.md` | 说明技术路线、数据流、测试策略和风险 | plan |
| `04-tasks.md` | 拆成可执行任务，按 TDD 顺序排列 | tasks |
| `05-implementation-notes.md` | 记录真实提交里代码如何落地 | implement |
| `06-verification.md` | 记录验证命令、证据和覆盖范围 | verify |
| `07-doc-impact.md` | 解释哪些内容进入长期项目记忆 | doc impact |
| `answer-key.md` | 自测题、参考答案、常见错误 | review |

## 和真实主项目的关系

主项目仍然以这个路径为事实源：

```text
/home/xieyx/projects/tu-share-stock-screener
```

本 demo 只记录学习材料。不要从这里反向覆盖主项目代码。
如果后续主项目的 `daily-run-status` 功能继续演进，应优先更新主项目的
`docs/features/` 和 `docs/project/`，然后再决定是否更新这个 learning demo。

## 和 Specify 的关系

这个案例不是由 `specify init` 直接生成的真实 feature 目录。
它是一个“Specify-shaped reconstruction”：

- 用 Spec Kit 的思路重写真实功能交付过程。
- 使用 `spec.md`、`plan.md`、`tasks.md` 的思维方式组织材料。
- 保留主项目自定义的 `Feature Spec Impact` 和 `Project Doc Impact` 纪律。
- 让你学习后可以判断：如果当时正式用 Spec Kit，应该如何写这些工件。

以后你可以把它迁移成真实的 Spec Kit feature 目录，例如：

```text
specs/001-daily-run-status/
  spec.md
  plan.md
  tasks.md
```

当前先保持在 `docs/learning/demos/`，避免把学习案例误认为
Spec Kit 仓库自己的产品功能。

## 学习时怎么用

建议做三轮练习。

第一轮：只看 `01-feature-request.md`，自己尝试写一个 spec。
写完后对照 `02-spec.md`，检查是否漏掉用户故事、边界、验收条件或非目标。

第二轮：只看 `02-spec.md`，自己尝试写 plan 和 tasks。
写完后对照 `03-plan.md` 和 `04-tasks.md`，检查是否真的按测试先行拆任务。

第三轮：看真实提交 `9cc1b86`，判断实现是否满足 spec。
再对照 `06-verification.md` 和 `07-doc-impact.md`，检查是否能说明
“验证覆盖了什么”以及“哪些项目记忆必须更新”。

## 判断你是否学会了

你可以在不看 `answer-key.md` 的情况下回答这些问题：

1. 为什么这个功能需要 feature spec，而不是只改代码？
2. 哪些内容属于短期 feature spec，哪些内容属于长期 project memory？
3. 为什么 daily status API 必须是只读？
4. 为什么不能新增 SQLite 表？
5. 后端测试应该先覆盖哪两个业务分支？
6. 前端测试为什么不能只断言“每日状态”标题出现？
7. `backup` 状态为什么用 mtime，测试为什么要显式设置 mtime？
8. 什么证据可以证明这个功能已经完成？
9. 如果以后要正式迁移到 Spec Kit，你会把这些文件放在哪里？

如果你能清楚回答这些问题，并能指出对应文件和证据，说明你已经掌握了这个案例。
