# DSH Mojobox

Mojobox 是 DSH 生态的组件目录、插件整合包和兼容证据项目。仓库同时提供机器可读目录、
可复现的离线 `.dshpack` 和一个纯静态浏览网站；安装与回滚由 EAC 等宿主自己的 Adapter
负责。

> Manifest 描述插件，Pack 表达组合意图，Lock 固定发布事实，Evidence 记录验证结果，
> Adapter 负责宿主差异。

## 当前状态

- 阶段：`0.1.0-alpha.0`，协议和目录仍可能发生不兼容调整。
- 目录：10 个插件、2 个维护 Pack、3 条生产 `Parsed` Evidence。
- 网站：支持浏览、搜索、筛选和下载 Manifest、Lock 与离线 `.dshpack`。
- EAC：已有独立的只读计划与事务安装实现，本仓库不包含 EAC 的私有安装代码。
- TUI：当前仅保留固定 revision 的多宿主 fixture 和 admission 映射，不代表实时兼容认证。

没有发布产物的插件会标记为 `unpublished`，不会进入 Pack Lock。插件没有作者提供的
`dsh-plugin.json` 时，目录记录必须用 `x-mojobox-maintenance.source = registry-maintained`
说明它由目录维护，不能伪装成作者声明。

## 五分钟开始开发

环境要求：Node.js 22 或更高版本，npm 使用仓库内的 lockfile。

```bash
git clone https://github.com/lanyun077/dsh-mojobox.git
cd dsh-mojobox
npm ci
npm test
```

启动静态网站：

```bash
npm run dev
```

构建发布目录：

```bash
npm run build
npm run preview
```

`npm run build` 和 `npm run dev` 会按 Pack Lock 下载插件 tarball，并在写入 `.dshpack`
前验证 SHA-256；首次运行需要网络。下载缓存位于 `.cache/artifacts/`，构建结果位于
`dist/`，两者都不是协议事实源。

## 从哪里开始改

| 目标 | 主要位置 | 完成前检查 |
| --- | --- | --- |
| 添加或更新插件 | `catalog/plugins/` | 来源、精确版本、Manifest 与 artifact 摘要 |
| 添加或更新整合包 | `catalog/packs/` | Pack/Lock 成对，组件版本与摘要一致 |
| 添加兼容证据 | `catalog/evidence/` | subject、宿主、suite、revision 和 digest 全部绑定 |
| 修改协议格式 | `schemas/`、`fixtures/` | 同时提供合法与非法 fixture，说明兼容影响 |
| 修改宿主准入映射 | `profiles/`、`spec-revisions.json` | 固定上游 revision，不改写历史 Evidence |
| 修改网站 | `site/`、`vite.config.mjs` | 根路径和仓库子路径都能构建 |
| 修改目录校验 | `scripts/validate.mjs` | 错误输入必须失败，不能只测成功路径 |
| 修改离线包生成 | `scripts/build-site.mjs` | 精确 Lock、摘要校验和确定性 ZIP 保持成立 |

具体步骤见 [贡献指南](CONTRIBUTING.md)，模块边界和数据流见
[架构说明](docs/architecture.md)。

## 仓库结构

```text
catalog/
├── plugins/       dsh-std 插件 Manifest 或明确标记的目录维护记录
├── packs/         Pack Manifest 与对应的精确 Pack Lock
└── evidence/      与产物、宿主和测试套件绑定的兼容证据
fixtures/
├── valid/         必须通过的协议样本
└── invalid/       必须被拒绝的协议样本
profiles/          EAC/TUI 等宿主的准入与能力快照
schemas/           Mojobox Pack、Pack Lock、Evidence Schema
scripts/           校验器和确定性站点/离线包生成器
site/              无后端的目录界面
vendor/dsh-std/    固定 revision 的上游 Manifest Schema 与许可证
spec-revisions.json  所有上游协议和证据输入的固定坐标
```

## 数据与发布流程

```text
catalog + schemas + profiles + spec revisions
                    │
          npm test  │  校验格式、引用、摘要和跨宿主证据
                    ▼
            scripts/build-site.mjs
              ├─ catalog.json
              ├─ Manifest / Pack / Lock 下载文件
              └─ deterministic .dshpack
                    │
                    ▼
                 Vite dist
                    │
             GitHub Pages / EAC snapshot
```

`catalog/` 才是事实源。`site/public/generated/` 与 `dist/` 都是生成物，不应手工编辑或提交。

## 协议边界

- [`dsh-std`](https://github.com/Yan-Zero/dsh-std) 定义插件 Manifest、facet、权限和运行时交互。
- [`dsh-distribution`](https://github.com/T-Auto/dsh-distribution) 定义完整 DSH 环境的身份、组成、数据管理和迁移；Mojobox 不重复定义这些语义。
- [`dsh-ecosystem-spec`](https://github.com/T-Auto/dsh-ecosystem-spec) 是生态入口。其旧 TUI admission 材料只作为已有 Evidence 的固定输入。
- Mojobox 只拥有 Catalog、Pack、Pack Lock、Evidence 和 `.dshpack`。
- EAC、TUI 或其他宿主拥有本地能力判断、用户确认、安装、文件锁、启动验证和回滚。

`.dshpack` 是插件组合的离线运输容器，不是完整 DSH 环境备份，也不包含 profile、会话、
用户数据或凭据。Mojobox Evidence 是有范围的测试记录，不是安全认证或永久兼容承诺。

## 固定上游版本

精确 revision、Schema 路径、suite digest 和 Host Descriptor digest 统一记录在
[`spec-revisions.json`](spec-revisions.json)。升级上游规范时必须显式更新相关 Schema、fixture
与 Evidence，不能跟随浮动 `main`，也不能用新 revision 改写旧证据。

当前 Mojobox Manifest 基线与旧 TUI admission fixture 使用的 `dsh-std` revision 不同；这是
已有证据的历史事实，不应通过填写空字段强行统一。详见[架构说明](docs/architecture.md)。

## 校验范围

`npm test` 会检查：

- 所有目录文件和合法 fixture 通过对应 Schema；
- 非法 fixture 被拒绝；
- Pack 与 Lock 的 ID、版本、组件集合一致；
- Lock 只包含精确 npm 版本；
- Manifest 原始字节摘要、artifact 摘要和 Lock 一致；
- Evidence 绑定精确 subject、Manifest、Host、suite 与规范 revision；
- 多宿主 fixture 使用相同 artifact，但保留独立 issuer、Host 和 Admission Profile。

校验通过只证明仓库内已声明的内容自洽，不证明插件安全，也不证明某宿主已经实现安装。

## 参与贡献

提交前至少运行：

```bash
npm test
npm run build
git diff --check
```

请使用任务分支和 Pull Request，不直接修改共享 `main`。详细的数据修改、摘要更新和 PR
要求见 [CONTRIBUTING.md](CONTRIBUTING.md)。

## 当前不做

Mojobox 当前不提供账号、评分、通用依赖求解、PKI、插件二进制上传、云端安装、宿主安装
事务或完整环境迁移。具体路线以 issue、PR 和可验证实现为准，不以 README 中的设想作为已交付能力。

## 许可证

[MIT](LICENSE)
