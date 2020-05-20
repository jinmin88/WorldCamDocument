# WorldCam API基礎說明
## API 呼叫方式
每個Restful API在呼叫時，都必須加入以下`Header`
```html 
Content-Type : application/json; charset=UTF-8
```
若該API需要登入後才能使用，則必須加入以下`Header`
```html
Authorization : Bearer [jwt_token]
```
其中`[jwt_token]`是由登入API(請見`[POST] api/JWT`)獲得。

每個api都可以設定語系，基本上都在api/後面加上語系即可，如`api/zh-TW/jwt`，目前支援`en`、`zh-TW`、`zh-CN`三種語系。

## Facebook登入
當使用Facebook登入時，需取得access_token後，把access_token於呼叫api/jwt時順便丟給server進行身分驗證。

而facebook的相關參數為：appId: `2116137285265889  `

## Google+登入
與facebook相同，client端一樣需取得access_token。

Google+相關參數為：clientId: `946399151159-2bfguo8hba3j8bt52iu8k06rlcrdf3d9.apps.googleusercontent.com`

## 帳號註冊時的ReCaptcha驗證
註冊帳號時，需進行Google ReCaptcha驗證，證明使用者並非機器人，故註冊帳號時，也須先取得recaptcha的response後當作註冊中的其中一個參數丟給server。

iOS(使用Invisible) : data-sitekey: `6Lc3jiAUAAAAANgiTfq9s75eqCQ_52D5i3ekMyde`

ReCaptcha相關參數為： data-sitekey: `6LegjyAUAAAAAPiileOTDOd6ePAqSXfBL2JtTbNa`

Android(使用Android Recaptcha)：data-sitekey: `6LcelWwUAAAAALBz55ji8NQJj-tTwGoBPPW8kBJz`

## 名詞介紹

WorldCam每個使用者都會有多個**專案**。而一個專案可以有很多**場景**，一個場景會有多組不同種類的**標記**。基本上，每上傳一張全景圖，等於多一個新的場景，而在場景內增加轉場連結連到另外一個場景則稱作**轉場標記**。

# 流程呼叫情境介紹
## 註冊帳號
呼叫 ``[POST] api/{culture}/user`` 進行帳號註冊
請見 Model ``JUserPost``資料結構
```json
{
  "userAccount": "test_account@penpower.com.tw",
  "password": "test1234",
  "nickname": "test account",
  "reCaptchaResponse": "03AHqfIOmdG-e8hGk_iNMvtvHva6JuoCwXh33VEH6xMEW61VjGVuw3LG4uub3QebW1lcctY_DYNgfQTx2gyJEEgDCO-MK4T7XZlkkCZSntQqxKFkr5nw5oFdXef2XXGKm1Hpq3Hr4V49WzEFk-GqS61K-Kq7Ej7FZey63L3WI6WzyKkF-o8Ul9C1_4_aD1aE6R7-sZ0OJcISqeBpc2ScfumORuaYt8RX4ddFYKI7xkgA5yMU-tLLVbZ3SyHsVaWegEYSJrErhVzMroZGQIHL013Mz2LsI4CClS6tX3TH-3Z1jIRRErRecTWXM0BSoFFn2OHzrPQBK_Vl9XQE0CkbzHEpHasugbCNg0Yw65IEW6WDZPsUVY3Yp6xpIj5IPiL8LXz4eCdt5EoxI5qOh6QpTZ7EF9kWWwGBHUNQ000aLOhOX2fD-4Oq5Ve5B6wIbr7QKmBvHkl5_fHIu0"
}
```
其中，`reCaptchaResponse`是透過Google Recaptcha元件於使用者勾選我不是機器人之後，自動產生的token

## 登入系統
登入系統目前總共有三種方式，第一種是使用**帳號密碼登入**，第二種是使用**Facebook登入**，第三種是使用**Google+登入**，
登入需使用``api/{culture}/jwt`` API進行登入，根據swagger文件我們知道此api還需要參數``JCredential``物件，此物件根據狀況不同會有以下幾種行為。
* 登入
  * 帳密登入
  ```json
  {
    "grant_type": "password",
    "username": "test@pp.com",
    "password": "test123"
  }
  ```
  * Google+或Facebook登入
  ```json
  {
    "grant_type": "facebook",
    "access_token": "1234567"
  }
  ```
* 回傳值意義
    當登入成功後，系統會回傳`JResponse<JAcessToken>`資料結構給Client，如以下範例。
    ```json
    {
        "error_codes": 0,
        "message": "",
        "data": {
            "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJqaW5taW4ubGl1QGdtYWlsLmNvbSIsImlzcyI6IlBlbnBvd2VyVG9rZW5TZXJ2ZXIiLCJhdWQiOlsiaHR0cHM6Ly90ZXN0LndvcmxkY2FtMzYwLmNvbSIsImh0dHBzOi8vdGVzdC53b3JsZGNhbTM2MC5jb20iXSwianRpIjoiZmFiYjg0NGUtMWUxYy00MWNlLThkY2YtMDhmNzZhN2Y0MjVjIiwidXNlcm5hbWUiOiJqaW5taW4ubGl1QGdtYWlsLmNvbSIsIm5iZiI6MTUzNDI5ODUzMiwiZXhwIjoxNTM0MzAyMTMyfQ.uDUF53WN6DsBxgpHAHNzuDq4ZE5JeCCzK08z2bbB2Cs",
            "expires_in": 3600,
            "userId": "03018a99-018a-4572-4a5a-08d4f6a058e8",
            "userProfile": {
                "id": "03018a99-018a-4572-4a5a-08d4f6a058e8",
                "userAccount": "jinmin.liu@gmail.com",
                "nickname": "jinmin",
                "lastLoginTime": "2018-08-15T02:02:12.7809616Z",
                "lastLoginIP": "127.0.0.1",
                "authType": 0,
                "roleType": 1,
                "acctStatus": 1,
                "usedSize": 202068155,
                "packageSize": 1073741824,
                "createTime": "2017-09-08T09:59:51.250062",
                "modifyTime": "2017-12-05T09:11:52.804027",
                "activeStatus": 1,
                "expiredTime": "2019-03-30T10:01:29.838212",
                "productCode": "01640AFD",
                "isPeriodActive": false,
                "isUseVIPService": false
            }
        }
    }
    ```
    error_codes回傳OK(0)代表登入成功，data欄位則是存放`JAcessToken`物件。
