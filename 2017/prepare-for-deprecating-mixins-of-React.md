## Prepare for deprecating mixins of React

去年年中 [Dan Abramov](https://twitter.com/dan_abramov) 在 React 部落格發表的[文章](https://facebook.github.io/react/blog/2016/07/13/mixins-considered-harmful.html)提到，隨著 React codebase 的持續演化，mixins 已經漸漸不符合團隊的期待，正式宣告 ES6 的語法不再支援。以下節錄部分相關段落:

> To ease the initial adoption and learning, we included certain escape hatches into React. The mixin system was one of those escape hatches, and its goal was to give you a way to reuse code between components when you aren’t sure how to solve the same problem with composition.

這段是說在一看始不知道怎麼用 composition 解決這類問題的時候，先用 mixin 來做 code reuse

> Three years passed since React was released. The landscape has changed. Multiple view libraries now adopt a component model similar to React. Using composition over inheritance to build declarative user interfaces is no longer a novelty. We are also more confident in the React component model, and we have seen many creative uses of it both internally and in the community.

簡單來說就是隨著 composition 在生態系逐漸成熟，mixin 因為不夠直觀和其它的缺點開始被大家所嫌棄。

這篇記錄我實作 [fluxxor-wrapper](https://github.com/frozenfung/fluxxor-wrapper) 的構想和用他替換專案裡面有用到 mixin 的 React component 的過程。 


### What is Fluxxor

[Fluxxor](https://github.com/BinaryMuse/fluxxor) 是基於 Facebook 提出的 Flux architecture 的實作，是一套用來搭配 React 管理 state 的工具。有某些概念跟 Redux 類似，像是都用 dispatcher 搭配 actions 來橋接使用者的行為和更新 state 的方法。不過也有些地方不同，像是不支援 middleware。

### What do mixins play between Fluxxor and React

一般來說 mixins 有下面幾個用途:

- Performance Optimizations
- Subscriptions and Side Effects
- Rendering Logic
- Context
- Utility Methods

React 跟 Fluxxor 的整合一開始的設計是透過 StoreWatchMixin 和 FluxMixin 兩個 mixins 實作 subscription 和 context 的功能。StoreWatchMixin 使得每當 fluxxor store 裡面的的值改變，就會對 component 觸發 `setState` 進而 re-render。而 FluxMixin 則是把 flux 透過 context 變成偽全域，讓每一層的 component 可以直接拿到 flux，而不用透過 props 一層層傳進去。

### Implement fluxxor-wrapper

我的想法很簡單，用一個 hoc 來實現原本透過 mixin 來達成的功能。

StoreWatchMixin 的部分其實沒有我想像中那麼複雜:

1. 定義 `setStateFromFlux` 用來更新 hoc 本身的 state
2. 定義 `getFlux` 做為 flux instance 的 getter
3. 在 `componentDidMount` 裡面把要觀察的 store 跟 `setStateFromFlux` 做綁定
4. 把傳進 hoc 的 props 跟 hoc 本身的 state 傳進 component
5. 把 `getFlux` 也傳進 component

FluxMixin 我沒有實作。主要是因為從目前專案的結構來看，大部份的 action 都是被第一層 component 呼叫，很少透過底層的 component 呼叫，所以我選擇要用的時候再透過 props 傳下去。

### Wrap it!

如何使用的部分在 Repo 有詳細的教學，在這裡我就不解釋了。請參考 [README](https://github.com/frozenfung/fluxxor-wrapper#getting-started) 

### Conclusion

設計與實作的過程最大的收穫就是讓我開始了解 hoc 的價值和運作邏輯。看完如果覺得有任何想法，歡迎在下方留言。你也可以用 [@frozenfung](https://twitter.com/frozenfung) 再 twitter 上找到我。

### Reference
- [Why FluxComponent > fluxMixin](https://github.com/acdlite/flummox/blob/v3.5.1/docs/docs/guides/why-flux-component-is-better-than-flux-mixin.md)
- [Mixins Considered Harmful](https://facebook.github.io/react/blog/2016/07/13/mixins-considered-harmful.html#migrating-from-mixins)