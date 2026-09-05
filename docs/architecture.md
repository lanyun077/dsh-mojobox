# Mojobox 架构与协议边界

本文面向准备修改目录、协议、构建流程或宿主 Adapter 的开发者。字段的规范性定义以 JSON
Schema 和 fixture 为准；本文解释模块如何协作，不另造一套 wire contract。

## 1. 产品边界

Mojobox 有三个职责：

1. 维护可审阅的 DSH 组件目录；
2. 用 Pack/Lock 表达并固定可复现的插件组合；
3. 聚合与精确产物、宿主和测试套件绑定的兼容证据。

网站只是这些数据的只读视图。安装、卸载、快照、文件锁和回滚始终属于宿主 Adapter。

## 2. 协议所有权

| 信息 | 事实源 | Mojobox 的处理方式 |
| --- | --- | --- |
| 插件能力、权限、facet、entrypoint | `dsh-std` Manifest | 引用并校验，不重新定义 |
| 插件集合与必需/可选关系 | Mojobox Pack | 本仓库定义 |
| 精确版本、来源与摘要 | Mojobox Pack Lock | 本仓库定义 |
| 测试对象、环境、suite 与结果 | Mojobox Evidence | 本仓库定义 |
| 整套环境身份、布局与迁移 | `dsh-distribution` | 不放入 Pack/Lock |
| 本地安装、启动验证与回滚 | EAC/TUI/其他 Host Adapter | 公共数据不携带私有实现 |

如果一个字段已有明确的上游所有者，应使用引用或 Evidence 绑定，不在 Mojobox Schema 中复制。

## 3. 数据流

### 校验

`scripts/validate.mjs` 读取 `catalog/`、`fixtures/`、`profiles/`、vendored Schema 和
`spec-revisions.json`，执行结构与跨文件一致性检查。它不访问网络，也不执行插件代码。

### 构建

`scripts/build-site.mjs` 从 `catalog/` 重新生成站点数据。对于 Pack 中的组件，它会：

1. 按 Lock 找到对应 Manifest；
2. 验证 Manifest 原始字节 SHA-256；
3. 下载精确 npm tarball 并验证 artifact SHA-256；
4. 按固定时间、稳定顺序写入 `.dshpack`；
5. 生成供静态页面读取的 `catalog.json`。

Vite 只负责把 `site/` 和生成数据输出到 `dist/`。生成目录可以删除并重建，不承载人工修改。

### 宿主消费

EAC 使用同一 catalog 的内置快照和在线缓存生成本地计划。EAC 私有的 profile 路径、snapshot
ID、RPC 和事务 journal 不进入本仓库的公共 Pack 格式。其他宿主可以只消费 Catalog，也可以
实现自己的计划或事务能力。

## 4. 文档类型

### Plugin Manifest

优先使用插件作者发布的 `dsh-plugin.json`。目录代维护记录必须明确标记
`registry-maintained`，并只陈述可核验事实。

### Pack

表达维护者希望提供的组件组合、必需性与宿主要求。Pack 不固定下载地址或 artifact 摘要。

### Pack Lock

记录已经解析并测试的精确版本、来源、Manifest 路径与摘要。一个 Pack 必须有同名 Lock，
两者的 ID、版本和组件集合必须一致。

### Evidence

记录一次有边界的验证。它必须绑定精确 artifact、Manifest、Host Descriptor、测试 suite 和
规范 revision。`evidenceLevel` 描述证据深度，`result` 描述本次结果，两者不能混为一谈。

## 5. 上游 revision 策略

`spec-revisions.json` 是上游坐标的单一入口。上游升级是显式迁移：先比较规范变化，再更新
vendored Schema、profile、fixture 和受影响 Evidence，最后运行完整校验与构建。

历史 Evidence 必须保留它签发时使用的 revision。除非重新运行 suite 并重新签发，否则不能
只替换 revision 字符串。

## 6. 设计约束

- 不执行 catalog 中的插件代码来完成校验。
- 不接受浮动版本或未验证的 artifact 替换。
- 不把目录维护内容表示成插件作者声明。
- 不把一个宿主的私有要求提升为所有宿主的公共要求。
- 不让网站读取用户本机环境或直接执行安装。
- 不把 `.dshpack` 描述成完整环境备份。
- 不在没有真实实现和证据时声明兼容能力。

## 7. 与 EAC 的同步

EAC 内置的 Mojobox 插件和 catalog snapshot 位于 EAC 仓库，不以 Git submodule 方式耦合本仓库。
公共 catalog 发生变化后，当前需要更新 EAC 快照并运行 EAC 的 Mojobox 集成测试；自动
sync/check 命令尚未实现。Mojobox 仓库不能反向 import EAC 私有源码或路径。