Client端需要自行儲存接收到的`access_token`，`access_token`是未來再使用需要權限的
API時，所需要放在Header的參數，使用方式為在Header加入Key:`Authorization` Value:`Bearer [access_token]`

另外會收到expires_in的欄位，意思是此token在接收到之後，幾秒後會過期，這個範例寫的是3600秒(一小時)。除了access_token之外，您必須記錄登入成功收到
JAcessToken的時間，以及expires_in。為了防止token過期，您可以自行撰寫迴圈來判斷是否要刷新token。比如說您可以在快到期五分鐘前才呼叫刷新Token API。
* 刷新Token
```json

  {
    "grant_type": "refresh_token",
    "refresh_token": "123456780",
  }
```

## 取得專案列表
您可以呼叫 `[GET] api/{culture}/Project` API取得Project列表。或是呼叫api/{locale}/Project/{project_id}取得單一的完整Project。針對回應的JProject結構，重點是的描述畫面上欄位的對應方式。

* 預覽縮圖：請存取``proj.defaultPano.PreviewImage``的縮圖屬性取得完整縮圖URL。
* 專案狀態：請存取``proj.Status``，若為0(Closed)則關閉、1(Open)則開啟
* 專案名稱：請存取``proj.Name``
* 專案大小：請存取``proj.ProjectSize``，請欄位單位為byte。
* 建立日期：請存取``proj.CreateTime``，使用UTC+0，需自行轉成Client所在時區的時間。
* 檢視專案：請使用WebView存取以下網址。``{api_server_base_url}/editor/PanoViewer.html?projectid={projectid}&jwt_token={access_token}&jwt_expires_in={expires_in}&lang={zh-TW}``
* 產生分享連結：請使用``[POST] api/{culture}/Share``產生該專案的分享連結。

## PanoViewer.html  (action javascript)
透過雙方定義好的javascript來通知app，例如:按下離開``Android 是 Mobile.action(MobileActionEnum.Exit)``。
* Android：Mobile.action(MobileActionEnum)。
* IOS： webkit.messageHandlers.messageHandler_action.postMessage(MobileActionEnum);。

## 建立專案
呼叫``[POST] api/{culture}/Project`` 建立空的新專案。

## 建立場景
建立場景則需要以下一連串的動作
1. 首先根據全景圖建立一張縮圖。
2. 使用``[POST] api/{culture}/Image``上傳縮圖，得到previewImageId。
3. 使用``[POST] api/{culture}/Image``上傳全景圖原檔，得到panoImageId。
4. 呼叫``[POST] api/{culture}/Project/{project_id}/pano``新增場景，回傳新的Project結果。

## Google Map 選擇WebView
呼叫範例
``[GET] https://test.worldcam360.com/editor/editgooglemap.html?lang=zh-TW&lat=6.86635829081027&lng=-75.17623924999998&address=Colombia``
您可輸入以下參數
1. lang: 語系 zh-TW / zh-CN / en
2. lat: 經度
3. lng: 緯度
4. address: 地址

當使用者搜尋某個地址之後，網址的lat/lng/address會自動變化。

## 使用者Profile
點選一般資訊時，需使用 ``[GET] api/{culture}/User/{user_id}`` 重新抓取自己的資訊(JUser)。填入以下對應欄位：
1. 用戶名稱：user.NickName
2. 用戶角色：user.RoleType
3. 啟用序號：user.ProductCode
4. 已使用空間：user.UsedSize
5. 總空間：user.PackageSize
6. 到期日：user.ExpiredTime (須從UTC+0轉成Client端時區)
7. 訂閱狀態：user.IsPeriodActive

## 顯示訂閱按鈕以及訂閱WebView
使用者基本資訊的訂閱狀態，顯示訂閱按鈕的規則如下
1. user.RoleType = 0 (一般測試用戶) : 顯示訂閱按鈕
1. user.RoleType = 1 (付費用戶)： 顯示訂閱按鈕
1. user.RoleType = 2,9 (企業用戶/管理員) : 皆不顯示訂閱按鈕

最後訂閱按鈕連到以下的webView ``{base_url}/MobilePricing?jwt_token={token}``

## 回應物件基本結構
基本上，所有api的回應都會使用`JResponse`物件包住，`JResponse`物件包含兩個欄位，`error_codes`代表執行的結果，通常如果API執行正確，都會RETURN OK(0)，若發生驗證資料錯誤(如新增帳號時傳入的帳號格式不合法)，Server則會根據您傳入的語系`{locale}`將錯誤的訊息內容回傳到`message`欄位給client，通常只要alert告知使用者錯誤的內容為何即可。

