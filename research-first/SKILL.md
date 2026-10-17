---
name: research-first
description: Look up prior art — web search, GitHub code/issues, official docs, changelogs — before designing, debugging, or picking a tool for a non-obvious problem. Use when the problem is unfamiliar, version-sensitive, has likely been solved publicly (an error message, a library API, a wiring question, a build failure), or when a first attempt already failed. Skip routine edits, simple lookups, and anything the workspace or your own knowledge already settles.
metadata:
  short-description: 遇到复杂问题先查现成方案
---

# Research First

目标：复杂问题上少猜、少重复造轮子；简单问题上不浪费轮次。

## 一、先判断该不该查

命中任意一条就**先查**：

- 报错/堆栈看不懂，或看起来自相矛盾
- 要用的库、API、配置项、构建工具，不确定**当前版本**的正确写法（版本敏感的东西尤其容易记错）
- 「有没有现成方案」类问题：选库、选工具、找官方示例、找可复用的实现
- 从零做新功能、新界面或新架构，要定交互和取舍 —— 动手前先找 2–3 个同类产品或开源实现，看它们怎么做、明确不做什么
- 已经按自己的判断试过一次，还是不对
- 出错代价高：鉴权、加密、数据迁移、并发、删改数据、CI/发布流程
- 要解释别人代码为什么这么写、或某个行为的成因

不查，直接动手：

- 常规增删改、重命名、格式化、写模板化的代码
- 答案就在当前工作区里 —— 先 `rg` / 读文件 / 看锁文件，比联网快得多
- 纯语法、纯算法、纯逻辑推导
- 用户已经把做法或规格定死了
- 查只是「再确认一下」自己的判断 —— 那就省掉

**顺序永远是：本地 → 官方文档 → 社区。** 仓库里的代码、`package.json` / `requirements.txt`、CHANGELOG、已有注释往往就是答案。

## 二、去哪里查

| 问题类型 | 首选 | 命令 |
|---------|------|------|
| 具体报错、异常、行为怪癖 | GitHub Issues | `gh search issues "<报错里最有辨识度的一段>" --repo <owner>/<repo> --limit 10 --json title,url,state` |
| 别人怎么实现某个功能 | GitHub 代码搜索 | `gh search code "<符号名>" --repo <owner>/<repo> --limit 10 --json path,url` |
| 官方用法 / API / 配置项 | 官方文档正文 | `curl -s https://r.jina.ai/<文档URL>` |
| 选型、方案对比、最佳实践 | Exa 网页搜索 | `mcporter call exa.web_search_exa query="<问题>" numResults=5` |
| 同类产品、交互设计参考 | 仓库主页 + README 的 Features / 用户手册 | `gh search repos "<关键词>" --limit 10 --json fullName,description,stargazersCount` |
| 版本变更、兼容性、破坏性改动 | Release / Changelog | `gh api repos/<owner>/<repo>/releases --jq '.[0:5][].tag_name'` |
| 中文社区场景、国内工具问题 | agent-reach | 读 `~/.agents/skills/agent-reach/SKILL.md` 与对应 `references/*.md` |

不知道仓库就先去掉 `--repo` 全局搜一轮（代码搜索可以加 `--language <lang>` 收窄），再挑命中的仓库限定重搜。需要结构化结果时用 `search_code` / `search_issues` / `search_pull_requests` 等 GitHub MCP 工具，比 CLI 更适合按仓库、语言、路径过滤。

## 三、搜索的正确姿势

- **少词，别堆词**：`gh search` 的多词是 **AND 合取**，词越多越容易归零。实测同一个问题：
  `field_validator` 命中 50+ 条 → `field_validator not called` 只有 2 条 → `pydantic field_validator not called` 直接 0 条。
  所以挑报错里**最有辨识度的一段**（如 `No module named 'xxx'`、`Cannot read properties of undefined`），控制在 2–4 个词。
- **靠 `--repo` 限定范围，不要把库名和版本写进 query**：`gh search issues "field_validator" --repo pydantic/pydantic` 比把 `pydantic v2` 塞进搜索词精确得多 —— 在库自己的仓库里，issue 标题通常不会出现库名。
- **空结果不等于没有先例**：`gh search` 查不到东西时输出为空、退出码仍是 `0`，不会报错。别把空结果读成「没人做过」，先减词、加引号、换成更独特的符号名再试一轮。
- **来源分级**：官方文档 → 源码 / CHANGELOG → maintainer 确认过的 issue 回复 → 高赞社区答案 → 博客。低可信来源只当线索，不当结论。
- **设计阶段先看成品**：找同类工具时先读 README 的 Features 和用户手册，别只看 star。记三条：它替用户省掉了哪一步、状态信息放在界面哪里、它明确声明不做什么。抄交互和取舍，不抄范围。
- **跟当前环境对齐**：查到写法后先看本地版本（`python -V`、`node -v`、`pip show <pkg>`、`cat package.json`）。别把最新版写法套到旧版本上。
- **换词不重喂**：同一 query 反复搜没有意义；换关键词、换语言、换平台（英文社区 → 中文社区，或反过来）。
- **输出要压缩**：`--json` + `--jq` 只取标题和链接；不要把整页 HTML、整篇文档灌进上下文。
- **查到的立刻落成文字**：结论 + 依据链接 + 在你的场景里怎么用。不要在多轮里反复重查同一个东西。

## 四、停止条件和汇报

- 同一问题查 2–3 轮还没有明确结论，就停下来：说清「目前查到什么 / 有哪些候选 / 卡在哪」，挑最合理的一条继续做，或把选择权交给用户。
- 参考了外部资料，就在答复里点一句来源（给链接），不要让用户以为是你凭记忆写的。照抄的方案必须说明它来自哪里。
- 搜索是为了把事情做对，不是为了交一份调研报告 —— 交付物永远是完成的任务，不是一堆链接。

## 五、边界

用户明确说「帮我搜索 / 调研一下 / 查查 X / 看看别人怎么说」时，那是 `agent-reach` 的职责（多平台、需要登录态的渠道都在那边），不要用本 skill 的命令硬凑。

本 skill 只管一件事：**你自己干活时，遇到复杂问题主动去补课**。
