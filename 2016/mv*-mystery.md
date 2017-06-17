## MV* 之謎

[資料來源] (https://github.com/livoras/blog/issues/11)

### MVC 要解決的問題
圖形介面（GUI）提供使用者直觀的操作介面，而使用者透過載具（滑鼠、鍵盤）執行應用邏輯（送出訂單），應用邏輯可能會觸發商業邏輯變動（在資料庫成立訂單）或是應用狀態改變（訂單清單依照時間排序）。為了方便維護以及管理，把應用程層。管理使用者介面稱為 view，應用數據層稱為 model，依據不同的業務邏輯 model 會提供相對應的接口。

### MVC 的祖先
- Smalltalk-80 MVC
- GoF

### 正版 MVC 
- 流程
	- 使用者操作 view
	- view 把處理權交給 controller
	- controller 會對從 view 來的數據進行預處理，決定交給哪個 model 處理
	- model 變更之後運行觀察者模式(observe pattern)
	- view 透過觀察者模式收到通知之後，會向 model 請求新的數據。或是 view 主動發送 	- request 索取新資料
- 什麼是觀察者模式
	- subject 擁有各式各樣的狀態
	- observer 可以向 subject 註冊各種狀態變化，但必須提供 update
	- 狀態改變時，檢查註冊表中有哪一些 observer 有註冊，發送相對應的 update
- 優點
	- 業務邏輯跟商務邏輯分開，模組化程度高
	- 透過觀察者模式達成多視圖更新
- 缺點
	- view 與 model 設計習習相關，難以組成可重複使用的元件。因為不同的應用 domain model 的設計也不一樣。

### 後端 MVC ( MVC model 2 )
- 流程
	- dispatcher / routes 負責解析 url 並且分配給相對應的 controller
	- controller 負責跟 model 要資料
	- model 跟 db 要資料回來整理過後給 controller
	- controller 拿整理過的資料塞進指定的 view 裡面
- 不是 MVC 的原因
	- 由於 view 無法感知 model 變動，因此不算是正版的 MVC

### MVVM
- 流程
	- 使用者操作 view 
	- view 發送訊息給 ViewModel
	- ViewModel 透過 binder 同時更新 Model 和 View
- ViewModel
	- 在 render view 的過程裡除了 domain model 的資料之外，可能還包含著 domain model 裡面沒有的資訊，像是 sort 的順序、filter 的方式等，這個部分由 ViewModel 負責。

