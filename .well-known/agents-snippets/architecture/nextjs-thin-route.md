# Next.js App Router 薄路由层

## 核心原则

根目录 `app/` 是 Next.js App Router 路由层，不是业务代码目录。除 `app/api/**` 外，`app/**` 应保持薄层。

## 路由文件只放什么

- `page.tsx` — 页面入口，只做数据获取和组件组合。
- `layout.tsx` — 布局定义。
- `loading.tsx` — 加载状态。
- `error.tsx` — 错误边界。
- `not-found.tsx` — 404 页面。
- `metadata` 导出 — SEO 元数据。

## 业务代码委托

路由文件中的业务实现必须委托给 `src/` 下的对应模块：

```typescript
// 正确：路由文件只做委托
import { SpecEditorPage } from "@/pages/spec-editor";

export default function Page() {
  return <SpecEditorPage />;
}

// 禁止：在路由文件中写业务逻辑
export default function Page() {
  const [data, setData] = useState(null);
  // ... 大量业务逻辑
}
```

## 后端入口

- `app/api/**` 是后端入口，API 端点及服务端处理逻辑在此。
- 如果项目有独立的后端服务目录（如 `src/shared/lib/actions/`），API 路由只做请求转发和响应格式化。

## 禁止事项

- 禁止在 `app/`（不含 `app/api/`）中直接写业务组件、状态管理或数据转换逻辑。
- 禁止在 `app/` 中创建 `components/`、`hooks/`、`utils/` 等子目录；这些属于 `src/`。
- 禁止在路由文件中直接访问数据库或外部服务；委托给 `src/` 的对应模块。
