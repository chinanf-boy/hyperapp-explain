# hyperapp

1 KB JavaScript 库 - 构建 web 应用. 

[![explain](http://llever.com/explain.svg)](https://github.com/chinanf-boy/Source-Explain)
    
Explanation

> "version": "1.0.0"

[github source](https://github.com/)

~~[english](./README.en.md)~~

---

😯只有1kb的应用框架

---

## 本目录

- [不先使用何谈解释](#使用-hyperapp)

- [认识-`package.json`](#package-json)

---

## 使用-hyperapp

``` js
import { h, app } from "hyperapp" // 导入

const state = { // 初始化-存储
  count: 0
}

const actions = { // 动作行为-定义
  down: value => state => ({ count: state.count - value }),
  up: value => state => ({ count: state.count + value })
}

const view = (state, actions) => ( // 组件定义
  <div>
    <h1>{state.count}</h1>
    <button onclick={() => actions.down(1)}>-</button>
    <button onclick={() => actions.up(1)}>+</button>
  </div>
)

app(state, actions, view, document.body) // 挂载-> document.body
```

> [codepen-hyperapp 一系列的示例](https://codepen.io/hyperapp/)

---

## package-json

作为js项目的根本

``` json
  "main": "dist/hyperapp.js", // node 导入主文件
  "module": "src/index.js", // 模块
  "typings": "hyperapp.d.ts", // ts 类型
```

## src-index

只有一个文件`index.js`

我们先看看示例的引入

``` js
import { h, app } from "hyperapp" 
```

- [h](#h-jsx语法糖)

- app

## h-jsx语法糖

代码 1-26

``` js
export function h(name, props) {
  var node
  var rest = []
  var children = []
  var length = arguments.length

  while (length-- > 2) rest.push(arguments[length])

  while (rest.length) {
    if (Array.isArray((node = rest.pop()))) {
      for (length = node.length; length--; ) {
        rest.push(node[length])
      }
    } else if (node != null && node !== true && node !== false) {
      children.push(node)
    }
  }

  return typeof name === "function"
    ? name(props || {}, children)
    : {
        name: name,
        props: props || {},
        children: children
      }
}
```