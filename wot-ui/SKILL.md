---
name: wot-design-uni-dev
description: wot-design-uni 组件库开发技能 - Vue3+TS+uni-app 跨端组件库开发、BEM 命名规范、CSS 变量主题化、Composables 架构、Props 工厂函数。当涉及 wot-design-uni/wot-design-uni、wd-button/wd-input 等 wd-* 组件开发、主题定制、BEM SCSS 编写时必须使用此技能。
---

# wot-design-uni 组件库开发技能

> 基于 Vue3 + TypeScript + uni-app 的跨端组件库，70+ 组件，MIT 协议。
> 官网：https://wot-ui.cn | GitHub：https://github.com/Moonofweisheng/wot-design-uni

## 快速规则

> **[组件开发清单]**
> ① 每个组件 = `wd-*.vue` + `types.ts` + `index.scss`，三文件缺一不可
> ② Props 必须用 `makeBooleanProp` / `makeStringProp` 等工厂函数，禁止手写 `{ type: Boolean }`
> ③ SCSS 用 BEM 混编：`@include b(block) { @include e(element) { ... } @include m(modifier) { ... } @include when(state) { ... } }`
> ④ 样式穿透用 `@include edeep(element)`，状态类用 `@include when(state)`
> ⑤ 主题变量用 `--wot-*` CSS 变量，`$` 前缀的 SCSS 变量在 `variable.scss` 中定义
> ⑥ 必须开启 `virtualHost: true` 和 `addGlobalClass: true`

---

## 仓库结构

```
src/uni_modules/wot-design-uni/
  components/
    common/
      abstracts/
        _config.scss       # BEM 命名空间配置：$namespace: 'wd'
        _function.scss     # SCSS 辅助函数（containsModifier/containsPseudo/主题色）
        _mixin.scss        # BEM 混编宏：b/e/m/when/edeep/mdeep
        variable.scss      # 所有 CSS 变量定义（80KB+，所有组件的主题 token）
      props.ts             # Props 工厂函数：makeBooleanProp / makeStringProp / ...
      util.ts              # 通用工具函数：isDef / objToStyle / pause / isEqual / ...
      base64.ts            # Base64 编解码
      clickoutside.ts       # 点击外部指令
      event.ts             # 事件总线
      interceptor.ts       # 拦截器
      canvasHelper.ts       # Canvas 辅助
      AbortablePromise.ts    # 可取消 Promise
    composables/
      index.ts             # 统一导出
      useChildren.ts       # 父组件收集子组件实例
      useParent.ts         # 子组件获取父组件实例
      useConfigProvider.ts # 主题配置注入
      useCountDown.ts      # 倒计时
      useLockScroll.ts     # 禁止滚动
      useTouch.ts          # 触摸手势
      usePopover.ts        # 气泡定位
      useTranslate.ts      # 国际化翻译
      useQueue.ts          # 请求队列
      useRaf.ts            # RAF 动画
      useUpload.ts         # 上传封装
      useCell.ts           # Cell 单元格行为
    wd-*/                  # 70+ 业务组件
      wd-*.vue             # 组件主文件
      types.ts             # Props 类型定义
      index.scss           # 组件样式
  package.json             # dev:dev:h5 / dev:mp-weixin 等脚本
```

---

## BEM 命名规范

### 命名空间配置
```scss
// common/abstracts/_config.scss
$namespace: 'wd';           // 块前缀
$elementSeparator: '__';     // 元素分隔符
$modifierSeparator: '--';    // 修饰符分隔符
$state-prefix: 'is-';       // 状态类前缀
```

### 命名对照表

| 类型 | 语法 | 示例 |
|------|------|------|
| 块（Block） | `@include b(button)` | `.wd-button` |
| 元素（Element） | `@include e(content)` | `.wd-button__content` |
| 修饰符（Modifier） | `@include m(primary)` | `.wd-button--primary` |
| 状态（State） | `@include when(disabled)` | `.wd-button.is-disabled` |
| 穿透样式（Deep Element） | `@include edeep(icon)` | `:deep(.wd-button__icon)` |
| 穿透修饰符（Deep Modifier） | `@include mdeep(disabled)` | `:deep(.wd-button--disabled)` |

### SCSS BEM 混编完整示例

```scss
@import './../common/abstracts/_mixin.scss';
@import './../common/abstracts/variable.scss';

@include b(button) {
  // 基础样式
  display: inline-flex;
  align-items: center;
  justify-content: center;

  // 元素
  @include e(content) {
    display: flex;
    align-items: center;
  }

  @include e(loading) {
    width: 20px;
    height: 20px;
  }

  // 修饰符
  @include m(primary) {
    background: $-button-primary-bg-color;
    color: $-button-primary-color;
  }

  @include m(small) {
    height: $-button-small-height;
    font-size: $-button-small-fs;
  }

  // 状态（is-*）
  @include when(disabled) {
    opacity: $-button-disabled-opacity;
    cursor: not-allowed;
  }

  @include when(round) {
    border-radius: 999px;
  }

  // 穿透样式（小程序组件内样式）
  @include edeep(icon) {
    margin-right: 6px;
    font-size: 16px;
  }
}

// 暗黑模式
.wot-theme-dark {
  @include b(button) {
    @include when(primary) {
      background: $-dark-button-primary-bg;
      color: $-dark-button-primary-color;
    }
  }
}
```

