# cloudflare_temp_email_worker

这是一个基于 [cloudflare_temp_email](https://github.com/dreamhunter2333/cloudflare_temp_email) 后端进行魔改后的 `worker.js` 包。

当前由 LeonMinGod 维护：

- 完整源码与仙侠风前端：<https://github.com/928886540/cloudflare_temp_email>
- 独立 Worker 部署包：<https://github.com/928886540/cloudflare_temp_email_worker>

本仓库的 `worker.js` 与完整源码仓库的 `worker/dist_worker.js` 保持一致，Telegram 邮件通知时间显示为北京时间（GMT+8）。

当前部署包由 v1.13.0 源码构建，包含数据库容量与管理配置接口。完整源码仓库执行 `cd worker && pnpm build` 后可同步生成的 `dist_worker.js` 到本仓库，避免前后端版本不一致导致 404。

## 这是什么

这个仓库不包含完整项目源码，也不是完整部署包。
它的用途很简单：

- 先按原项目教程部署好 `cloudflare_temp_email`
- 保留你原来的 D1、KV、环境变量、前端等配置
- 然后把原项目后端 Worker 替换成这里提供的魔改版 `worker.js`

## 适用场景

适合已经部署或准备部署原版 `cloudflare_temp_email`，但希望后端支持：

- 根域配置
- 启用配置后接受该根域下的泛子域邮箱

例如当你配置根域：

```json
["example.com"]
```

后端将支持：

- `example.com`
- `a.example.com`
- `b.example.com`
- `x.y.example.com`

注意：这里的意思是**后端域名校验支持泛子域**。

## 使用方法

### 第一步：先部署原项目

请先参考原项目文档完成完整部署：

- GitHub 项目：<https://github.com/dreamhunter2333/cloudflare_temp_email>
- 原项目文档：<https://temp-mail-docs.awsl.uk>

你需要先把这些内容部署好：

- 前端
- Cloudflare Worker
- D1
- KV
- 环境变量
- 其它你自己的 Cloudflare 配置

### 第二步：确认环境变量

这个魔改版：
- 不要写 `*.` 通配符格式

正确示例：

```json
["12313123.eu.cc"]
```

不建议这样写：

```json
["*.12313123.eu.cc"]
```

升级旧版部署包时，请在 Worker 环境变量中设置 `ENABLE_CREATE_ADDRESS_SUBDOMAIN_MATCH = true`，保留根域及其子域的创建能力。需要前端支持随机或自定义子域时，增加 `RANDOM_SUBDOMAIN_DOMAINS`，值为允许使用此功能的根域数组。

Wrangler 配置示例：

```toml
[vars]
DOMAINS = ["example.com"]
ENABLE_CREATE_ADDRESS_SUBDOMAIN_MATCH = true
RANDOM_SUBDOMAIN_DOMAINS = ["example.com"]
```

### 第三步：替换 Worker 后端代码

在你原本已经部署好的 `cloudflare_temp_email` Worker 中：

1. 打开 Cloudflare Workers 控制台
2. 找到原项目对应的 Worker
3. 进入代码编辑界面
4. 用本仓库中的 `worker.js` 替换原有 Worker 代码
5. 保存并重新部署

## 说明

- 这是对原项目后端 Worker 的魔改版本
- 部署包从完整源码生成，泛子域行为通过环境变量配置
- 前端默认界面如果还是只显示根域，这是正常的；该魔改主要是后端支持泛子域

## 已知说明

- 升级前请备份 D1；已有表但没有版本标记时，在新版后台执行数据库迁移即可补齐结构和版本记录，已有邮件和账号会保留
- 容量套餐选项只用于显示使用率，不会创建数据库、增加免费额度或开通付费服务
- 前端如果没有额外修改，默认仍可能只允许通过界面选择根域
- 如需测试泛子域，可直接调用后端创建地址接口并传入子域
- Cloudflare 路由是否能真正把子域邮件投递进来，不属于这个 `worker.js` 本身负责的范围

## 致谢

本魔改基于原项目：

- <https://github.com/dreamhunter2333/cloudflare_temp_email>

## Upgrade notes (English)

This bundle is built from the maintained v1.13.0 source and includes the admin config and database capacity APIs. Run `pnpm build` in the full repository's `worker` directory and copy `dist_worker.js` here as `worker.js` to keep them synchronized.

Set `ENABLE_CREATE_ADDRESS_SUBDOMAIN_MATCH = true` when upgrading from the legacy wildcard bundle. Configure `RANDOM_SUBDOMAIN_DOMAINS` with your root domains to enable random/custom subdomain controls. Cloudflare Email Routing must still be configured separately.

Back up D1 before upgrading. Existing databases without a version marker can be migrated from the admin page; migration adds missing schema objects and preserves mail and accounts. The capacity plan selector only changes the displayed limit; it does not create a database or purchase a plan.
