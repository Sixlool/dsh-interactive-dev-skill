# DSH Interactive Dev Skill（交互式开发工作流）

让 AI 用**结构化选择题**和你确认需求、分阶段推进开发的 DeepSeek Harness Skill。

## 这是什么

把一次真实的 DSH 桌宠插件实战（[dsh-desktop-pet](https://github.com/Sixlool/dsh-desktop-pet)）中验证有效的方法论沉淀成可复用 Skill：

- **选择题式需求收集**：外观/交互/联动/持久化/位置，一次问完一组，每题带推荐项
- **文档先行**：需求基线 → 架构设计 → 逐条对照实现
- **先预览后实装**：图形/UI 用无头浏览器渲染验证，满意才写进代码
- **修复迭代**：追加不可变包 + update，先诊断再改
- **发布**：整理成标准仓库上传 GitHub

## 内容

```
dsh-interactive-dev-skill/
├── SKILL.md                    # 完整方法论（何时用/工作流/提问规范/坑/清单）
├── examples/
│   ├── 01-需求五问示例.md       # 需求收集选择题模板
│   ├── 02-架构决策点示例.md     # 决策点收敛提问
│   └── 03-视觉预览验证示例.md   # 渲染→复查→实装循环
└── README.md
```

## 用法

在 DSH 会话中让 Agent 加载本 Skill（或参考 `SKILL.md` 自行执行），
然后从一句模糊需求开始（如"我想做一个能显示图片的桌宠"），Agent 会：
问一组选择题 → 写需求基线 → 查运行时契约写架构 → 实现 → 视觉预览验证 → 发布。

## 环境

- DeepSeek Harness（Web 界面）
- 依赖 DSH 的 Inspect / Slots / Events / Cordis 插件系统
- 视觉验证依赖 vision 工具链（vision_html_screenshot / vision_describe）

## License

[MIT](./LICENSE)