---

## Props 工厂函数

```typescript
// components/common/props.ts
import { numericProp } from '../common/props'

// 布尔属性（推荐所有布尔 prop 都用这个）
export const makeBooleanProp = <T>(defaultVal: T) => ({
  type: Boolean,
  default: defaultVal
})

// 字符串属性
export const makeStringProp = <T>(defaultVal: T) => ({
  type: String as unknown as PropType<T>,
  default: defaultVal
})

// 数字属性
export const makeNumberProp = <T>(defaultVal: T) => ({
  type: Number,
  default: defaultVal
})

// 数字或字符串属性（最常用）
export const makeNumericProp = <T>(defaultVal: T) => ({
  type: numericProp, // [Number, String]
  default: defaultVal
})

// 数组属性
export const makeArrayProp = <T = any>() => ({
  type: Array as PropType<T[]>,
  default: () => []
})

// 必填属性
export const makeRequiredProp = <T>(type: T) => ({
  type,
  required: true as const
})

// 基础属性（每个组件都要 spread）
export const baseProps = {
  customStyle: makeStringProp(''),
  customClass: makeStringProp('')
}
```

### types.ts 完整示例

```typescript
// components/wd-example/types.ts
import type { ExtractPropTypes, PropType } from 'vue'
import { baseProps, makeBooleanProp, makeStringProp, makeNumberProp } from '../common/props'

// 类型别名
export type ExampleType = 'primary' | 'success' | 'info' | 'warning' | 'error'
export type ExampleSize = 'small' | 'medium' | 'large'

export const exampleProps = {
  ...baseProps,

  // 布尔：默认为 false
  disabled: makeBooleanProp(false),
  loading: makeBooleanProp(false),
  round: makeBooleanProp(true),
  block: makeBooleanProp(false),

  // 字符串：有默认值
  type: makeStringProp('primary'),
  size: makeStringProp('medium'),

  // 数字
  max: makeNumberProp(100),

  // 可选值（联合类型）
  color: String as PropType<string>,

  // slot 前的属性定义顺序：基础属性 > bool > string > number > 联合类型
}

export type ExampleProps = ExtractPropTypes<typeof exampleProps>
```

---

## 组件主文件模板

```vue
<template>
  <!-- 根节点：staticClass + customClass，style + customStyle -->
  <view
    :class="[`wd-example', customClass]"
    :style="customStyle"
  >
    <view :class="rootClass">
      <!-- 内容 -->
      <view :class="[`wd-example__content', contentClass]">
        <slot />
      </view>

      <!-- 加载状态 -->
      <view v-if="loading" :class="`wd-example__loading`" />

      <!-- 插槽 -->
      <slot name="extra" />
    </view>
  </view>
</template>

<script setup lang="ts">
import { computed, ref, watch } from 'vue'
import { exampleProps } from './types'

const props = defineProps(exampleProps)
const emit = defineEmits<{
  (e: 'click', event: any): void
  (e: 'update:modelValue', value: any): void
  (e: 'change', value: any): void
}>()

// 响应式状态
const internalValue = ref(props.modelValue)

// 计算属性
const rootClass = computed(() => {
  return [
    'wd-example',
    `wd-example--${props.type}`,
    `wd-example--${props.size}`,
    props.disabled ? 'is-disabled' : '',
    props.loading ? 'is-loading' : '',
    props.round ? 'is-round' : '',
  ].filter(Boolean).join(' ')
})

// Watch
watch(
  () => props.modelValue,
  (val) => {
    internalValue.value = val
  }
)

// 方法
function handleClick(event: any) {
  if (!props.disabled && !props.loading) {
    emit('click', event)
  }
}
</script>

<style lang="scss" scoped>
@import './index.scss';
</style>
```

### options 配置（必须）

```javascript
export default {
  name: 'wd-example',
  options: {
    virtualHost: true,       // 启用虚拟主机，组件根节点变成真实 div
    addGlobalClass: true,    // 允许穿透到组件内部的全局类名
    styleIsolation: 'shared' // 样式与其他组件共享
  }
}
```

---

## Theming 主题系统

### CSS 变量体系

所有主题变量以 `--wot-` 为前缀，定义在 `variable.scss`（80KB+）。驼峰转横线：
- `buttonPrimaryBgColor` -> `--wot-button-primary-bg-color`
- `colorSuccess` -> `--wot-color-success`

### 通过 wd-config-provider 注入主题

```vue
<wd-config-provider :theme="theme" :theme-vars="themeVars">
  <App />
