# Vue 3 Reactivity 最佳实践

## 核心原则

**最小源状态，派生数据用 computed，副作用用 watch。**

```
ref / reactive（源状态）→ computed（派生数据）→ watch / watchEffect（副作用）
```

## 选择指南

### ref vs reactive

| 场景 | 推荐 | 原因 |
|------|------|------|
| string / number / boolean | `ref` | 自动 `.value` 解包 |
| 对象 / 数组 | `ref` | 解构 `reactive` 会丢失响应式 |
| 表单对象 | `ref({...})` | 整体替换时不会丢失响应式 |
| Map / Set | `reactive(new Map())` | `ref` 对 Map/Set 支持不完整 |

### computed vs watch

| 场景 | 推荐 |
|------|------|
| 从已有数据派生新值 | `computed` |
| 编排副作用（API 调用、日志） | `watch` |
| 多个源变化后触发一次性操作 | `watch([a, b], ...)` |
| 需要访问旧值 | `watch` |

## 常见陷阱

### 1. reactive 解构丢失响应式

```typescript
// ❌ 错误
const state = reactive({ count: 0, name: 'foo' });
const { count, name } = state;  // count 和 name 失去响应式

// ✅ 正确：保持原始引用
const count = computed(() => state.count);

// ✅ 或使用 toRefs
const { count, name } = toRefs(state);
```

### 2. ref 在 reactive 中自动解包

```typescript
const count = ref(0);
const state = reactive({ count });
// state.count 是 number 类型（自动解包），不需要 .value

// 注意：只有 ref 嵌套在 reactive 中会自动解包
// 嵌套在普通对象中不会
```

### 3. shallowRef 性能优化

```typescript
// 大型只读列表 → shallowRef 避免深层代理
const bigList = shallowRef<Item[]>([]);

// 更新时需要 triggerRef 通知变化
bigList.value = newItems;
triggerRef(bigList);
```

### 4. watch 立即执行

```typescript
// 需要立即执行一次 + 后续变化时触发
watch(
  () => props.id,
  (newId) => fetchData(newId),
  { immediate: true }
);
```

### 5. 模板中避免复杂计算

```typescript
// ❌ 模板中的复杂计算——每次渲染都执行
// {{ items.filter(i => i.active).map(i => i.name).join(', ') }}

// ✅ 提取到 computed
const activeNames = computed(() =>
  items.value.filter(i => i.active).map(i => i.name).join(', ')
);
```
