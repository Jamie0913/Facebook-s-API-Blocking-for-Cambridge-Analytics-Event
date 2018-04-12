# Facebook-s-API-Blocking-for-Cambridge-Analytics-Event

Facebook Graph API
Graph API 是 Facebook 讓程式設計師可以用程式化的方式存取 Facebook 資料的系統化介面，舉凡使用者在 Facebook 上留下的任何資料、行為、足跡，都可以透過 Graph API 在適當的授權下存取。這是一個非常龐大且複雜的 API，它的版本更迭也非常快速。為了避免造成開發者的困擾，在 API 版本演進的同時，舊有的 API 版本仍會持續留存一段時間，好讓開發者有時間能轉移到新的 API 版本上。但所謂的 breaking change，就是 “立刻生效” 的 API 更新，而且它會同時影響所有的 API 版本，不論新或舊。
￼
由於 breaking change 會立即影響所有版本的 API，對開發者來說能應變的時間非常短，因此 Facebook 自己也傾向盡可能不發布 breaking change，而使用 API 的版本更迭來替代。
但這次在 4 月 4 日發布的 breaking change 它的規模之大、影響範圍之廣，算是史上罕見。

在 breaking change 的說明中，Facebook 洋洋灑灑列出了所有受到影響的 API:
* App Insights API
* Events API
* Facebook Login
* Games
* Groups API
* Invitable Friends API
* Messenger Platform
* Open Graph
* Pages API
* Search API
* Taggable Friends
* Tagged Users
* User Node

咦？還有什麼 API 沒有被影響到呢？
再仔細進去看，大部分的 API 都拿掉了非常多原先可以讓開發者用 HTTP GET 查詢的 edge 和 field。
Edge 和 field 是 Facebook Graph API 的特殊用語。Facebook Graph API 這個名字中的 “Graph” 指的是 “社交關係圖” (social graph)，它是 Facebook 上呈現資料的一種形式。這個 graph 中包含三種元素：
* Node (節點) – 社交圖中的基本物件，如使用者、粉專、留言等
* Edge (關係連線) – 節點之間的關係連線，如粉專上的留言或使用者牆上的留言、貼圖等
* Field (欄位) – 物件的相關資料或屬性，如使用者的生日、社團的名稱等
 
Graph API 的威力
舉例來說，如果我們想知道 “創投阿麗莎” 這個粉專 (它是一個 node) 的粉專簡介 (它是這個 node 的某個 field)，我們可以透過 Graph API 送出一個 HTTP get 的請求:
"https://graph.facebook.com/v2.12/vcalyssa?fields=about&access_token=EAEos.......e7BldHOIHeQXhn4JQxq"
我們甚至可以直接在瀏覽器的網址列上輸入上述網址，就會收到來自 Graph API server 的回應。這是一個包成 JSON 格式的回應:
{
  "about": "不用明星臉，不用開分身，行不改名坐不改姓，創投阿麗莎就是我",
  "id": "186642488407218"
}
上面那個網址中有一個參數叫做 token，它是存取 Graph API 的關鍵。這裡的 token 並不是幣圈的那個 token，你不需要花費加密貨幣才能存取 Graph API。這裡的 token 是一種代表你有某種權力的標誌，在傳統關於通訊領域的計算機科學詞彙中，token 常被翻譯成 “權杖” 或 “令牌” (有人記得 Token Ring 網路嗎 ?)，Facebook 自己官方的中文文件則翻譯成權杖。你必須要用一個 Facebook ID 授權產生這組編碼過的數字，它代表存取這組 API 時的身分。
如果我們在呼叫 API 時沒有附上有效的 token，就會得到這樣的回應：
{
   "error": {
      "message": "An access token is required to request this resource.",
      "type": "OAuthException",
      "code": 104,
      "fbtrace_id": "Ajl5BVW9JQS"
   }
}
幾年前在舊版的 API 中，還有些 API 可以不需要 token 就能存取如頭像等公開的資料，但現在幾乎所有的 API，不管存取的是不是公開資料，都需要一個有效的 token。
Token 有幾種不同的類型，但最常見的 token 就是代表使用者身分的 “用戶存取權杖” (User Access Token)，它必須經由使用者授權產生。我們在 Facebook 上玩那些心理測驗的小程式時，會跳出應用程式要求授權的視窗來，當你同意並授權那些應用程式之後，它們就會拿到你的 Facebook ID 所授權的 token，只要在你授權的範圍內，使用這個 token 就可以以你的 Facebook 身分來呼叫 Graph API。
Facebook 身分在 Graph API 的使用上，是非常重要的一個因素，因為它決定了你可以看得到什麼、不能看到什麼；也決定了你可以做什麼、不能做什麼。舉例來說，如果你跟某人是朋友，當你用你的 token 呼叫 GET /user/albums 這個 edge 的查詢時，你就會看到 Graph API 回傳他在 Facebook 上有哪些相簿、相簿的名稱、以及每一個相簿的 node ID 等資料。當你有了每個相簿的 node ID，你就可以繼續用 photos 這個 edge 去查詢每個相簿裡有哪些照片，然後再用 picture 這個 field 拿到每一張照片的 URL，看到圖檔。以上這些動作，都可以透過 Graph API 輕輕鬆鬆用幾行程式碼自動完成，換句話說，透過 Graph API，你可以輕輕鬆鬆把某人貼在 Facebook 上的照片全部抓一份下來。前提是他跟你必須是 Facebook 上的好友，而且他有對好友開放他的相簿。如果你跟對方不是好友，而你企圖用 Graph API 去存取他未公開的相簿，你什麼資料都拿不到。
看出來了嗎？Graph API 雖然強大，但是它不會讓你存取超過你的 Facebook ID 權限所能看到的資料。
Graph API 強大的地方不在於它能看到什麼，而在於它能讓你用程式化、自動化的方法完成許多工作。
以社團 (group) 的 API 來說，你可以對社團的 node 查詢 /members 這個 edge，就能一次列出參加這個社團的所有成員。Facebook 上有許多動輒數萬人甚至數十萬人的大型社團，用這個方法，你可以輕輕鬆鬆一次拿到數萬人的 Facebook user node 資料。雖然你跟這些人不見得是好友，但光是透過 user node 查詢他們公開的資料，如關於 (about)、政治傾向 (political)、宗教 (religion)、婚姻狀態 (relationship_status)、家鄉 (hometiwn) 這些欄位，就能大概拼湊出這個使用者的族群或某些分類的輪廓。這可以拿來幹嘛？光是拿來投放廣告，對某些族群來說就有不得了的精準度了。
更有甚者，如果你的 app 要求使用者授權存取你的 profile，而他同意了，那麼不光是公開的資料，連他只對朋友開放的欄位都拿得到，你就可以更精確地了解這個使用者。這事情聽起來有點耳熟？前陣子鬧得滿城風雨的 Cambridge Analytica 醜聞，就是這麼運作的。
 