</wd-config-provider>
```

```typescript
import type { ConfigProviderThemeVars } from 'wot-design-uni'

const themeVars: ConfigProviderThemeVars = {
  // 按钮主题
  buttonPrimaryBgColor: '#07c160',
  buttonPrimaryColor: '#ffffff',
  buttonSmallHeight: '72rpx',
  buttonMediumHeight: '88rpx',
  buttonLargeHeight: '96rpx',

  // 颜色
  colorSuccess: '#07c160',
  colorWarning: '#ff976a',
  colorError: '#ed6a0c',

  // 输入框
  inputHeight: '88rpx',
  inputBg: '#f5f5f5',
}

const theme = ref<'light' | 'dark'>('light')
```

### 通过 CSS 直接覆盖

```css
/* H5: 在 :root 上覆盖 */
:root,
page {
  --wot-button-primary-bg-color: #07c160;
  --wot-color-success: #07c160;
}

/* 小程序: 在 page 节点上覆盖 */
page {
  --wot-button-primary-bg-color: #07c160;
}
```

### 暗黑模式

```vue
<!-- 全局暗黑 -->
<wd-config-provider theme="dark">
  <App />
</wd-config-provider>

<!-- 动态切换 -->
<wd-config-provider :theme="theme">
  <App />
</wd-config-provider>
```

暗黑模式需要在 `index.scss` 中配合写：

```scss
.wot-theme-dark {
  @include b(button) {
    @include when(primary) {
      background: $-dark-button-primary-bg;
      color: $-dark-button-primary-color;
    }
  }
}
```

### 通过 useConfigProvider 注入主题（JS 方式）

```typescript
import { useConfigProvider } from 'wot-design-uni'

// 在组件 setup 中调用，传递主题变量给插槽内的深层组件
useConfigProvider({ themeVars: { buttonPrimaryBgColor: '#07c160' } })
```

---

## Composables 合集

### useChildren / useParent（父子通信）

父组件通过 `useChildren` 收集子组件实例，子组件通过 `useParent` 获取父组件。

```typescript
// wd-tabs/types.ts - 定义 InjectionKey
import type { InjectionKey } from 'vue'
export const TABS_KEY: InjectionKey<TabsProvide> = Symbol('tabs')

// wd-tabs/wd-tabs.vue - 父组件
import { useChildren } from '../composables/useChildren'

const { children, linkChildren } = useChildren(TABS_KEY)

const provideValue = {
  currentIndex: currentIndex.value,
  scrollx: scrollx.value,
  scroll: (index: number) => handleScroll(index)
}
linkChildren(provideValue)

// wd-tab-panel/wd-tab-panel.vue - 子组件
import { useParent } from '../composables/useParent'
import { TABS_KEY } from '../wd-tabs/types'

const { parent } = useParent(TABS_KEY)

// 通过 parent.xxx 访问父组件的属性和方法
console.log(parent?.currentIndex)
```

### useCountDown（倒计时）

```typescript
import { useCountDown } from '../composables/useCountDown'

const { start, pause, reset, currentRemainTime } = useCountDown({
  time: 60 * 1000,
  onChange: ({ remainingTime }) => {
    console.log('剩余:', remainingTime)
  },
  onFinish: () => {
    console.log('倒计时结束')
  }
})

start()
```

### useLockScroll（禁止滚动）

```typescript
import { useLockScroll } from '../composables/useLockScroll'

const { lock, unlock } = useLockScroll()
// lock() 禁止滚动
// unlock() 恢复滚动
```

### useTouch（触摸手势）

```typescript
import { useTouch } from '../composables/useTouch'

const touch = useTouch()
```

### useTranslate（国际化）

```typescript
import { useTranslate } from '../composables/useTranslate'

const { t } = useTranslate('button')
// 使用: t('close') 或 t('confirm')
```

### useQueue（请求队列）

```typescript
import { useQueue } from '../composables/useQueue'

const queue = useQueue()
// queue.add(promise)
// queue.clear()
```

### useRaf（RAF 动画）

```typescript
import { useRaf } from '../composables/useRaf'

const { start, stop, isRunning } = useRaf((timestamp) => {
  // 动画逻辑
})
```

---

## 常用工具函数（util.ts）

```typescript
import { isDef, objToStyle, pause, isEqual, range, getEnv } from '../common/util'

// 判断是否定义（非 null/undefined）
isDef(value) // true/false

// 对象转行内样式字符串
objToStyle({ color: 'red', fontSize: '14px' })
// => 'color: red; font-size: 14px;'

// 异步延时（用于清空操作等）
await pause(150)  // 150ms

// 深度相等比较
isEqual(a, b)

// 数值范围限制
range(5, 0, 10) // => 5
range(-1, 0, 10) // => 0

// 判断平台
getEnv() // 'mp-weixin' | 'h5' | 'app' | ...

// 小数精度计算
add(num1, num2)
subtract(num1, num2)
```

---

## Icon 图标使用

```vue
<!-- 内置图标 -->
<wd-icon name="add-circle" color="#0083ff" size="20px" />

