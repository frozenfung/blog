## First npm package of Polydice

### 自己用的 package 自己包

最近子站總是鬼打牆的出現主站實作過的功能，於是就想要把幾個站共用的 [jquery-tinyslide](https://github.com/polydice/jquery-tinyslide) 拉出來成為一個獨立的 npm package，實踐 DRY（Don't Repeat Yourself）精神。


### 開始實作之前

先簡單的列出這次主要的任務

- Move code & test from project to package
- Make it compatible with both npm and bower
- Write readme
- Create a simple demo

然後簡單的寫下 readme 的結構

- Install
- Dependency
- Usage
- Options
- Test
- Demo
- Contribute
- Licence

### Hands on

包 npm package 的第一步，當然是先參考網路上各種大神們的各種作品。大概是最近都在寫 React 的關係，第一個想到的就是 [classnames](https://github.com/JedWatson/classnames)，一款由民間開發，火了之後升級成 [React 官方認證](https://facebook.github.io/react/docs/class-name-manipulation.html)的 CSS class management API。主要參考他們的**檔案配置**和**前端環境設定**。

#### 檔案配置

`LISENSE` 的部分主要參考了下面兩個資源，第一個網站介紹幾種 opensource 常用的 license，依照自己的需求去挑選即可，一般來說如果你開發的是單純的 opensource 專案，選 MIT 即可。第二個網站是由 GitHub 發起的 opensource-friendly 計畫，裡面提供了我需要的 MIT license 的 template。

- [Open Source Initiative](https://opensource.org)
- [Choose an open source license](http://choosealicense.com)

`.npmignore` 這個檔案有點像是 `.gitignore` 的 npm 版。讓你把某些開發用的檔案濾掉，安裝 package 的時候就不會裝進來，常見的像是 `.gitignore` 或是 `test`。  

`package.json` 裡面記載了關於一切跟 npm package 有關的資訊，直接下 `npm init` 指令無痛完成。

`bower.json` 跟 `package.json` 類似，下 `bower init` 然後依照指令完成。 


#### 安裝相容性

鑑於目前前端套件的一百種安裝方式，如果只支援 `npm install` 有違反社會善良風俗之嫌。所以通常在設計 index.js（專案進入點）的時候會讓支援幾種現在社會上常用的套件管理框架。

```
if (typeof module !== 'undefined' && module.exports) {
  module.exports = TinySlide;
} else if (typeof define === 'function' && typeof define.amd === 'object' && define.amd) {
  define('TinySlide', [], function () {
    return TinySlide;
  });
} else {
  window.TinySlide = TinySlide;
}

```

第一個條件支援 Node Module Management (npm)，第二個條件支援 RequireJS，最後一個直接宣告成 global，支援不使用 module manage tool 的玩家。

#### 微調整

完成了上面的設定後，又參考了幾個別的 npm packages，做了一些調整

- 加上 npm version 的 badge 
- 由於這是個 jquery plugin，所以特別在 readme 裡面提醒大家要記得先裝 jquery 
- bower install 或是 npm install 實際上是到 GitHub 的 repo 抓程式，因此記得要記得把 repo 設成 public，才算是註冊完成。

#### Problem with es6

完成了微調整之後，原本以為可以順利導入到各個專案裡面。一開始在本地測試也很順利，然而，一直到準備上 production 之前，docker build image 跑到 `bundle exec rake assets:precompile RAILS_ENV=production` 這行的時候會有來自 `execJS` 通報的問題。跟 Richard 討論了一下，因為我們是採用 browserify + sprocket 來管理 js 的檔案，後來覺得有可能是 browserify 在從 `node_module` 裡面 require `jquery-tinyslide` 的時候無法解析檔案裡面的 es6 語法。

除了上述的原因，也考慮到安裝 package 的使用者不一定會有相容 es6 的開發環境。參考了兩篇用 es6 開發 npm package 的文章

- [How to publish a module written in ES6 to NPM?](http://stackoverflow.com/questions/29738381/how-to-publish-a-module-written-in-es6-to-npm)
- [How to build and Publish ES6 npm Modules Today, with Babel](https://booker.codes/how-to-build-and-publish-es6-npm-modules-today-with-babel/)

稍微調整了開發的[流程](https://github.com/polydice/jquery-tinyslide/blob/master/package.json#L8-L9)，改成在發佈之前先用 babel 把 es6 開發的程式編譯成 es5，然後發佈 es5 的版本。將專案改成安裝 es5 的版本之後，docker build image 就沒有再遇到問題，順利 deploy。

### 最後

做 opensource 是一件愉快的事，如果有人跟你一起做的話就更棒了。如果大家看完這篇對於這個專案有什麼想法，歡迎在下面留言。或是直接到 [GitHub Repo](https://github.com/polydice/jquery-tinyslide) 上面發 issue、開 PR，跟我們更直接的互動。


`