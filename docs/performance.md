# 我該使用 Recompose 嗎？Performance 和其他問題的擔憂

我相信使用 higher-order composition helper 可以更精巧，更能關注在 component，並提供一個比 class 更好的 programming model，像是 `mapProps()` 或 `shouldUpdate()` - 它們本質不是 class-y。

話雖如此，對現有 API 的任何抽像都可以得到權衡。當引入一個新的 component 到 tree 時，會有一個 performance 的開銷。我猜想透過 blocking subtree 使用 `shouldComponentUpdate()` 來重新 render 得到的 cost 結果是微不足道的 - 使用 Recompose 的 `shouldUpdate()` 和 `onlyUpdateForKeys()` helper 讓這些變得更容易。未來我們將做一些 benchmark，所以可以知道我們處理了些什麼。

然而，許多 Recompose 的 higher-order component heleper 使用 stateless function component 被實作，而不是 class component。React 最終將會包含 stateless component 的最佳化。
