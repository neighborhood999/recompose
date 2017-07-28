# 我該使用 Recompose 嗎？Performance 和其他問題的擔憂

我相信使用 higher-order composition helper 可以更精巧，更能關注在 component，並提供一個比 class 更好的 programming model，像是 `mapProps()` 或 `shouldUpdate()` - 它們本質不是 class-y。

話雖如此，對現有 API 的任何抽像都可以得到權衡。當引入一個新的 component 到 tree 時，會有一個 performance 的開銷。我猜想透過 blocking subtree 使用 `shouldComponentUpdate()` 來重新 render 得到的 cost 結果是微不足道的 - 使用 Recompose 的 `shouldUpdate()` 和 `onlyUpdateForKeys()` helper 讓這些變得更容易。未來我們將做一些 benchmark，所以可以知道我們處理了些什麼。

然而，許多 Recompose 的 higher-order component helper 使用了 stateless function component 來實作，而不是 class component。最終，React 將包含 stateless component 的優化。在此之前，我們可以利用參考透明來做本身的優化。換句話說，從一個 stateless function 建立一個元素是有效的*，相同於呼叫 function 並回傳 output。

\* *如果 Stateless function component 它們接受 context 或使用預設 props，就不是參考透明（referentially transparent）；我們透過檢查 `contextTypes` 和` defaultProps` 的存在來確認。*

為了要完成這個，Recompose 使用一個特別的 `createElement()` 版本，它回傳 stateless function 的 output，而不是建立一個新元素。對於 class component，它使用內建的 `React.createElement()`。

我不推薦這個方法在你的 app 中大部分的 stateless function component。首先，你會失去使用 JSX 語法，除非你對 `React.createElement()` 做了 monkey-patch，但不是個好主意。再來，你會失去惰性求值（lazy evaluation）。考慮這兩個 component 之間的差別，因為 `Comments` 是一個 stateless function component：

```js
// 具有 lazy evaluation
const Post = ({ title, content, comments, showComments }) => {
  const theComments = <Comments comments={comments} />;
  return (
    <article>
      <h1>title</h1>
      <div>{content}</div>
      {showComments ? theComments : null}
    </article>
  );
});

// 沒有 lazy evaluation
const Post = ({ title, content, comments, showComments }) => {
  const theComments = Comments({ comments });
  return (
    <article>
      <h1>title</h1>
      <div>{content}</div>
      {showComments ? theComments : null}
    </article>
  );
});
```

在第一個範例，`Comments` function 用來建立一個 React 元素，如果 `showComments` 為 true，只*透過 React* 被計算。在第二個範例，`Comments` function 在每次 render 的 `Post` 被計算，不管 `showComments` 的值為何。可以把 `Comments` 呼叫放在三元運算子中來解決，但很容易忽略這種區別，並產生 performance 問題。基本上，你應該總是建立一個元素。

所以為什麼 Recompose 要打破這個規則？因為它是一個 utility library，而不是一個應用程式。就像 lodash 去使用 for-loops 作為 helper function 實作細節一樣，對於 Recompose 避免中間 React 元素作為一個（暫時）performance 優化也是可行的。
