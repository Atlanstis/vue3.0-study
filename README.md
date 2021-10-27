# Vue.js 3.0

## 基本介绍

> [Vue3.0 中文文档](https://v3.cn.vuejs.org/)

### 源码组织方式

- 源码采用 TypeScript 重写
- 使用 Monorepo 管理项目结构（单项目管理多个包）

```
package 目录结构
|-- compiler-core       与平台无关的的编译器
|-- compiler-dom        浏览器环境下的编译器，依赖于 compiler-core
|-- compiler-sfc        编译单文件组件，依赖于 compiler-dom 与 compiler-core
|-- compiler-ssr        服务端渲染编译器，依赖于 compiler-dom
|-- reactivity          数据响应式系统
|-- runtime-core        与平台无关的运行时
|-- runtime-dom         与浏览器相关的运行时，处理原生 DOM API，事件等
|-- runtime-test        为测试编写的运行时
|-- server-renderer     用于服务器渲染
|-- shared              Vue 内部使用的公共 API
|-- size-check          私有，tree shaking 后，检查包的大小
|-- template-explorer   浏览器环境里运行的实时编译组件，输出 render 函数
|-- vue                 构建完整版的 vue
```

### Composition API

- Vue.js 3.0 新增的一组 API
- 一组基于函数的 API
- 可以更灵活的组织组件的逻辑

### 性能提升

#### 响应式系统升级

- Vue.js 2.x 中响应式系统的核心 defineProperty
- Vue.js 3.0 中使用 Proxy 对象重写响应式系统
  - 可以监听动态新增的属性
  - 可以监听删除的属性
  - 可以监听数组的索引及 length 属性

#### 编译优化

- Vue.js 2.x 中通过标记静态根节点，优化 diff 的过程
- Vue.js 3.0 中标记和提升所有的静态根节点，diff 的时候只需要对比动态节点内容
  - Fragments
  - 静态提升
  - Patch flag
  - 缓存事件处理函数

#### 优化打包体积

- Vue.js 3.0 中移除了一些不常用的 API
  - 例如：inline-template，filter 等
- Tree-shaking

### Vite

基于现代浏览器可直接运行 ES Module 的特性。

#### Vite as Vue-CLI

- Vite 在开发模式下不需要打包可以直接运行
- Vue-CLI 在开发模式下必须对项目打包才可以运行
- Vite 在生产环境下使用 Rollup 打包
  - 基于 ES Module 的方式打包
- Vue-CLI 在生产环境下使用 Webpack 打包

#### 特点

- 快速冷启动
- 按需编译
- 模块热更新

## Composition API

> [Composition API 案例](https://github.com/Atlanstis/vue3.0-study/tree/main/todolist)

## 响应式系统

### 回顾

- Proxy 对象实现属性监听
- 多层属性嵌套，在访问属性过程中处理下一级属性
- 默认监听动态添加的属性
- 默认监听属性的删除操作
- 默认监听数组索引和 length 属性
- 可以作为单独的模块使用

### 实现

> [响应式简易实现](https://github.com/Atlanstis/vue3.0-study/tree/main/reactivity)

#### reactive

- 接受一个参数，判断这参数是否为对象（只能为对象）
- 创建拦截器对象 handler，设置 get/set/deleteProperty
- 返回 Proxy 对象

#### reactive vs ref

- ref 可以把基本数据类型数据，转成响应式对象
- ref 返回的对象，重新赋值成对象也是响应式的
- reactive 返回的对象，重新赋值会丢失响应式特性
- reactive 返回的对象不可以解构

#### toRefs

- 将 reactive 创建的对象，转换成 ref 创建的对象

