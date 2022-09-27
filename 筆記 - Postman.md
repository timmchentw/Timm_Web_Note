# Postman

## Environment

右上角眼睛Icon可設定Environment

1. 新增Environment & Set Variable Values
   ![image](/images/postman/1.png)
2. 眼睛Icon旁邊可下拉切換Environments
3. 在要使用的地方輸入{{your_var_name}}即可根據當下Environment填入相對應的值 (URL, Parameters, Headers, Token皆可以用)
   ![image](/images/postman/2.png)
4. 使用以下script在"Tests"頁籤，即可將response body中的token設定到該Environment的Variable

    ```javascript
    var response = pm.response.json();
    pm.environment.set("api_token", response.access_token); // access_token名稱跟隨API回傳格式而定
    ```

5. 在API要使用token時，填入{{api_token}}自動讀取