若api有回傳其他物件，則會放在`data`欄位中，如`[POST] api/jwt`登入API回傳的型態為`JResponse<JAccessToken>`，若登入成功，則會收到以下的Response：
```json
{
    "error_codes": 0,
    "message": "",
     "data": {
        "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJqaW5taW4ubGl1QGdtYWlsLmNvbSIsImlzcyI6IlBlbnBvd2VyVG9rZW5TZXJ2ZXIiLCJhdWQiOlsiaHR0cHM6Ly90ZXN0LndvcmxkY2FtMzYwLmNvbSIsImh0dHBzOi8vdGVzdC53b3JsZGNhbTM2MC5jb20iXSwianRpIjoiZmFiYjg0NGUtMWUxYy00MWNlLThkY2YtMDhmNzZhN2Y0MjVjIiwidXNlcm5hbWUiOiJqaW5taW4ubGl1QGdtYWlsLmNvbSIsIm5iZiI6MTUzNDI5ODUzMiwiZXhwIjoxNTM0MzAyMTMyfQ.uDUF53WN6DsBxgpHAHNzuDq4ZE5JeCCzK08z2bbB2Cs",
        "expires_in": 3600,
        "userId": "03018a99-018a-4572-4a5a-08d4f6a058e8",
        "userProfile": {
            "id": "03018a99-018a-4572-4a5a-08d4f6a058e8",
            "userAccount": "jinmin.liu@gmail.com",
            "nickname": "jinmin",
            "lastLoginTime": "2018-08-15T02:02:12.7809616Z",
            "lastLoginIP": "127.0.0.1",
            "authType": 0,
            "roleType": 1,
            "acctStatus": 1,
            "usedSize": 202068155,
            "packageSize": 1073741824,
            "createTime": "2017-09-08T09:59:51.250062",
            "modifyTime": "2017-12-05T09:11:52.804027",
            "activeStatus": 1,
            "expiredTime": "2019-03-30T10:01:29.838212",
            "productCode": "01640AFD",
            "isPeriodActive": false,
            "isUseVIPService": false
        }
    }
}
```

## app編輯用的網頁
panoeditormobile.html這個網頁是手機編輯專用的網頁。
請使用WebView存取以下網址。``{api_server_base_url}/editor/panoeditormobile.html?projectid={projectid}&jwt_token={access_token}&lang={zh-TW }``
1. lang: 語系 zh-TW / zh-CN / en
2. jwt_token: token
3. projectid: 專案id
4. updatesceneid:用於MobileActionEnum.UpdatePano 告訴app 目前更新那個全景的id
5. looksceneid: 指定一開始要看那個全景,用於MobileActionEnum.UpdatePano,當app選好圖上傳完圖後,返回webview,可以看到更新後的全景
6. lookmapid: 指定一開始要看那個平面圖,用於MobileActionEnum.AddPlan,當app選好圖上傳完圖後,返回webview,可以直接返回編輯平面圖

## 訂閱流程
描述個人資訊頁面上的訂閱按鈕之行為。
1. 按鈕顯示條件，只有當以下條件成立才顯示，其他狀況則不顯示。
    1. user.RoleType == RoleTypeEnum.BasicMember || user.RoleType == RoleTypeEnum.PaidMember
1. 按下訂閱按鈕流程：
    1. 開啟WebView並連至``{api_server_base_url}/{culture}/MobilePricing?jwt_token={jwt_token}``
    1. 若付費成功，偵測目前WebView的Url是否有 ``return-agreement`` 或 ``return-payment``或``stripe/return``，代表訂閱成功，當使用者關掉WebView之後使用GET api/User API取得使用者的新資料。
    1. 若付費失敗或使用者自行取消，會回到 ``cancel-agreement``或``cancel-payment``或``stripe/cancel``，代表交易取消，使用者關掉WebView之後，不須重新撈取資料
    
## 序號流程
描述個人資訊頁面上的序號按鈕之行為。
1. 按鈕顯示條件，只有以下狀況顯示
    1. user.RoleType == RoleTypeEnum.BasicMember || user.RoleType == RoleTypeEnum.PaidMember
1. 按下輸入序號流程
    1. 使用修改使用者資料api 填入ProductCode之後修改即可。
    1. 若修改成功，重新抓取使用者資料更新之
    1. 若失敗，彈出server給的訊息即可
    
# asteroom 2.0.0修改事項
## 會員個人資訊，新增電話以及個人大頭照圖片
1. 原本會員JUser資料結構新增 Phone/ProfileImage資料結構分別顯示客戶的電話以及大頭照
1. 使用[PUT] api/{culture}/User/{user_id} API修改資料，JUserPut資料結構新增兩個欄位 phone以及profileImageId，如果塞null則代表不更新，大頭照的上傳流程與之前一樣，先使用image post 傳圖取得imageId之後塞至profileImageId。

## Leadgen管理
1. Leadgen管理提供了三個槽狀資料結構JLeadgenMain/JLeadgenProjGrop/JLeadgen 達成顯示此功能的所有資訊。 
1. JProject資料結構新增一個欄位LeadgenMainCount，用於顯示每個專案的Leads數字。
1. 前台Viewer客戶要填寫客人資料時，請呼叫[POST] api/{culture}/Leadgen，body填入JLeadgenPost資料結構，新增Leadgen資料
1. 當點選專案的Leads按鈕時，呼叫 [GET] api/{culture}/LeadgenMain，查詢LeadgenMain列表，詳細查法請看swagger 此api介紹
1. 當需要更新LeadgenMain的電話/客戶姓名/備註時，請呼叫 [PUT] api/{culture}/LeadgenMain 使用JLeadgenMainPut物件更新欄位內容

## 轉移專案按鈕
1. 轉移按鈕一律是Enabled
1. Client端點擊此按鈕之後，檢查Client端User的條件，若角色為BasicMember或是帳號已經過期，請跳出請訂閱之類的對話方塊。
1. 若User的AcctStatus為Active且RoleType為PaidMember/EnterpriseMember的話，才開始轉移專案的流程
1. 跳出一個Dialog讓使用者鍵入要轉移的目的帳號，這個Dialog同時有一個textbox與一個comboBox，comboBox使用取得常用帳號的api取出最近常用的帳號列表
1. 取得喜好帳號列表API：[HTTPGET] api/{culture}/User/PerferredAccount，會回傳List<JUserPerferredAccount>資料結構並顯示在combobox
1. 使用者可透過textbox輸入帳號或由combox選擇常用帳號(擇一)輸入to_account
1. 呼叫轉移專案API: [HTTPPUT] api/{culture}/Project/{project_id}/Transfer?to_account={to_account}
1. API或執行失敗：則跳出server給的錯誤訊息
1. API若執行成功：則重新抓取User Profile(因為UsedSize會變小)以及重新抓取專案列表(因為少一個專案)
1. 以上UI還需等JASON設計

