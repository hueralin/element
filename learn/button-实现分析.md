# ElementUI Button 组件实现分析

## 1. 组件源码结构

- 入口文件：`packages/button/index.js`，用于注册组件。
- 主体实现：`packages/button/src/button.vue`，包含模板、props、计算属性、方法。
- 组合组件：`packages/button-group/index.js` 和 `packages/button/src/button-group.vue`，用于按钮组。

## 2. 主要 Props 说明

| Prop       | 类型    | 默认值    | 说明           |
| ---------- | ------- | --------- | -------------- |
| type       | String  | 'default' | 按钮类型       |
| size       | String  |           | 按钮尺寸       |
| icon       | String  | ''        | 图标类名       |
| nativeType | String  | 'button'  | 原生 type 属性 |
| loading    | Boolean | false     | 是否加载中     |
| disabled   | Boolean | false     | 是否禁用       |
| plain      | Boolean | false     | 是否朴素按钮   |
| autofocus  | Boolean | false     | 是否自动聚焦   |
| round      | Boolean | false     | 是否圆角按钮   |
| circle     | Boolean | false     | 是否圆形按钮   |

## 3. 依赖与注入

- **依赖注入**：
  - `elForm`、`elFormItem`：用于与表单组件联动，自动获取禁用、尺寸等状态。
- **全局配置**：
  - 通过 `this.$ELEMENT` 获取全局 size 配置。
- **无 mixin、无自定义指令**：button 组件本身未直接使用 mixin 或自定义指令。

## 4. 计算属性与核心逻辑

- `buttonSize`：优先级为 props.size > elFormItem.elFormItemSize > $ELEMENT.size。
- `buttonDisabled`：优先级为 props.disabled > elForm.disabled。
- `handleClick` 方法：触发 click 事件。
- loading 状态下，icon 被 loading 图标替换。

### 4.1 样式实现与 class 绑定分析

ElementUI Button 组件的样式实现主要依赖于动态 class 绑定和 SCSS 样式文件：

#### 4.1.1 class 绑定逻辑

在 `button.vue` 的模板中，按钮的 class 属性会根据不同的 props 动态添加样式类：

- `el-button--{type}`：按钮类型（如 primary、success、danger 等）
- `el-button--{size}`：按钮尺寸（如 medium、small、mini）
- `is-disabled`：禁用态
- `is-loading`：加载态
- `is-plain`：朴素按钮
- `is-round`：圆角按钮
- `is-circle`：圆形按钮

这些 class 会根据 props 的值自动组合，形成丰富的按钮样式。

#### 4.1.2 样式文件位置

所有按钮相关的样式定义在 `packages/theme-chalk/src/button.scss` 文件中。该文件包含了按钮的基础样式、不同类型、尺寸、状态下的样式表现。

#### 4.1.3 主要样式类说明

- `.el-button`：按钮的基础样式，包括布局、边框、字体等。
- `.el-button--primary`、`.el-button--success` 等：不同类型按钮的配色方案。
- `.el-button--medium`、`.el-button--small`、`.el-button--mini`：不同尺寸按钮的高度、字体大小等。
- `.is-disabled`：禁用态样式，通常会有灰色背景、不可点击等效果。
- `.is-loading`：加载态样式，显示 loading 图标并禁用按钮。
- `.is-plain`：朴素按钮样式，通常无背景色，边框色与字体色一致。
- `.is-round`：圆角按钮样式。
- `.is-circle`：圆形按钮样式。

#### 4.1.4 交互相关样式

- 按钮在 `disabled` 或 `loading` 状态下不可点击，并有相应的视觉反馈。
- 加载时显示 `.el-icon-loading` 图标，并隐藏原有 icon 和文本。
- 只有在有默认插槽内容时才渲染文本，方便 loading 态下只显示 loading 图标。

> 你可以在 `packages/theme-chalk/src/button.scss` 文件中查看更多详细的样式实现细节。

#### 4.1.5 朴素按钮高亮与失焦机制

朴素按钮（plain）在点击时的高亮效果，完全依赖于浏览器的 `:active` 伪类和 CSS 样式：

