# DSH Mojobox

Mojobox 是面向 DSH 生态的组件目录、整合包和兼容证据平台。当前仓库已包含
`v0.1` 规格样本和静态目录网站，不执行安装，也不读取用户环境。

核心边界：清单负责描述，Lock 负责固定发布事实，Evidence 负责记录验证结果；宿主 Adapter
负责环境判断、安装事务和回滚。

## 当前内容

```text
catalog/
├── plugins/      dsh-std v0.15 插件 Manifest
├── packs/        Pack Manifest 与 Pack Lock
└── evidence/     兼容证据
fixtures/
├── valid/        必须通过的输入
└── invalid/      必须拒绝的输入
schemas/          Pack、Pack Lock、Evidence JSON Schema
profiles/         宿主 Admission Profile
scripts/          最小目录校验器
site/             插件与 Pack 静态目录
vendor/dsh-std/   固定 revision 的 Manifest Schema
```

当前目录包含 10 条插件记录、2 个项目维护 Pack 和 3 条 `Parsed` 生产证据。插件未发布作者
`dsh-plugin.json` 时，记录使用 `x-mojobox-maintenance.source = registry-maintained` 明确标记，
不把目录元数据伪装成作者声明。没有可下载产物的记录标记为 `unpublished`，不会进入 Pack Lock。

## 校验

需要 Node.js 22 或更高版本：

```bash
npm ci
npm test
npm run build
npm run dev
```

校验器会检查：

- catalog 与合法 fixtures 通过对应 Schema；
- 非法 fixtures 被拒绝；
- Pack 与 Lock 的 id、版本和组件集合一致；
- Lock 只使用精确 npm 版本；
- Manifest 与记录的 npm tarball SHA-256 和 Lock 一致；
- Evidence 绑定精确 Manifest、artifact、suite 和 `dsh-std` revision。
- 多宿主 fixture 必须绑定同一 artifact/Manifest，并保留不同 issuer、host 和 Admission Profile；fixture 不计为生产兼容结论。
- 多宿主 fixture 复用 `dsh-ecosystem-spec` 的真实 fixture 路径、摘要和 requirement ID；EAC 使用自己的检查 ID 映射，不把 TUI 私有准入要求冒充为跨宿主标准。

`manifestDigest` 是对应 JSON 文件原始字节的 SHA-256。`artifactDigest` 是 Lock 中下载产物
原始字节的 SHA-256；当前四个已发布产物均由精确 npm tarball 实算，仓库不托管二进制副本。

静态网站提供插件与 Pack 搜索、筛选、详情、来源、哈希和证据展示，并下载 Manifest、Lock
与离线 `.dshpack`。构建阶段按 Lock 下载 npm tarball、校验 SHA-256 后写入
`objects/sha256/<content-hash>`。目录维护的 Manifest 也按其 digest 写入 objects，确保离线包可复验；
生成内容位于 `dist/`，不作为协议事实源提交。

## 固定规范

- `dsh-std`: `3df054302468d2091859db4b3bd079042d33f100`，Manifest `0.15`
- `dsh-ecosystem-spec`: `d28c267fe7fd775428ec2dccd65b0b7efd4dacee`，仅作为
  `tui-admission/0.15` Profile 参考，不作为公共 Pack 的全局要求
- `dsh-TUI`: `c8a59a3d88066f61c36b9dbf4817fb4a5fb68031`，其固定的
  `dsh-ecosystem-spec` revision 与上项一致

该生态规范和 TUI 固定 `dsh-std` `614dfa1ac168db79fcf4577cf0ebb34e2e3b944b`。Mojobox 暂时保留
较早的 `3df054302468d2091859db4b3bd079042d33f100`：新 schema 强制作者级的
`requires`、`permissions`、`contributes` 和 `subscriptions` 声明，不能替目录维护记录擅自补空值。
阶段 4 fixture 使用的 `valid-plugin.json` 同时通过两个 revision 的 Manifest schema。

精确 revision 记录在 [`spec-revisions.json`](spec-revisions.json)。规范升级必须显式修改该文件、
vendored Schema、fixtures 和相关证据，不能跟随浮动主分支。

## 暂不包含

当前版本不提供账号、通用依赖求解、签名基础设施、云端安装或宿主安装事务。
EAC 的 Host Descriptor、只读适用性计划和可靠安装属于后续阶段，并继续保留在 EAC 仓库。