# asteroom 2.2修改事項
## 專案的GA統計報表
1. 需要使用 [GET] api/{culture}/GAReport/{project_id} 取得報表資料(需要登入，請參見GAReport)
1. 相關資料結構請參閱Swagger, 可以先使用https://test.asteroom.com/api/GAReport/4d21ecf4-e41e-4fe9-99e4-26698531ddc2 觀看大概產出的JSON內容。
1. 待CLIENT端撰寫完成之後，會改回只能檢視自己的專案GA結果

## 加入FCM cloud messaging功能
1. 原本更新User基本資料的api  [PUT] api/{locale}/User/{user_id} 新增三個欄位webFCMToken / androidFCMToken / iosFCMToken，各平台登入之後取完fcm token或刷新fcm token時需要呼叫此api更新各自平台的fcm token.
1. 各平台相關設定檔會另外再寄信通知

# asteroom 2.5修改事項
1. 原企業版改為收取120美金一個月，擁有10G的空間以及Agent管理功能
1. 會員的角色新增一級：客製版(RoleType=7)
  1. 客製版會員可由後台調整  
  1. App的購買按鈕當為客製版時不顯示
  
# asteroom 2.5修改事項
## Dollhouse購買流程
1. 專案列表API所回傳的資料結構，增加dollTasks陣列，代表這個project目前的建立Dollhouse任務的清單
1. 第二個dollFloors資料結構，代表這個project目前有的Dollhouse (如果dollFloors.Count>0代表有Dollhouse)
1. 撰寫findActiveTask(Project proj)
    ```csharp
    public DollTask findActiveTask(Project proj) {
        foreach (var task in proj.dollTasks) {
            if (task.Status == TaskStatus.Acceppted ||
                task.Status == TaskStatus.Completed ||
                task.Status == TaskStatus.Feedback) {
                return task;
            }
        }
        return null;
    }
    ```
1. 專案列表中，每個專案圖片右上角，使用以下邏輯顯示小ICON
   ```csharp
   var activeTask = findActiveTask(project);
   if (activeTask != null) {
       if (dollTask.DollFloors.Count > 0) {
           //顯示Dollhouse建置中圖示
           //顯示Dollhouse按鈕
       }
       else {
           //顯示Dollhouse建置中圖示
           //顯示Dollhouse按鈕
       }
   }
   else {
       if (dollTask.DollFloors.Count > 0) {
           //顯示Dollhouse已完成圖示
           //選單不顯示任何按鈕
       }
       else {
           //不顯示任何圖示
           //選單顯示Dollhouse按鈕
       }
   }
   ```
1. Dollhouse按鈕行為
    * 使用上一節的方式找activeTask
    * 當project.DollFloors.Count == 0且project的activeTask == null
        * 顯示第一次購買Doorhouse視窗，上面有個購買按鈕
        * 呼叫[POST] /api/{locale}/Order with body ```{"pricing_item_id" : "P_DOLLHOUSE", "project_id": "{project_id}"}```    
        * 會回傳JResponse<string>結構，如果失敗則顯示訊息，成功的話data回傳一串URL，此時將網頁導向這個url進入購買流程
    * 當project.DollFloors.Count == 0 且 activeTask.Status != null
        * 如果activeTask.Status == TaskStatus.Accepted
            * 顯示任務處理中視窗
        * 如果activeTask.Status == TaskStatus.Feedback
            * 顯示任務處理中視窗 (強調是處理Feedback案件)
        * 如果activeTask.Status == TaskStatus.Completed
            * 顯示結案及Feedback視窗
                * 選結案 : 使用 ```[PUT] api/DollTask/{id}``` api更新狀態
                * 填入字串後選Feedback : 使用 ```[PUT] api/DollTask/{id}``` api更新狀態
    * 當project.DollFloors.Count > 0 且 activeTask == null
        * 不顯示Dollhouse按鈕
    * 當project.DollFloors.Count > 0 且 activeTask != null
        * 如果activeTask.Status == TaskStatus.Feedback
            * 顯示處理中視窗
        * 如果activeTask.Status == TaskStatus.Completed
            * 顯示結案及Feedback視窗
                * 選結案 : 使用 ```[PUT] api/DollTask/{id}``` api更新狀態
                * 填入字串後選Feedback : 使用 ```[PUT] api/DollTask/{id}``` api更新狀態

# asteroom 2.6修改 事項
1. 原本的api/DollTask/{project_id} 取回的資料結構中，新增一個int類型的欄位panoCount，目的是顯示目前有幾張全景圖。
2. MobileActionEnum列舉新增BuyDollhouse = 17,//顯示購買dollhouse視窗
3. panoeditormobile.html 新增本版參數 ex: version=2019-09-02 (yyyy-mm-dd) 年月日就可以了

# asteroom 2.8 修改事項
1. Pricing方案修改
    * 未來版本分成下列幾種版本：
        * 專業版：每月收取$19.9USD，一次購買一年$199USD，有5個Active Project和10GB storage。
        * 企業版(25)：每月收取$59USD，一次購買一年$590USD，有25個Active Project和100GB storage，並且最多可建立10個子帳號(含自己)。
        * 企業版(50)：每月收取$99USD，一次購買一年$990USD，有50個Active Project和100GB storage，並且最多可建立25個子帳號(含自己)。
        * 企業版(100)：每月收取$99USD，一次購買一年$990USD，有100個Active Project和100GB storage，並且最多可建立50個子帳號(含自己)。
    * 收費頁面原本可以選1至12個月，現在改成可選1至6個月
    * 收費頁面的Return URL改為返回第一次進入Payment頁面的Referer URL。
