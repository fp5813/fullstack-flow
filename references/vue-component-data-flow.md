# Vue 3 组件数据流最佳实践

## 核心模型

```
Props down（父 → 子）
Events up（子 → 父）
v-model（双向，用于输入类组件）
provide/inject（深层依赖，不常用）
Pinia（跨特征共享状态，按需）
```

## Props

### 显式类型声明（必须）

```typescript
// ✅ 使用泛型参数声明类型
const props = defineProps<{
  items: Item[];
  loading?: boolean;
  title: string;
}>();

// ✅ 使用 withDefaults 提供默认值
const props = withDefaults(defineProps<{
  items?: Item[];
  title: string;
}>(), {
  items: () => [],
});

// ❌ 避免运行时声明（不支持泛型）
defineProps({
  items: Array,
  title: String,
});
```

### Props 命名规范

- camelCase 命名
- Boolean props 默认 false，不需要默认值
- 可选 props 使用 `?` 标记

## Emits

### 显式类型声明（必须）

```typescript
const emit = defineEmits<{
  (e: 'select', id: string): void;
  (e: 'update:modelValue', value: string): void;
  (e: 'delete', id: string): void;
}>();
```

- 事件名用 kebab-case（模板中）或 camelCase（代码中）
- 每个事件标注参数类型

## v-model

### 自定义组件 v-model

```typescript
// 子组件
const props = defineProps<{ modelValue: string }>();
const emit = defineEmits<{ (e: 'update:modelValue', value: string): void }>();

function onInput(e: Event) {
  emit('update:modelValue', (e.target as HTMLInputElement).value);
}
```

```vue
<!-- 父组件使用 -->
<MyInput v-model="searchText" />
```

### 多个 v-model（Vue 3+）

```typescript
const props = defineProps<{ search: string; page: number }>();
const emit = defineEmits<{
  (e: 'update:search', value: string): void;
  (e: 'update:page', value: number): void;
}>();
```

```vue
<MyFilter v-model:search="search" v-model:page="page" />
```

## provide / inject

### 使用时机

- 主题、语言、用户信息等**全局上下文**
- 组件树深度 > 3 层时需要透传的数据
- **不适合**：频繁变化的数据（改用 Pinia 或 props）

### 类型安全

```typescript
// keys.ts
import type { InjectionKey, Ref } from 'vue';

export const userKey: InjectionKey<Ref<User | null>> = Symbol('user');

// 父组件 provide
import { userKey } from './keys';
provide(userKey, ref(currentUser));

// 子组件 inject
import { userKey } from './keys';
const user = inject(userKey);  // 类型：Ref<User | null> | undefined
```

## 数据流决策树

```
数据需要跨组件传递？
├── 父子关系 → Props down / Events up
├── 深度树（>3层）→ provide / inject
├── 跨页面/跨特征 → Pinia store
└── 输入类组件双向绑定 → v-model
```
