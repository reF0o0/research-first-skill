# research-first skill

research-first 是一个 Codex skill，管的是 agent 干活时的一个习惯：遇到复杂问题先上网查现成做法，别硬猜；简单问题照旧直接动手，不要为了查而查。

陌生报错、版本敏感的 API、选型、从零设计新功能，以及自己试过一次没搞定的情况会自动加载，也可以显式 `$research-first` 调用。

## 能力

- **先判断再动手**：把「该查」和「不该查」写成了两列判据。该查的是报错看不懂、API 用法和版本对不上、要选型找轮子、从零设计功能或界面要参考同类实现、已经失手一次、出错代价高（鉴权 / 加密 / 数据迁移 / CI）；不该查的是常规增删改、纯语法算法、答案就在当前仓库里。既防止漏查，也防止每件小事都去联网。
- **顺序固定为 本地 → 官方文档 → 社区**：先 `rg` 仓库、读锁文件和 CHANGELOG，本地能回答的绝不联网，本地答不了才出去查。
- **按问题类型分流渠道**：报错丢给 GitHub Issues，实现方式丢给 GitHub 代码搜索，官方用法用 jina 抓文档正文，选型和方案对比用 Exa，版本兼容看 Release，同类产品看仓库主页的 Features 和用户手册。
- **新功能先看同类实现**：从零设计功能、界面或架构时，先找 2–3 个同类产品，读它们的 Features 和用户手册，记下可借鉴的交互、状态摆法，以及它们明确声明不做什么。
- **搜索词要少**：`gh search` 的多词是严格 AND，词越多越容易搜成 0 条。同一个问题从 `field_validator` 的 50+ 条、到 `field_validator not called` 的 2 条、再到 `pydantic field_validator not called` 的 0 条。正确姿势是 query 只留 2–4 个最有辨识度的词，范围靠 `--repo owner/repo` 限定。
- **空结果不等于没有先例**：`gh search` 查不到时输出为空、退出码仍是 0，不报错。这种情况按「查询太窄」处理——减词、加引号、换成更独特的符号名重试，而不是当成「没人做过」的结论。
- **来源分级**：官方文档 → 源码 / CHANGELOG → maintainer 确认过的 issue 回复 → 高赞社区答案 → 博客。低可信来源只当线索，不当结论。
- **跟本机环境对齐**：查到写法先看本地版本（`python -V`、`node -v`、`pip show`、`cat package.json`），不把最新版写法套到旧版本上。
- **有成本上限**：同一问题查 2–3 轮还没结论就停下来汇报候选方案，不允许无限查下去；引用外部资料要在答复里给出链接。
- **跟 agent-reach 划清边界**：用户明确说「调研 / 查一下 / 看看别人怎么说」走 agent-reach（多平台和登录态渠道都归它）；这个 skill 只管 agent 自己干活时主动补课。

## 安装

1  把[仓库链接](https://github.com/reF0o0/research-first-skill)发给 agent，说“下载并安装这个 skill”，agent 会自动完成下载、安装。

2  从 [Releases 页面](https://github.com/reF0o0/research-first-skill/releases) 下载压缩包，解压后把其中的 `research-first` 目录放到 skill 目录：

   - Codex：`~/.codex/skills/research-first/`

3  也可以从源码装：

   ```bash
   git clone https://github.com/reF0o0/research-first-skill.git
   cp -R research-first-skill/research-first ~/.codex/skills/
   ```

更新重做一遍即可：重新下压缩包覆盖，或 `git pull` 后重新 `cp -R`。

## 使用

1  平时不用管它怎么触发。遇到「这个报错没见过」「这个库的写法不确定」「有没有现成的轮子」「这个新功能别人是怎么做的」这四类情况时它会自动被加载，也可以显式 `$research-first` 调用。

2  查报错和查实现方式，都用 `gh`，都不用打开浏览器：

   ```bash
   gh search issues "field_validator" --repo pydantic/pydantic --limit 3 --json title,url
   gh search code '"normalize_embeddings"' --limit 3
   ```

   实测输出（同一台机器、同一个 `gh` 登录态）：

   ```
   [{"title":"field_validator in Pydantic_dataclass not inherited from parent", ...},
    {"title":"field_validator changing validation order", ...}]

   AIAnytime/Document-Buddy-App:vectors.py: encode_kwargs: dict = {"normalize_embeddings": True},
   AbeEstrada/mlx-rag:main.py: encode_kwargs={"normalize_embeddings": True},
   FareedKhan-dev/rag-zero-hallucinations:README.md: kw = {"normalize_embeddings": True, ...}
   ```

   注意 `--repo` 的位置：限定到库自己的仓库时 issue 标题往往根本不写库名，所以范围靠参数给，不要靠搜索词给。

3  读官方文档正文、以及对没有明确文档的问题做方案搜索：

   ```bash
   curl -s "https://r.jina.ai/https://pypi.org/project/requests/"
   mcporter call exa.web_search_exa query="python asyncio task was destroyed but it is pending" numResults=3
   ```

   实测输出：

   ```
   Title: requests
   URL Source: https://pypi.org/project/requests/
   Markdown Content:
   **Requests** is a simple, yet elegant, HTTP library.

   Title: Please explain "Task was destroyed but it is pending!" after cancelling tasks
   URL:    https://stackoverflow.com/questions/40897428/...
   ```

   前者把网页转成 Markdown 正文，省掉整页 HTML 进上下文；后者适合「这类问题一般怎么做」而不是「这个符号在哪」。

4  怀疑是版本差异时，直接看 Release：

   ```bash
   gh api repos/pallets/flask/releases --jq '.[0:5][].tag_name'
   ```

   实测输出：

   ```
   3.1.3
   3.1.2
   3.1.1
   3.1.0
   3.0.3
   ```

## 搜索词长度

这条是踩出来的，也是 skill 里改动最大的一处：`gh search` 的多词是严格 AND，不是模糊匹配。同一台机器、同一时间实测：

| 搜索词 | 命中数 |
|---|---|
| `field_validator` | 50+（打满 `--limit 50`） |
| `field_validator not called` | 2 |
| `pydantic field_validator not called` | 0 |
| `pydantic v2 field_validator not called` | 0 |

代码搜索一样，词堆多了直接归零：

| 搜索词 | 结果 |
|---|---|
| `SentenceTransformer encode normalize_embeddings --language python` | 空 |
| `normalize_embeddings` | 有结果 |
| `"normalize_embeddings"` | 有结果，且都是真实用法 |

所以「报错原文 + 库名 + 版本」这种看起来很自然的写法反而最容易搜空。skill 里定的是：**query 只留 2–4 个最有辨识度的词，范围靠 `--repo` 给，结果为空先减词再判断。**

## 环境要求

- [Codex](https://openai.com/zh-Hans-CN/codex/) App 或 CLI
- `gh` CLI 且已 `gh auth login`（Issues 搜索、代码搜索、Release 查询都走它）
- 可选：`mcporter`（Exa 网页搜索）、`curl`（抓文档正文，macOS 自带）
- 不需要装任何 Python 依赖，这个 skill 里只有指令，没有脚本

## 注意事项

- `gh search` 的空结果退出码是 `0`，脚本化和 agent 判断时别把「查不到」当成「不存在」。
- Exa 那条依赖 `mcporter`。没装的话用 `agent-reach` 走同样的搜索，或者直接 `curl` 官方文档。
- 这个 skill 不替代读代码。工作区里能回答的问题，先读工作区。
- 用户明确要求「调研一下」时应该由 `agent-reach` 接手，不是这个 skill。
- 仓库里不含任何 API key、本机用户名或会话数据。
