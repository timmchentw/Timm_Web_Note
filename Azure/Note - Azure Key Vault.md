# Azure Key Vault

## Soft Delete

刪除Key後會在一段時間保留檔案

* [Purge Protection](https://learn.microsoft.com/zh-tw/azure/key-vault/general/soft-delete-overview?WT.mc_id=Portal-Microsoft_Azure_KeyVault#permitted-purge)

    預設不開啟 (開啟後不可重設)，強制不能刪除Soft delete的Key，直到到期後才會自動刪除，適合資料加密等保護情境

## [Access model](https://learn.microsoft.com/en-us/azure/key-vault/general/security-features?WT.mc_id=Portal-Microsoft_Azure_KeyVault#access-model-overview)


* Vault access policy
  
    每個User各自設定Key, Secret, Certificate權限

* Azure role-based access control

  有Role的概念，方便批次調整

## Resource Access

允許下方資源直接取得Key Vault資源

* Azure VM
* Azure Resource Manager
* Azure Disk Encryption

## Network

可限制存取IP、網段等

## Secret

* 可儲存密碼文字
* 可上傳Certificate檔案
* 可設定Enabled, 起始 & 失效時間

## Key

## Certificate

## [Best Practice](https://learn.microsoft.com/en-us/azure/key-vault/general/best-practices)

    * 不同環境使用不同Vaults
    * 開啟Logger與Warning
    * 備份
    * IP限制來保護


## Application

* Secret Manager