<!-- 带图标的按钮 -->
<wd-button icon="edit-outline">编辑</wd-button>

<!-- 自定义图标（通过 classPrefix） -->
<wd-icon class-prefix="fish" name="kehuishouwu" />
```

自定义图标需要引入 iconfont CSS：

```css
@font-face {
  font-family: "fish";
  src: url('...') format('woff2');
}
.fish { font-family: "fish" !important; }
.fish-kohuishouwu::before { content: "\e627"; }
```

---

## 开发调试命令

```bash
# H5 开发
pnpm dev:h5

# 微信小程序
pnpm dev:mp-weixin

# App
pnpm dev:app

# 类型检查
pnpm type-check

# 测试
pnpm test:h5
pnpm test:mp-weixin

# 构建
pnpm build:h5
pnpm build:mp-weixin
```

---

## 组件选型决策指南

> 当你拿到设计稿或需求描述时，按以下流程快速匹配组件。不确定时优先查官网 https://wot-ui.cn 确认组件能力。

### 决策流程图

```
设计稿分析
    │
    ├─ 表单类 ──────────────────────────────────────────
    │    ├─ 单行文本输入 ──→ wd-input
    │    ├─ 多行文本输入 ──→ wd-textarea
    │    ├─ 数字输入（带+/-按钮）─→ wd-input-number
    │    ├─ 复选框 ──→ wd-checkbox
    │    ├─ 单选框 ──→ wd-radio / wd-radio-group
    │    ├─ 开关 ──→ wd-switch
    │    ├─ 滑动条 ──→ wd-slider
    │    ├─ 选择器（日期）──→ wd-date-picker
    │    ├─ 选择器（地区）──→ wd-area-picker
    │    ├─ 选择器（单项）──→ wd-picker / wd-scroll-view（自定义）
    │    ├─ 评分/星级 ──→ wd-rate
    │    └─ 搜索框 ──→ wd-search
    │
    ├─ 展示类 ──────────────────────────────────────────
    │    ├─ 列表项（带标题+描述+箭头）──→ wd-cell / wd-cell-group
    │    ├─ 图片 ──→ wd-img / wd-lazyload
    │    ├─ 徽标/角标 ──→ wd-badge
    │    ├─ 标签/胶囊 ──→ wd-tag
    │    ├─ 头像 ──→ wd-avatar / wd-avatar-group
    │    ├─ 进度条 ──→ wd-progress
    │    ├─ 步骤条 ──→ wd-steps
    │    ├─ 时间线 ──→ wd-timeline
    │    ├─ 分割线 ──→ wd-divider
    │    ├─ 空状态 ──→ wd-status-tip
    │    └─ 滚动公告 ──→ wd-notice-bar
    │
    ├─ 反馈类 ──────────────────────────────────────────
    │    ├─ 轻提示/Toast ──→ wd-toast
    │    ├─ 对话框/Modal ──→ wd-dialog
    │    ├─ 底部弹出操作表 ──→ wd-action-sheet
    │    ├─ 气泡/工具提示 ──→ wd-popover
    │    ├─ 气泡确认框 ──→ wd-popover（type=confirm）
    │    ├─ 加载状态 ──→ wd-loading
    │    └─ 顶部进度条 ──→ wd-progress（type=linear）
    │
    ├─ 导航类 ──────────────────────────────────────────
    │    ├─ Tab 切换 ──→ wd-tabs + wd-tab
    │    ├─ 标签栏 ──→ wd-tabbar
    │    ├─ 顶部导航栏 ──→ wd-navbar
    │    ├─ 侧边索引栏 ──→ wd-index-anchor + wd-scroll-view
    │    ├─ 面包屑 ──→ wd-goods-category（业务）/ 自定义
    │    └─ 分页器 ──→ wd-pagination
    │
    ├─ 布局类 ──────────────────────────────────────────
    │    ├─ 弹层/抽屉 ──→ wd-drawer
    │    ├─ 弹出层/Modal ──→ wd-popup
    │    ├─ 结果页/成功失败页 ──→ wd-result
    │    ├─ 回到顶部 ──→ wd-back-top
    │    ├─ 吸底/固定底部 ──→ CSS position:fixed + wd-button
    │    └─ 卡片布局 ──→ wd-card + wd-grid
    │
    ├─ 选择类 ──────────────────────────────────────────
    │    ├─ 日历选择 ──→ wd-calendar
    │    ├─ 级联选择 ──→ wd-column-picker
    │    ├─ 上传图片/文件 ──→ wd-upload
    │    └─ 筛选/过滤面板 ──→ wd-filter-view / wd-popup
    │
    └─ 业务组合类 ──────────────────────────────────────
         ├─ 商品卡片 ──→ wd-goods-card
         ├─ 商品属性选择 ──→ wd-goods-route + wd-sku
         └─ 地址选择 ──→ wd-address-list
