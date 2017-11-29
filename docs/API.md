# API

在這份 API 文件中，一個 **higher-order component** (HOC) 指的是一個 function 接受一個單一的 React component 並回傳一個新的 React component。

```js
const EnhancedComponent = hoc(BaseComponent)
```

這種形式讓 HOC（有時候稱為 **enhancers**）可以被組合：

```js
const composedHoc = compose(hoc1, hoc2, hoc3)

// 相同於
const composedHoc = BaseComponent => hoc1(hoc2(hoc3(BaseComponent)))
```

大部分的 Recompose helper 都是 **function，並回傳 higher-order component**：

```js
const hoc = mapProps(ownerProps => childProps)
const EnhancedComponent = hoc(BaseComponent)

// 相同於
const EnhancedComponent = mapProps(ownerProps => childProps)(BaseComponent)
```

有一些像是 `pure` 本身就是 higher-order component：

```js
const PureComponent = pure(BaseComponent)
```

## TOC

* [Higher-order components](#higher-order-components)
  + [`mapProps()`](#mapprops)
  + [`withProps()`](#withprops)
  + [`withPropsOnChange()`](#withpropsonchange)
  + [`withHandlers()`](#withhandlers)
  + [`defaultProps()`](#defaultprops)
  + [`renameProp()`](#renameprop)
  + [`renameProps()`](#renameprops)
  + [`flattenProp()`](#flattenprop)
  + [`withState()`](#withstate)
  + [`withStateHandlers()`](#withstatehandlers)
  + [`withReducer()`](#withreducer)
  + [`branch()`](#branch)
  + [`renderComponent()`](#rendercomponent)
  + [`renderNothing()`](#rendernothing)
  + [`shouldUpdate()`](#shouldupdate)
  + [`pure()`](#pure)
  + [`onlyUpdateForKeys()`](#onlyupdateforkeys)
  + [`onlyUpdateForPropTypes()`](#onlyupdateforproptypes)
  + [`withContext()`](#withcontext)
  + [`getContext()`](#getcontext)
  + [`lifecycle()`](#lifecycle)
  + [`toClass()`](#toclass)
* [Static property helpers](#static-property-helpers)
  + [`setStatic()`](#setstatic)
  + [`setPropTypes()`](#setproptypes)
  + [`setDisplayName()`](#setdisplayname)
* [Utilities](#utilities)
  + [`compose()`](#compose)
  + [`getDisplayName()`](#getdisplayname)
  + [`wrapDisplayName()`](#wrapdisplayname)
  + [`shallowEqual()`](#shallowequal)
  + [`isClassComponent()`](#isclasscomponent)
  + [`createSink()`](#createsink)
  + [`componentFromProp()`](#componentfromprop)
  + [`nest()`](#nest)
  + [`hoistStatics()`](#hoiststatics)
* [Observable utilities](#observable-utilities)
  + [`componentFromStream()`](#componentfromstream)
  + [`componentFromStreamWithConfig()`](#componentfromstreamwithconfig)
  + [`mapPropsStream()`](#mappropsstream)
  + [`mapPropsStreamWithConfig()`](#mappropsstreamwithconfig)
  + [`createEventHandler()`](#createeventhandler)
  + [`createEventHandlerWithConfig()`](#createeventhandlerwithconfig)
  + [`setObservableConfig()`](#setobservableconfig)

## Higher-order components

### `mapProps()`

```js
mapProps(
  propsMapper: (ownerProps: Object) => Object,
): HigherOrderComponent
```

接受一個 function 將所有 props map 到 base component 成為一個新的 props 集合。

`mapProps()` 和 functional 函式庫像是 [lodash/fp](https://github.com/lodash/lodash/tree/npm/fp) 可以做很好的搭配。例如，Recompose 沒有一個 `omitProps() function`，但你可以使用 loadsh-fp 的 `omit()` 輕鬆地建立一個：

```js
const omitProps = keys => mapProps(props => omit(keys, props))

// 因為 loadsh-fp 的 curry 化，這相同於
const omitProps = compose(mapProps, omit)
```

### `withProps()`

```js
withProps(
  createProps: (ownerProps: Object) => Object | Object
): HigherOrderComponent
```

除了建立新的 props 被 merge 到 owner props，相似於 `mapProps()`。

你也可以直接傳送一個 props object 而不是 function。在這個形式下，除了被提供的 props 優先於 owner 的 props，它類似於 `defaultProps()`。


### `withPropsOnChange()`

```js
withPropsOnChange(
  shouldMapOrKeys: Array<string> | (props: Object, nextProps: Object) => boolean,
  createProps: (ownerProps: Object) => Object
): HigherOrderComponent
```

類似 `withProps()`，當其中一個 owner prop 透過 `shouldMapOrKeys` 被指定時，僅建立新的 prop。這可以確保不必要的昂貴計算，當必要時才會在內部執行 `createProps()`。

除了一個 prop keys 的陣列，第一個參數也可以是一個回傳布林值的 function，給定目前的 props 和 next props。當 `createProps()` 應該被呼叫時，允許你自訂 props。

### `withHandlers()`

```js
withHandlers(
  handlerCreators: {
    [handlerName: string]: (props: Object) => Function
  } |
  handlerCreatorsFactory: (initialProps) => {
    [handlerName: string]: (props: Object) => Function
  }
): HigherOrderComponent
```

接受 handler creator 或是 factory function 的 object map。這些都是 higher-order function，接受一組 props 並回傳一個 function handler：

允許 handler 經由 closure 去存取目前的 props，而不需要去改變它的 signature。

Handler 作為 immutable props 被傳入到 base component，它們的身份通過 render 被保留。這避免在一個 functional component 在內部建立 handler 的陷阱，導致每次 render 都會有新的 handler，打破了依賴於 prop 相等的 `shouldComponentUpdate()` 優化。這是使用 `withHandlers` 主要的理由來建立 handler，而不是使用 `mapProps` 或 `withProps` 在每次 update 時建立新的 handler。

使用範例：

```js
const enhance = compose(
  withState('value', 'updateValue', ''),
  withHandlers({
    onChange: props => event => {
      props.updateValue(event.target.value)
    },
    onSubmit: props => event => {
      event.preventDefault()
      submitForm(props.value)
    }
  })
)

const Form = enhance(({ value, onChange, onSubmit }) =>
  <form onSubmit={onSubmit}>
    <label>Value
      <input type="text" value={value} onChange={onChange} />
    </label>
  </form>
)
```

### `defaultProps()`

```js
defaultProps(
  props: Object
): HigherOrderComponent
```

指定預設情況下傳送給 base component 的 props。類似於 `withProps()`，除了 owner props 優先於被提供的 props。

雖然它有類似的效果，使用 `defaultProps()` HoC *不* 等同於在 component 上直接設定 static `defaultProps` 屬性。


### `renameProp()`

```js
renameProp(
  oldName: string,
  newName: string
): HigherOrderComponent
```

重新命名一個 prop。

### `renameProps()`

```js
renameProps(
  nameMap: { [key: string]: string }
): HigherOrderComponent
```

重新命名多個 prop，使用一個 map 將舊的 prop 名稱更新為新的 prop 名稱。

### `flattenProp()`

```js
flattenProp(
  propName: string
): HigherOrderComponent
```

扁平化一個 prop，所以它的欄位會被展開（spread）到 props object。

```js
const enhance = compose(
  withProps({
    object: { a: 'a', b: 'b' },
    c: 'c'
  }),
  flattenProp('object')
)
const Abc = enhance(BaseComponent)

// Base component 接收到的 props： { a: 'a', b: 'b', c: 'c', object: { a: 'a', b: 'b' } }
```

另一個使用範例是當 `flattenProp()` 從 Relay 接收 fragment data。Relay fragment 被作為一個 props object 傳遞，你經常需要扁平化成它的組成欄位：

```js
// The `post` prop 是一個 object，有 title、author 和 content 欄位
const enhance = flattenProp('post')
const Post = enhance(({ title, content, author }) =>
  <article>
    <h1>{title}</h1>
    <h2>By {author.name}</h2>
    <div>{content}</div>
  </article>
)
```

### `withState()`

```js
withState(
  stateName: string,
  stateUpdaterName: string,
  initialState: any | (props: Object) => any
): HigherOrderComponent
```

傳送兩個額外的 props 到 base component：一個 state 和一個 function 來更新 state 的值。state updateer 具有以下的 signature：

```js
stateUpdater<T>((prevValue: T) => T, ?callback: Function): void
stateUpdater(newValue: any, ?callback: Function): void
```

第一個形式接受一個 function，它 map 先前的 state 成為一個新的 state。你可能想要隨著 `withHandlers()` 使用這個 state updater 來建立具體的 updater function。例如，要建立一個基礎計數功能的 Higher-Order Component：

```js
const addCounting = compose(
  withState('counter', 'setCounter', 0),
  withHandlers({
    increment: ({ setCounter }) => () => setCounter(n => n + 1),
    decrement: ({ setCounter }) => () =>  setCounter(n => n - 1),
    reset: ({ setCounter }) => () => setCounter(0)
  })
)
```

第二個形式接受一個單一的值，作為新的 state。

兩種形式都接受可選的第二個參數，一旦 `setState()` 完成且 component 重新 render 將會執行一個 callback function。

初始的 state 是必須的。它可以是 state 值本身，或是回傳一個 function 給定初始 props 的 state。

### `withStateHandlers()`

```js
withStateHandlers(
  initialState: Object | (props: Object) => any,
  stateUpdaters: {
    [key: string]: (state:Object, props:Object) => (...payload: any[]) => Object
  }
)

```

傳送 state object 屬性和 `(...payload: any[]) => Object` 形式的 immutable updater function 到 base component。

每個 state updater function 接受 state、props 和 payload 並必須回傳一個新的 state 或是 undefined。新的 state 只是與先前的 state 被 shallow merge。
回傳 undefined component 不會 render 出來。

範例：

```js
  const Counter = withStateHandlers(
    ({ initialCounter = 0 }) => ({
      counter: initialCounter,
    }),
    {
      incrementOn: ({ counter }) => (value) => ({
        counter: counter + value,
      }),
      decrementOn: ({ counter }) => (value) => ({
        counter: counter - value,
      }),
      resetCounter: (_, { initialCounter = 0 }) => () => ({
        counter: initialCounter,
      }),
    }
  )(
    ({ counter, incrementOn, decrementOn, resetCounter }) =>
      <div>
        <Button onClick={() => incrementOn(2)}>Inc</Button>
        <Button onClick={() => decrementOn(3)}>Dec</Button>
        <Button onClick={resetCounter}>Reset</Button>
      </div>
  )
```

### `withReducer()`

```js
withReducer<S, A>(
  stateName: string,
  dispatchName: string,
  reducer: (state: S, action: A) => S,
  initialState: S | (ownerProps: Object) => S
): HigherOrderComponent
```

類似於 `withState()`，但是使用 reducer function 來更新 state。一個 reducer 是一個 function，接受一個 state 和一個 action，並回傳一個新的 state。

傳送兩個額外的 prop 到 base component：一個 state 的值和一個 dispatch 方法。dispatch 方法傳送一個 action 到 reducer 並回傳計算後的新 state。

### `branch()`

```js
branch(
  test: (props: Object) => boolean,
  left: HigherOrderComponent,
  right: ?HigherOrderComponent
): HigherOrderComponent
```

接受一個 test function 和兩個 higher-order component。test function 從 owner 傳送 props。 如果回傳 true，`left` higher-order component 會被 apply 到 `BaseComponent`；反之是 `right` higher-order component。如果沒有提供 `right`，它將預設為被 wrap 的 component。

### `renderComponent()`

```js
renderComponent(
  Component: ReactClass | ReactFunctionalComponent | string
): HigherOrderComponent
```

得到一個 component 並回傳一個該 component 的 higher-order 版本 component。

這和另一個需要 higher-order component 的 helper 結合非常的有用，像是 `branch()`：

```js
// `isLoading()` 是一個 function，回傳 component 的 state 是否為 loading
const spinnerWhileLoading = isLoading =>
  branch(
    isLoading,
    renderComponent(Spinner) // `Spinner` 是一個 React component
  )

// 現在使用 `spinnerWhileLoading()` helper 來新增一個 loading spinner 到任何的 base component
const enhance = spinnerWhileLoading(
  props => !(props.title && props.author && props.content)
)
const Post = enhance(({ title, author, content }) =>
  <article>
    <h1>{title}</h1>
    <h2>By {author.name}</h2>
    <div>{content}</div>
  </article>
)
```

### `renderNothing()`

```js
renderNothing: HigherOrderComponent
```

一個 higher-order component 總是回傳 `null`。

這對於結合其他的 higher-order component 非常有用，例如 `branch()`：

```js
// `hasNoData()` 是一個 function，如果 component 沒有資料回傳 true
const hideIfNoData = hasNoData =>
  branch(
    hasNoData,
    renderNothing
  )

// 現在使用 `hideIfNoData()` helper 來隱藏任何的 base component
const enhance = hideIfNoData(
  props => !(props.title && props.author && props.content)
)
const Post = enhance(({ title, author, content }) =>
  <article>
    <h1>{title}</h1>
    <h2>By {author.name}</h2>
    <div>{content}</div>
  </article>
)
```

### `shouldUpdate()`

```js
shouldUpdate(
  test: (props: Object, nextProps: Object) => boolean
): HigherOrderComponent
```

Higher-order component 版本的 [`shouldComponentUpdate()`](https://facebook.github.io/react/docs/react-component.html#shouldcomponentupdate)。test function 接受 current props 和 next props 兩者。

### `pure()`

```js
pure: HigherOrderComponent
```

防止 component 更新，除非 prop 被改變。利用 `shallowEqual()` 來測試 prop 的改變。

### `onlyUpdateForKeys()`

```js
onlyUpdateForKeys(
  propKeys: Array<string>
): HigherOrderComponent
```

防止 component 更新，除非指定的 key 所對應的 prop 被更新。使用 `shallowEqual()` 來測試改變。

這對於常見使用的 PureRenderMixin、`shouldPureComponentUpdate()` 或者是 Recompose 本身的 `pure()` helper 方法有更好的優化，因為這些 tool 比較*每個* prop，而 `onlyUpdateForKeys()` 只關心所指定的 prop。

範例：

```js
/**
 * 如果 owner 傳送不必要的 props（例如：一個 comment 的陣列），
 * 它不會導致 render 生命週期的浪費。
 *
 * 這是很好的解構，因為很清楚該 component 實際所關心的 props。
 */
const enhance = onlyUpdateForKeys(['title', 'content', 'author'])
const Post = enhance(({ title, content, author }) =>
  <article>
    <h1>{title}</h1>
    <h2>By {author.name}</h2>
    <div>{content}</div>
  </article>
)
```

### `onlyUpdateForPropTypes()`

```js
onlyUpdateForPropTypes: HigherOrderComponent
```

與 `onlyUpdateForKeys()` 工作方式一樣，但是從 base component 的 `propTypes` 來推斷 prop key。與 `setPropTypes()` 結合非常有用。

如果 base component 沒有任何的 `propTypes`，component 將不會接收到任何的更新。這可能會有不是預期的行為，所以會在 console 出現警告。

```js
import PropTypes from 'prop-types'; // You need to import prop-types. See https://facebook.github.io/react/docs/typechecking-with-proptypes.html

const enhance = compose(
  onlyUpdateForPropTypes,
  setPropTypes({
    title: PropTypes.string.isRequired,
    content: PropTypes.string.isRequired,
    author: PropTypes.object.isRequired
  })
)

const Post = enhance(({ title, content, author }) =>
  <article>
    <h1>{title}</h1>
    <h2>By {author.name}</h2>
    <div>{content}</div>
  </article>
)
```

### `withContext()`

```js
withContext(
  childContextTypes: Object,
  getChildContext: (props: Object) => Object
): HigherOrderComponent
```

提供 context 到 component 的 children。`childContextTypes` 是一個 React prop 類型的 object。 `getChildContext()` 是一個 function，回傳 child 的 context。與 `getContext()` 一起使用。

### `getContext()`

```js
getContext(
  contextTypes: Object
): HigherOrderComponent
```

從 context 取得值並作為 props 傳送。與 `withContext()` 一起使用。

### `lifecycle()`

```js
lifecycle(
  spec: Object,
): HigherOrderComponent
```

一個 [`React.Component()`](https://facebook.github.io/react/docs/react-api.html#react.component) 版本的 higher-order component。它支援完整的 `Component` API，除了 `render()` 方法，透過 default 來被實作（如果被指定 override 的話，錯誤將會被 log 在 console）。在你需要存取 lifecycle 方法的情況下，你應該使用這個 helper 作為一個方案。

透過 `setState` 任何 state 可以在 lifecycle 方法做改變，將 state 作為 props 被 wrap 到 component 被傳遞。

範例：
```js
const PostsList = ({ posts }) => (
  <ul>{posts.map(p => <li>{p.title}</li>)}</ul>
)

const PostsListWithData = lifecycle({
  componentDidMount() {
    fetchPosts().then(posts => {
      this.setState({ posts });
    })
  }
})(PostsList);
```

### `toClass()`

```js
toClass: HigherOrderComponent
```

將一個 function component wrap 到 class。這可以被 library 作為一個 fallback 需要 ref 一個 component，像是 Relay。

如果 base component 已經是 class，它回傳給定的 component。

## Static property helpers

這些 function 看起來像 higher-order component helpers — 這些 component 最後被 curry。然而，不是回傳一個新 component，這些 helper 透過設定或是 override 一個 static 屬性來 mutate base component。

### `setStatic()`

```js
setStatic(
  key: string,
  value: any
): HigherOrderComponent
```

在 base component 分配一個靜態屬性的值。

### `setPropTypes()`

```js
setPropTypes(
  propTypes: Object
): HigherOrderComponent
```

在 base component 分配 `propTypes` 屬性。

### `setDisplayName()`

```js
setDisplayName(
  displayName: string
): HigherOrderComponent
```

在 base component 分配 `displayName` 屬性。

## Utilities

Recompose 也包含一些額外的 helpers，它們不是 higher-order component，但是依然非常有用。

### `compose()`

```js
compose(...functions: Array<Function>): Function
```

compose 將多個 higher-order component 組合為單一的 higher-order component。這與 Redux 中的 compose 功能完全一樣，或是 lodash 的 `flowRight()`。

### `getDisplayName()`

```js
getDisplayName(
  component: ReactClass | ReactFunctionalComponent
): string
```

回傳 React component 的名稱。

### `wrapDisplayName()`

```js
wrapDisplayName(
  component: ReactClass | ReactFunctionalComponent,
  wrapperName: string
): string
```

回傳一個包裝版本名稱的 React component。例如，如果 `component` 的名稱是 `'Post'`，且 `wrapperName` 是 `'mapProps'`，回傳的值是 `'mapProps(Post)'`。大部分的 Recompose higher-order-component 都使用 `wrapDisplayName()`。

### `shallowEqual()`

```js
shallowEqual(a: Object, b: Object): boolean
```

回傳 object 的 shallow equal 的布林值。

### `isClassComponent()`

```js
isClassComponent(value: any): boolean
```

給定的值是否為 React component class，回傳布林值。

### `createSink()`

```js
createSink(callback: (props: Object) => void): ReactClass
```

建立一個不 render 任何東西的 component，但是當接收到新的 props 呼叫一個 callback。

### `componentFromProp()`

```js
componentFromProp(propName: string): ReactFunctionalComponent
```

建立一個接受 component 作為一個 prop 的 component，並 render component 和剩下的 props。

範例：

```js
const enhance = defaultProps({ component: 'button' })
const Button = enhance(componentFromProp('component'))

<Button foo="bar" /> // renders <button foo="bar" />
<Button component="a" foo="bar" />  // renders <a foo="bar" />
<Button component={Link} foo="bar" />  // renders <Link foo="bar" />
```

### `nest()`

```js
nest(
  ...Components: Array<ReactClass | ReactFunctionalComponent | string>
): ReactClass
```

通過巢狀化的內容來組合 component。例如：

```js
// 給定 component A、B 和 C
const ABC = nest(A, B, C)
<ABC pass="through">Child</ABC>

// 效果相同於
<A pass="through">
  <B pass="through">
    <C pass="through">
      Child
    </C>
  </B>
</A>
```

### `hoistStatics()`

```js
hoistStatics(hoc: HigherOrderComponent): HigherOrderComponent
```

增強一個 higher-order component，以便在使用時它複製非 react static 屬性從 base component 到新的 component。這對於當使用 Recompose 和像是 Relay 之類的 library 的時候非常有用。

注意這只 hoist _非 react_ 的 static 屬性。以下的 static 屬性將不會被 hoist：`childContextTypes`、`contextTypes`、`defaultProps`、`displayName`、`getDefaultProps`、`mixins`、`propTypes` 和 `type`。以下原生的 static 方法也會被忽略：`name`、`length`、`prototype`、`caller`、`arguments` 和 `arity`。

## Observable utilities

事實證明大部分的 React Component API 可以用 observable 的方式來表示：

- 合併多個 stream 在一起來代替 `setState()`。
- 使用 `startWith()` 或 `concat()` 來代替 `getInitialState()`。
- 使用 `distinctUntilChanged()`、`debounce()` 等等來代替 `shouldComponentUpdate()`。

其他的好處包含：

- 不需要區別 state 和 props - 所有的東西都是 stream。
- 不需要擔心從事件 listener 取消訂閱。Subscription 會幫你處理。
- Sideways 載入資料是不需要的 - 只需要合併 props stream 與一個外部的 stream。
- Access to an ecosystem of observable libraries, such as RxJS.


**Recompose's observable utilities 可以被設定與其他的 observable 或是 stream-like 的 library 一起使用。參考 [`setObservableConfig()`](#setobservableconfig) 以下詳細資料。**

### `componentFromStream()`

```js
componentFromStream(
  propsToReactNode: (props$: Observable<object>) => Observable<ReactNode>
): ReactComponent
```

透過 map 一個 props 的 observable stream 成一個 React nodes（vdom）stream 來建立一個 React component。

你可以將 `propsToReactNode` 看作為一個 `f` function：

```js
const vdom$ = f(props$)
```

`props$` 是 props 的 stream 且 `vdom$` 是 React nodes 的 stream。這個表達方式類似於 React views 作為一個 function 的流行概念，經常被溝通作為：

```
v = f(d)
```

範例：

```js
const Counter = componentFromStream(props$ => {
  const { handler: increment, stream: increment$ } = createEventHandler()
  const { handler: decrement, stream: decrement$ } = createEventHandler()
  const count$ = Observable.merge(
      increment$.mapTo(1),
      decrement$.mapTo(-1)
    )
    .startWith(0)
    .scan((count, n) => count + n, 0)

  return props$.combineLatest(
    count$,
    (props, count) =>
      <div {...props}>
        Count: {count}
        <button onClick={increment}>+</button>
        <button onClick={decrement}>-</button>
      </div>
  )
})
```

### `componentFromStreamWithConfig()`

```js
componentFromStreamWithConfig<Stream>(
  config: {
    fromESObservable<T>: ?(observable: Observable<T>) => Stream<T>,
    toESObservable<T>: ?(stream: Stream<T>) => Observable<T>,
  }
) => (
  propsToReactNode: (props$: Stream<object>) => Stream<ReactNode>
): ReactComponent
```

替代 `componentFromStream()` 的 helper，它接受一個 observable config，並回傳一個自訂的 `componentFromStream()`，使用指定的 observable library。

**注意：以下的設定模組不包含在主要的 export。你必須獨立 import 它們，如範例所示。**

#### RxJS

```js
import rxjsConfig from 'recompose/rxjsObservableConfig'
const componentFromStream = componentFromStreamWithConfig(rxjsConfig)
```

#### RxJS 4 (legacy)

```js
import rxjs4Config from 'recompose/rxjs4ObservableConfig'
const componentFromStream = componentFromStreamWithConfig(rxjs4Config)
```

#### most

```js
import mostConfig from 'recompose/mostObservableConfig'
const componentFromStream = componentFromStreamWithConfig(mostConfig)
```

#### xstream

```js
import xstreamConfig from 'recompose/xstreamObservableConfig'
const componentFromStream = componentFromStreamWithConfig(xstreamConfig)
```

#### Bacon

```js
import baconConfig from 'recompose/baconObservableConfig'
const componentFromStream = componentFromStreamWithConfig(baconConfig)
```

#### Kefir

```js
import kefirConfig from 'recompose/kefirObservableConfig'
const componentFromStream = componentFromStreamWithConfig(kefirConfig)
```

#### Flyd

```js
import flydConfig from 'recompose/flydObservableConfig'
const componentFromStream = componentFromStreamWithConfig(flydConfig)
```

### `mapPropsStream()`

```js
mapPropsStream(
  ownerPropsToChildProps: (props$: Observable<object>) => Observable<object>,
  BaseComponent: ReactElementType
): ReactComponent
```

`componentFromStream()` 版本的 higher-order component - 接受一個 function，map 一個 owner props 的 observable stream 到一個子 props 的 stream，而不是直接到一個 React nodes 的 stream。然後將子 props 傳送到 base component。

你可能想要使用此版本與其他 Recompose higher-order component helper 進行互相操作。

### `mapPropsStreamWithConfig()`
```js
mapPropsStreamWithConfig<Stream>(
  config: {
    fromESObservable<T>: ?(observable: Observable<T>) => Stream<T>,
    toESObservable<T>: ?(stream: Stream<T>) => Observable<T>,
  },
) => (
  ownerPropsToChildProps: (props$: Stream<object>) => Stream<object>,
  BaseComponent: ReactElementType
): ReactComponent
```

`mapPropsStream()` 的替代方案，接受一個 observable 設定並回傳一個使用指定 observable library 自訂的 `mapPropsStream()`。參考上方的 `componentFromStreamWithConfig()`。

```js
const enhance = mapPropsStream(props$ => {
  const timeElapsed$ = Observable.interval(1000)
  return props$.combineLatest(timeElapsed$, (props, timeElapsed) => ({
    ...props,
    timeElapsed
  }))
})

const Timer = enhance(({ timeElapsed }) =>
  <div>Time elapsed: {timeElapsed}</div>
)
```

### `createEventHandler()`

```js
createEventHandler<T>(): {
  handler: (value: T) => void,
  stream: Observable<T>
}
```

回傳一個有 `handler` 和 `stream` 屬性的 object。`stream` 是一個 observable 的序列，而且 `handler` 是一個 function，push 新的值到序列上。對於建立 event handler 像是 `onClick` 非常的有用。

### `createEventHandlerWithConfig()`
```js
createEventHandlerWithConfig<T>(
  config: {
    fromESObservable<T>: ?(observable: Observable<T>) => Stream<T>,
    toESObservable<T>: ?(stream: Stream<T>) => Observable<T>,
  }
) => (): {
  handler: (value: T) => void,
  stream: Observable<T>
}
```

另一個 `createEventHandler()` 接受一個 observable 的設定，並回傳使用指定 observable library 客製化的 `createEventHandler()`。參考上方 `componentFromStreamWithConfig()`。

### `setObservableConfig()`

```js
setObservableConfig<Stream>({
  fromESObservable<T>: ?(observable: Observable<T>) => Stream<T>,
  toESObservable<T>: ?(stream: Stream<T>) => Observable<T>
})
```

**注意：`setObservableConfig()` 使用 global state，如果打算共享使用內部的 package，可能會造成 app break。參考 `componentFromStreamWithConfig()` 和 `mapPropsStreamWithConfig()` 作為 package author 的替代。**

在 Recompose 的 Observable 是純 object，符合 [ES Observable proposal](https://github.com/zenparsing/es-observable)。通常，你可能會想要與 observable library 像是 RxJS 的一起使用它們。預設上，要求你在 apply 任何轉換前，convert 由 Recompose 提供的 observable。

```js
mapPropsStream(props$ => {
  const rxjsProps$ = Rx.Observable.from(props$)
  // ...現在你可以 map、 filter、scan 等等
  return transformedProps$
})
```

這很快就會變得很乏味。`setObservableConfig()` 設定一個 global observable 轉換讓你可以自動的 apply，而不個別對每個 stream 執行轉換。

```js
import Rx from 'rxjs'
import { setObservableConfig } from 'recompose'

setObservableConfig({
  // 轉換一個純 ES observable 成一個 RxJS 5 observable
  fromESObservable: Rx.Observable.from
})
```

除了 `fromESObservable`，config object 也接受 `toESObservable`，轉換一個 stream 為一個 ES observable。因為 RxJS 5 observable 已經符合 ES observable spec，在上面的範例 `toESObservable` 不是必要的。然而，對於像是 RxJS 4 或 xstream 的 library 是需要的，這些 stream 不符合 ES observable spec。

幸運的是，你不需要擔心對於你喜愛的 stream library 要去如何設定 Recompose，因為 Recompose 為你提供了設定。

**注意：以下的設定 modules 不包含主要的 export。你必須個別的 import 它們，如範例所示。**

#### RxJS

```js
import rxjsconfig from 'recompose/rxjsObservableConfig'
setObservableConfig(rxjsconfig)
```

#### RxJS 4 (legacy)

```js
import rxjs4config from 'recompose/rxjs4ObservableConfig'
setObservableConfig(rxjs4config)
```

#### most

```js
import mostConfig from 'recompose/mostObservableConfig'
setObservableConfig(mostConfig)
```

#### xstream

```js
import xstreamConfig from 'recompose/xstreamObservableConfig'
setObservableConfig(xstreamConfig)
```

#### Bacon

```js
import baconConfig from 'recompose/baconObservableConfig'
setObservableConfig(baconConfig)
```

#### Kefir

```js
import kefirConfig from 'recompose/kefirObservableConfig'
setObservableConfig(kefirConfig)
```

#### Flyd

```js
import flydConfig from 'recompose/flydObservableConfig'
setObservableConfig(flydConfig)
```