2. 企業版子帳號修改
    * RoleTypeEnum新增一個列舉Child=6，代表這個User的角色是子帳號。
        * **請各個Client端使用新的RoleTypeEnum顯示版本資訊顯示的版本字串**
        * **當RoleType=6時不要顯示購買按鈕**
    * AppUser資料表新增一個ParentId的欄位，指的是父親帳號的UserId。
    * 廢除原本的Agent Management，改用子帳號維護的方式運作。
    * 新增/api/ChildAccount等CRUD API，提供Web端實作子帳號的CRUD。
    * JUser對應子帳號增加以下欄位
        * int ChildAccount：假設是企業版帳號，此欄位代表可以建立多少子帳號。
        * int ActiveCount：代表這個帳號可以開啟多少個專案的分享狀態。
        * int IsActiveShareable：代表這個帳號是否與母帳號分享ActiveCount個數(假設子帳號開1個，母帳號開2個，則母帳號與子帳號可視為已經開啟3個)
        * int EntActiveCount：代表企業版帳號最多可以Active的Project Count。
        * int ProjCount：目前此帳號所有的專案數量
        * int OpenProjCount：目前此帳號所有專案Status=Open的數量
        * int IsNeedResetPassword：是否需要重設密碼
            * **請各Client當登入之後發現JUser.IsNeedResetPassword為True的狀態下，請強制跳出更換密碼的UI，當用戶重設完密碼之後，此Flag會自動變成false，下次就不需要強制更換密碼了。**
    * 已分享的專案數 / 可分享的專案數 顯示邏輯如下
        * 當為企業版時：顯示 uesr.OpenProjCount / user.ActiveCount
        * 當為Child版本且IsActiveShareable==true時：顯示 user.Parent.OpenProjCount / user.Parent.ActiveCount
        * 當為Child版本且IsActiveShareable==false時：顯示 user.OpenProjCount / user.ActiveCount
        * 當為其他角色時：顯示 user.OpenProjCount / user.ActiveCount
3. **廢除Agent Management，JProject新增一個Bool IsShowUserProfile取代**
   * **原本新增專案時，選擇Agent時使用JProjectPost.IsShowUserProfile來代表此Project是否要顯示經紀人資訊。**
   * **原本修改專案時，選擇Agent時使用JProjectPut.IsShowUserProfile代表修改是否要顯示經紀人資訊。**
4. **各個Client的Version增加顯示Maximum number of active tours (當roleType=6時，判斷IsActiveShareable==true時使用user.Parent.ActiveCount，否的話用user.ActiveCount，當RoleType為其他時採用user.ActiveCount)，並放在第一列**
5. **各個Client的Version顯示的容量，當帳號角色為6(企業版子帳號)時，採用user.Parent.UsedSize取代原本的user.UsedSize**
6. **當專案為空專案時，就算用戶有上傳自己專案的封面圖，我們也不顯示出來**
7. 所有需要Authrize的API以及refresh token時，需判斷回傳的JResponse.ErrorCodes執行以下流程(原本的ErrorCodes 10改成UserSuspend)：
    * 當response.error_code == ErrorCodes.UserNotExist(9) 或 ErrorCodes.UserSuspend(10)，顯示訊息之後幫用戶登出並回到登入畫面。
8. 所有client在專案列表進入leangen頁面之後，若呼叫ga api得到 response.error_code == ErrorCodes.ProjectNotExist(8)時，請關閉目前ga畫面並回到專案列表中(需要重整專案列表)
9. 修改個人資訊的版本，顯示**已啟用專案數/配額專案數(剩餘可分享專案數)**，對應到user.OpenProjCount/user.ActiveCount(user.AvaiableProjCount)。
10. 當用戶所使用的企業版為50人以上(含50)，取消顯示Powerer by asteroom。

# asteroom 2.9 修改事項
1. 新增免費送Dollhouse功能，異動如下：
    * JUser新增一個欄位，叫做FreeDollhouseCount，紀錄還有多少免費製作Dollhouse的次數。
    * 原本按下購買Dollhouse的API([GET]/api/DollTask/{project_id})取回資料的時候，JDollTaskData多一個欄位bool IsFree，代表此次購買是否可以免費。
    * 當IsFree為false的時候走舊有的流程：下訂單後付款完才產生Dollhouse任務單。
    * 當IsFree為true時，Pricing顯示$0，原本購買的按鈕改成顯示"免費"，按下之後呼叫以下的api：
    * [HttpPost]/api/DollTask，Body傳入購買的project_id，假設成功之後，請顯示成功訊息後，重整專案狀態，並回到列表。
    * 測試方法：
        * 可以到測試後台=>行銷管理=>GiftCard管理=>選"一個月GiftCard"的活動=>複製一組可用的序號後，登入試用帳號後輸入此序號，即可以升級成專業版並得到一個免費製作Dollhouse的機會
        * 可以請Jeff幫你的帳號增加FreeDollhouseCount值
