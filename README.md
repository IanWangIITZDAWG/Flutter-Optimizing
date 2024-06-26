# 優化你的 Flutter APP
##### ***翻譯自https://www.linkedin.com/pulse/optimizing-flutter-ui-performance-strategies-smooth-o5jwf***

## 前言

你知道有 79% 的使用者會因為應用程式不流暢而放棄使用嗎?
如今這個手機當道的世界中，一款流暢且快速的應用程式已經是基礎標準
尤其對於 Flutter 應用程式，順暢的動畫、保持高幀率是保住使用者的關鍵
但要達到這樣的標準仍然是件不簡單的事，不論你是剛接觸 Flutter 的新手或是目標為增進效能的老手，了解如何優化你的 UI 會是關鍵。

此篇文章主要是教你如何在任何裝置上使你的應用程式擁有流暢與高品質，帶著你了解普遍效能瓶頸問題，與優化組件圖像動畫的技巧。

OK，讓我們來探索這充滿潛力的提升效能技巧吧！

## 理解 Flutter 渲染過程

Flutter是使用Widget為基礎構建UI  
每個Widget，像是按鈕、文字、圖像等等都是UI中的一塊拼圖  
而每個Widget背後都有一個相對應的RenderObject，變成RenderObject後才會有明確的布局來繪製在畫面上  
讓我們來窺探這其中的機制：渲染過程。  
![pipeline.jpg](/images/pipeline.jpg)

這個渲染機制大致上有三個階段：
1. **Build**：在這階段中Widget會開始構建，定義其中的屬性。
2. **Layout**：在這個階段元素會確定各自的尺寸、位置。
3. **Painting**：在這階段元素會被繪製出來到你的眼中，文字、顏色、效果都會被套用。

### 各個渲染引擎的差別

- **Flutter Engine**：最主要的引擎，會被用在Android、iOS、Windows、macOS、Linux中。
- **CanvasKit**：在web中的會用到的另外一個引擎，使其在web中具有渲染能力。
- **Impeller**：試驗型引擎，目標為增進效能，使用了預編譯以及繪圖API(Vulcan)。

然而這個進程並不是毫無缺陷的，當Widget重新Build的時候會使整個進程重新執行，影響著效能
例如你只不過調整Widget 在畫面上的位置，其他Widget也會被連著移動與重新計算大小。如果沒有經過優化，這種情況在Flutter中會常常發生。

## 了解效能瓶頸

即使Flutter有著令人驚嘆的效能，但如果不經過好好的優化，漂亮的視覺效果與動畫也會因為不流暢而顯得粗糙
為了能夠有效的優化，需要先確定導致效能低落的原因，讓我們看看一些常見的問題：

1. **Widget Rebuild**：Flutter會藉由Widget建構出UI，每個Widget都是畫面中其中一角，但其中如果有Widget進行了不必要的重新建構，造成一連串反應而導致其他Widget進行了不必要的重新構建，這將會拖累渲染系統，導致卡頓。
2. **布局複雜**：複雜的布局導致巢狀Widget影響Flutter的計算畫面中各元素的位置與大小的效率。
3. **圖像處理**：像素高的圖片會拖慢載入速度，使用調整大小與壓縮來優化效能。
4. **動畫**：動畫可以增加互動性，但如果使用不當，很容易造成反效果，時機不當或高密度的動會導致卡頓，破壞使用者體驗。
5. **網路請求**：使用非同步獲取資料會有等待時間，如果處理不當，會讓使用者等待更長的時間，例如網路慢、參數多、頻繁請求等等。

上面只是幾個常見例子，每個人遇到的問題都不太一樣，主要還是要看各自的APP架構與執行方式
然而只要你能夠識別出這些問題的問題點，就代表你已經邁出重要的一步了。

## 優化方法

現在我們學到了Flutter渲染流程，以及Widget Rebuild陷阱，接下來讓我們一起學習優化他們的方法吧。

#### 減少Widget Rebuild

- **當個Rebuild偵探**：使用DevTools來找出會常常造成Widget Rebuild的資料，減少他們的發生來讓你的APP更流暢。
- **善用不可變**：屬性不會變的 Widget 請使用`const`關鍵詞，這會減少不必要的Rebuld與增進效能。
- **聰明的使用狀態管理**：選擇性的使用`setState`，只更新會觸發Rebuild所需要的最少資料。如果資料太複雜，可以考慮使用狀態管理庫。
- **使用Statusless簡化**：如果Widget的外觀都不會變動，請將此Widget聲明為`StatelessWidget`，這樣可以減少不必要的Rebuild開銷。

