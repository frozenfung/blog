## How React Lifecycle Methods Work

### Intro

學習 React 的路上，看到 [Understanding the React Component Lifecycle](http://busypeoples.github.io/post/react-component-lifecycle/) 這邊文章驚為天人，這篇文章除了帶我認識 Lifecycle Methods 之外，更重要的是它的**使用時機以及呼叫順序**。把文章的幾個重點記下來，再加上自己的學習心得，整理成這篇方便自己以後複習。

### Prologue 

這個系列的上一篇 [Introduction to React Lifecycle Methods](https://medium.com/@frozenfung/introduction-to-react-lifecycle-methods-eca8bb90c278#.rtqm1onoo) 裡面，討論了**這些 API 是什麼**以及**為什麼要用這些 API**，對於**如何使用這些 API**沒有更深入的討論。這一篇想要透過解構 component 常有的行為以及每個行為觸發的 lifecycle methods，可以比較清楚了解怎麼把 lifecycle methods 應用在實作上。

### Movement

主樂章分成兩個部分

* Common behaviours of component
* Default chain of methods triggered by behaviour

先介紹 component 常見的行為，再來討論這些行為觸發的 lifecycle methods。

#### Common behaviours of component

component 裡面常見的行為有四種：

* initialize - 初始化 component，`React.render` 或是 `ReactDOM.render` 時觸發。
* change state - 改變 state，`this.setState` 時觸發。
* change props - 改變 props，從上一層 component 傳入新的資料時觸發。 
* unmount - component 被移出 DOM 時觸發。

#### Default chain of methods triggered by behaviour

上面四種行為，React 幫我們制定了一系列後續觸發的方法和這些方法被觸發的順序。

**initialize**

1. `getDefaultProps`
2. `getInitialState`
3. `componentWillMount`
4. `render`
5. `componentDidMount`

當一個 component 透過 `React.render` 或是其他的方式初始化，首先會被呼叫兩個方法是 `getDefaultProps` 和 `getInitialState`。這兩個方法只會在 component 被初始化的時候被呼叫一次，也是唯一的一次。

```
getDefaultProps() {
	/* Return all props you need */
	return {
		a: 1,
		b: 2,
	};
}
```

第一個被呼叫的是 `getDefaultProps`，通常會在這裡設定 props 的初始值。之後就可以透過 `this.props` 取得 props 的值。


```
getInitialState() {
	/* Initialize state here */
	return { 
		c: 3,
		d: 4,
	};
}
```

接著被呼叫的是 `getInitialState`，通常會在這裡設定 state 的初始值。之後就可以透過 `this.state` 取得 state 的值。

初始化完 state 跟 props 之後，接著是 `componentWillMount`。有趣的是如果在`componentWillMount` 裡面用 `this.setState` 重新設定 state 的值，這時並不會觸發額外的 render，而把在這個階段的 state 變更併入這入原本的 state 變更，等到 進入下一個階段（render）一起變更。

```
render() {
	return <ul>
		<li>1</li>
		<li>2</li>
		<li>3</li>
	</ul>;
}
```

`render` 階段最重要的任務是自定義 component 的基本 html template。當然也可以不回傳任何值，直接`return false`。

最後是 `componentDidMount`，由於在上一個階段 DOM 的結構已經被渲染（render）出來，這個階段通常會跟 server 要資料或是對已經載入資料的 DOM 做簡單的操作。

**change state**

1. `shouldComponentUpdate`
2. `componentWillUpdate`
3. `render`
4. `compoentDidUpdate`

```
shouldComponentUpdate(nextProps, nextState) {
	// return a boolean value
	return true;
}
```

`shouldComponentUpdate` 會在 `render` 之前被呼叫，透過回傳的布林值來決定接下來的動作會繼續或是被省略。預設傳入的兩個參數 `nextProps` 和 `nextState`，我通常用來跟 `this.state` 和 `this.props` 做比較，再依據比較的結果決定回傳的布林值。

```
componentWillUpdate(nextProps, nextState) {
	// perform any preparations for an upcoming update
}
```

一旦 `shouldComponentUpdate` 回傳 `true` 之後，`componentWillUpdate` 就會接著執行。在這個階段我通常依照 `nextState` 和 `nextProps` 的值針對應用邏輯做需要的調整和確認，準備好就可以進入 `render` 階段。不過每個應用的邏輯和需求不同，在這個階段可能完全不需要做任何事，那麼就直接把這個方法省略。

`render` 階段回傳自定義的 html template。

```
componentDidUpdate(prevProps, prevState) {
    // do what ever you want
}
```

`render` 結束，代表著需要被更新的資料都已經更新完成，如果有需要的話，這個時候我會透過 `componentDidUpdate` 用來對更新後的 DOM 做一些簡單操作。

**change props**

1. `componentWillReceiveProps`
2. `shouldComponentUpdate`
3. `componentWillUpdate`
4. `render`
5. `componentDidUpdate`

```
componentWillReceiveProps(nextProps) {
  this.setState({
    // set state with nextProps
  });
}
```

**change props** 跟 **change state** 觸發的 lifecycle methods 基本上大同小異，除了 **change props** 在一開始會呼叫 `componentWillReceiveProps`。而`componentWillReceiveProps` 唯一被呼叫的時機是當 props 改變。在這個階段我通常會利用 `nextProps` 或是 `this.props` 取得 props 的值，經過調整之後透過`this.setState`設定 state，這時候 `this.setState` 不會觸發額外的 render。

剩下的 lifecycle methods 應用請見 **change state** 章節。

**unmount**

1. `componentWillUnmount`

在 unmount 只會觸發一個 lifecycle method。也就是還沒有討論到的 `componentWillUnmount`，它特別的是只有當 component 從 DOM 移除之前才會被呼叫，我通常用它來清除這個 component 相關的資料。

#### Postlude

圍繞著這些行為上面學習 lifecycle methods，對我在實作中正確的使用這些方法很有幫助。不過關於使用細節，以及內部的運作方式，不是這一篇的重點所以就沒有紀錄太多。如果需要更多的資訊，[官方文件](https://facebook.github.io/react/docs/component-specs.html)對於細節有更多的討論。

### Reference

[Understanding the React Component Lifecycle](http://busypeoples.github.io/post/react-component-lifecycle/)