# Vue 3 Composables 最佳实践

## 快速参考

| 规范 | 说明 |
|------|------|
| 命名 | `use{Feature}.ts` |
| 输入参数 | 支持 `MaybeRef<T>` / `MaybeRefOrGetter<T>` |
| 返回 | 按字母序，`ref` + 函数 |
| 副作用 | 在 composable 内统一处理 |
| 外部只读 | 返回 `readonly(data)` 而非裸 `data` |

## 定义

Composable 是一个利用 Vue Composition API 封装**有状态逻辑**的函数。

```
适合提取为 composable：
- 多个组件共享的逻辑
- 有状态的逻辑（loading / error / data）
- 有副作用的逻辑（API 调用、事件监听、定时器）
- 复杂的业务逻辑

不适合：
- 纯工具函数（放在 utils/ 即可）
- 仅单个组件使用的简单逻辑（内联即可）
```

## 命名规范

```typescript
// composables/useOrder.ts
// composables/useAuth.ts
// composables/usePagination.ts
```

- 文件名：`use{Feature}.ts`
- 函数名：`use{Feature}`
- 返回对象按**字母序**排列

## 输入模式

### 支持 MaybeRef 输入

使用 `MaybeRef<T>` 或 `MaybeRefOrGetter<T>` 作为输入参数，让 composable 同时支持响应式和非响应式输入：

```typescript
import { ref, toValue, type MaybeRef } from 'vue';
import type { Ref } from 'vue';

export function useSearch(query: MaybeRef<string>) {
  // 在计算/watch 内部使用 toValue 解包
  const results = ref<string[]>([]);

  watchEffect(() => {
    // toValue 自动解包 ref / getter / 普通值
    const q = toValue(query);
    if (q) {
      results.value = performSearch(q);
    }
  });

  return { results: readonly(results) };
}
```

### 使用方式

```typescript
// 传入普通值
const { results } = useSearch('initial');

// 传入 ref（响应式）
const searchText = ref('');
const { results } = useSearch(searchText);
```

## 返回模式

```typescript
export function useCounter(initial: MaybeRef<number> = 0) {
  const count = ref(toValue(initial));

  function increment() { count.value++; }
  function reset() { count.value = toValue(initial); }

  return {
    count: readonly(count),  // 外部只读（通过函数修改）
    increment,
    reset,
  };
}
```

- 状态用 `readonly` 暴露，通过返回的函数修改
- 防止外部直接修改内部状态

## 组合 composable

```typescript
export function useOrderList() {
  const { orders, loading, fetch } = useFetch<Order[]>('/api/orders');
  const { page, pageSize, pagination } = usePagination(1, 20);

  // 组合多个 composable 的返回
  async function fetchPage() {
    await fetch({ page: page.value, pageSize: pageSize.value });
  }

  // 监听分页变化重新请求
  watch([page, pageSize], fetchPage);

  return {
    orders: readonly(orders),
    loading: readonly(loading),
    page,
    pageSize,
    pagination,
    fetchPage,
  };
}
```

## 生命周期管理

### 自动清理副作用

```typescript
import { onUnmounted } from 'vue';

export function useInterval(fn: () => void, ms: MaybeRef<number>) {
  const timer = ref<ReturnType<typeof setInterval> | null>(null);

  watchEffect(() => {
    clearInterval(timer.value ?? undefined);
    timer.value = setInterval(fn, toValue(ms));
  });

  // 组件卸载时自动清理
  onUnmounted(() => {
    if (timer.value) clearInterval(timer.value);
  });
}
```

## 检查清单

- [ ] 函数名 `use{Feature}` 格式
- [ ] 输入参数使用 `MaybeRef<T>` / `MaybeRefOrGetter<T>`
- [ ] 内部使用 `toValue()` 解包输入
- [ ] 返回状态用 `readonly` 包装
- [ ] 返回按字母序排列
- [ ] 组件卸载时清理副作用（`onUnmounted` / `watchEffect` 自动清理）