#### 有效率的布局

- **神奇的 Container**：聰明的使用`Container`，他提供常用的布局屬性(padding、margin、aligment etc)，減少巢狀Widget的使用。
- **Row 與 Column 的力量**：如果是簡單的縱橫排列布局，請多使用`Row`與`Column`，兩者都提供效率很好的排列能力。
- **Sliver 使用**：如果遇到大型資料需要渲染，請使用Sliver Widget(`SliverList`、`SliverGrid` etc)，Sliver Widget可以有效的控制顯示滾動區域內的Widget，來減少效能低落。
- **規劃佈局**：將複雜的布局劃分為更好管理的小型布局，這會改善渲染效率並減少牽一髮動全身的風險。

#### 優化圖像

- **大小問題**：調整圖像的大小以符合顯示需求，不要載入只占畫面一點點部分的的肥大圖檔。
- **使用快取**：使用`CachedNetworkImage` 來快取已經下載的網路圖檔，這可以避免多餘的網路圖檔請求，提高之後相同的圖檔載入速度。
- **壓縮的藝術**：考慮使用圖像壓縮套件像是`flutter_image_compress`，既可以降低圖像的大小也不會犧牲圖像的品質。

#### 製作流暢的動畫

- **動畫類型**：了解隱性動畫(狀態改變)與顯性動畫(手動控制)，選擇適合自己的方式。
- **Tweeing插值**：掌握tweeing差值的概念來製作流暢且令人賞心悅目的動畫。
- **使用可表現複雜動畫的套件**：對於複雜的動畫表現可以使用`rive`或是`flare`套件，兩者都提供強大的功能與工具來製作進階動畫。
- **小技巧**：動畫中避免不必要的重繪，使用`vsync`實現幀率同步，確保動畫與設備的刷新率保持一致，從而獲得無縫體驗。

優化是門不斷更新的技術，記得常常使用DevTools來查看效能並不斷完善你的程式，來獲得最好的回饋。

## 調適與偵錯

即使經過了細心的優化，效能問題還是會出現，此時 DevTools 就會是你的好夥伴，他可以偵測出問題來讓你的 UI 保持完美。以下便是DevTools的使用方式:

- **Performance 頁面**：這張直觀的表格可以實時顯示 APP 的效能表現，可查看幀持續時間、識別慢幀並定位出需要關注的部分。
![performance.png](/images/performance.png)
- **CPU FlameChart**：深入研究可視化 APP 執行時間線的 FlameChart，他可以識別導致效能瓶頸的確切 Function 與程式碼。
![cpu-flame.png](/images/cpu-flame.png)
- **Inspector**：查看 Widget tree 如何建構、識別會 Rebuild 的 Widget、以及分析屬性。
![inspector.png](/images/inspector.png)

你可以使用這些DevTools來：

- **識別瓶頸**：使用DevTools來確定APP中哪個部分花過多的時間渲染、表現動畫、處理資料。
- **查看優化成效**：比較舊執行效能與新執行效能來追蹤你的優化成效，這樣可以確保你付出的努力不是白費的。
- **除錯更有效率**：結合DevTools的調適資料與一般除錯技巧(breakpoint、logs etc)來定位效能問題的根本原因，加速你的除錯過程。

### 額外除錯技巧：

- **善用 Logging**：在關注的程式碼中印出資訊可以讓你更好的追蹤執行進度與識別潛在的效能問題。
- **使用斷言**：添加斷言來驗證預計的程式執行結果，能夠盡早發現問題。
- **使用熱重載**：可以馬上的測試更動以及觀察成效，讓優化更有效率。

調適與除錯對於任何階段的開發者都是很有價值的技巧，使用這些工具都需要一點熟練度，盡早掌握它吧！

## 結論

在這裡我們已經探索了Flutter渲染過程、優化UI、處理潛在的效能問題，以及教你如何使用工具與技巧處理這些問題
記住，優化是一個持續不中斷的過程，相信在這裡所學的技巧可以幫助你完成一個流暢且優雅的使用者體驗。

#### 重點：

- **優先注重效率**：減少不必要的Widget Rebuild、優化布局、優化圖像與動畫。
- **善用工具**：使用DevTools來調適與除錯，利用Logging與斷言盡早識別效能問題。
- **測試與進化**：持續的追蹤分析你的優化成效來獲得最大的成果。

相信透過在這裡所學的知識已經可以讓你作出一款不僅能讓使用者著迷而且效能優異的APP
記住，一款流暢且高效的APP早已經不是那些大公司所擁有的專利品，而是讓你能在各種APP中競爭的門票而已。