2. 新增影像處理流程，須連Dollhouse流程一併修改
    * 購買按鈕顯示與否邏輯
    ```csharp
    bool IsFree = user.FreeDollhouseCount > 0 ? true : false;
    bool IsShowDollhouseItem = project.DollFloors.Count > 0 ? false : (project.DollTasks.Count(a=>a.Status == TaskStatus.Accepted || a.Status == Task.FeedBack || a.Status == Task.Completed) > 0 ? false : true);
    bool IsShowImageProcessingItem = project.HasImageProcessing == true ? false  : (project.ImageTasks.Count(a=>a.Status == TaskStatus.Accepted) > 0 ? false : true);
    if (IsShowDollhouseItem || IsShowImageProcessingItem) {
        //Display the buy button
    }
    ```
    * 顯示第一個畫面
        * 如果IsShowDollhouseItem == true，則呼叫 [GET]/api/DollTask/{project_id}取得商品價格以及任務所需的處理時間等資訊後，加入到List中。
        * 如果IsShowImageProcessingItem == true，則呼叫[GET]/api/ImageTask/{project_id}取得商品價格以及任務所需的處理時間等資訊後，加入到List中。
    * 顯示第二個畫面
        * 假設IsFree為True
            * 當IsShowDollhouseItem==true且IsShowImageProcessingItem==true時，勾選行為為一起勾選或一起取消勾選，按鈕字樣為免費，按下之後呼叫購買免費Dollhouse API。
            * 當IsShowDollhouseItem==true且IsShowImageProcessingItem==false時，按鈕字樣為免費，按下之後呼叫購買免費Dollhouse API。
            * 當IsShowDollhouseItem==false且IsShowImageProcessingItem==true時，按鈕字樣為購買，按下之後呼叫購買影像處理項目API。
        * 假設IsFree為False
            * 勾選行為不作連動，勾甚麼則呼叫對應的購買API。
    * 購買API介紹
        * 免費Dollhouse API：[POST]/api/DollTask with JSON body {"project_id" : "{project_id}" }
        * 只購買Dollhouse API：[POST]/api/Order with JSON body {"pricing_item_id":"P_DOLLHOUSE","project_id" : "{project_id}"}
        * 只購買ImageProcessing API：[POST]/api/Order with JSON body {"pricing_item_id": "P_IMAGEPROCESSING", "project_id": "{project_id}" }
        * 兩者一起購買：[POST]/api/Order with JSON body {"pricing_item_id": "P_DOLLHOUSE_AND_IMAGEPROCESSING", "project_id": "{project_id}" }
        * 呼叫完之後redirect url到api給定的訂單網頁即可

# asteroom 3.0 客製版修改事項
1. RoleType新增腳色7(Custom)，為客製版腳色，與企業版相同，可以建立子帳號，但是差異在於客製版和其子帳號的收費方式(專案分享/Dollhouse/影像處理)，為每個月1號收上個月所使用的量來收費。
2. 可於測試後台的帳號明細頁，將您的帳號升級為客製版帳號。升級之後，會將所有的專案關閉，重新計算價錢。
3. 版本資訊頁，若角色為7(Custom)，顯示"客製版"，若為6(Child，Parent的角色為7)，則顯示"客製版"字串。
4. 與企業版相同，需要顯示子帳戶管理功能，與企業版的管理功能一模一樣。
5. [GET] api/DollTask與[GET] api/ImageTask 回傳的資料多一欄isCustom，可讓client端判斷是否為客製版。
6. 顯示Dollhouse以及PanoRetouch時，使用[GET]api/DollTask/{project_id}以及[GET]api/ImageTask/{project_id}取得Dollhouse以及PanoRetouch的IsFree/Price/ProcessDays/PanoCount四個值，當IsFree=false時，跟以前一樣呼叫[POST]api/Order新增Dollhouse以及訂單。若IsFree為true的話，執行第6步的步驟。
7. 當IsFree=true時，有買Dollhouse呼叫[POST]api/DollTask/v3 新增Dollhose任務單，使用以下輸入
```json
{
  "projectId": "xxxxxxxxxxxxxxxxxxxx",
  "pricing_item_id" : "P_DOLLHOUSE_AND_IMAGEPROCESSING"   //P_DOLLHOUSE_AND_IMAGEPROCESSING:兩者都買, P_DOLLHOUSE:只買DOLLHOUSE, P_IMAGEPROCESSING:只買影像處理
}
```
8. 當RoleType=7(客製版)，個人資訊新增一項Billing Info，使用[GET] api/Invoice 取得客製版帳號已經開完的所有Invoice。
9. 當您開啟專案/購買Dollhouse/購買影像處理訂單時，都會自動建立一筆InvoiceItem到資料庫中，這些InvoiceItem會在下個月一號，進行開立Invoice的動作，所以為了測試第七點，不可能等到下個月才看結果，所以這邊提供一個api可以把帳號目前尚未開立的InvoiceItem一起開立一張Invoice。此api請呼叫[GET] api/Stripe/create?userAccount={your_account}，呼叫完會馬上開立Invoice，這時第7部的api才能取到資料。

# asteroom 3.0 新增顯示email修改事項
1. JUser新增一個DisplayEmail欄位，顯示用戶要秀給client的聯絡用email
2. [PUT]api/User/{user_id} 新增displayEmail欄位，可以提供修改
3. 請在分享專案時，顯示此displayEmail

# asteroom 3.0 客製版本購買dollhouse修改
1. [GET] api/DollTask/{project_id} 增加兩個欄位IsEditable以及Hint兩個欄位
2. Dollhouse以及PanoRetouch的價格顯示都依照回傳的Amount欄位顯示
3. IsEditable意思代表此Dollhouse或PanoRetouching是否可以讓user取消選取
4. Hint欄位如果不是null，則在Dollhouse和PanoRetouching下面顯示Hint字串
5. IsFree欄位代表要call哪個api, true的時候call [POST]api/DollTask/v3 , false時call [POST]api/Order
6. 若想測試IsEditable=false的客製板狀態，請將客製版的設定Project price以及PanoRetouching的價格設定為0，dollhouse的設定設成1-30-60,2.5

# 列舉型態
- ## <a name="AcctStatusEnum"></a>AcctStatusEnum (帳號啟用狀態)
    ```csharp
    public enum AcctStatusEnum
    {
        UnConfirm = 0,  //帳號還未確認信箱
        Active = 1,     //帳號已經啟用信箱
        Suspend = 2,    //帳號已被停用
        Expired = 3     //帳號已經過期
    }
    ```
