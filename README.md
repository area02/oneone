# AREA 02 x ONEONE 整合指南

## 簡介

本文件說明如何在 AREA 02 嵌入 ONEONE 活動頁面。主要涵蓋登入驗證、頁面導航和參數傳遞等功能。
需先提供 ONEONE 的活動網址，以供白名單設定。

## 基本設定

### AREA 02 來源設定

請先設定 AREA 02 的來源網址，範例為測試站：

```js
const area02Origin = 'https://staging.area02.site';
```

## 功能說明

### 1. 登入驗證

使用 `CHECK_LOGIN` 事件來驗證用戶登入狀態。

```js
window.parent.postMessage({
    type: "CHECK_LOGIN",
    query: {
        customParam1: 'value1',
        customParam2: 'value2',
    }
}, area02Origin);
```

### 2. 頁面導航

使用 `NAVIGATE` 事件進行頁面跳轉，支援內部路徑和外部連結。

```js
window.parent.postMessage({
    type: 'NAVIGATE',
    path: '/some-path' || 'https://www.example.com',
    query: {
        customParam1: 'value1',
        customParam2: 'value2'
    }
}, area02Origin);
```

### 3. URL 參數更新

使用 `UPDATE_URL_PARAMS` 事件動態更新 URL 參數。

```js
window.parent.postMessage({
    type: 'UPDATE_URL_PARAMS',
    query: {
        customParam1: 'value1',
        customParam2: 'value2'
    }
}, area02Origin);
```

## 事件監聽與處理

### 監聽父窗口事件

設置事件監聽器來處理來自父窗口的回應：

```js
window.addEventListener("message", (event) => {
    
    // 驗證消息來源
    if (event.origin !== area02Origin) return;

    try {

        const { type, data } = event.data;

        switch (type) {

            case 'PAGE_ENTER':

                // 頁面載入完成事件（最多等待 3000ms）

                const { userToken, urlParams } = data;
                console.log('User Token:', userToken);
                console.log('URL Parameters:', urlParams);

                // 在此處理頁面初始化邏輯

                break;

            case 'LOGIN_STATUS':

                // 登入狀態回應
                const { userToken: loginToken } = data;
                console.log("User token:", loginToken);

                // 在此處理登入狀態

                break;

        }

    } 
    catch (error) {

        console.error("訊息處理失敗:", error);

    }

});
```

## 事件回應說明

### PAGE_ENTER 事件

- 觸發時機：頁面載入完成後自動觸發
- 回傳資料：
  - `userToken`: 用戶驗證令牌，為空時表示未登入
  - `urlParams`: URL 查詢參數

### LOGIN_STATUS 事件

- 觸發時機：回應 `CHECK_LOGIN` 請求
- 回傳資料：
  - `userToken`: 用戶驗證令牌，為空時表示未登入

### 注意事項

1. 所有 postMessage 操作都必須指定正確的 `area02Origin`
2. 建議實作錯誤處理機制
3. 確保在使用 `userToken` 前進行有效性驗證

## JWT 說明

### 簽發

JWT 由 AREA 02 簽發，並透過 `userToken` 回傳給 ONEONE。

### 驗證

payload 包含：

```json
{"iss":"AREA 02","sub":"12345","aud":"oneone","iat":1747619228,"exp":1747705628}
```

- `iss`: jwt 發行者，固定為 `AREA 02`
- `sub`: token（AREA02 使用者唯一值）
- `aud`: 接收方，固定為 `oneone`
- `iat`: 發行時間（timestamp）
- `exp`: token 到期時間（timestamp）

演算法為：

```json
{
  "typ": "JWT",
  "alg": "RS256"
}
```

## 技術支援

如有任何技術問題，請聯繫 AREA 02 技術支援團隊。
