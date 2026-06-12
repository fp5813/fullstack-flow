# Vue 3 前端测试最佳实践

## 快速参考

| 测试类型 | 工具 | 文件命名 |
|---------|------|---------|
| 组件测试 | Vitest + Vue Test Utils | `__tests__/Comp.spec.ts` |
| Composable 测试 | Vitest（直接调用） | `__tests__/useXxx.spec.ts` |
| E2E | Playwright | `e2e/*.spec.ts` |

**运行命令**: `npx vitest run`（CI） / `npx vitest`（watch 模式）

## 技术栈

| 工具 | 用途 | 安装 |
|------|------|------|
| Vitest | 测试运行器（兼容 Vite 配置） | `npm install -D vitest` |
| Vue Test Utils | 组件挂载和交互 | `npm install -D @vue/test-utils` |
| jsdom | DOM 环境模拟 | `npm install -D jsdom` |

## Vitest 配置

```typescript
// vite.config.ts
import { defineConfig } from 'vitest/config';
import vue from '@vitejs/plugin-vue';

export default defineConfig({
  plugins: [vue()],
  test: {
    environment: 'jsdom',
    globals: true,
    setupFiles: './src/test/setup.ts',
  },
});
```

## 组件测试

### 基础挂载

```typescript
import { mount } from '@vue/test-utils';
import { describe, it, expect } from 'vitest';
import OrderCard from './OrderCard.vue';

describe('OrderCard', () => {
  it('should render order title', () => {
    const wrapper = mount(OrderCard, {
      props: { title: 'Test Order', amount: 100 },
    });
    expect(wrapper.text()).toContain('Test Order');
    expect(wrapper.text()).toContain('100');
  });
});
```

### 交互测试

```typescript
it('should emit click when button is pressed', async () => {
  const wrapper = mount(OrderCard, {
    props: { title: 'Test', onDelete: () => {} },
  });
  await wrapper.find('.delete-btn').trigger('click');
  expect(wrapper.emitted('delete')).toBeTruthy();
});
```

### 异步组件测试

```typescript
import { flushPromises } from '@vue/test-utils';

it('should load data on mount', async () => {
  const wrapper = mount(OrderList);
  // 等待所有异步操作完成
  await flushPromises();
  expect(wrapper.findAll('.order-item').length).toBeGreaterThan(0);
});
```

## Composable 测试

```typescript
import { ref } from 'vue';
import { describe, it, expect } from 'vitest';
import { useSearch } from './useSearch';

describe('useSearch', () => {
  it('should return results for query', async () => {
    const query = ref('');
    const { results, search } = useSearch(query);

    query.value = 'test';
    await search();

    expect(results.value.length).toBeGreaterThan(0);
  });

  it('should handle empty query', async () => {
    const { results, search } = useSearch('');
    await search();
    expect(results.value).toEqual([]);
  });
});
```

## 目录结构

```
src/
├── __tests__/              ← 测试文件统一目录（可选）
│   ├── components/
│   │   └── OrderCard.spec.ts
│   └── composables/
│       └── useSearch.spec.ts
└── components/
    └── OrderCard.vue

// 或与组件同目录（推荐）
src/
└── components/
    ├── OrderCard.vue
    └── OrderCard.spec.ts
```

## 命名规范

| 测试类型 | 命名模式 | 示例 |
|---------|---------|------|
| 渲染测试 | `should_xxx_when_yyy` | `should_render_title_when_provided` |
| 交互测试 | `should_emit_xxx_when_yyy` | `should_emit_delete_when_clicked` |
| 异步测试 | `should_xxx_after_yyy` | `should_load_data_after_mount` |
| 异常测试 | `should_handle_xxx` | `should_handle_empty_data` |

## 覆盖率要求

| 层级 | 要求 | 说明 |
|------|------|------|
| Composables | ≥ 80% | 核心业务逻辑 |
| 组件 | ≥ 60% | 主要渲染路径 + 交互 |
| 异常路径 | 100% | 每类异常至少 1 条 |

## Playwright E2E（可选）

```typescript
// e2e/login.spec.ts
import { test, expect } from '@playwright/test';

test('should login successfully', async ({ page }) => {
  await page.goto('/login');
  await page.fill('[data-testid="username"]', 'admin');
  await page.fill('[data-testid="password"]', 'password');
  await page.click('[data-testid="login-btn"]');
  await expect(page).toHaveURL('/dashboard');
});
```

E2E 测试适用于关键用户流程（登录、下单、支付），不需要全覆盖。