劍橋分析事件
劍橋大學的一位老師 Aleksandr Kogan 以研究為由，發布了一支叫做 “thisisyourdigitallife” 的 app，它就像我們整天在 Facebook 上會看到心理測驗、你像哪個大明星、稱號產生器之類的小程式一樣，數以萬計的使用者不疑有他，安裝了這個 app，同意了它存取 Facebook profile，也玩了它所提供的 “測驗”。因此 Kogan 就透過這個 app 拿到了非常多使用者的 Facebook profile。
Aleksandr Kogan 的公司叫做 GSR (Global Science Research)，它成功地藉由這個 app 取得了大量的使用者資料。據稱 GSR 將這批資料賣給了 Cambridge Analytica 的母公司 SCL (Strategic Communication Laboratories)，而 SCL 再讓 Cambridge Analytica 使用這批資料。
Cambridge Analytica 分析、挖掘這些資料之後，對這些人中的特定族群投放廣告，據稱 Cambridge Analytica 藉此操作影響了英國脫歐公投以及美國總統團川普的選舉。不得不說，這其實是 retargeting 廣告的極致操作。
根據 Facebook 自己的估計，有 27 萬左右的使用者安裝了 thisisyourdigitallife，而由於這個 app 除了要求存取使用者本身的 profile 外，也會要求存取朋友的資料，因此粗估遭收集資料的使用者人數在 5000 萬之譜，相當驚人。
但仔細來看這整件事情，説 Facebook 有數千萬 “個資外洩” 其實言過於實，因為這個 app 確實是在使用者的同意跟授權之下才透過 Graph API 拿到使用者的資料，但它沒有按照收集資料時所承諾的方式使用資料，大概是這整件事裡最醜陋的地方。部分下載 thisisyourdigitallife 的使用者可能認為他們的個資會被用於學術研究，而更大部分的使用者可能根本沒有意識到他們在做這個測驗的同時，資料會被收集甚至轉賣。
這事情一次惹毛了英國和美國兩頭獅子。英國國會下議院 (House of Commons) 要求 Mark Zukerberg 出席 “數位、文化、媒體及運動委員會 (Digital, Culture, Media and Sport Committee) 所舉辦的聽證會，但 Zuckerberg 不打算親自出席，而打算派 CTOcMike Schroepfer 或 CPO Chris Cox 去面對英國佬。
在美國本土，參議院的司法委員會 (Judiciary Committee) 及商業、科學、交通委員會 (Commerce, Science, and Transportation Committee) 以 “Facebook, Social Media Privacy, and the Use and Abuse of Data” （Facebook、社群媒體隱私、使用以及濫用資料” 為題舉行聯合聽證會，邀請 Mark Zuckerberg 於 4 月 10 日到場作證。無獨有偶，眾議院的能源及商業委員會 (Energy and Commerce Committee) 也舉辦了以 “Facebook: Transparency and Use of Consumer Data” (Facebook: 透明以及消費者資料的使用) 為題的監督聽證會 (oversight hearing) 聽證會，邀請 Mark Zuckerberg 於 4 月 11 日到場作證。看來接下來這段時間，Mark 會有接不完的通告。
眼看著火越燒越大，Facebook 不得不下猛藥來止血，於是就有了 4 月 4 日的 Graph API 重大變更。
在這次的 breaking change 中，除了我們前面舉例的那些透過 Graph API 能做的事大半被封鎖之外，最大的衝擊是這個：
￼
我相信大部分的開發者看到這個公告都傻眼了。
它的大意是：從 4 月 4 日起，所有的 app 都必須經過 app review 的程序，才能存取活動、社團、和粉絲專頁的 API，包含那些之前已經被核准的 app。用到活動和社團 API 的 app 會馬上被停用 API，直到 app review 通過；而使用粉專 API 的 app 則可以寬限到 app review 開始時。為什麼要等到 app review 開始時呢 ? 因為現在 app review 被暫停了!!! 什麼時候會恢復呢 ? “in few weeks”，官方這麼說。但沒有一個確切的日期。
美國發生 911 恐怖攻擊事件時，由於不知道到底還有沒有更多來自民航機劫機潛在的威脅，為了控制損害，只好先命令美國境內所有的民航機停飛。
Facebook 此時的舉動也很類似。Facebook 上還有成千上萬像 thisisyourdigitallife 這樣的心理測驗小遊戲在流竄，誰也說不準裡面還有沒有其它的程式在收集和濫用個資，為了 Mark Zuckerberg 即將出席的聽證會，Facebook 只好先把這些 app 會用到的 API 直接關了。等風頭過去之後，再慢慢用人工審核的方法開放它們使用 API。這固然是個非常時期不得不的做法，但對於老老實實依賴 Graph API 正派經營某些業務 (如粉專活動管理、社團團購等) 的開發者來說，卻變成最大的惡夢。App 一夜之間失靈，恢復之日遙遙無期。
這真的是 Facebook app 開發者的世界末日了嗎 ? 其實未必。
 
