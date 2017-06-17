## Server-render ReactJS with Rails

自從愛料理 2013 年導入 ReactJS 以來，有一大部分前端功能都是透過 ReactJS 開發。而為了提高可維護性與降低程式複雜度，從去年開始，陸陸續續的把從前用 AngularJS 開發的程式用 ReactJS 改寫，也即將在最近全數改寫完成。不論是新開發的 ReactJS 專案，或是舊的 AngularJS 專案改寫，站上有越來越多的功能是透過 React component 來實踐，也讓我們開始考慮 server-render 的可能。剛好不久之前看到 Airbnb 開源的一套 Rails- friendly 的 server-render 框架 [hypernova](https://github.com/airbnb/hypernova)，在參考過官方[範例](https://github.com/airbnb/hypernova/tree/master/examples/simple)之後，覺得或許值得一試。 

### Projects get involved in

在描寫實作的細節之前，先簡單介紹這次的專案們，這幾個專案都是於 2016 年 6 月由 Airbnb 開源:

**hypernova** 

[hypernova](https://github.com/airbnb/hypernova) 是一個 express-based 的 node server，負責接收 client 送來的 component，透過 React 的 renderToString 這隻 API 把 component 轉換成 html 字串，最後將 html 字串送回 client。

**hypernova-ruby**

[hypernova-ruby](https://github.com/airbnb/hypernova-ruby) 是 Ruby 的 client，提供的 view helper 讓我們能夠把想要 server-render 的 component 整合到 Rails 的 view template 裡面。除此之外還負責處理在 server-render 失敗的時候的狀況。

**hypernova-react**

[hypernova-react](https://github.com/airbnb/hypernova-react) 是 React component 的 wrapper，讓同一個 component 在 client 端和 server 端可以用不同的方式運作。


### Hands On

接下來會用[愛料理部落格](https://blog.icook.tw/)當成範例，說明 server-render 遇到的一些限制，以及針對這些限制所作出的調整。接下來會提到的兩個 component，一個是[標籤](https://blog.icook.tw/tags)，另一個是[食譜 Widget](https://blog.icook.tw/posts/97942)。

**Configure Rails application**

考慮到每個應用的狀況都不一樣，關於詳細的設定過程就不贅述了，請參考官方的 [Get Started](https://github.com/airbnb/hypernova#get-started)。

**Pre-compile files instead of browserify them**

依照官方的指示設定完成之後，遇到的第一個問題是 server-side 跟 client-side 程式不相容的問題。原本透過 browserify-rails 幫我們編譯 ES6 的 JS，而這在 JS 只在 client 執行的時候不會有問題，但是當我們想要在 server 的環境下導入 JS 的時候就不太方便。所以後來決定把需要 server-render 的檔案拉到 browserify 之外，在部署之前先透過 babel 編譯成 ES5，這樣無論是 server 端的 hypernova 或是 sprocket 都可以抓到可執行的 JS 檔案。 

**Server rendering isn't free/cheap**

調整好檔案被導入的形式之後，我們先從標籤開始嘗試。基本上 client 端的 fallback 不會有什麼問題，不過 server-render 的情形卻時好時壞。經過了一連串的實驗跟討論之後，猜測可能是這個 component 一次想要 render 的內容太多造成的不穩定。決定到 hypernova 的專案上面開了 [issue](https://github.com/airbnb/hypernova/issues/36) 發問，維護者的回覆也證實了我們的猜測。

> Yes, it sounds like you're server rendering too much information. Server rendering isn't free/cheap so just render what you need to and defer the rest. 

把需要 render 的 html tags 的數量從 4k 降到 1k 左右之後就沒有遇到問題了。

**Lower level API is rock!**

接著要 server-render 的是食譜 Widget。這裡遇到的問題是，食譜 Widget 是由 [html-pipeline](https://github.com/jch/html-pipeline) filter 解析文章內的食譜編號動態產生，沒辦法用 view helper 的方式載入。跟 [Richard]() 討論之後，決定採用更有彈性的 [lower level API](https://github.com/airbnb/hypernova-ruby#configuration)。而原本在 componentDidMount 裡面執行的抓取食譜相關資訊的程式，也因為 server-render 的原因，必須提早執行。

最終的流程如下：

1. 掃描文章內容搜集出現的食譜編號
2. 用食譜編號跟外部資源拿到食譜相關資訊
3. 把食譜資訊整理成符合 [hypernova job](https://github.com/airbnb/hypernova-ruby/blob/master/lib/hypernova/controller_helpers.rb#L45-L48) 的形式，並且 enqueue 進 batch instance，等待 submit
4. 呼叫 batch.submit!，拿到 render 好的 html container 包著 component 字串。如果 node server 回應錯誤訊息或是沒有反應，則呼叫 batch.submit_fallback! 產生空的 html container 等待 client 端 render
5. 把文章內容出現的食譜編號位置替換成對應的 html container  

Lower level API 成功的讓我跨越 view helper 沒辦法動態載入的限制。

### Almost Done

到目前為止已經完成 95%，只剩下一些地方需要調整。

**Move client specific operations into ComponentDidMount**

為了讓 component 在 server 和 client 都能夠執行，針對 component 做了一些簡單的調整。主要是把只有在 client 才能完成的一些操作（跟 DOM 相關、docuemnt & window）搬到 componentDidMount 執行。

**Img tag onLoad event could never be triggered when server-render**

發現一個有趣 bug 是 server-render 的 img tag 很有可能不會觸發 onLoad event，發生的條件是當圖片下載完成的時候，React 還沒有把 onLoad 事件掛上去，
詳細情形以及解法可以參考這篇在 stackoverflow 上的[問答](http://stackoverflow.com/questions/39777833/image-onload-event-in-isomorphic-react-register-event-after-image-is-loaded)。

### Conclusion

server-render 對愛料理帶來的幫助可以分成兩個面向來看。對於使用者來說，從一打開頁面是一片空白，到打開頁面就能顯示已經初始化好的資訊，提供了更好的使用者體驗。從搜尋引擎來看，server 提供更完整的 html，讓這個頁面在搜尋引擎建索引的時候能提供更充分的資訊， 對於 SEO 很有幫助。  

這次踩到了好幾個的雷，最終都很順利的解決了。大部分要感謝跟同事討論激發出的靈感，以及網路上社群不吝嗇的分享，讓我少走很多彎路。如果對這篇文章有任何問題或是建議，歡迎在下面留言。
