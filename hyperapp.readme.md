
# <img height=24 src=https://cdn.rawgit.com/jorgebucaran/f53d2c00bafcf36e84ffd862f0dc2950/raw/882f20c970ff7d61aa04d44b92fc3530fa758bc0/Hyperapp.svg>Hyperapp

[![Travis CI](https://img.shields.io/travis/hyperapp/hyperapp/master.svg)](https://travis-ci.org/hyperapp/hyperapp)
[![Codecov](https://img.shields.io/codecov/c/github/hyperapp/hyperapp/master.svg)](https://codecov.io/gh/hyperapp/hyperapp)
[![npm](https://img.shields.io/npm/v/hyperapp.svg)](https://www.npmjs.org/package/hyperapp)
[![Slack](https://hyperappjs.herokuapp.com/badge.svg)](https://hyperappjs.herokuapp.com "Join us")

Hyperapp是一个用于构建Web应用程序的JavaScript微框架. 

-   **最小**- 我们积极地将您需要理解的概念最小化,以提高工作效率,同时保持与其他框架相同的功能. 
-   **务实**- 在管理状态时,Hyperapp在功能编程方面保持坚定,但采取实用的方法来允许副作用,异步操作和DOM操作. 
-   **独立**- 少花钱多办事. Hyperapp将状态管理与支持键控更新和生命周期事件的虚拟DOM引擎相结合 - 所有这些都没有依赖关系. 

## 入门

我们的第一个例子是一个可以递增或递减的计数器. 来吧[在线尝试](https://codepen.io/hyperapp/pen/zNxZLP/left/?editors=0010). 

```jsx
import { h, app } from "hyperapp"

const state = {
  count: 0
}

const actions = {
  down: value => state => ({ count: state.count - value }),
  up: value => state => ({ count: state.count + value })
}

const view = (state, actions) => (
  <div>
    <h1>{state.count}</h1>
    <button onclick={() => actions.down(1)}>-</button>
    <button onclick={() => actions.up(1)}>+</button>
  </div>
)

app(state, actions, view, document.body)
```

Hyperapp由双功能API组成. <samp>hyperapp.h</samp>返回一个新的[虚拟DOM](#view)节点树和<samp>hyperapp.app</samp> [坐骑](#mounting)指定DOM元素中的新应用程序. 如果没有元素,可以使用Hyperapp"无头",这在单元测试程序时非常有用. 

此示例假定您使用的是JavaScript编译器[巴别塔](https://babeljs.io)要么[打字稿](https://www.typescriptlang.org)和模块捆绑器一样[包](https://parceljs.org),[的WebPack](https://webpack.js.org)如果您使用的是JSX,您只需安装JSX即可[转换插件](https://babeljs.io/docs/plugins/transform-react-jsx)并将pragma选项添加到您的<samp>.babelrc</samp>文件. 

```json
{
  "plugins": [["transform-react-jsx", { "pragma": "h" }]]
}
```

JSX是一种语言语法扩展,允许您编写散布在JavaScript中的HTML标记. 因为浏览器不了解JSX,所以我们使用编译器将其转换为<samp>hyperapp.h</samp>引擎盖下的功能调用. 

```jsx
const view = (state, actions) =>
  h("div", {}, [
    h("h1", {}, state.count),
    h("button", { onclick: () => actions.down(1) }, "-"),
    h("button", { onclick: () => actions.up(1) }, "+")
  ])
```

请注意,使用Hyperapp构建应用程序不需要JSX. 您可以使用<samp>hyperapp.h</samp>直接和没有如上所示的编译步骤. JSX的其他替代方案包括[@ hyperapp / HTML](https://github.com/hyperapp/html),[骇客神条](https://github.com/substack/hyperx),[t7](https://github.com/trueadm/t7)和[ijk](https://github.com/lukejacksonn/ijk). 

## 安装

使用npm或Yarn安装. 

<pre>
npm i <a href=https://www.npmjs.com/package/hyperapp>hyperapp</a>
</pre>

然后用模块捆绑器之类的[卷起](https://rollupjs.org)要么[的WebPack](https://webpack.js.org),像使用其他任何东西一样使用. 

```js
import { h, app } from "hyperapp"
```

如果您不想设置构建环境,可以从CDN下载Hyperapp[unpkg.com](https://unpkg.com/hyperapp)它将通过全球可用<samp>window.hyperapp</samp>目的. 我们支持所有符合ES5标准的浏览器,包括Internet Explorer 10及更高版本. 

```html
<script src="https://unpkg.com/hyperapp"></script>
```

## 概观

Hyperapp应用程序由三个相互关联的部分组成: [州](#state),[视图](#view),和[行动](#actions). 

初始化后,您的应用程序将在连续循环中执行,从用户或外部事件接收操作,更新状态,并通过虚拟DOM模型表示视图中的更改. 将操作视为通知Hyperapp更新状态并安排下一个视图重绘的信号. 在处理动作之后,将新状态呈现给用户. 

### 州

状态是一个简单的JavaScript对象,用于描述整个程序. 它包含在执行期间在应用程序中移动的所有动态数据. 一旦创建状态,就不能对状态进行变异. 我们必须使用操作来更新它. 

```js
const state = {
  count: 0
}
```

与任何JavaScript对象一样,状态可以是嵌套的对象树. 我们将状态中的嵌套对象称为部分状态. 单个状态树不会与模块化冲突 - 请参阅[嵌套操作](#nested-actions)了解如何更新深层嵌套对象并拆分状态和操作. 

```js
const state = {
  top: {
    count: 0
  },
  bottom: {
    count: 0
  }
}
```

因为Hyperapp在更新状态时执行浅合并,顶级状态必须是普通的JavaScript对象,除此之外,您可以使用任何类型,包括数组,ES6[地图](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map),[集](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set),[Immutable.js](https://facebook.github.io/immutable-js/)结构等

### 操作

改变状态的唯一方法是通过行动. 动作是期望有效载荷的一元函数 (接受单个参数) . 有效负载可以是您想要传递给操作的任何内容. 

要更新状态,操作必须返回部分状态对象. 新状态将是此对象与当前状态之间浅层合并的结果. 在引擎盖下,Hyperapp连接您的操作中的每个功能,以便在状态发生变化时安排视图重绘. 

```js
const actions = {
  setValue: value => ({ value })
}
```

操作可以返回一个函数,该函数将当前状态和操作作为参数返回并返回部分状态对象,而不是直接返回部分状态对象. 

```js
const actions = {
  down: value => state => ({ count: state.count - value }),
  up: value => state => ({ count: state.count + value })
}
```

状态更新始终是不可变的. 不要在动作中改变状态对象参数并返回它 - 结果不是您所期望的 (例如,视图不会被重绘) . 

不变性使得时间旅行调试成为可能,通过使状态变化更加可预测,有助于防止引入难以追踪的错误,并允许使用浅层平等对组件进行廉价的记忆<samp>===</samp>检查. 

#### 异步动作

用于副作用的操作 (写入数据库,向服务器发送请求等) 不需要具有返回值. 您可以从另一个操作或回调函数中调用操作. 返回Promise的行为,<samp>未定义</samp>要么<samp>空值</samp>不会触发重绘或更新状态. 

```js
const actions = {
  upLater: value => (state, actions) => {
    setTimeout(actions.up, 1000, value)
  },
  up: value => state => ({ count: state.count + value })
}
```

一个动作可以是一个<samp>[异步](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function)</samp>功能. 因为<samp>异步</samp>函数返回一个Promise,而不是一个部分状态对象,需要调用另一个动作才能更新状态. 

```js
const actions = {
  upLater: () => async (state, actions) => {
    await new Promise(done => setTimeout(done, 1000))
    actions.up(10)
  },
  up: value => state => ({ count: state.count + value })
}
```

#### 嵌套操作

动作可以嵌套在命名空间内. 更新深度嵌套状态就像在与要更新的状态部分相同的路径中声明对象内部的操作一样简单. 

```jsx
const state = {
  counter: {
    count: 0
  }
}

const actions = {
  counter: {
    down: value => state => ({ count: state.count - value }),
    up: value => state => ({ count: state.count + value })
  }
}
```

#### 互通性

该<samp>应用</samp>函数返回您的操作的副本,其中每个函数都连接到状态的更改. 将此对象暴露给外部世界对于从其他程序或框架操作应用程序,订阅全局事件,监听鼠标和键盘输入等非常有用. 

要查看此操作,请修改示例[入门](#getting-started)将有线操作保存到变量并尝试使用它们. 您应该相应地看到计数器更新. 

```jsx
const main = app(state, actions, view, document.body)

setInterval(main.up, 250, 1)
setInterval(main.down, 500, 1)
```

包含返回state参数的动作可能很有用. 由于状态更新始终是不可变的,因此返回对当前状态的引用将不会安排视图重绘. 

```jsx
const actions = {
  getState: () => state => state
}
```

### 视图

每次应用程序状态更改时,都会调用视图函数,以便您可以根据新状态指定DOM的外观. 视图以称为虚拟DOM的纯JavaScript对象的形式返回您的规范,Hyperapp负责更新实际DOM以匹配它. 

```js
import { h } from "hyperapp"

export const view = (state, actions) =>
  h("div", {}, [
    h("h1", {}, state.count),
    h("button", { onclick: () => actions.down(1) }, "-"),
    h("button", { onclick: () => actions.up(1) }, "+")
  ])
```

虚拟DOM是使用称为虚拟节点的嵌套JavaScript对象树描述DOM的外观. 可以将其视为DOM的轻量级表示. 在该示例中,view函数返回这样的对象. 

```jsx
{
  nodeName: "div",
  attributes: {},
  children: [
    {
      nodeName: "h1",
      attributes: {},
      children: [0]
    },
    {
      nodeName: "button",
      attributes: { ... },
      children: ["-"]
    },
    {
      nodeName:   "button",
      attributes: { ... },
      children: ["+"]
    }
  ]
}
```

虚拟DOM模型允许我们编写代码,好像整个文档被丢弃并在每次更改时重建,而我们只更新实际更改的内容. 我们尝试通过将新虚拟DOM与前一个虚拟DOM进行比较,以尽可能少的步骤执行此操作. 这导致高效率,因为通常只有一小部分节点需要改变,并且与重新计算虚拟DOM相比,改变真实DOM节点是昂贵的. 

抛弃旧的虚拟DOM并在每次更新时完全重新创建它可能看起来很浪费`-`更不用说在任何时候,Hyperapp都会在内存中保留两个虚拟DOM树,但事实证明,浏览器可以非常快速地创建数十万个对象. 另一方面,修改DOM要贵几个数量级. 

### 安装

要在页面中安装应用程序,我们需要一个DOM元素. 此元素称为应用程序容器. 使用Hyperapp构建的应用程序始终只有一个容器. 

```jsx
app(state, actions, view, container)
```

Hyperapp还将尝试重用容器内的现有元素,从而实现SEO优化并改善您的网站交互时间. 该过程包括与您的应用程序一起提供完全呈现的页面. 然后,我们将您的DOM节点转换为开箱即用的交互式应用程序,而不是丢弃现有内容. 

这就是我们如何从之前的计数器示例中回收服务器呈现的内容. 看到[入门](#getting-started)对于应用程序代码. 

```html
<!doctype html>
<html>
<head>
  <meta charset="utf-8">
  <script defer src="bundle.js"></script>
</head>

<body>
  <div>
    <h1>0</h1>
    <button>-</button>
    <button>+</button>
  </div>
</body>
</html>
```

### 组件

组件是返回虚拟节点的纯函数. 与视图功能不同,组件未连接到您的应用程序状态或操作. 组件是愚蠢的,可重用的代码块,它们封装了属于一起的标记,样式和行为. 

这里介绍了如何在应用程序中使用组件. 该应用程序是典型的To-Do管理器. 来吧[在这里试试吧](https://codepen.io/hyperapp/pen/zNxRLy). 

```jsx
import { h } from "hyperapp"

const TodoItem = ({ id, value, done, toggle }) => (
  <li
    class={done && "done"}
    onclick={() =>
      toggle({
        value: done,
        id: id
      })
    }
  >
    {value}
  </li>
)

export const view = (state, actions) => (
  <div>
    <h1>Todo</h1>
    <ul>
      {state.todos.map(({ id, value, done }) => (
        <TodoItem id={id} value={value} done={done} toggle={actions.toggle} />
      ))}
    </ul>
  </div>
)
```

如果您不知道要提前放入组件的所有属性,可以使用[传播语法](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_operator). 请注意,Hyperapp组件可以返回元素数组,如以下示例所示. 此技术允许您对子项列表进行分组,而无需向DOM添加额外节点. 

```jsx
const TodoList = ({ todos, toggle }) =>
  todos.map(todo => <TodoItem {...todo} toggle={toggle} />)
```

#### 懒惰组件

组件只能从其父组件接收属性和子项. 与顶级视图函数类似,惰性组件将传递给您的应用程序全局状态和操作. 要创建惰性组件,请从常规组件返回视图函数. 

```jsx
import { h } from "hyperapp"

export const Up = ({ by }) => (state, actions) => (
  <button onclick={() => actions.up(by)}>+ {by}</button>
)

export const Down = ({ by }) => (state, actions) => (
  <button onclick={() => actions.down(by)}>- {by}</button>
)

export const Double = () => (state, actions) => (
  <button onclick={() => actions.up(state.count)}>+ {state.count}</button>
)

export const view = (state, actions) => (
  <main>
    <h1>{state.count}</h1>
    <Up by={2} />
    <Down by={1} />
    <Double />
  </main>
)
```

#### 儿童组成

组件通过第二个参数接收子元素,允许您和其他组件将任意子组件传递给它们. 

```jsx
const Box = ({ color }, children) => (
  <div class={`box box-${color}`}>{children}</div>
)

const HelloBox = ({ name }) => (
  <Box color="green">
    <h1 class="title">Hello, {name}!</h1>
  </Box>
)
```

## 支持的属性

支持的属性包括[HTML属性](https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes),[SVG属性](https://developer.mozilla.org/en-US/docs/Web/SVG/Attribute),[DOM事件](https://developer.mozilla.org/en-US/docs/Web/Events),[生命周期事件](#lifecycle-events),和[按键](#keys). 请注意,不支持非标准HTML属性名称,<samp>的onclick</samp>和<samp>类</samp>是有效的,但是<samp>的onClick</samp>要么<samp>班级名称</samp>不是. 

### 样式

该<samp>样式</samp>attribute需要普通对象而不是HTML中的字符串. 每个声明都包含一个用于编写的样式名称属性<samp>骆驼香烟盒</samp>和一个价值. 也支持CSS变量. 

将对各个样式属性进行区分和映射<samp>[HTMLElement.style](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/style)</samp>DOM元素的属性成员 - 因此您应该使用JavaScript样式对象[财产名称](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Properties_Reference),例如<samp>背景颜色</samp>而不是<samp>背景颜色</samp>. 

```jsx
import { h } from "hyperapp"

export const Jumbotron = ({ text }) => (
  <div
    style={{
      color: "white",
      fontSize: "32px",
      textAlign: center,
      backgroundImage: `url(${imgUrl})`
    }}
  >
    {text}
  </div>
)
```

### 生命周期事件

当通过生命周期事件创建,更新或删除虚拟DOM管理的元素时,可以通知您. 将它们用于动画,数据获取,包装第三方库,清理资源等. 

请注意,生命周期事件附加到虚拟DOM节点而不是组件. 考虑添加一个键以确保事件附加到特定的DOM元素,而不是循环的元素. 

#### 在OnCreate

创建元素并将其附加到DOM后会触发此事件. 使用它直接操作DOM节点,发出网络请求,创建幻灯片/淡入淡出动画等. 

```jsx
import { h } from "hyperapp"

export const Textbox = ({ placeholder }) => (
  <input
    type="text"
    placeholder={placeholder}
    oncreate={element => element.focus()}
  />
)
```

#### 的onupdate

每次更新元素属性时都会触发此事件. 使用<samp>oldAttributes</samp>在事件处理程序内部检查是否有任何属性发生了变化. 

```jsx
import { h } from "hyperapp"

export const Textbox = ({ placeholder }) => (
  <input
    type="text"
    placeholder={placeholder}
    onupdate={(element, oldAttributes) => {
      if (oldAttributes.placeholder !== placeholder) {
        // Handle changes here!
      }
    }}
  />
)
```

#### onremove

在从DOM中删除元素之前触发此事件. 用它来创建幻灯片/淡出动画. 呼叫<samp>done</samp>在函数内部删除元素. 不会在其子元素中调用此事件. 

```jsx
import { h } from "hyperapp"

export const MessageWithFadeout = ({ title }) => (
  <div onremove={(element, done) => fadeout(element).then(done)}>
    <h1>{title}</h1>
  </div>
)
```

#### 的OnDestroy

在从DOM中删除元素后直接或由于父项被删除而触发此事件. 用它来使计时器无效,取消网络请求,删除全局事件监听器等. 

```jsx
import { h } from "hyperapp"

export const Camera = ({ onerror }) => (
  <video
    poster="loading.png"
    oncreate={element => {
      navigator.mediaDevices
        .getUserMedia({ video: true })
        .then(stream => (element.srcObject = stream))
        .catch(onerror)
    }}
    ondestroy={element => element.srcObject.getTracks()[0].stop()}
  />
)
```

### 按键

每次更新DOM时,Keys都有助于识别节点. 通过设置<samp>键</samp>在虚拟节点上,您声明该节点应该对应于特定的DOM元素. 如果位置发生变化,这允许我们将元素重新排序到新位置,而不是冒险破坏它. 

```jsx
import { h } from "hyperapp"

export const ImageGallery = ({ images }) =>
  images.map(({ hash, url, description }) => (
    <li key={hash}>
      <img src={url} alt={description} />
    </li>
  ))
```

键必须在兄弟节点中是唯一的. 如果索引还指定了兄弟节点的顺序,请不要将数组索引用作键. 如果列表中项目的位置和数量是固定的,则没有区别,但如果列表是动态的,则每次重建树时密钥都会更改. 

```jsx
import { h } from "hyperapp"

export const PlayerList = ({ players }) =>
  players
    .slice()
    .sort((player, nextPlayer) => nextPlayer.score - player.score)
    .map(player => (
      <li key={player.username} class={player.isAlive ? "alive" : "dead"}>
        <PlayerProfile {...player} />
      </li>
    ))
```

#### 顶级节点

密钥未在视图的顶级节点上注册. 如果要切换顶级视图,并且必须使用密钥,请将它们包装在不变的节点中. 

## 链接

-   [松弛](https://hyperappjs.herokuapp.com)
-   [推特](https://twitter.com/hyperappJS)
-   [例子](https://codepen.io/search/pens/?q=hyperapp)
-   [/ R / hyperapp](https://www.reddit.com/r/hyperapp)

## 执照

Hyperapp是MIT许可的. 看到[执照](LICENSE.md). 