- `:active` 伪类在鼠标按下时自动添加，松开或点击其他地方时自动移除，无需 JS 事件辅助。
- 相关高亮样式在 `button.scss` 的 `@include when(plain)` 下定义。
- 这种实现方式简单高效，充分利用了浏览器原生的交互机制。

### 4.2 button.scss 样式实现与 BEM 规范详解

ElementUI 的样式实现采用了 BEM（Block-Element-Modifier）命名规范，并结合 SCSS 的 mixin 机制，实现了高可维护性和灵活的样式扩展。

#### 4.2.1 BEM 命名规范简介

- **Block（块）**：独立的功能模块，如 `el-button`。
- **Element（元素）**：块的组成部分，用双下划线连接，如 `el-button__icon`（本组件未用到）。
- **Modifier（修饰符）**：用于定义块或元素的不同状态或变体，用双中划线连接，如 `el-button--primary`、`el-button--small`。

BEM 的好处是让样式结构清晰、可复用、易于维护。

#### 4.2.2 SCSS mixin 与 @include 的用法

- `@include b(button) { ... }`：生成以 `.el-button` 为前缀的样式块。
- `@include m(primary) { ... }`：生成 `.el-button--primary` 这样的修饰符样式。
- `@include when(disabled) { ... }`：生成 `.is-disabled` 这样的状态类样式。
- `@include button-size(...)`、`@include button-variant(...)`：调用自定义的 mixin，批量设置尺寸、配色等属性。

##### 4.2.2.1 例子：
```scss
@include b(button) {
  ... // 这里是 .el-button 的基础样式
  @include m(primary) {
    ... // 这里是 .el-button--primary 的样式
  }
  @include when(disabled) {
    ... // 这里是 .is-disabled 的样式
  }
}
```

#### 4.2.3 主要样式实现机制

- **基础样式**：通过 `.el-button` 设置按钮的布局、边框、颜色等。
- **类型修饰符**：如 `.el-button--primary`，通过 `@include m(primary)` 调用 mixin，设置不同类型的配色方案。
- **尺寸修饰符**：如 `.el-button--small`，通过 `@include m(small)` 调用 mixin，设置不同尺寸的 padding、字体大小等。
- **状态修饰符**：如 `.is-disabled`、`.is-loading`，通过 `@include when(disabled)`、`@include when(loading)`，设置禁用、加载等状态下的样式。
- **圆角/圆形**：如 `.is-round`、`.is-circle`，通过 `@include when(round)`、`@include when(circle)`，设置圆角或圆形按钮。

#### 4.2.4 代码片段举例说明

```scss
@include b(button) {
  // 基础样式
  display: inline-block;
  ...
  @include m(primary) {
    // 主按钮样式
    @include button-variant(...);
  }
  @include m(small) {
    // 小尺寸按钮样式
    @include button-size(...);
  }
  @include when(disabled) {
    // 禁用态样式
    color: $--button-disabled-font-color;
    cursor: not-allowed;
    ...
  }
}
```

#### 4.2.5 总结

通过 BEM 命名和 SCSS mixin，ElementUI 的 button 样式实现既规范又灵活，便于扩展和维护。即使不熟悉 BEM，也可以通过观察 class 结构和 SCSS 组织方式，快速理解和定制按钮样式。

## 5. 与全局配置的关系

- 全局配置通过 `Vue.prototype.$ELEMENT` 注入，包含 size、zIndex 等。
- 组件内部通过 `this.$ELEMENT.size` 获取全局默认尺寸。

## 6. button-group 简析

- 结构简单，仅包裹 slot 内容，class 为 `el-button-group`。
- 主要用于按钮组合的样式处理。

## 7. 相关国际化

- button 组件本身无独立文案，国际化主要由外部传入 slot 内容。

## 8. 相关变更记录（摘自 CHANGELOG）

- 2.15.7：修复 disabled 优先级 (#21375 by @cs1707)

---

> 本文档基于源码自动分析生成，适合初学者理解 ElementUI Button 组件的实现方式。
