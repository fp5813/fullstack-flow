---
name: fullstack-flow/vue-standards
description: "Vue 3 前端开发规范（fullstack-flow 子技能）— Composition API + `<script setup lang=\"ts\">`，涵盖组件拆分、Reactivity、数据流、Composables、测试和审查清单。Agent FE 在 Phase 5/5.5 时必需加载。"
user-invocable: false
---

# Vue 3 前端开发规范

> 本规范基于 vuejs-ai/skills 社区最佳实践，适用于 Vue 3 + Composition API + TypeScript + `<script setup lang="ts">` 项目。
> 具体项目的 UI 框架（Element Plus / Ant Design Vue 等）和 API 风格（defHttp / axios 等）从 `CODEBUDDY.md` 获取。

---

## 1. 核心架构约定

### 1.1 默认技术栈

| 技术 | 默认选型 |
|------|---------|
| 框架 | Vue 3 + Composition API |
| 脚本 | `<script setup lang="ts">` |
| TypeScript | 严格模式（`strict: true`） |
| 路由 | Vue Router 4 |
| 状态管理 | Pinia（可选，仅跨组件共享状态时） |
| 构建工具 | Vite |
| CSS | `<style scoped>` |

> 具体 UI 库/请求库等查看 `CODEBUDDY.md` 或项目现有代码。

### 1.2 组件边界规划（Phase 4 阶段完成）

进入 Phase 5 编码前，先确认组件拆分方案：

**路由级视图组件** — 保持轻薄，仅做：
- 页面布局 / app shell
- Provider 装配（`provide` / `Pinia` store 注入）
- 特征组件的组合编排

**特征组件拆分客观条件**（满足任一条即应拆分）：

| 条件 | 做法 |
|------|------|
| 组件有 3+ 个独立 UI 区域（表单/筛选/列表/状态） | 提取 UI 区域为子组件 |
| 模板块可复用（行/卡片/列表项） | 提取为独立的子组件 |
| 同时负责编排/状态 + 大量展示标记 | 提取逻辑到 composable |
| 单个 SFC 超过 300 行 | 拆分组件或提取 composable |

---

## 2. 核心编码规范

### 2.1 响应式（Reactivity）

**源状态 → 派生 → 副作用：**

```
ref / reactive（源状态）→ computed（派生数据）→ watch / watchEffect（副作用）
```

| 场景 | 推荐 | 避免 |
|------|------|------|
| 基础类型（string/number/boolean） | `ref` | `reactive` |
| 对象/数组 | `ref`（自动解包 `.value`） | `reactive`（解包行为不一致） |
| 派生数据 | `computed` | `watch` + 手动变量 |
| 大型只读列表 | `shallowRef` + `triggerRef` | `ref`（深层代理开销） |
| 模板内复杂计算 | 提取到 `computed` | 模板内 `{{ items.filter(...) }}` |

**常见陷阱：**

```typescript
// ❌ 错误：解构 reactive 会丢失响应式
const { name, age } = reactive({ name: 'foo', age: 18 });

// ✅ 正确：使用 ref 或 toRefs
const person = ref({ name: 'foo', age: 18 });
const { name, age } = toRefs(person.value);

// ✅ 正确：保持 ref 的 .value 访问
const count = ref(0);
const double = computed(() => count.value * 2);
```

### 2.2 SFC 结构与模板

**文件结构顺序：**

```vue
<script setup lang="ts">
// 导入 → 状态 → computed → 函数 → watch → 生命周期
</script>

<template>
  <!-- 声明式模板，逻辑放在 script 中 -->
</template>

<style scoped>
/* 组件样式，scoped 隔离 */
</style>
```

**模板规则：**

| 规则 | 说明 |
|------|------|
| `v-for` + `:key` | `:key="item.id"`（使用唯一 id，不推荐 index） |
| `v-if` 和 `v-for` 分开 | `v-if` 放在外层 `<template>`，`v-for` 在内层 |
| `v-html` 慎重 | 内容必须已转义，防止 XSS |
| 分支/推导提到 script | 模板仅做渲染，逻辑放 script |
| 事件处理器 | `@click="handler"` 不写内联复杂表达式 |

### 2.3 组件数据流

**主模式：Props down / Events up**

```typescript
// 子组件 — 显式类型声明
const props = defineProps<{
  items: Item[];
  loading?: boolean;
}>();

const emit = defineEmits<{
  (e: 'select', id: string): void;
  (e: 'delete', id: string): void;
}>();
```

| 模式 | 使用场景 |
|------|---------|
| `defineProps` / `defineEmits` | 默认 |
| `v-model` | 真正的双向绑定组件（输入框、选择器等） |
| `provide` / `inject` | 深层依赖传递（主题、配置等），配合 `InjectionKey` |
| 全局状态（Pinia） | 跨特征/跨页面的共享状态 |

### 2.4 Composables

**命名与结构：**

```typescript
// composables/useFeature.ts

import { ref, computed, type Ref } from 'vue';

export function useFeature(input: MaybeRef<string>) {
  const data = ref<string[]>([]);
  const loading = ref(false);

  async function fetch() {
    loading.value = true;
    try {
      data.value = await api.fetchData(toValue(input));
    } finally {
      loading.value = false;
    }
  }

  return {
    data: readonly(data),  // 外部只读
    loading: readonly(loading),
    fetch,
  };
}
```

