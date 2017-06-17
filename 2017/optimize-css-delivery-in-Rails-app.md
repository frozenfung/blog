## Optimize CSS delivery in Rails app

這是一篇實作紀錄，紀錄前陣子透過 Google 的 [PageSpeed Tools](https://developers.google.com/speed/pagespeed/) 發現 `清除前幾行內容中的禁止轉譯 JavaScript 和 CSS` 這項被扣分，遵循 [Google 官方的建議改善方式](https://developers.google.com/speed/docs/insights/OptimizeCSSDelivery)，用 [loadCSS](https://github.com/filamentgroup/loadCSS) 搭配 [critical-path-css-rails](https://github.com/mudbugmedia/critical-path-css-rails) 順利解決這個問題。接下來我會比較詳細描述問題和實作的過程。


### Big CSS file block page render

一般來說用 Rails 開發網站，用 Sprocket 配上 Tilt 來做 asset pipeline 算是蠻常見的方式。在 http2 尚未普及的年代，透過 Sprocket 整併 assets 能夠大幅減少需要的 CSS/JS request 數量。不過隨著網站越來越大，CSS 的大小也隨之增加。等待下載 CSS 的時間變長，或是[一次解析過多 CSS](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/render-blocking-css) 而造成不必要的載入延遲。

### Best practice

理想的狀況是把每個頁面的第一屏（above-the-fold）需要用到的樣式用 `<style>` 包起來，放在 `<head>` 裡面，剩餘的樣式用 async 的方式載入。這樣可以在大致上不影響 UI 下用最快的速度刷出第一屏需要的元素給使用者。

總結要做的事情

- Preload external resources
- Dynamic inline style for each endpoint

### Preload external resources

Preload 是 W3C 制定的一種跟 `<link>` 有關的 [規格](https://w3c.github.io/preload/)，下面從官網節錄部分介紹

> This specification defines the preload keyword that may be used with link elements. This keyword provides a declarative fetch primitive that initiates an early fetch and separates fetching from resource execution.

簡單來說透過設定 `<link>` 的 rel attriubte 為 `preload`，可以讓瀏覽器暫時忽略 html 順序的限制先開始下載你想要的資源。

`<link rel="preload" href="stylesheet.css" />`


比較可惜的是瀏覽器的支援度比較差（圖一），所以我額外用 [loadCSS](https://github.com/filamentgroup/loadCSS) 補強還不支援的瀏覽器。（2017.5.17） 

（圖）


### Dynamic inline style for each endpoint

再來要處理動態載入樣式，這個階段我們選擇 [critical-path-css-rails](https://github.com/mudbugmedia/critical-path-css-rails) 這個 Gem 來實作。安裝過程有點繁瑣我就不贅述了，簡單來說它做了三件事:

1. 建立 `critical_path_css` rake task，並且透過 `Rake::Task['assets:precompile'].enhance { Rake::Task['critical_path_css:generate'].invoke }` 把這個 task 註冊到`assets:precompile`
2. `critical_path_css` 的 task 會讀 `config/critical_path_css.yml` 裡面的設定的路徑，用 [phantomjs](https://github.com/ariya/phantomjs) 搭配 [penthouse](https://github.com/pocketjoso/penthouse) 分別產生每個路徑的 critical css
3. 將上一步產生的 critical css 存到 redis

之後只需要 `CriticalPathCss.fetch(request.path)` 抓到該路徑的 critical css，最後放進 `<head>` 裡面就大功告成。如果不想用 rake task 也可以試試看用 [ActiveJob](https://gist.github.com/taranda/1597e97ccf24c978b59aef9249666c77) 。 


### Conclusion

上線之後再用 PageSpeed Tool 再測試一次，這次已經沒有因為過多的 CSS 而被扣分了。整個實作不會花太多的時間，上線就能夠馬上看到成效，唯一的缺點是 `penthouse` 在執行蒐集 critical css 的環境是 JS disabled，所以在 client 端用 JS 產生的元素的樣式會抓不到。

有任何想法或是建議，歡迎在下面留言跟我們討論。

### Reference

* [W3C Working Draft - Preload](https://www.w3.org/TR/preload/)
* [Render Blocking CSS](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/render-blocking-css)