```

### 常用组件速查表

#### 表单

| 场景 | 组件 | 关键 Props |
|------|------|-----------|
| 普通输入框 | `wd-input` | `v-model`, `type`, `placeholder`, `disabled`, `clearable`, `prefix-icon`, `suffix-icon` |
| 多行文本 | `wd-textarea` | `v-model`, `maxlength`, `autosize` |
| 数字步进器 | `wd-input-number` | `v-model`, `min`, `max`, `step`, `disabled` |
| 复选框 | `wd-checkbox` | `v-model`, `shape`（square/round）, `disabled` |
| 单选组 | `wd-radio-group` + `wd-radio` | `v-model`, `direction`（row/column） |
| 开关 | `wd-switch` | `v-model`, `active-color`, `inactive-color`, `loading` |
| 滑动条 | `wd-slider` | `v-model`, `range`, `step`, `disabled` |
| 日期选择 | `wd-date-picker` | `v-model`, `columns-type`, `min-date`, `max-date` |
| 地区选择 | `wd-area-picker` | `v-model`, `area-data` |
| 搜索框 | `wd-search` | `v-model`, `placeholder`, `hide-cancel` |
| 评分 | `wd-rate` | `v-model`, `count`, `size`, `allow-half` |
| 表单整体 | `wd-form` + `wd-form-item` | `model`, `rules`（配合 `prop` 使用） |

#### 布局与展示

| 场景 | 组件 | 关键 Props |
|------|------|-----------|
| 列表项 | `wd-cell-group` + `wd-cell` | `title`, `label`, `is-link`, `clickable` |
| 卡片 | `wd-card` | `goods`（商品对象）, `no-thumb` |
| 图片 | `wd-img` | `src`, `mode`, `lazy-load`, `round`, `width/height` |
| 头像 | `wd-avatar` | `src`, `size`, `shape`（circle/square）, `icon` |
| 头像组 | `wd-avatar-group` | `urls`, `max-count`, `size` |
| 徽标 | `wd-badge` | `value`, `type`, `dot` |
| 标签 | `wd-tag` | `type`, `plain`, `round`, `size`, `mark` |
| 进度条 | `wd-progress` | `percentage`, `color`, `stroke-width`, `show-text` |
| 步骤条 | `wd-steps` + `wd-step` | `active`, `direction` |
| 空状态 | `wd-status-tip` | `type`（noData/noNetwork/noResult） |
| 分割线 | `wd-divider` | `direction`, `hairline`, `content-position` |

#### 导航

| 场景 | 组件 | 关键 Props |
|------|------|-----------|
| Tab 切换 | `wd-tabs` + `wd-tab` | `v-model`, `sticky`, `swipeable`, `animation` |
| 标签栏 | `wd-tabbar` | `v-model`, `fixed`, `border`, `safe-area-inset-bottom` |
| 顶部导航 | `wd-navbar` | `title`, `left-text`, `right-text`, `fixed`, `safe-area-inset-top` |
| 分页 | `wd-pagination` | `v-model`, `total`, `page-size`, `force-ellipses` |

#### 弹层与反馈

| 场景 | 组件 | 关键 API / Props |
|------|------|----------------|
| 轻提示 | `wd-toast` | `wdToast.show({ message, type, duration })` |
| 对话框 | `wd-dialog` | `wdDialog.show({ title, message, showCancel })` 或组件用法 |
| 操作菜单 | `wd-action-sheet` | `v-model`, `actions`, `cancel-text` |
| 弹出层 | `wd-popup` | `v-model`, `position`, `closeable`, `round` |
| 抽屉 | `wd-drawer` | `v-model`, `side`（left/right）, `with-header` |
| 加载中 | `wd-loading` | `type`, `fullscreen` |

### 快速开发模板

#### 模板 1：表单页面
```vue
<wd-form :model="formData" :rules="rules" ref="formRef">
  <wd-form-item prop="name">
    <wd-input v-model="formData.name" label="名称" placeholder="请输入" required />
  </wd-form-item>
  <wd-form-item prop="type">
    <wd-radio-group v-model="formData.type" direction="column">
      <wd-radio value="1">类型一</wd-radio>
      <wd-radio value="2">类型二</wd-radio>
    </wd-radio-group>
  </wd-form-item>
  <wd-button @click="handleSubmit" block type="primary">提交</wd-button>
</wd-form>
```

#### 模板 2：列表页面（带 cell）
```vue
<wd-cell-group title="基本信息">
  <wd-cell title="标题" :value="info.title" is-link @click="onEdit" />
  <wd-cell title="状态">
    <wd-tag type="success">{{ info.status }}</wd-tag>
  </wd-cell>
</wd-cell-group>
<wd-cell-group title="操作">
  <wd-cell title="删除" is-link @click="onDelete" />
</wd-cell-group>
```

#### 模板 3：弹层抽屉
```vue
<!-- 按钮触发 -->
<wd-button @click="drawerVisible = true">打开抽屉</wd-button>

