# Spec Kit 学习文档

这个目录集中存放面向 `spec-kit` 源码学习的中文材料。

## 当前约定

- 主工作区：WSL 内的 `/home/xieyx/projects/spec-kit`。
- Windows 目录：`D:\studyProject\spec-kit` 只作为同步镜像；后续以 WSL 修改、验证、提交为准。
- 学习资料边界：只维护 `docs/learning/`，不修改官方文档入口文件 `docs/toc.yml`、`docs/index.md`、`docs/README.md`。
- 实战 demo 边界：后续用于学习、演练或验证 SDD 流程的 demo、草稿和记录都放在 `docs/learning/` 内，优先使用 `docs/learning/demos/`。
- 实战目标：把 Spec Kit 的 SDD 流程逐步应用到同级项目 `/home/xieyx/projects/tu-share-stock-screener`。

| 文档 | 用途 | 建议读法 |
| --- | --- | --- |
| [深度学习计划](learning-plan.md) | 安排学习顺序、练习和验收标准 | 先读，按周推进 |
| [使用原理与进阶技巧](spec-kit-principles-and-techniques.md) | 解释源码心智模型、执行链路、模块边界和调试方法 | 第 1 周后配合源码阅读 |
| [实战 demo 目录](demos/README.md) | 存放后续学习 demo、演练草稿和实践记录 | 做实战前先确认命名和边界 |

如果只想快速建立全局认识，先读学习计划的“学习路线总览”和进阶技巧的“总体心智模型”。如果目标是贡献代码，重点完成 integration、preset/extension、workflow、bundle 和测试章节的练习。
