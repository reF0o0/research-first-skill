# CLAUDE.md

这个仓库是一个 Codex skill：`research-first/`。

## 项目概述

让 agent 干活时遇到复杂问题先上网查现成方案，而不是硬猜；简单问题照旧直接做。三条线：先按判据决定该不该查（该查的七条、不该查的五条）；再按问题类型分流渠道（GitHub Issues 查报错、GitHub 代码搜索查实现、jina 抓官方文档正文、Exa 查选型、Release 查版本、仓库主页和 README 查同类产品的设计）；最后给搜索本身设成本上限（2–3 轮没结论就停下来汇报）。

两个结论是实测出来的，改 skill 时别丢：`gh search` 的多词是严格 AND，`field_validator` 命中 50+ 条而 `pydantic field_validator not called` 是 0 条，所以 query 要短、范围靠 `--repo` 给；`gh search` 查不到东西时输出为空而退出码仍是 0，不能当成「没有先例」。

## 目录结构

- `research-first/SKILL.md` — agent 读的入口：该不该查的判据、渠道路由表、搜索姿势、停止条件、与 agent-reach 的边界
- `research-first/agents/openai.yaml` — UI 元数据
- `README.md` — 面向用户的安装与使用文档

## 约定

- 这个 skill 只有指令，没有脚本，也不引入任何依赖。命令必须能在本机直接跑通再写进 `SKILL.md`。
- `SKILL.md` 里的命令示例都要保留可复制的完整形式（含 `--limit`、`--json` 字段列表），不要写成伪代码。
- 「少词、靠 `--repo` 限定、空结果先减词」这三条是实测结论，改动前要有新的实测数据。
- `SKILL.md` 里的渠道要和环境里真实存在的工具对齐：`gh`、`mcporter`（Exa）、`curl`。加新渠道前先确认它在本机装好了。
- 和 `agent-reach` 的分工要一直留着：用户明确要求调研归 `agent-reach`，这个 skill 只管 agent 自己补课。别把两边写成互相替代。
- 提交前确认没有本机用户名、密钥、真实会话数据混进来。
- README 的风格跟 [deepseek-vision-skill](https://github.com/reF0o0/deepseek-vision-skill) 对齐：能力用 `**要点**：说明。` 列表、安装和使用用 `1  ` 起头的编号段、实测结论放表格；README 不放目录树（目录结构写在本文件里）。
- 发布：打 `v*` tag 会触发 workflow 把 `research-first/` 打包成 zip 挂到 Release。
