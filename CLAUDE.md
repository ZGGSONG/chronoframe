# CLAUDE.md

> AI 开发文档见 [docs/ai/](docs/ai/)

## 快速命令

```bash
pnpm dev              # 开发服务器
pnpm build            # 生产构建（修改后必须执行）
pnpm lint             # ESLint（提交前必须执行）
pnpm db:generate      # 生成迁移文件
```

## AI 文档导航

| 文档 | 说明 |
|-----|------|
| [docs/ai/index.md](docs/ai/index.md) | 总览 |
| [docs/ai/backend.md](docs/ai/backend.md) | 后端开发规范 |
| [docs/ai/frontend.md](docs/ai/frontend.md) | 前端开发规范 |
| [docs/ai/workflow.md](docs/ai/workflow.md) | 开发流程规则 |

## 项目结构

```
app/                    # 前端（Vue/Nuxt）
  components/           # 组件（自动导入）
  pages/                # 路由页面
server/
  api/                  # API 路由（自动导入）
  database/schema.ts    # Drizzle schema
  services/             # 业务逻辑
packages/webgl-image/   # WebGL 包
shared/types/           # 共享类型
```

## 禁止修改

- `packages/webgl-image/dist/`
- `data/`
- `pnpm-lock.yaml`
