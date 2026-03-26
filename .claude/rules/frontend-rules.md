---
paths:
  - "app/**"
---

# Frontend 规则

## 组件命名

自动导入规则：
- `app/components/photo/Viewer.vue` → `<PhotoViewer />`
- `app/components/ui/Button.vue` → `<Button />`（UI 组件无前缀）
- `.client.vue` 后缀表示仅客户端渲染

## 目录结构

- `app/components/` — Vue 组件
- `app/composables/` — 组合式函数
- `app/pages/` — 路由页面（文件路由）
- `app/stores/` — Pinia 状态管理

## 组件引用

```vue
<!-- 无需手动 import，Nuxt 自动导入 -->
<PhotoViewer :photo="photo" />
<Button variant="primary">提交</Button>
```
