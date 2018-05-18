# Flow 支援 recompose

## 它是如何運作的

在大部分情況下，你需要宣告一個被 enhance 的 Component 的 prop type。
Flow 將會推斷所有其他你需要的類型。

例如：

```javascript
import type { HOC } from 'recompose';

type EnhancedComponentProps = {
  text?: string,
};

const baseComponent = ({ text }) => <div>{text}</div>;

const enhance:HOC<*, EnhancedComponentProps> = compose(
  defaultProps({
    text: 'world',
  }),
  withProps(({ text }) => ({
    text: `Hello ${text}`
  }))
);

export default enhance(baseComponent);

```

觀看 recompose flow 的運作。

![recompose-flow](https://user-images.githubusercontent.com/5077042/28116959-0c96ae2c-6714-11e7-930e-b1454c629908.gif)

## 如何開始使用

最簡單的方式就是從範例開始。

查看[這個](http://grader-meets-16837.netlify.com/)應用程式的[原始碼](../types/flow-example)


## 支援

recompose 的 HOCs 的類型定義被分為兩個部分。

### Part 1 - HOCs 與良好的 flow 支援

在大多數情況下，你使用它們不會有什麼大問題。
類型推斷和錯誤檢測可以良好的運作。

這些 HOCs 是：*defaultProps, mapProps, withProps, withStateHandlers, withHandlers, pure, onlyUpdateForKeys, shouldUpdate, renderNothing, renderComponent, branch, withPropsOnChange, onlyUpdateForPropTypes, toClass, withContext, getContext, setStatic, setPropTypes, setDisplayName*

#### 「良好」 HOCs 的已知問題

參考 `test_mapProps.js` - 在 HOCs 的類型推斷不會檢測到類型的錯誤

### Part 2 - 其他 HOCs

要使用這些 HOCs - 你需要提供類型資訊（不是類型自動推斷）。
You must be a good voodoo dancer.

參考 `test_voodoo.js` 的想法。

一些建議：

- *flattenProp,renameProp, renameProps* 可以輕鬆的被 _withProps_ 替換
- *withReducer, withState* -> 使用 _withStateHandlers_ 替代
- _lifecycle_ -> 如果需要一個 _lifecycle_，你不需要 recompose，而是直接使用 React 的 class
- _mapPropsStream_ -> 參考 `test_mapPropsStream.js`

#### 上述 HOCs 已知的問題

參考 `test_voodoo.js`、`test_mapPropsStream.js`

### Utils

*getDisplayName, wrapDisplayName, shallowEqual,isClassComponent, createSink, componentFromProp, nest, hoistStatics.*

### 文章

[Typing Higher-order Components in Recompose With Flow](https://medium.com/flow-type/flow-support-in-recompose-1b76f58f4cfc)

### 常見問題

為什麼使用 `HOC <*，Blbla>` 的存在類型不可避免這種情況？

*我嘗試使用類型別名，但還沒有找到如何讓他運作。*

## 致謝

特別致謝 [@gcanti](https://github.com/gcanti) 在 PR [#241](https://github.com/acdlite/recompose/pull/241) 的貢獻，對於目前的定義，它是相當明確的基礎。
