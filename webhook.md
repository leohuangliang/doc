# 1.5.1.4、卡授权交易通知回调 
## 简要描述 
• 卡授权交易通知回调 
• Message-Type:VirtualAuthorizations 
## 请求 URL 
• {{ baseUrl }}/webhook 
## 请求方式 
• POST 
## 请求类型 
• Content-Type application/json 
#### 参数 
参数名 类型 说明 
authorization_id string 交易 id 
virtual_id string 卡 id 
card_number string 卡号 
amount string 金额 
currency string 币种 
transaction_type string 交易类型(Auth,Reversal,Return） 
transaction_status string 交易状态 状态 Enums in (Failed,Success) 
fail_reason string 失败原因 
authorization_time string 授权时间 
#### 返回示例 
{ 
 "code": 200, 
 "msg": "OK", 
 "result": { 
 "code": "0000", 
 "data": { 
 "amount": "30.00", 
 "auth_code": "", 
 "authorization_id": "172630089001571614", 
 "authorization_time": "2024-08-09 18:06:38", 
 "card_number": "223467XXX5791", 
 "currency": "USD", 
 "fail_reason": "", 
 "merchant_country": "", 
 "merchant_mcc": "10261", 
 "merchant_name": "zqc_test222", 
 "platform_amount": "11.11", 
 "platform_currency": "USD", 
 "transaction_status": "Success", 
 "transaction_type": "Auth", 
 "virtual_id": "1723191314704889" 
 }, 
 "message": "成功", 
 "sign":"4b6d86f5eaf416af7c4d892ecbd7f86ad28c5fd16bd057c991cc
07333602902a" 
 } 
} 