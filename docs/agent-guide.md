# 使用 coding agent 开发 Mojobox

仓库根目录的 `AGENTS.md` 已为支持仓库指令的 coding agent 提供默认边界。本指南给出任务描述
模板，让 Agent 能更快定位事实源、完成校验并交付可审阅差异。

## 通用任务模板

```text
请先完整读取 AGENTS.md，并检查当前分支和工作区。

任务：<明确的单一目标>
输入：<上游仓库、npm 包、issue 或现有文件>
允许修改：<路径列表>
保持不变：<协议/行为边界>
完成标准：<可观察结果>

实现前读取目标 Schema、一个合法 fixture、一个非法 fixture和一个同类生产样本。
只陈述可核验事实，不编造兼容性或摘要。完成后执行 AGENTS.md 对应的最低验证，
并报告修改文件、实际结果、未验证项和摘要/Evidence 影响。不要 commit 或 push。
```

“允许修改”和“保持不变”可以省略，但对协议或跨仓库任务建议明确填写。

## 添加插件记录

```text
请先读取 AGENTS.md，然后为 <插件名> 添加 Mojobox 目录记录。

上游来源：<GitHub URL>
npm 包：<精确包名和版本；没有则写未发布>

核对作者 Manifest、package.json、license、repository 和实际 artifact。
有作者 dsh-plugin.json 时优先采用；没有时使用 registry-maintained，并且不补造作者权限、
facet 或运行时能力。已发布 artifact 计算真实 SHA-256，未发布则不得加入 Pack Lock。
不要修改公共 Schema。完成后运行 npm test。
```

## 创建或更新 Pack

```text
请先读取 AGENTS.md，为 <用途> 创建/更新一个 Mojobox Pack 和对应 Pack Lock。

组件：
- <插件 ID>@<精确版本>，required=<true|false>

只使用 catalog 中已有且具有真实 artifact 的组件。核对 Pack/Lock 的 ID、版本和组件集合，
从真实 Manifest 字节和 npm tarball 计算 SHA-256。不要实现依赖求解器，也不要改变插件
Manifest。完成后运行 npm test 和 npm run build，并报告生成的 .dshpack 名称。
```

## 签发 Evidence

```text
请先读取 AGENTS.md，为 <subject ID>@<version> 生成 <Host> 的 <Evidence level> Evidence。

测试输入：<suite/profile/host descriptor 路径或 revision>
实际测试结果：<命令和结果来源>

Evidence 必须绑定精确 artifact、Manifest、Host Descriptor、suite 和规范 revision。
不要把 fixture 当生产证据，不要把 Schema 通过写成 Tested/Observed，不要复用其他 issuer 的
结论。缺少真实测试输入时停止在差距报告，不生成虚假 Evidence。完成后运行 npm test。
```

## 修改协议

```text
请先读取 AGENTS.md，评估并实现以下 Mojobox wire contract 变化：<变化>。

先列出受影响的 Schema、fixtures、Validator、Catalog、Lock、Evidence 和网站消费者。
给出最小兼容方案；如果现有字段已经能表达需求，不新增字段。实现时至少增加一个 valid 和
一个 invalid fixture，并更新必要的语义校验。不要引入网络校验或执行插件。完成后运行
npm test、npm run build 和 git diff --check，并说明迁移与回退方式。
```

## 修改静态网站

```text
请先读取 AGENTS.md，修改 Mojobox 静态网站以实现：<用户工作流>。

保持 Catalog 为唯一数据源，不新增后端、不读取本机环境、不直接执行安装。
覆盖 loading、empty、error 和正常状态；检查窄屏、桌面、键盘焦点和长文本。
完成后运行 npm test、npm run build、BASE_PATH=/dsh-mojobox/ npm run build，
并提供关键视口截图和交互结果。不要提交生成目录。
```

## 审查 Agent 结果

合作者不应只看 Agent 的“已完成”结论。至少确认：

1. `git diff` 只包含任务相关修改。
2. 新数据通过 Schema 且与生产样本语义一致。
3. 摘要来自真实文件，不是占位符或复制值。
4. Evidence 等级与实际执行证据相符。
5. `npm test` 和所需构建命令确实运行成功。
6. `dist/`、`.cache/`、`site/public/generated/` 未进入提交。
7. Agent 没有把 Mojobox、`dsh-std`、`dsh-distribution` 和 Host Adapter 的职责混在一起。