無法隱藏的數位足跡
我們前面說過，Graph API 能拿到的資料，不會超過 Facebook ID 所授權存取的資料。換句話說，Graph API 看得到的資料，使用者用瀏覽器登入 Facebook 都看得到。而在 Graph API 被如此粗暴地關閉一大部分之後，透過使用者介面能看到的東西反而遠比 Graph API 要多得多。
因此，有許多開發者已經在研究如何不依賴 Graph API，而直接透過程式化操作瀏覽器的介面來達到一樣的功能。
比如我們前面舉的那個例子：把某位好友的相簿中所有的照片都抓下來，就可以用瀏覽器做到。當然不是叫你一本一本相簿點進去看、一張一張照片按右鍵下載，而是透過像 Puppeteer 或 Phantomjs 之類的 headless web browsing framework 來做。所謂 headless 就是 “沒有人頭”，其實就是一種自動化的網頁瀏覽工具，常見的網頁爬蟲其實就是 headless browsing 的最好例子。當然，這樣做除了會有效能上的問題外，最大的風險就是 Facebook UI 的改版。每次 Facebook 的網頁只要一改版，人都不會操作了，更何況機器 ? 至於 Facebook 的網頁介面有沒有防堵 headless browsing 的機制，我相信有。我們偶爾還是會看到 Facebook 跳出 CAPTCHA，但它有多強硬、是不是真的很難繞過，則還需要更多的測試才能得知。
其實，我們是無法阻擋使用者持續在 Facebook 上留下並揭露他們的數位足跡的。就算沒有了 Graph API，這些東西一樣看得到。
當你乖乖地照著 Facebook 的建議，編輯個人簡介、工作經歷、學歷、居住城市，你就已經把自己靈魂的一部分賣給 Facebook 了。如果你更聽話，在 profile 中告訴 Facebook 你喜歡聽什麼音樂、看什麼電影、在追什麼劇、喜歡哪個球隊，你就已經變成 Facebook 的數位資產。當你在 Facebook 上舉辦活動、邀請朋友、參加活動、管理行程，你就讓 Facebook 更知道你喜歡跟誰一起做什麼事、參加什麼活動。更不用說 Facebook 近期推出的找工作和拍賣等功能，幾乎是完完全全要將你的生活數位化地複製一份在它上面。
就像台灣的 104 人力銀行所擁有的資料可能比中華民國政府的戶政資料庫還豐富一樣，Facebook 掌握了全世界二十億人的數位足跡，它的規模之大、影響力之深，在人類歷史上絕對是前所未見。這次的劍橋分析事件只是個開端，它只是暴露出這個系統的威力，以及某些面相中它的脆弱之處。
要如何在數位生活的便利和個人隱私之間取得平衡，這個挑戰，才剛開始。
 
