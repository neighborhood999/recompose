Recompose
-----

[![build status](https://img.shields.io/travis/acdlite/recompose/master.svg?style=flat-square)](https://travis-ci.org/acdlite/recompose)
[![coverage](https://img.shields.io/codecov/c/github/acdlite/recompose.svg?style=flat-square)](https://codecov.io/github/acdlite/recompose)
[![code climate](https://img.shields.io/codeclimate/github/acdlite/recompose.svg?style=flat-square)](https://codeclimate.com/github/acdlite/recompose)
[![npm version](https://img.shields.io/npm/v/recompose.svg?style=flat-square)](https://www.npmjs.com/package/recompose)
[![npm downloads](https://img.shields.io/npm/dm/recompose.svg?style=flat-square)](https://www.npmjs.com/package/recompose)

Recompose 是一個 React utility 用於 function component 和 higher-order component。可以把它想為像是給 React 使用的 lodash。

[**完整 API 文件**](docs/API.md) - 了解每個 helper

[**Recompose Base Fiddle**](https://jsfiddle.net/samsch/p3vsmrvo/24/) - 深入淺出 Recompose

```
npm install recompose --save
```

**📺 觀看 Andrew [在 React Europe 上的 Recompose talk](https://www.youtube.com/watch?v=zD_judE-bXk)。**  
*(注意：Andrew 在 React Europe 上的 Recompose talk 被移除了，更多資訊請參考[這裡](https://github.com/acdlite/recompose/releases/tag/v0.26.0))*

### 相關 module

[**recompose-relay**](src/packages/recompose-relay) — Relay 的 Recompose helper

## 你可以使用 Recompose 在...

### ...提升 state 到 functional wrapper

像是 `withState()` 和 `withReducer()` helper 提供一個很好的方式來表達 state 的更新：

```js
const enhance = withState('counter', 'setCounter', 0)
const Counter = enhance(({ counter, setCounter }) =>
  <div>
    Count: {counter}
    <button onClick={() => setCounter(n => n + 1)}>Increment</button>
    <button onClick={() => setCounter(n => n - 1)}>Decrement</button>
  </div>
)
```

或者是一個 Redux 風格的 reducer：

```js
const counterReducer = (count, action) => {
  switch (action.type) {
  case INCREMENT:
    return count + 1
  case DECREMENT:
    return count - 1
  default:
    return count
  }
}

const enhance = withReducer('counter', 'dispatch', counterReducer, 0)
const Counter = enhance(({ counter, dispatch }) =>
  <div>
    Count: {counter}
    <button onClick={() => dispatch({ type: INCREMENT })}>Increment</button>
    <button onClick={() => dispatch({ type: DECREMENT })}>Decrement</button>
  </div>
)
```

### ...執行大部分 React 常見的 pattern

像是 `componentFromProp()` 和 `withContext()` helper 封裝常見的 React pattern 到一個簡單的 functional interface：

```js
const enhance = defaultProps({ component: 'button' })
const Button = enhance(componentFromProp('component'))

<Button /> // renders <button>
<Button component={Link} /> // renders <Link />
```

```js
const provide = store => withContext(
  { store: PropTypes.object },
  () => ({ store })
)

// Apply 到 base component
// App 的子節點可以存取到 context.store
const AppWithContext = provide(store)(App)
```

### ...優化 render 效能

不需要轉移寫一個新的 class 來實作 `shouldComponentUpdate()`。像是 `pure()` 和 `onlyUpdateForKeys()` 的 Recompose helper 會幫你完成：

```js
// 一個 render 成本很高的 component
const ExpensiveComponent = ({ propA, propB }) => {...}

// 相同的 component 的優化版本，使用 props 的 shallow comparison
// 效果相同於 extend React.PureComponent
const OptimizedComponent = pure(ExpensiveComponent)

// 更多的優化：如果指定的 props key 改變了才做更新
const HyperOptimizedComponent = onlyUpdateForKeys(['propA', 'propB'])(ExpensiveComponent)
```

### ...與其他的 library 相互操作

Recompose helper 整合了非常棒的外部 library。像是 Relay、Redux 和 RxJS

```js
const enhance = compose(
  // 由 recompose-relay 所提供，這是 Recompose 版本的 Relay.createContainer()
  createContainer({
    fragments: {
      post: () => Relay.QL`
        fragment on Post {
          title,
          content
        }
      `
    }
  }),
  flattenProp('post')
)

const Post = enhance(({ title, content }) =>
  <article>
    <h1>{title}</h1>
    <div>{content}</div>
  </article>
)
```

### ...建立你自己的 library

許多 React library 重複實作了相同 utility，像是 `shallowEqual()` 和 `getDisplayName()`。Recompose 也提供了這些 utility 給你使用。

```js
// 任何 Recompose module 可以被獨立的被 import
import getDisplayName from 'recompose/getDisplayName'
ConnectedComponent.displayName = `connect(${getDisplayName(BaseComponent)})`

// 或是甚至更好的：
import wrapDisplayName from 'recompose/wrapDisplayName'
ConnectedComponent.displayName = wrapDisplayName(BaseComponent, 'connect')

import toClass from 'recompose/toClass'
// 轉換一個 function component 成為一個 class component，例如，它可以給定一個 ref，
// 回傳 class component。
const ClassComponent = toClass(FunctionComponent)
```

### ...更多

## API 文件

[閱讀文件](docs/API.md)

## 支援 Flow

[閱讀文件](types)

## Translation

[Traditional Chinese](https://github.com/neighborhood999/recompose)

## Why

忘了 ES6 class 和 `createClass()` 吧！

React 應用程式主要慣用 function component 組合而成。

```js
const Greeting = props =>
  <p>
    Hello, {props.name}!
  </p>
```

Function component 有許多關鍵優勢：

- 它們可以防止濫用 `setState()` API，以 props 作為替代。
- 它們鼓勵 [「smart」 和 「dumb」 component pattern](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0)。
- 它們鼓勵程式碼應該可以有更多的複用性和模組化。
- 它們阻止 component 的增長，變得複雜且負責太多的職責。
- 將來，他們將允許 React 透過不必要的檢查和 memory 分配來做 performance 的優化。

（注意！雖然 Recompose 鼓勵盡可能使用 function component 操作，但它還是可以和正常的 React components 相互工作。）

### Higher-order component 讓一切更簡單

大部分我們在 React 談論關於 composition，都是關於 component 的 composition。例如，一個  `<Blog>` 可能有多個 `<Post>` component 組合而成，它們由許多 `<Comment>` component 組合而成。

Recompose 專注在另一個 composition 單元：**higher-order components** （HoCs）。HoCs 是 function，接受一個 base component 並回傳一個附加新功能的 component。它們可用於將常見的抽象任務成為一個可複用的部分。

Recompose 提供一個 helper function 的工具包來建立 higher-order component。

## [我應該使用 Recompose 嗎？Performance 和其他問題的擔憂](docs/performance.md)

## 使用方法

所有 function 在頂層被 export 後可以使用。

```js
import { compose, mapProps, withState /* ... */ } from 'recompose'
```

**注意：** Recompose 的_對等依賴（peer dependency）_ 是 `react`。如果你使用 `preact`，新增以下到你的 `webpack.config.js`：

```js
resolve: {
  alias: {
    react: "preact"
  }
}
```

### Composition

Recompose helper 被設計為可以被組合的：

```js
const BaseComponent = props => {...}

// 這是可以使用的，但是有點乏味
let EnhancedComponent = pure(BaseComponent)
EnhancedComponent = mapProps(/*...args*/)(EnhancedComponent)
EnhancedComponent = withState(/*...args*/)(EnhancedComponent)

// 取而代之
// 注意！排序將會被反轉 - props 的 flow 是由頂部到底部
const enhance = compose(
  withState(/*...args*/),
  mapProps(/*...args*/),
  pure
)
const EnhancedComponent = enhance(BaseComponent)
```

技術上，這也意味你可以使用他們作為 decorators（取自於你的決定）：

```js
@withState(/*...args*/)
@mapProps(/*...args*/)
@pure
class Component extends React.Component {...}
```

### 優化 bundle size

由於 `0.23.1` 版本的 recompose 得到了 ES2015 module 的支援。
要減少 bundle 大小，你需要使用支援 tree shaking 的 bundler 像是 [webpack 2](https://github.com/webpack/webpack) 或 [Rollup](https://github.com/rollup/rollup)。

#### 使用 babel-plugin-lodash

[babel-plugin-lodash](https://github.com/lodash/babel-plugin-lodash) 不僅限於 [lodash](https://github.com/lodash/lodash)。它也可以和 `recompose` 一起使用。

可以透過在 `.babelrc` 來更新 `lodash` 完成設定。

```diff
 {
-  "plugins": ["lodash"]
+  "plugins": [
+    ["lodash", { "id": ["lodash", "recompose"] }]
+  ]
 }
```

之後，你可以像以下 import 需要的部分，而不需實際 import 整個 libray。

```js
import { compose, mapProps, withState } from 'recompose'
```

### Debugging

如何在 HOC 之間追蹤 `props` 可能非常的困難。一個有用的技巧是你可以建立一個 debug HOC 來列印出 props，它不會修改 base component。

建立：

```js
const debug = withProps(console.log)
```

然後在 HOC 之間使用它

```js
const enhance = compose(
  withState(/*...args*/),
  debug, // 在這裡列印出 props
  mapProps(/*...args*/),
  pure
)
```


## 誰在使用 Recompose
如果你的公司或是專案使用 Recompose，請透過[編輯](https://github.com/acdlite/recompose/wiki/Sites-Using-Recompose/_edit) wiki 頁面自行新增到[官方用戶名單](https://github.com/acdlite/recompose/wiki/Sites-Using-Recompose)。

## Recipes 的靈感
我們有一個 community-driven Recipes 的頁面。它是一個分享和看到 recompose pattern 靈感的地方。請新增到 [Recipes](https://github.com/acdlite/recompose/wiki/Recipes)！

## 需要 Feedback

Project 處於在早期的階段。如果你有任何建議，請提出 issue 或是送出 PR！或者在 [Twitter](https://twitter.com/acdlite)（Andrew Clark）上聯繫我。


## 取得幫助

**對於像是「我該如何在使用 X 與 Recompose」或「我的程式碼不能執行」之類的使用問題，請優先在 [StackOverflow 使用 Recompose tag](http://stackoverflow.com/questions/tagged/recompose?sort=votes&pageSize=50) 搜尋並提問。**

我們請你這麼做是因為 StackOverflow 可以讓更常見的問題可以被看見。不幸的是，好的答案可能在 GitHub 上丟失並過時。

有一些問題需要很長時間才能得到答案。**如果你的問題被關閉，或是你在很長的時間沒辦法在 StackOverflow 得到回覆，** 我們鼓勵你貼上你的 issue 並連結到你的問題。我們將關閉你的 issue，但是這將讓其他人有機會看到在個問題並在 StackOverflow 上回覆。

請體諒我們這麼做, 因為這不是 issue tracker 的主要目的。

### 幫助我們也幫助你

在這兩個網站上，用一種容易閱讀的方式來結構化你的程式碼和問題，來吸引他人來回答，這是一個很好的辦法。例如，我們鼓勵你使用語法高亮、縮排和分割段落。

請記住，其他人花了時間來嘗試幫助你，如果你可以提供相關的 library 版本並執行一個小的 project 來重現你的問題，可以讓這一切變得容易。你可以將你的程式碼放在 [JSBin](http://jsbin.com) 或者是較大的 project 在 GitHub。確保所有必要 dependency 都被宣告在 `package.json`，任何人執行 `npm install && npm start` 都可以重現問題。
