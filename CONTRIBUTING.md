# 参与 Mojobox 开发

Mojobox 的改动通常落在目录数据、协议、网站或构建工具之一。请保持改动聚焦，不要在同一个
PR 中顺带重排全部 JSON 或升级无关依赖。

## 开发准备

需要 Node.js 22 或更高版本。

```bash
npm ci
npm test
```

从任务分支开始工作。提交使用 Conventional Commits，例如：

```text
feat(catalog): 添加 example 插件记录
fix(pack): 修正 focus-kit 的 artifact 摘要
docs: 补充 Evidence 贡献说明
```

## 添加或更新插件

1. 在 `catalog/plugins/` 新增或修改一个以稳定插件 ID 命名的 JSON 文件。
2. 优先引用作者发布的 Manifest；目录代维护时标记 `registry-maintained`。
3. 仅填写能从代码、包元数据或上游文档核验的能力。
4. 已发布产物必须使用精确版本、固定 URL 和真实 SHA-256。
5. 未发布插件标记为 `unpublished`，不要写入 Pack Lock。
6. 运行 `npm test`。

更新 Manifest 的任何原始字节都会改变 `manifestDigest`。修改后必须同步所有引用它的 Lock
和 Evidence；不要通过重新格式化 JSON 制造无意义的摘要变化。

## 添加或更新 Pack

Pack 与 Lock 必须成对：

```text
catalog/packs/<id>.pack.json
catalog/packs/<id>.lock.json
```

Pack 写组合意图，Lock 写精确发布事实。提交前确认：

- Pack ID 和版本与 Lock 的 `pack` 一致；
- 每个组件都存在于 `catalog/plugins/`；
- 组件版本完全一致，不使用 range 或 tag；
- `source` 是精确 npm 包版本；
- `manifestDigest` 来自仓库文件的原始字节；
- `artifactDigest` 来自实际下载的 tarball；
- 所有必需组件都有可下载产物。

随后运行 `npm test` 和 `npm run build`。构建会重新下载或读取缓存产物并校验摘要。

## 添加 Evidence

Evidence 不是兼容口号，而是一条可复验记录。至少核对：

- subject ID、版本和 artifact digest；
- Manifest digest；
- issuer、Host、Adapter 和 DSH 版本；
- Admission Profile 或 suite ID、版本和 digest；
- 使用的规范 revision；
- 每项 check 的真实结果；
- 签发时间、有效期和撤回状态。

fixture 用于验证协议形状，不计为生产证据。不同宿主对同一产物签发 Evidence 时必须保留
各自 issuer、Host Descriptor 和适用范围。

## 修改 Schema 或校验器

Schema 改动至少同时包含：

- 一个应通过的 `fixtures/valid/` 样本；
- 一个应拒绝的 `fixtures/invalid/` 样本；
- `scripts/validate.mjs` 中必要的跨文件语义检查；
- PR 中对兼容影响和旧数据迁移方式的说明。

不要为了单个目录条目放宽公共 Schema。无法由 Schema 表达的跨文件约束应放入校验器，
但不要把网络访问、插件执行或宿主安装塞进校验阶段。

## 修改网站

网站必须保持纯静态并以 catalog 为唯一数据源。不要手工修改
`site/public/generated/` 或 `dist/`。

```bash
npm run dev
npm run build
BASE_PATH=/dsh-mojobox/ npm run build
npm run preview
```

检查搜索、筛选、详情和所有下载链接。布局变化还应验证窄屏与桌面视口、键盘焦点、空状态
和下载失败状态。

## 更新上游规范

不要把 `spec-revisions.json` 中的 commit 改成分支名或 tag。升级步骤是：

1. 阅读上游变更并确定影响范围；
2. 更新精确 revision 和 vendored Schema/许可证；
3. 更新受影响的 profile 与 fixture；
4. 重新运行 suite 后才重新签发 Evidence；
5. 运行 `npm test` 和 `npm run build`。

旧 Evidence 应保留旧 revision。新的生态入口 revision 不会自动让历史 TUI admission 证据失效，
也不能自动证明它符合新协议。

## 提交前检查

```bash
npm test
npm run build
git diff --check
git status --short
```

确认没有提交以下内容：

- `node_modules/`、`.cache/`、`dist/` 或 `site/public/generated/`；
- token、cookie、凭据、用户目录或本机绝对路径；
- 未解释的生成文件或二进制产物；
- 与当前任务无关的格式化和依赖升级。

## Pull Request

PR 标题和说明使用中文，列出目的、协议影响、验证命令和实际结果。目录、Evidence 或页面变化
应附关键前后对比；协议变化必须说明兼容性和回退方式。