<!-- 抽屉内容 -->
<wd-drawer v-model="drawerVisible" title="操作" side="right">
  <wd-button @click="onConfirm">确认</wd-button>
</wd-drawer>
```

#### 模板 4：操作反馈
```typescript
import { wdToast, wdDialog } from 'wot-design-uni'

// 轻提示
wdToast.show({ message: '保存成功', type: 'success' })

// 确认对话框
wdDialog.confirm({
  title: '确认删除',
  message: '删除后无法恢复，是否确认？',
}).then(() => {
  // 用户点了确认
})

// 操作菜单
wdActionSheet.show({
  actions: [
    { name: '编辑', action: () => handleEdit() },
    { name: '删除', color: '#fa4350', action: () => handleDelete() },
  ],
  cancelText: '取消'
})
```

## 组件文档链接索引

> 快速查找某个 `wd-*` 组件的详细 API 和示例。
> GitHub 源码：`https://github.com/Moonofweisheng/wot-design-uni/tree/master/src/uni_modules/wot-design-uni/components/wd-{name}/`
> 官网文档：`https://wot-ui.cn/component/{name}.html`

### 表单类
| 组件 | GitHub | 官网 |
|------|--------|------|
| `wd-button` | [wd-button](https://github.com/Moonofweisheng/wot-design-uni/tree/master/src/uni_modules/wot-design-uni/components/wd-button) | [button](https://wot-ui.cn/component/button.html) |
| `wd-input` | [wd-input](https://github.com/Moonofweisheng/wot-design-uni/tree/master/src/uni_modules/wot-design-uni/components/wd-input) | [input](https://wot-ui.cn/component/input.html) |
| `wd-textarea` | [wd-textarea](https://github.com/Moonofweisheng/wot-design-uni/tree/master/src/uni_modules/wot-design-uni/components/wd-textarea) | [textarea](https://wot-ui.cn/component/textarea.html) |
| `wd-input-number` | [wd-input-number](https://github.com/Moonofweisheng/wot-design-uni/tree/master/src/uni_modules/wot-design-uni/components/wd-input-number) | [input-number](https://wot-ui.cn/component/input-number.html) |
| `wd-checkbox` | [wd-checkbox](https://github.com/Moonofweisheng/wot-design-uni/tree/master/src/uni_modules/wot-design-uni/components/wd-checkbox) | [checkbox](https://wot-ui.cn/component/checkbox.html) |
| `wd-radio-group` | [wd-radio-group](https://github.com/Moonofweisheng/wot-design-uni/tree/master/src/uni_modules/wot-design-uni/components/wd-radio-group) | [radio](https://wot-ui.cn/component/radio.html) |
| `wd-switch` | [wd-switch](https://github.com/Moonofweisheng/wot-design-uni/tree/master/src/uni_modules/wot-design-uni/components/wd-switch) | [switch](https://wot-ui.cn/component/switch.html) |
| `wd-slider` | [wd-slider](https://github.com/Moonofweisheng/wot-design-uni/tree/master/src/uni_modules/wot-design-uni/components/wd-slider) | [slider](https://wot-ui.cn/component/slider.html) |
| `wd-search` | [wd-search](https://github.com/Moonofweisheng/wot-design-uni/tree/master/src/uni_modules/wot-design-uni/components/wd-search) | [search](https://wot-ui.cn/component/search.html) |
| `wd-rate` | [wd-rate](https://github.com/Moonofweisheng/wot-design-uni/tree/master/src/uni_modules/wot-design-uni/components/wd-rate) | [rate](https://wot-ui.cn/component/rate.html) |
| `wd-picker` | [wd-picker](https://github.com/Moonofweisheng/wot-design-uni/tree/master/src/uni_modules/wot-design-uni/components/wd-picker) | [picker](https://wot-ui.cn/component/picker.html) |
| `wd-date-picker` | [wd-date-picker](https://github.com/Moonofweisheng/wot-design-uni/tree/master/src/uni_modules/wot-design-uni/components/wd-date-picker) | [date-picker](https://wot-ui.cn/component/date-picker.html) |
| `wd-area-picker` | [wd-area-picker](https://github.com/Moonofweisheng/wot-design-uni/tree/master/src/uni_modules/wot-design-uni/components/wd-area-picker) | [area-picker](https://wot-ui.cn/component/area-picker.html) |
| `wd-calendar` | [wd-calendar](https://github.com/Moonofweisheng/wot-design-uni/tree/master/src/uni_modules/wot-design-uni/components/wd-calendar) | [calendar](https://wot-ui.cn/component/calendar.html) |
| `wd-upload` | [wd-upload](https://github.com/Moonofweisheng/wot-design-uni/tree/master/src/uni_modules/wot-design-uni/components/wd-upload) | [upload](https://wot-ui.cn/component/upload.html) |
| `wd-form` | [wd-form](https://github.com/Moonofweisheng/wot-design-uni/tree/master/src/uni_modules/wot-design-uni/components/wd-form) | [form](https://wot-ui.cn/component/form.html) |

### 布局展示类
| 组件 | GitHub | 官网 |
|------|--------|------|
| `wd-cell-group` | [wd-cell-group](https://github.com/Moonofweisheng/wot-design-uni/tree/master/src/uni_modules/wot-design-uni/components/wd-cell-group) | [cell](https://wot-ui.cn/component/cell.html) |
| `wd-card` | [wd-card](https://github.com/Moonofweisheng/wot-design-uni/tree/master/src/uni_modules/wot-design-uni/components/wd-card) | [card](https://wot-ui.cn/component/card.html) |
| `wd-grid` | [wd-grid](https://github.com/Moonofweisheng/wot-design-uni/tree/master/src/uni_modules/wot-design-uni/components/wd-grid) | [grid](https://wot-ui.cn/component/grid.html) |
| `wd-img` | [wd-img](https://github.com/Moonofweisheng/wot-design-uni/tree/master/src/uni_modules/wot-design-uni/components/wd-img) | [img](https://wot-ui.cn/component/img.html) |
| `wd-badge` | [wd-badge](https://github.com/Moonofweisheng/wot-design-uni/tree/master/src/uni_modules/wot-design-uni/components/wd-badge) | [badge](https://wot-ui.cn/component/badge.html) |
| `wd-tag` | [wd-tag](https://github.com/Moonofweisheng/wot-design-uni/tree/master/src/uni_modules/wot-design-uni/components/wd-tag) | [tag](https://wot-ui.cn/component/tag.html) |
| `wd-avatar` | [wd-avatar](https://github.com/Moonofweisheng/wot-design-uni/tree/master/src/uni_modules/wot-design-uni/components/wd-avatar) | [avatar](https://wot-ui.cn/component/avatar.html) |
| `wd-progress` | [wd-progress](https://github.com/Moonofweisheng/wot-design-uni/tree/master/src/uni_modules/wot-design-uni/components/wd-progress) | [progress](https://wot-ui.cn/component/progress.html) |
| `wd-steps` | [wd-steps](https://github.com/Moonofweisheng/wot-design-uni/tree/master/src/uni_modules/wot-design-uni/components/wd-steps) | [steps](https://wot-ui.cn/component/steps.html) |
| `wd-divider` | [wd-divider](https://github.com/Moonofweisheng/wot-design-uni/tree/master/src/uni_modules/wot-design-uni/components/wd-divider) | [divider](https://wot-ui.cn/component/divider.html) |
| `wd-status-tip` | [wd-status-tip](https://github.com/Moonofweisheng/wot-design-uni/tree/master/src/uni_modules/wot-design-uni/components/wd-status-tip) | [status-tip](https://wot-ui.cn/component/status-tip.html) |
| `wd-notice-bar` | [wd-notice-bar](https://github.com/Moonofweisheng/wot-design-uni/tree/master/src/uni_modules/wot-design-uni/components/wd-notice-bar) | [notice-bar](https://wot-ui.cn/component/notice-bar.html) |
| `wd-timeline` | [wd-timeline](https://github.com/Moonofweisheng/wot-design-uni/tree/master/src/uni_modules/wot-design-uni/components/wd-timeline) | [timeline](https://wot-ui.cn/component/timeline.html) |

### 导航类
| 组件 | GitHub | 官网 |
|------|--------|------|
| `wd-tabs` | [wd-tabs](https://github.com/Moonofweisheng/wot-design-uni/tree/master/src/uni_modules/wot-design-uni/components/wd-tabs) | [tabs](https://wot-ui.cn/component/tabs.html) |
| `wd-tabbar` | [wd-tabbar](https://github.com/Moonofweisheng/wot-design-uni/tree/master/src/uni_modules/wot-design-uni/components/wd-tabbar) | [tabbar](https://wot-ui.cn/component/tabbar.html) |
| `wd-navbar` | [wd-navbar](https://github.com/Moonofweisheng/wot-design-uni/tree/master/src/uni_modules/wot-design-uni/components/wd-navbar) | [navbar](https://wot-ui.cn/component/navbar.html) |
| `wd-pagination` | [wd-pagination](https://github.com/Moonofweisheng/wot-design-uni/tree/master/src/uni_modules/wot-design-uni/components/wd-pagination) | [pagination](https://wot-ui.cn/component/pagination.html) |
| `wd-index-anchor` | [wd-index-anchor](https://github.com/Moonofweisheng/wot-design-uni/tree/master/src/uni_modules/wot-design-uni/components/wd-index-anchor) | [index-anchor](https://wot-ui.cn/component/index-anchor.html) |

### 弹层反馈类
| 组件 | GitHub | 官网 |
|------|--------|------|
| `wd-toast` | [wd-toast](https://github.com/Moonofweisheng/wot-design-uni/tree/master/src/uni_modules/wot-design-uni/components/wd-toast) | [toast](https://wot-ui.cn/component/toast.html) |
| `wd-dialog` | [wd-dialog](https://github.com/Moonofweisheng/wot-design-uni/tree/master/src/uni_modules/wot-design-uni/components/wd-dialog) | [dialog](https://wot-ui.cn/component/dialog.html) |
| `wd-action-sheet` | [wd-action-sheet](https://github.com/Moonofweisheng/wot-design-uni/tree/master/src/uni_modules/wot-design-uni/components/wd-action-sheet) | [action-sheet](https://wot-ui.cn/component/action-sheet.html) |
| `wd-popup` | [wd-popup](https://github.com/Moonofweisheng/wot-design-uni/tree/master/src/uni_modules/wot-design-uni/components/wd-popup) | [popup](https://wot-ui.cn/component/popup.html) |
| `wd-drawer` | [wd-drawer](https://github.com/Moonofweisheng/wot-design-uni/tree/master/src/uni_modules/wot-design-uni/components/wd-drawer) | [drawer](https://wot-ui.cn/component/drawer.html) |
| `wd-loading` | [wd-loading](https://github.com/Moonofweisheng/wot-design-uni/tree/master/src/uni_modules/wot-design-uni/components/wd-loading) | [loading](https://wot-ui.cn/component/loading.html) |
| `wd-popover` | [wd-popover](https://github.com/Moonofweisheng/wot-design-uni/tree/master/src/uni_modules/wot-design-uni/components/wd-popover) | [popover](https://wot-ui.cn/component/popover.html) |
| `wd-back-top` | [wd-back-top](https://github.com/Moonofweisheng/wot-design-uni/tree/master/src/uni_modules/wot-design-uni/components/wd-back-top) | [back-top](https://wot-ui.cn/component/back-top.html) |
| `wd-result` | [wd-result](https://github.com/Moonofweisheng/wot-design-uni/tree/master/src/uni_modules/wot-design-uni/components/wd-result) | [result](https://wot-ui.cn/component/result.html) |

### 配置类
| 组件 | GitHub | 官网 |
|------|--------|------|
| `wd-config-provider` | [wd-config-provider](https://github.com/Moonofweisheng/wot-design-uni/tree/master/src/uni_modules/wot-design-uni/components/wd-config-provider) | [config-provider](https://wot-ui.cn/component/config-provider.html) |
| `wd-icon` | [wd-icon](https://github.com/Moonofweisheng/wot-design-uni/tree/master/src/uni_modules/wot-design-uni/components/wd-icon) | [icon](https://wot-ui.cn/component/icon.html) |

### 业务组合类
| 组件 | GitHub | 官网 |
|------|--------|------|
| `wd-goods-card` | [wd-goods-card](https://github.com/Moonofweisheng/wot-design-uni/tree/master/src/uni_modules/wot-design-uni/components/wd-goods-card) | [goods-card](https://wot-ui.cn/component/goods-card.html) |
| `wd-sku` | [wd-sku](https://github.com/Moonofweisheng/wot-design-uni/tree/master/src/uni_modules/wot-design-uni/components/wd-sku) | [sku](https://wot-ui.cn/component/sku.html) |
| `wd-goods-route` | [wd-goods-route](https://github.com/Moonofweisheng/wot-design-uni/tree/master/src/uni_modules/wot-design-uni/components/wd-goods-route) | [goods-route](https://wot-ui.cn/component/goods-route.html) |
| `wd-address-list` | [wd-address-list](https://github.com/Moonofweisheng/wot-design-uni/tree/master/src/uni_modules/wot-design-uni/components/wd-address-list) | [address-list](https://wot-ui.cn/component/address-list.html) |

> 所有 70+ 组件的完整列表请访问 GitHub 目录：[components](https://github.com/Moonofweisheng/wot-design-uni/tree/master/src/uni_modules/wot-design-uni/components)

---

## 约束清单

| 规则 | 说明 |
|------|------|
| 每个组件 3 文件 | 必有 `wd-*.vue`、`types.ts`、`index.scss` |
| Props 用工厂函数 | 禁止手写 `{ type: Boolean }`，用 `makeBooleanProp(false)` |
| BEM 命名 | 块+元素用 `@include b/e`，修饰符用 `@include m`，状态用 `@include when` |
| SCSS 导入顺序 | `@import '../common/abstracts/_mixin.scss'` -> `@import '../common/abstracts/variable.scss'` |
| 样式穿透 | 小程序用 `@include edeep(element)`，H5 直接用 `:deep()` |
| options 配置 | 必须有 `virtualHost: true`、`addGlobalClass: true` |
| 暗黑模式 | 在 `.wot-theme-dark` 包裹块内写对应样式 |
| customClass/customStyle | 组件根节点必须接受并应用这两个来自 baseProps 的属性 |
| 主题变量名 | CSS 变量用 `--wot-xxx`，SCSS 变量用 `$-button-xxx`，不要混淆 |
| emit 类型定义 | 使用 Vue 3.4+ 的 `defineEmits<{ (e: 'event', payload): void }>()` 泛型语法 |