- ## <a name="AuthTypeEnum"></a>AuthTypeEnum (帳號登入方式)
    ```csharp
    public enum AuthTypeEnum
    {
        Custom = 0,         //一般使用帳號密碼登入
        Facebook = 2,       //使用Facebook登入
        GooglePlus = 3      //使用Google+登入
        Apple = 4,          //使用Apple登入
    }
    ```
- ## <a name="DeliveryStatus"></a>DeliveryStatus (訂單運送狀態)
    ```csharp
    public enum DeliveryStatus
    {
        Unfulfilled = 0,    //尚未出貨
        Shipped = 1,        //已經寄出
    }
    ```
- ## <a name="DiscountType"></a>DiscountType (折扣類型)
    ```csharp
    public enum DiscountType
    {
        FixedAmount = 0,    //定額折扣  10 => Product.Price - 10
        Percentage = 1      //百分比折扣  80% => Product.Price * 80%
    }
    ```
- ## <a name="DisplayUnit"></a>DisplayUnit (顯示量尺單位)
    ```csharp
    public enum DisplayUnit
    {
        Meter = 0,  //公尺
        Foot = 1    //英尺
    }
    ```
- ## <a name="DollFaceType"></a>DollFaceType (Dollhouse中，房間內每面牆壁的種類)
    ```csharp
    public enum DollFaceType
    {
        Floor = 0,  //底板
        Ceil = 1,   //天花板
        Wall = 2    //牆壁
    }
    ```    
- ## <a name="ErrorCodes"></a>ErrorCodes (錯誤代碼)
    ```csharp
    public enum ErrorCodes
    {
        OK = 0,                             //無任何錯誤
        CustomError = 1,                    //一般錯誤，請見Message錯誤訊息
        AccessTokenExpired = 2,             //存取Token過期
        InvalidParameter = 3,               //錯誤的參數
        InsufficientPermissions = 4,        //權限不足
        CaptchaError = 5,                   //Captcha錯誤
        SocialAccountNeedProductCode = 6,   //使用SSN登入需要註冊碼
        AccountUnConfirm = 7,               //帳號尚未認證
        ProjectNotExist = 8,                //專案不存在
        UserNotExist = 9,                   //帳號不存在
        UserSuspend = 10,                   //帳號逾期
        SyncNextround = 11,                 //代表資料與Client端預期的已經不同，請再同步一次
        LocalServerTimeExceed = 12,         //Local端與Server端時間超過15分鐘
        PaypalAgreementCancelWhenPending = 101, //Paypal定期定額訂單取消已通知但是目前為Pending        
        UnknownError = 9999                 //未知的錯誤
    }
    ```
- ## <a name="FCMMessageType"></a>FCMMessageType (FCM通知類型)
    ```csharp
    public enum FCMMessageType
    {
        TransferProjectSuccess = 1, //轉移專案成功
        NewIncomingLeadgen = 2,     //有新的Leadgen訊息進來
        DollTaskCreated = 3,        //Dollhouse Task已經建立 (有人下Dollhouse訂單)
        DollTaskCompleted = 4       //Dollhouse Task已經完成
        ImageTaskCreated = 5,
        ImageTaskClosed = 6
    }
    ```
- ## <a name="MailType"></a>MailType (信件類型)
    ```csharp
    public enum MailType
    {
ACTIVATION = 0,             //帳號啟用信
        RESET_PWD = 1,              //密碼重設信
        WELCOME = 2,                //歡迎信
        TENDAY_NOTIFICATION = 3,    //十天通知信
        THREEDAY_NOTIFICATION = 4,  //三天通知信
        REPORT = 5,                 //檢舉信
        ORDER_CONFIRMATION = 6,     //訂單確認信
        UPGRADE_TO_PRO = 7,         //升級成專業版通知信
        LEARN_HOWTO = 8,            //Learn Howto
        WELL_DONE_SHOOTING = 9,     //Well donw shooting first panorama
        CHECK_OUT_OTHER = 10,       //Check out other saying about asteroom
        HAVE_TROUBLE_STITCHING = 11,//Have trouble stitching
        WELL_DONE_STITCHING = 12,   //Well done stitching
        GETTING_STUCK = 13,         //Getting stuck
        YOU_HAD_A_VISITOR = 14,     //You had a visitor
        WAY_TO_GET_MORE = 15,       //Way to get more
        DOLLHOUSE_COMPLETE = 16,    //Dollhosue is complete
        ORDER_IN_PROCESS = 17,      //Dolltask or imagetask start to process
        PANORETOUCH_COMPLETE = 18,  //PanoRetouch order is complete
        TRIAL_21_CREATE_TOUR_IN_MINUTES = 21,
        TRIAL_22_3TIPS_TO_STANDOUT = 22,
        TRIAL_31_STANDOUT_WITH_VR = 23,
        TRIAL_41_STANDOUT_IN_DIGITAL_MARKETING = 24,
        TRIAL_42_FLOOR_PLAN = 25,
        TRIAL_51_INACTIVE_USER = 26,
        TRIAL_62_D10_NOTICE = 27,
        TRIAL_72_D3_NOTICE = 28,
        ACTIVATION_RDC_INSTA360 = 31,
        WELCOME_RDC_INSTA360 = 32,
        SYSTEM_EMAIL = 99
    }
    ```
- ## <a name="NoticeTypeEnum"></a>NoticeTypeEnum (通知類型)
    ```csharp
    public enum NoticeTypeEnum
    {
        ProjectComment,     //專案的評論
        ObjectComment,      //物件評論
        StorageFullAlert,   //背包快滿的通知
        ExpiredAlert,       //逾期通知
        NewTrasferProject,  //有新的轉移專案
        DollTaskCreated,    //有新的Dollhouse task
        DollTaskCompleted,  //Dollhouse task剛完成通知
        ImageProcessingCreated, //有新的影像處理task
        ImageProcessingClosed   //影像處理task結案
    }
    ```
