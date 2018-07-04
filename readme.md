# hyperapp [![explain](http://llever.com/explain.svg)](https://github.com/chinanf-boy/Source-Explain)

1 KB JavaScript 库 - 构建 web 应用. 

> "version": "1.2.6"

[github source](https://github.com/hyperapp/hyperapp/tree/1.2.6)

[中文](./readme.md) |  ~~[english explain](./README.en.md)~~

欢迎 `Issue` 和 `Pull` ❤️, 最好 `Pull` 👏

---

😯只有1kb的应用框架

- [ ] [校对🀄️: 原文 readme.md 🇨🇳翻译](./hyperapp.readme.md)

---

## 本目录

- [x] [不先使用何谈解释](#使用-hyperapp)

- [x] [认识-`package.json`](#package-json)

- [x] [src/index 主文件](#src-index)

- [ ] [ h - jsx语法糖](#h-jsx语法糖)

- [ ] [app - 结合与挂载](#app-结合与挂载)

---

## 使用-hyperapp

---

这个例子假设你正在使用像`Babel`或`TypeScript`这样的JavaScript编译器，以及像`Parcel`，`Rollup`，`Webpack`等模块捆绑器。通常，你需要做的就是`安装JSX 转换插件`，并将编译选项添加到你的.babelrc 文件。

``` json
{
  "plugins": [["transform-react-jsx", { "pragma": "h" }]]
}
```

[更多-可以看](https://github.com/hyperapp/hyperapp#getting-started)

---

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

// 使用-上述-JSX 转换插件后，我们可以-上面所写的

// 不然我们需要，下面这样写

// const view = (state, actions) =>
//   h("div", {}, [
//     h("h1", {}, state.count),
//     h("button", { onclick: () => actions.down(1) }, "-"),
//     h("button", { onclick: () => actions.up(1) }, "+")
//   ])

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

---

## src-index

只有一个文件[index.js](./hyperapp/src/index.js)

我们先看看示例的引入

``` js
import { h, app } from "hyperapp" 
```

- [h > jsx语法糖](#h-jsx语法糖)

- [app > 结合与挂载](#app-结合与挂载)

---

## h-jsx语法糖

代码 1-26

 `<button onclick={() => actions.down(1)}>-</button>`

==

`h("button", { onclick: () => actions.down(1) }, "-"),`

> `h` 不仅作为-`JSX语法糖`-且-作为规范`组件结构`的作用

``` js
export function h(name, attributes) {
  var rest = []
  var children = []
  var length = arguments.length

  while (length-- > 2) rest.push(arguments[length])

  while (rest.length) {
    var node = rest.pop()
    if (node && node.pop) {
      for (length = node.length; length--; ) {
        rest.push(node[length])
      }
    } else if (node != null && node !== true && node !== false) {
      children.push(node)
    }
  }

  return typeof name === "function"
    ? name(attributes || {}, children)
    : {
        nodeName: name,
        attributes: attributes || {},
        children: children,
        key: attributes && attributes.key
      }
}
```

---

## app-结合与挂载

使用-> `app(state, actions, view, document.body)`

- state `存储`

- actions `动作函数`

- view `组件`

- document.body `挂载目标`

代码 28-315

``` js
export function h(name, attributes) {
  var rest = []
  var children = []
  var length = arguments.length

  while (length-- > 2) rest.push(arguments[length])

  while (rest.length) {
    var node = rest.pop()
    if (node && node.pop) {
      for (length = node.length; length--; ) {
        rest.push(node[length])
      }
    } else if (node != null && node !== true && node !== false) {
      children.push(node)
    }
  }

  return typeof name === "function"
    ? name(attributes || {}, children)
    : {
        nodeName: name,
        attributes: attributes || {},
        children: children,
        key: attributes && attributes.key
      }
}

export function app(state, actions, view, container) {
  var map = [].map
  var rootElement = (container && container.children[0]) || null
  var oldNode = rootElement && recycleElement(rootElement)
  var lifecycle = []
  var skipRender
  var isRecycling = true
  var globalState = clone(state)
  var wiredActions = wireStateToActions([], globalState, clone(actions))

  scheduleRender()

  return wiredActions

  function recycleElement(element) {
    return {
      nodeName: element.nodeName.toLowerCase(),
      attributes: {},
      children: map.call(element.childNodes, function(element) {
        return element.nodeType === 3 // Node.TEXT_NODE
          ? element.nodeValue
          : recycleElement(element)
      })
    }
  }

  function resolveNode(node) {
    return typeof node === "function"
      ? resolveNode(node(globalState, wiredActions))
      : node != null
        ? node
        : ""
  }

  function render() {
    skipRender = !skipRender

    var node = resolveNode(view)

    if (container && !skipRender) {
      rootElement = patch(container, rootElement, oldNode, (oldNode = node))
    }

    isRecycling = false

    while (lifecycle.length) lifecycle.pop()()
  }

  function scheduleRender() {
    if (!skipRender) {
      skipRender = true
      setTimeout(render)
    }
  }

  function clone(target, source) {
    var out = {}

    for (var i in target) out[i] = target[i]
    for (var i in source) out[i] = source[i]

    return out
  }

  function setPartialState(path, value, source) {
    var target = {}
    if (path.length) {
      target[path[0]] =
        path.length > 1
          ? setPartialState(path.slice(1), value, source[path[0]])
          : value
      return clone(source, target)
    }
    return value
  }

  function getPartialState(path, source) {
    var i = 0
    while (i < path.length) {
      source = source[path[i++]]
    }
    return source
  }

  function wireStateToActions(path, state, actions) {
    for (var key in actions) {
      typeof actions[key] === "function"
        ? (function(key, action) {
            actions[key] = function(data) {
              var result = action(data)

              if (typeof result === "function") {
                result = result(getPartialState(path, globalState), actions)
              }

              if (
                result &&
                result !== (state = getPartialState(path, globalState)) &&
                !result.then // !isPromise
              ) {
                scheduleRender(
                  (globalState = setPartialState(
                    path,
                    clone(state, result),
                    globalState
                  ))
                )
              }

              return result
            }
          })(key, actions[key])
        : wireStateToActions(
            path.concat(key),
            (state[key] = clone(state[key])),
            (actions[key] = clone(actions[key]))
          )
    }

    return actions
  }

  function getKey(node) {
    return node ? node.key : null
  }

  function eventListener(event) {
    return event.currentTarget.events[event.type](event)
  }

  function updateAttribute(element, name, value, oldValue, isSvg) {
    if (name === "key") {
    } else if (name === "style") {
      for (var i in clone(oldValue, value)) {
        var style = value == null || value[i] == null ? "" : value[i]
        if (i[0] === "-") {
          element[name].setProperty(i, style)
        } else {
          element[name][i] = style
        }
      }
    } else {
      if (name[0] === "o" && name[1] === "n") {
        name = name.slice(2)

        if (element.events) {
          if (!oldValue) oldValue = element.events[name]
        } else {
          element.events = {}
        }

        element.events[name] = value

        if (value) {
          if (!oldValue) {
            element.addEventListener(name, eventListener)
          }
        } else {
          element.removeEventListener(name, eventListener)
        }
      } else if (name in element && name !== "list" && !isSvg) {
        element[name] = value == null ? "" : value
      } else if (value != null && value !== false) {
        element.setAttribute(name, value)
      }

      if (value == null || value === false) {
        element.removeAttribute(name)
      }
    }
  }

  function createElement(node, isSvg) {
    var element =
      typeof node === "string" || typeof node === "number"
        ? document.createTextNode(node)
        : (isSvg = isSvg || node.nodeName === "svg")
          ? document.createElementNS(
              "http://www.w3.org/2000/svg",
              node.nodeName
            )
          : document.createElement(node.nodeName)

    var attributes = node.attributes
    if (attributes) {
      if (attributes.oncreate) {
        lifecycle.push(function() {
          attributes.oncreate(element)
        })
      }

      for (var i = 0; i < node.children.length; i++) {
        element.appendChild(
          createElement(
            (node.children[i] = resolveNode(node.children[i])),
            isSvg
          )
        )
      }

      for (var name in attributes) {
        updateAttribute(element, name, attributes[name], null, isSvg)
      }
    }

    return element
  }

  function updateElement(element, oldAttributes, attributes, isSvg) {
    for (var name in clone(oldAttributes, attributes)) {
      if (
        attributes[name] !==
        (name === "value" || name === "checked"
          ? element[name]
          : oldAttributes[name])
      ) {
        updateAttribute(
          element,
          name,
          attributes[name],
          oldAttributes[name],
          isSvg
        )
      }
    }

    var cb = isRecycling ? attributes.oncreate : attributes.onupdate
    if (cb) {
      lifecycle.push(function() {
        cb(element, oldAttributes)
      })
    }
  }

  function removeChildren(element, node) {
    var attributes = node.attributes
    if (attributes) {
      for (var i = 0; i < node.children.length; i++) {
        removeChildren(element.childNodes[i], node.children[i])
      }

      if (attributes.ondestroy) {
        attributes.ondestroy(element)
      }
    }
    return element
  }

  function removeElement(parent, element, node) {
    function done() {
      parent.removeChild(removeChildren(element, node))
    }

    var cb = node.attributes && node.attributes.onremove
    if (cb) {
      cb(element, done)
    } else {
      done()
    }
  }

  function patch(parent, element, oldNode, node, isSvg) {
    if (node === oldNode) {
    } else if (oldNode == null || oldNode.nodeName !== node.nodeName) {
      var newElement = createElement(node, isSvg)
      parent.insertBefore(newElement, element)

      if (oldNode != null) {
        removeElement(parent, element, oldNode)
      }

      element = newElement
    } else if (oldNode.nodeName == null) {
      element.nodeValue = node
    } else {
      updateElement(
        element,
        oldNode.attributes,
        node.attributes,
        (isSvg = isSvg || node.nodeName === "svg")
      )

      var oldKeyed = {}
      var newKeyed = {}
      var oldElements = []
      var oldChildren = oldNode.children
      var children = node.children

      for (var i = 0; i < oldChildren.length; i++) {
        oldElements[i] = element.childNodes[i]

        var oldKey = getKey(oldChildren[i])
        if (oldKey != null) {
          oldKeyed[oldKey] = [oldElements[i], oldChildren[i]]
        }
      }

      var i = 0
      var k = 0

      while (k < children.length) {
        var oldKey = getKey(oldChildren[i])
        var newKey = getKey((children[k] = resolveNode(children[k])))

        if (newKeyed[oldKey]) {
          i++
          continue
        }

        if (newKey != null && newKey === getKey(oldChildren[i + 1])) {
          if (oldKey == null) {
            removeElement(element, oldElements[i], oldChildren[i])
          }
          i++
          continue
        }

        if (newKey == null || isRecycling) {
          if (oldKey == null) {
            patch(element, oldElements[i], oldChildren[i], children[k], isSvg)
            k++
          }
          i++
        } else {
          var keyedNode = oldKeyed[newKey] || []

          if (oldKey === newKey) {
            patch(element, keyedNode[0], keyedNode[1], children[k], isSvg)
            i++
          } else if (keyedNode[0]) {
            patch(
              element,
              element.insertBefore(keyedNode[0], oldElements[i]),
              keyedNode[1],
              children[k],
              isSvg
            )
          } else {
            patch(element, oldElements[i], null, children[k], isSvg)
          }

          newKeyed[newKey] = children[k]
          k++
        }
      }

      while (i < oldChildren.length) {
        if (getKey(oldChildren[i]) == null) {
          removeElement(element, oldElements[i], oldChildren[i])
        }
        i++
      }

      for (var i in oldKeyed) {
        if (!newKeyed[i]) {
          removeElement(element, oldKeyed[i][0], oldKeyed[i][1])
        }
      }
    }
    return element
  }
}
```