| 规范 | 说明 |
|------|------|
| 文件名 | `use{Feature}.ts` |
| 输入参数 | 支持 `MaybeRef<T>` / `MaybeRefOrGetter<T>` 增强灵活性 |
| 返回 | 按字母序，`ref` + 函数 |
| 副作用 | 在 composable 内统一处理（loading / error） |
| 纯逻辑 | 不依赖 Vue 组件实例 |

> `toValue()` 是 Vue 3.3+ 工具函数，自动解包 `Ref` / `Getter` / 普通值。

### 2.5 API 调用模式

```typescript
// api/order.ts
import { defHttp } from '@/utils/http';

export function fetchOrderList(params: OrderQuery) {
  return defHttp.post<PageResult<OrderVO>>({ url: '/api/auth/orders/page', params });
}

export function fetchOrderDetail(orderNo: string) {
  return defHttp.get<OrderVO>({ url: '/api/auth/orders/detail', params: { orderNo } });
}
```

- 统一通过 `api/` 目录下的封装函数调用后端接口
- 错误处理在 composable 或 hook 层统一处理，而非在每个组件中 try-catch
- 请求函数返回类型必须显式声明

---

## 3. 性能优化（行为正确后引入）

> **原则**：性能优化是后置步骤。核心行为正确且验证通过后再进行。

| 场景 | 优化手段 |
|------|---------|
| 大列表渲染 | 虚拟滚动（`vue-virtual-scroller` 或自定义） |
| 静态子树 | `v-once` / `v-memo` 指令 |
| 热路径组件抽象 | 内联展开（避免不必要的组件包装） |
| 更新触发太频繁 | 使用 `updated` hook 检查或 `shallowRef` |

---

## 4. 前端测试规范

### 4.1 技术栈

| 工具 | 用途 |
|------|------|
| Vitest | 测试运行器（兼容 Vite 配置） |
| Vue Test Utils | 组件挂载和交互 |
| @vue/test-utils | 官方测试工具库 |
| jsdom | DOM 环境模拟 |

### 4.2 组件测试

```typescript
import { mount } from '@vue/test-utils';
import { describe, it, expect } from 'vitest';
import OrderList from './OrderList.vue';

describe('OrderList', () => {
  it('should render items', () => {
    const wrapper = mount(OrderList, {
      props: { items: [{ id: '1', name: 'Test' }] },
    });
    expect(wrapper.text()).toContain('Test');
  });

  it('should emit select when clicked', async () => {
    const wrapper = mount(OrderList, {
      props: { items: [{ id: '1', name: 'Test' }] },
    });
    await wrapper.find('.item').trigger('click');
    expect(wrapper.emitted('select')?.[0]).toEqual(['1']);
  });
});
```

### 4.3 Composable 测试

```typescript
import { renderComposable } from '@vue/test-utils';
import { useFeature } from './useFeature';

describe('useFeature', () => {
  it('should fetch data', async () => {
    const { data, fetch } = useFeature('test');
    await fetch();
    expect(data.value).toHaveLength(3);
  });
});
```

### 4.4 命名规范

| 测试类型 | 命名模式 | 示例 |
|---------|---------|------|
| 组件测试 | `should_xxx_when_yyy` | `should_render_items_when_provided` |
| Composable 测试 | `should_xxx_with_yyy` | `should_fetch_data_with_valid_input` |
| 交互测试 | `should_emit_xxx_when_yyy` | `should_emit_select_when_clicked` |

---

## 5. 前端代码审查清单

### 5.1 组件设计
- [ ] 组件拆分合理，无巨型组件（> 300 行应考虑拆分）
- [ ] 路由级视图组件保持轻薄
- [ ] 无重复模板块（应提取为子组件）

### 5.2 Reactivity
- [ ] 最小源状态（无多余 `ref`）
- [ ] 派生数据使用 `computed`，非手动变量
- [ ] 无 `reactive` 解构丢失响应式

### 5.3 数据流
- [ ] Props/Emits 类型完整显式声明
- [ ] `v-model` 仅用于真正的双向组件
- [ ] `provide/inject` 配合 `InjectionKey`
- [ ] API 错误处理完整

### 5.4 模板
- [ ] `v-for` 有 `:key`（唯一 id）
- [ ] 无 `v-if`+`v-for` 混用
- [ ] 模板无线内计算

### 5.5 Composables & 类型
- [ ] composable 命名 `use{Feature}`，输入支持 `MaybeRef`
- [ ] composable 返回按字母序
- [ ] TypeScript 类型完整（无 `any` 逃避）
- [ ] `api/` 目录下请求函数类型显式声明

### 5.6 性能与文档
- [ ] 性能优化仅在核心逻辑验证后引入
- [ ] `@Operation` / JSDoc 注释完整（新增 API 必加）
- [ ] `@AsyncLog` 操作日志完整（写操作必加）


> **使用方式**：Phase 5 时 Agent FE 加载本规范 + 3 个必读 reference（`vue-reactivity.md`、`vue-component-data-flow.md`、`vue-composables.md`）生成代码。
> Phase 5.5 时 Agent FE 按第 5 章审查清单逐一审核。`vue-testing.md` 按需加载。
> 项目特定 UI 库/请求库等查看 `CODEBUDDY.md`。
