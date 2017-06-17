## Introduction to React Lifecycle Methods 

### Intro

React 最重要的精神之一就是 component。為了讓 component 的開發更有彈性，React 提供了友善的 API，透過這些 API 讓開發者可以**準確地安排 component 在每個階段的行為。**

### Why

在介紹怎麼使用這些 API 之前，我想先談一下為什麼我們需要這些 API。
在網頁上，每一個被 render 出來的 component 都可以對應到 html 裡面的一個元素(element)，而每一個元素在一個網頁的生命週期裡可能會經歷下面三個階段：

* 出生 (mount)
* 改變 (updating)
* 死亡 (unmount)

而透過這些 API 讓我們可以精準地控制 component 在不同階段執行應該做的事。舉例來說：

* 在 shouldComponentUpdate 階段，比較新舊資料是不是相同之後決定要不要 render 這個 component。
* 在 componentDidMount 階段，發送 Ajax 向 server 取得需要的資料。

對我來說這些 API 某種程度上表達該 component 的運作流程，透過熟悉運用這些 API 呼叫的順序和時機，加速理解與開發 component 的速度。

### What

Lifecycle methods 提供了七個 API，依照上面提到的三個階段（出生、改變、死亡) 分類

#### 出生 (mount)

* componentWillMount
* componentDidMount

#### 改變 (updating)

* componentWillReceiveProps
* shouldComponentUpdate
* componentWillUpdate
* componentDidUpdate

#### 死亡 (unmount)

* componentWillUnmount

關於每個 API 的使用時機和注意事項，請參考官方的文件：

[Component Specs and Lifecycle] (https://facebook.github.io/react/docs/component-specs.html)

### How

關於如何在使用這些 API，我另外寫了一篇 [How React lifecycle methods work](#)。在這篇裡面將會列舉幾種常見的 component 行為，圍繞這些行為討論這些生命週期使用的時機和注意事項。



