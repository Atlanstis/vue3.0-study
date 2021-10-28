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

### 概念

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

### 实现

```js
const isObject = (val) => val !== null && typeof val === 'object'
const convert = (target) => (isObject(target) ? reactive(target) : target)
const hasOwnProperty = Object.prototype.hasOwnProperty
const hasOwn = (target, key) => hasOwnProperty.call(target, key)

export function reactive(target) {
  if (!isObject(target)) return target
  const handler = {
    get: (target, key, receiver) => {
      // 收集依赖
      track(target, key)
      const result = Reflect.get(target, key, receiver)
      return convert(result)
    },

    set: (target, key, value, receiver) => {
      const oldVal = Reflect.get(target, key, receiver)
      let result = true
      if (oldVal !== value) {
        result = Reflect.set(target, key, value, receiver)
        // 触发更新
        trigger(target, key)
      }
      return result
    },

    deleteProperty: (target, key) => {
      const hadKey = hasOwn(target, key)
      const result = Reflect.deleteProperty(target, key)
      if (hadKey && result) {
        // 触发更新
        trigger(target, key)
      }
      return result
    }
  }
  return new Proxy(target, handler)
}

let activeEffect = null

export function effect(callback) {
  activeEffect = callback
  callback() // 访问响应式对象属性，去收集依赖
  activeEffect = null
}

let targetMap = new WeakMap()

// 依赖收集的函数
export function track(target, key) {
  if (!activeEffect) return
  let depsMap = targetMap.get(target)
  if (!depsMap) {
    targetMap.set(target, (depsMap = new Map()))
  }
  let dep = depsMap.get(key)
  if (!dep) {
    depsMap.set(key, (dep = new Set()))
  }
  dep.add(activeEffect)
}

// 触发更新的函数
export function trigger(target, key) {
  const depsMap = targetMap.get(target)
  if (!depsMap) return
  const dep = depsMap.get(key)
  if (dep) {
    dep.forEach((effect) => {
      effect()
    })
  }
}

export function ref(raw) {
  // 判断 raw 是不是 ref 创建的对象，如果是，直接返回
  if (isObject(raw) && raw.__v_isRef) {
    return raw
  }
  let value = convert(raw)
  const r = {
    __v_isRef: true,
    get value() {
      track(r, 'value')
      return value
    },

    set value(newValue) {
      if (newValue !== value) {
        raw = newValue
        value = convert(raw)
        trigger(r, 'value')
      }
    }
  }

  return r
}

export function toRefs(proxy) {
  // 未对 reactive 创建的对象进行标记，此处先跳过判断是否是 reactive 创建的对象

  const ret = proxy instanceof Array ? new Array(proxy.length) : {}
  for (const key in proxy) {
    ret[key] = toProxyRef(proxy, key)
  }
  return ret
}

function toProxyRef(proxy, key) {
  const r = {
    __v_isRef: true,
    get value() {
      return proxy[key]
    },

    set value(newValue) {
      proxy[key] = newValue
    }
  }
  return r
}

export function computed(getter) {
  const result = ref()

  effect(() => {
    result.value = getter()
  })

  return result
}
```

## Vite

### 概念

- Vite 是一个面向现代浏览器的一个更轻更快的 Web 应用开发工具
- 基于 ECMAScript 标准原生模块系统（ES Modules）实现

## 核心功能

- 静态 Web 服务器
- 编译单文件组件
  - 拦截浏览器不识别的模块，并处理
- HMR