- ## <a name="OrderStatus"></a>OrderStatus (訂單狀態)
    ```csharp
    public enum OrderStatus
    {
        Pending = 10,       //擱置中
        Processing = 20,    //處理中
        Completed = 30,     //已完成
        Cancelled = 40,     //已取消
        Returned = 50,      //已退貨
        PartialReturned = 51//部分退貨
    }
    ```
- ## <a name="PeriodType"></a>PeriodType (定期類型)
    ```csharp
    public enum PeriodType
    {
        Daily = 0,    //每日
        Monthly = 1,  //每月
        Weekly = 2,   //每週
        Yearly = 3    //每年
    }
    ```
- ## <a name="PPUserOrderBy"></a>PPUserOrderBy (使用者排序方式)
    ```csharp
    public enum UserOrderBy
    {
        UserAccount = 0,    //根據帳號
        LastLoginTime = 1,  //根據上次登入時間
        CreateTime = 2      //根據帳號建立時間
    }
    ```
- ## <a name="ProductAttribute"></a>ProductAttribute (商品屬性)
    ```csharp
    public enum ProductAttribute
    {
        Buyout = 0,     //買斷式商品
        Period = 1      //期間式商品
    }
    ```
- ## <a name="ProductType"></a>ProductType (商品類型)
    ```csharp
    public enum ProductType
    {
        DigitalGoods = 0,   //數位商品
        PhysicalGoods = 1   //實體商品
    }
    ```
- ## <a name="ProjectOrderBy"></a>ProjectOrderBy (專案排序方式)
    ```csharp
    public enum ProjectOrderBy
    {
        Title,          //根據專案標題排序
        CreateTime,     //根據專案建立時間排序
        ViewCount,      //根據觀看次數排序
        LikeCount,      //根據按讚次數排序
    }
    ```
- ## <a name="ProjectStatusEnum"></a>ProjectStatusEnum (專案狀態)
    ```csharp
    public enum ProjectStatusEnum
    {
        Closed = 0,     //已關閉
        Open =1,        //開放中
    }
    ``` 
- ## <a name="RoleTypeEnum"></a>RoleTypeEnum (帳號角色)
    ```csharp
    public enum RoleTypeEnum
    {
        BasicMember = 0,        //試用版 (Trial User)
        PaidMember = 1,         //專業版 (Professional User
        EnterpriseMember = 2,   //企業版 (Enterprise User)
        Child = 6,              //子帳號 (Enterprise-Child User)
        Custom = 7,             //客製版用戶 (Custom User)
        Operator = 8,           //操作員 (Operator)
        Admin = 9               //管理員 (Administrator)
    }
    ```
- ## <a name="SetupActionEnum"></a>SetupActionEnum (場景內標示物件種類)
    ```csharp
    public enum SetupActionEnum
    {
        Transition = 0,     //場景轉移
        Description = 1,    //描述物件
        Text = 2,           //文字物件
        Image = 3,          //影像物件 
        Object3D = 4,       //3D物件
        Video = 5,          //影片物件
        Text3D = 6,         //3D文字物件
        Link = 7            //超連結物件
    }
    ```
- ## <a name="SetupIconEnum"></a>SetupIconEnum (場景內標記物件的圖示種類)
    ```csharp
    public enum SetupIconEnum
    {
        None = 0,           //無圖示
        Transition_Up,      //上
        Transition_Down,    //下
        Transition_Left,    //左
        Transition_Right,   //右
        Transition_Circle,  //圈圈
        Description,        //描述
        TextIcon,           //文字
        VideoIcon           //影像
    }
    ```
- ## <a name="UploadTypeEnum"></a>UploadTypeEnum (上傳種類)
    ```csharp
    public enum UploadTypeEnum
    {
        Local = 0,              //上傳到本地
        GoogleCloudStorage = 1  //上傳到GCS
    }
    ```
- ## <a name="VideoTypeEnum"></a>VideoTypeEnum (影片種類)
    ```csharp
    public enum VideoTypeEnum
    {
        File = 0,       //檔案
        Youtube = 1     //Youtube連結
    }
    ```
- ## <a name="PlatformTypeEnum"></a>PlatformTypeEnum (Client端平台種類)
    ```csharp
    /// <summary>
    /// Client平台類別
    /// </summary>
    public enum PlatformTypeEnum
    {
        Windows = 0,
        Android = 1,
        IOS =2
    }
    ```
    - ## <a name="MobileActionEnum"></a>MobileActionEnum 
    ```csharp
    public enum MobileActionEnum
    {
        Exit  = 0,  //離開APP
        GoVR = 1,    //進入VR模式
        ExitVR = 2,   //離開VR模式
        Unauthorized = 3,//401(token失效)
        AddPano = 4, //編輯模式按下新增全景圖
        AddPlan = 5, //編輯模式按下新增平面圖
        UpdatePano = 6,//編輯模式按下更新全景圖
        AddImageToImageObject = 7,//編輯模式按下新增圖片物件
        AddImageToDescObject = 8,/編輯模式,編輯描述物件時,按下新增圖片
        AddImageToLogoList = 9, //編輯模式,在腳架資訊時,按下新增圖片
        AddAudioToEditProjectInfo = 10,//編輯模式,在編輯專案資訊時,按下新增背景音樂
        ExitAndNeedUpdateProject = 11,//編輯模式,按下離開且有更新時,app需要更新目前專案資訊記憶體和UI
        ProjectDoesNotExist = 12,//編輯模式,因為專案不存在(當下可能被其他平台刪除),所以app收到這個訊息,要離開且更新目前專案資訊記憶體和UI
        ShareProject = 13,//檢視按下分享
        OpenWebLink = 14,//開啟網頁連結
        OpenVideoLink = 15,//開啟影片連結
        WebReday = 16,//網頁也經載好 
        BuyDollhouse = 17,//顯示購買dollhouse視窗
    }
    ```
