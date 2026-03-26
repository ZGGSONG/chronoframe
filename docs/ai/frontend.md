# 前端开发规范

## 项目结构

```
app/
  components/         # Vue 组件（自动导入）
  composables/        # 组合式函数
  pages/              # 路由页面（文件路由）
  stores/             # Pinia 状态管理
  assets/             # 静态资源
  plugins/            # Nuxt 插件
```

## 组件命名规则

自动导入规则：

| 文件路径 | 组件名 |
|---------|-------|
| `app/components/photo/Viewer.vue` | `<PhotoViewer />` |
| `app/components/ui/Button.vue` | `<Button />` |

- UI 组件（`app/components/ui/`）无前缀
- `.client.vue` 后缀表示仅客户端渲染

## 组件使用

无需手动 import，Nuxt 自动导入：

```vue
<template>
  <!-- 自动可用 -->
  <PhotoViewer :photo="photo" />
  <Button variant="primary">提交</Button>
</template>
```

## 关键目录

- `app/components/photo/` — 照片展示组件
- `app/components/masonry/` — 瀑布流布局组件
- `app/components/map/` — 地图相关组件
- `app/composables/` — 可复用逻辑（useXxx）

## 提交前检查

```bash
pnpm lint && pnpm build
```

必须全部通过。
