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
    1. 若付費成功，偵測目前WebView的Url是否有 ``return-agreement`` 或 ``return-payment``，代表訂閱成功，當使用者關掉WebView之後使用GET api/User API取得使用者的新資料。
    1. 若付費失敗或使用者自行取消，會回到 ``cancel-agreement``或``cancel-payment``，代表交易取消，使用者關掉WebView之後，不須重新撈取資料
    
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

# 列舉型態
- ## <a name="AcctStatusEnum"></a>AcctStatusEnum (帳號啟用狀態)
    ```csharp
    public enum AcctStatusEnum
    {
        UnConfirm = 0,  //帳號還未確認信箱
        Active = 1,     //帳號已經啟用信箱
        Suspend = 2,    //帳號已被管理員Suspend
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
        UserExpired = 10,                   //帳號逾期
        SyncNextround = 11,                 //代表資料與Client端預期的已經不同，請再同步一次
        LocalServerTimeExceed = 12,         //Local端與Server端時間超過15分鐘
        PaypalAgreementCancelWhenPending = 101, //Paypal定期定額訂單取消已通知但是目前為Pending        
        UnknownError = 9999                 //未知的錯誤
    }
    ```
- ## <a name="NoticeTypeEnum"></a>NoticeTypeEnum (通知類型)
    ```csharp
    public enum NoticeTypeEnum
    {
        ProjectComment,     //專案的評論
        ObjectComment,      //物件評論
        StorageFullAlert,   //背包快滿的通知
        ExpiredAlert        //逾期通知
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
        Returned = 50       //已退貨
    }
    ```
- ## <a name="PeriodType"></a>PeriodType (定期類型)
    ```csharp
    public enum PeriodType
    {
        Daily = 0,      //每日
        Monthly = 1,    //每月
        Yearly = 2      //每年
    }
    ```
- ## <a name="PPUserOrderBy"></a>PPUserOrderBy (使用者排序方式)
    ```csharp
    public enum PPUserOrderBy
    {
        UserAccount = 0,    //根據帳號
        CreateTime = 1,     //根據建立時間
        LastLoginTime = 2   //根據上次登入時間
    }
    ```
- ## <a name="ProductAttribute"></a>ProductAttribute (商品屬性)
    ```csharp
    public enum ProductAttribute
    {
        Buyout = 0,     //買斷式商品
        Period = 1,     //期間式商品
        Times = 2       //次數性商品
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
        BasicMember = 0,        //一般測試用戶
        PaidMember = 1,         //付費用戶
        EnterpriseMember = 2,   //企業用戶
        Admin = 9               //管理員
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
