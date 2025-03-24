# YouCard API 文档

## 1. 接口说明

### 1.1 概述

- YouCard 接口文档

### 1.2 提交地址

| 环境 | 链接 |
|------|------|
| 联调环境 | https://test.youcardx.com/api/v1/vcc |
| 生产环境 | https://gateway.youcardx.com/api/v1/vcc |

### 1.3 请求前需配置 API 权限

1. 白名单验证: 请求 API 前需要给商户分配允许访问白名单 IP(未设置则无法访问)
2. API 权限验证: 请求 API 前需要给商户分配 API 访问权限(未设置则无法访问)

### 1.4 注意部分接口 data 返回内容

- 部分详情接口 data 无数据的时候可能返回 `{}`

## 2. 附录

### 2.1 常用错误码

#### 系统错误码

| 枚举值 | 描述 |
|--------|------|
| 200 | OK |
| 204 | No Content |
| 400 | Bad Request |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not Found |
| 405 | Method Not Allowed |
| 500 | Internal Server Error |
| 503 | Service Unavailable |

#### 业务错误码

| 枚举值 | 中文描述 | 英文描述 |
|--------|----------|----------|
| 0000 | 请求成功 | Request is Success |
| 0348 | 无效商户 | Invalid Merchant |
| 1000 | 参数不合法 | Invalid Parameters |
| 4000 | 请求失败 | Request is Fail |

### 2.2 异步通知事件类型

| 枚举值(事件标识) | 说明 |
|------------------|------|
| VirtualBatch | 卡任务申请回调 |
| VirtualInfo | 卡信息通知回调 |
| VirtualOperate | 卡操作通知回调 |
| VirtualAuthorizations | 卡授权交易通知回调 |
| VirtualCancelCard | 销卡通知回调 |
| VirtualCidTransfer | 子账户交易通知回调 |
| VirtualCidActivate | 子账户激活成功回调 |

### 2.3 子账户结算类型

| 枚举值(类型) | 说明 |
|--------------|------|
| RechargeAmount | 充值金额 |
| TransferOutAmount | 转出金额 |
| CardTransferInAmount | 卡转入金额 |
| CardTransferOutAmount | 卡转出金额 |
| RefundRatioFee | 退值手续费 |
| DestroyCardFee | 销卡手续费 |
| CardOpeningFee | 卡片开通费 |
| MonthlyFee | 卡片月管理费 |
| SharedCardTransactionConsumption | 共享卡交易消耗 |
| TransactionSingleProcessingFee | 交易单笔处理费 |
| TransactionSmallProcessingFee | 小额交易费用 |
| TransactionRefundProcessingFee | 退款手续费 |
| TransactionWithdrawProcessingFee | 撤销手续费 |
| TransactionFailProcessingFee | 交易失败手续费 |
| CardRechargeProcessingFee | 卡充值手续费 |
| Other | 其他 |

### 2.4 交易类型枚举值

| 枚举值(事件标识) | 说明 |
|------------------|------|
| Auth | 授权 |
| Reversal | 授权撤销（清算前） |
| Return | 退款（结算后） |

## 3. 必需接口

### 3.1 获取 Token

#### 简要描述

- 获取 API 请求所需的 Token 串
- token 是商户平台全局唯一接口调用凭据，调用各接口时都需使用 token，需要进行妥善保存
- token 的有效期只有 2 小时，不需要定时刷新，重复获取不会导致上一次获取的 token 失效

#### 请求 URL

```
https://{host}/api/v1/vcc/token
```

#### 请求方式

- GET

#### 参数

- Basic Auth 认证 设置请求信息：
  1. HTTP request header：content-type: application/json
  2. HTTP Authorization Basic 认证：username: clientID, password：secret
     - 注：商户平台系统设定->API 信息中的 ClientID、Secret，每个商户都有分配对应的 ClientID、Secret

| 参数名称 | 是否必填 | 参数说明 |
|----------|----------|----------|
| client_id | M | 商户用户凭证(备注：固定长度 32 位) |
| secret | M | 商户唯一凭证密钥(备注：固定长度 32 位) |

#### 返回示例

##### 成功示例

```json
{
  "code": "200",
  "msg": "OK",
  "result": {
    "code": "0000",
    "message": "成功",
    "data": {
      "token_type": "Bearer",
      "access_token": "2b2449a87c2e528aee33defc6aa4a0a830d052e1",
      "expires_in": 4378
    }
  }
}
```

##### 失败示例

```json
{
  "code": "401",
  "msg": "Unauthorized",
  "result": {
    "code": "1003",
    "message": "身份认证失败:传值错误",
    "data": []
  }
}
```

#### 返回参数说明

- Content-Type: application/json Authorization 参数说明：token_type 和 access_token 值
- 返回的 token_type 和 access_token 需要作为各个 vcc 接口的请求头
- token_type：表示要填写的 token 类型
- access_token：表示要填写 token 的值
- 完整的例子：Bearer（英文空格）2b2449a87c2e528aee33defc6aa4a0a830d052e1

## 4. 业务接口

### 4.1 主账户

#### 4.1.1 获取主账户余额

##### 简要描述

- 获取主账户余额

##### 请求 URL

```
https://{host}/api/v1/vcc/balance/list
```

##### 请求方式

- GET

##### 返回示例

```json
{
  "code": "200",
  "msg": "OK",
  "result": {
    "code": "0000",
    "message": "成功",
    "data": [
      {
        "currency": "USD",
        "available_balance": "62149.82"
      }
    ]
  }
}
```

##### 返回参数说明

| 参数名 | 类型 | 说明 |
|--------|------|------|
| currency | string | 当前 mid 可用币种 |
| available_balance | string | 当前 mid 可用币种对应的账户余额 |

### 4.2 子账户

#### 4.2.1 申请子账户

##### 简要描述

- 申请子账户

##### 请求 URL

```
https://{host}/api/v1/vcc/cid/apply
```

##### 请求方式

- POST
- Content-Type: application/json

##### 返回示例

```json
{
  "code": "200",
  "msg": "OK",
  "result": {
    "code": "0000",
    "message": "成功",
    "data": {
      "cards_id": "C160103346595807012",
      "status": "Pending"
    }
  }
}
```

##### 返回参数说明

| 参数名 | 类型 | 说明 |
|--------|------|------|
| cards_id | string | 子账户 ID 用于卡接口 |
| status | string | 子账户状态值 生成后为待激活、需要激活成功后才可使用该子账户，状态 Enums in (Inactive,Active,Pending,Canceled) |

#### 4.2.2 获取子账户信息列表

##### 简要描述

- 获取子账户信息列表

##### 请求 URL

```
https://{host}/api/v1/vcc/cid/list
```

##### 请求方式

- GET

##### 参数

| 参数名 | 必选 | 类型 | 说明 |
|--------|------|------|------|
| cards_id | 否 | string | 子账户 |
| status | 否 | string | 状态值 状态 Enums in (Inactive,Active,Pending,Canceled) |
| start_page | 否 | string | 开始的页数 默认为 1 |
| row | 否 | string | 显示的条数 默认为 20 最大为 100 |

##### 返回示例

```json
{
  "code": "200",
  "msg": "OK",
  "result": {
    "code": "0000",
    "message": "成功",
    "data": {
      "total": 2,
      "list": [
        {
          "cards_id": "C24080900002002",
          "cards_name": "C24080900002002",
          "bins": [],
          "status": "Pending",
          "balance": "0.00",
          "usable_balance": "0.00",
          "created_at": "2024-08-22 03:28:54"
        },
        {
          "cards_id": "C24080900002001",
          "cards_name": "C24080900002002",
          "bins": [6601, 6666],
          "status": "Active",
          "balance": "8.86",
          "usable_balance": "0.86",
          "created_at": "2024-08-21 11:27:34"
        }
      ]
    }
  }
}
```

##### 返回参数说明

| 参数名 | 类型 | 说明 |
|--------|------|------|
| cards_id | string | 当前查询的子账户 |
| cards_name | string | 当前查询的子账户名称 |
| bins | array | 子账户的可用 bin |
| status | string | 子账户状态值 状态 Enums in (Inactive,Active,Pending,Canceled) |
| balance | string | 子账户剩余的余额 |
| usable_balance | string | 子账户的可用余额 |
| created_at | string | 当前子账户的创建时间 |

#### 4.2.3 子账户充值

##### 简要描述

- 子账户充值

##### 请求 URL

```
https://{host}/api/v1/vcc/cid/recharge
```

##### 请求方式

- POST
- Content-Type: application/json

##### 参数

| 参数名 | 必选 | 类型 | 说明 |
|--------|------|------|------|
| cards_id | 是 | string | 需要进行充值的子账户 |
| currency | 是 | string | 充值的币种 |
| amount | 是 | string | 充值的金额 |
| unique_order_id | 是 | string | 商户交易流水号 不可重复 |

##### 返回示例

```json
{
  "code": "200",
  "msg": "OK",
  "result": {
    "code": "0000",
    "message": "成功",
    "data": {
      "unique_order_id": "Test123458897",
      "status": "Pending"
    }
  }
}
```

##### 返回参数说明

| 参数名 | 类型 | 说明 |
|--------|------|------|
| unique_order_id | string | 商户交易流水号 |
| status | string | 充值成功与否的状态 状态 Enums in (Pending,Success,Failed) |

#### 4.2.4 子账户转出

##### 简要描述

- 子账户转出

##### 请求 URL

```
https://{host}/api/v1/vcc/cid/transfer
```

##### 请求方式

- POST
- Content-Type: application/json

##### 参数

| 参数名 | 必选 | 类型 | 说明 |
|--------|------|------|------|
| cards_id | 是 | string | 需要进行转出的子账户 |
| amount | 是 | string | 转出的金额 |
| unique_order_id | 是 | string | 商户交易流水号 不可重复 |

##### 返回示例

```json
{
  "code": "200",
  "msg": "OK",
  "result": {
    "code": "0000",
    "message": "成功",
    "data": {
      "unique_order_id": "test123456",
      "status": "Pending"
    }
  }
}
```

##### 返回参数说明

| 参数名 | 类型 | 说明 |
|--------|------|------|
| unique_order_id | string | 商户交易流水号 |
| status | string | 转出成功与否的状态 状态 Enums in (Pending,Success,Failed) |

#### 4.2.5 获取所有子账户充值信息

##### 简要描述

- 获取所有子账户充值信息

##### 请求 URL

```
https://{host}/api/v1/vcc/cid/order/list
```

##### 请求方式

- GET

##### 参数

| 参数名 | 必选 | 类型 | 说明 |
|--------|------|------|------|
| cards_id | 否 | string | 需要进行获取数据的子账户 |
| unique_order_id | 否 | string | 需要进行获取数据的商户交易流水号 |
| status | 否 | string | 状态值 状态 Enums in (Pending,Success,Failed) |
| start_page | 否 | string | 开始的页数 默认为 1 |
| row | 否 | string | 显示的条数 默认为 20 最大为 100 |

##### 返回示例

```json
{
  "code": "200",
  "msg": "OK",
  "result": {
    "code": "0000",
    "message": "成功",
    "data": {
      "total": 1,
      "list": [
        {
          "cards_id": "C171867875808958001",
          "unique_order_id": "test123457",
          "type": "Out",
          "currency": "USD",
          "amount": "9.01",
          "arrival_amount": "9.01",
          "fee": "0.00",
          "status": "Success",
          "remark": null,
          "created_at": "2024-06-18 07:45:48"
        }
      ]
    }
  }
}
```

##### 返回参数说明

| 参数名 | 类型 | 说明 |
|--------|------|------|
| cards_id | string | 子账户 |
| unique_order_id | string | 商户交易流水号 |
| type | string | 类型 充值 or 转出 |
| currency | string | 币种 |
| amount | string | 金额 |
| arrival_amount | string | 到账金额 |
| fee | string | 手续费 |
| status | string | 状态值 |
| remark | string | 备注 |
| created_at | string | 创建时间 |

#### 4.2.6 获取子账户结算明细

##### 简要描述

- 获取子账户结算明细

##### 请求 URL

```
https://{host}/api/v1/vcc/cid/settle/list
```

##### 请求方式

- GET

##### 参数

| 参数名 | 必选 | 类型 | 说明 |
|--------|------|------|------|
| cards_id | 否 | string | 需要进行获取数据的子账户 |
| start_page | 否 | string | 开始的页数 默认为 1 |
| row | 否 | string | 显示的条数 默认为 20 最大为 100 |

##### 返回示例

```json
{
  "code": "200",
  "msg": "OK",
  "result": {
    "code": "0000",
    "message": "成功",
    "data": {
      "total": 8,
      "list": [
        {
          "unique_id": "172430855079872228",
          "cards_id": "C24080900002001",
          "virtual_id": "1723187121458990",
          "amount": "-3.00",
          "type": "CardOpeningFee",
          "currency": "USD",
          "remarks": null,
          "settle_at": "2024-08-23 00:00:00",
          "created_at": "2024-08-22 06:35:50"
        },
        {
          "unique_id": "172430755960990061",
          "cards_id": "C24080900002001",
          "virtual_id": "1723187121458984",
          "amount": "-3.00",
          "type": "CardOpeningFee",
          "currency": "USD",
          "remarks": null,
          "settle_at": "2024-08-23 00:00:00",
          "created_at": "2024-08-22 06:19:19"
        },
        {
          "unique_id": "172430723222747151",
          "cards_id": "C24080900002001",
          "virtual_id": "1723187121458978",
          "amount": "-3.00",
          "type": "CardOpeningFee",
          "currency": "USD",
          "remarks": null,
          "settle_at": "2024-08-23 00:00:00",
          "created_at": "2024-08-22 06:13:52"
        },
        {
          "unique_id": "172429758365137422",
          "cards_id": "C24080900002001",
          "virtual_id": "-",
          "amount": "25.00",
          "type": "RechargeAmount",
          "currency": "USD",
          "remarks": null,
          "settle_at": "2024-08-23 00:00:00",
          "created_at": "2024-08-22 03:33:03"
        },
        {
          "unique_id": "172423995595125343",
          "cards_id": "C24080900002001",
          "virtual_id": "-",
          "amount": "-70.00",
          "type": "TransferOutAmount",
          "currency": "USD",
          "remarks": null,
          "settle_at": "2024-08-22 00:00:00",
          "created_at": "2024-08-21 11:32:35"
        },
        {
          "unique_id": "172423984839764978",
          "cards_id": "C24080900002001",
          "virtual_id": "-",
          "amount": "70.00",
          "type": "TransferOutAmount",
          "currency": "USD",
          "remarks": "子账户转出失败返还",
          "settle_at": "2024-08-22 00:00:00",
          "created_at": "2024-08-21 11:30:48"
        },
        {
          "unique_id": "172423979328298368",
          "cards_id": "C24080900002001",
          "virtual_id": "-",
          "amount": "-70.00",
          "type": "TransferOutAmount",
          "currency": "USD",
          "remarks": null,
          "settle_at": "2024-08-22 00:00:00",
          "created_at": "2024-08-21 11:29:53"
        },
        {
          "unique_id": "172423971769994960",
          "cards_id": "C24080900002001",
          "virtual_id": "-",
          "amount": "70.00",
          "type": "RechargeAmount",
          "currency": "USD",
          "remarks": null,
          "settle_at": "2024-08-22 00:00:00",
          "created_at": "2024-08-21 11:28:37"
        }
      ]
    }
  }
}
```

##### 返回参数说明

| 参数名 | 类型 | 说明 |
|--------|------|------|
| unique_id | string | 系统交易流水号 |
| cards_id | string | 子账户 |
| virtual_id | string | 卡 id(部分类型时为-) |
| amount | string | 金额 |
| type | string | 结算类型（详见附录子账户结算类型） |
| currency | string | 结算币种 固定 USD |
| remarks | string | 备注 |
| settle_at | string | 结算时间 |
| created_at | string | 创建时间 |

### 4.3 VirtualCard

#### 4.3.1 申请卡任务

##### 简要描述

- 申请卡任务

##### 请求 URL

```
https://{host}/api/v1/vcc/card/batch/apply
```

##### 请求方式

- POST
- Content-Type: application/json

##### 参数

| 参数名 | 必选 | 类型 | 说明 |
|--------|------|------|------|
| unique_order_id | 是 | string | 商户请求流水号 |
| cards_id | 是 | string | 申请卡任务的子账户 |
| bin | 是 | string | 申请卡的 bin |
| quantity | 是 | string | 申请卡任务的卡数量 |
| initial_amount | 否 | float | 申请卡初始金额 |
| day_amount_limit | 否 | string | 卡日限额 |
| total_amount_limit | 否 | string | 总限额 |
| is_use_default_cardholder | 否 | int | 是否使用默认持卡人（0:不使用;1:使用） |
| cardholder_ids | 否 | string | 当不使用默认持卡人时候填写持卡人 id,号分割 |
| card_type | 是 | string | 申请卡的类型 0:储值卡;1:共享卡 |

##### 返回示例

```json
{
  "code": "200",
  "msg": "OK",
  "result": {
    "code": "0000",
    "message": "成功",
    "data": {
      "batch_id": "2024061708200240290",
      "batch_status": "Pending"
    }
  }
}
```

##### 返回参数说明

| 参数名 | 类型 | 说明 |
|--------|------|------|
| batch_id | string | 卡任务 id |
| batch_status | string | 该卡任务申请状态 状态 Enums in (Disable,Active,Pending,Processing,CardExpired,Opening) |

#### 4.3.2 查询卡任务信息

##### 简要描述

- 查询卡任务信息

##### 请求 URL

```
https://{host}/api/v1/vcc/card/batch/info
```

##### 请求方式

- GET

##### 参数

| 参数名 | 必选 | 类型 | 说明 |
|--------|------|------|------|
| batch_id | 否 | string | 卡任务 id（二选一） |
| unique_order_id | 否 | string | 商户请求流水号（二选一） |

##### 返回示例

```json
{
  "code": "200",
  "msg": "OK",
  "result": {
    "code": "0000",
    "message": "成功",
    "data": {
      "batch_id": "2024061809495727129",
      "cards_id": "C171867875808958002",
      "card_type": "Shared",
      "initial_amount": "0.00",
      "quantity": 4,
      "day_amount_limit": "999.00",
      "batch_total_amount": "0.00",
      "success_number": 4,
      "batch_status": "Success"
    }
  }
}
```

##### 返回参数说明

| 参数名 | 类型 | 说明 |
|--------|------|------|
| batch_id | string | 当前查询卡任务 id |
| cards_id | string | 该卡任务 id 申请的 cid |
| card_type | string | 卡类型 |
| initial_amount | string | 卡初始金额 |
| quantity | string | 开卡数量 |
| day_amount_limit | string | 日交易限额 |
| batch_batch_total_amount | string | 批次总金额 |
| success_number | string | 开卡成功数 |
| batch_status | string | 当前开卡任务状态 状态 Enums in (Pending,Process,Fail,Success) |

#### 4.3.3 获取卡信息

##### 简要描述

- 获取卡信息

##### 请求 URL

```
https://{host}/api/v1/vcc/card/list
```

##### 请求方式

- GET

##### 参数

| 参数名 | 必选 | 类型 | 说明 |
|--------|------|------|------|
| batch_id | 否 | string | 卡批次 id |
| virtual_id | 否 | string | 卡 id |
| cards_id | 否 | string | Cid |
| start_page | 否 | string | 开始的页数 默认为 1 |
| row | 否 | string | 显示的条数 默认为 20 最大为 100 |

##### 返回示例

```json
{
  "code": "200",
  "msg": "OK",
  "result": {
    "code": "0000",
    "message": "成功",
    "data": {
      "total": 4,
      "list": [
        {
          "virtual_id": 1717730712076671,
          "cards_id": "C171867875808958002",
          "bin": 6602,
          "batch_id": "2024061809495727129",
          "card_type": "Shared",
          "card_number": "2234670327599365",
          "cvv": "319",
          "expiration_year": "2026",
          "expiration_month": "06",
          "allow_card_out": true,
          "is_transaction": true,
          "apply_for_cancellation": false,
          "status": "Active",
          "created_at": "2024-06-18 09:49:57",
          "day_amount_limit": "999.00"
        }
      ]
    }
  }
}
```

##### 返回参数说明

| 参数名 | 类型 | 说明 |
|--------|------|------|
| virtual_id | string | 卡 id |
| cards_id | string | 该卡对应的卡任务 id |
| bin | string | 卡 bin |
| batch_id | string | 卡对应的卡批次 id |
| card_type | string | 卡类型 |
| card_number | string | 卡号 |
| cvv | string | cvv |
| expiration_year | string | 有效年份 |
| expiration_month | string | 有效月份 |
| allow_card_out | string | 是否可转出 |
| is_transaction | string | 是否统计交易 |
| apply_for_cancellation | string | 销卡申请时间 |
| status | string | 卡状态 状态 Enums in (Disable,Active,Pending,Processing,CardExpired,Opening) |
| created_at | string | 卡创建时间 |
| day_amount_limit | string | 卡日交易限额 |

#### 4.3.4 卡充值

##### 简要描述

- 卡充值

##### 请求 URL

```
https://{host}/api/v1/vcc/card/recharge
```

##### 请求方式

- POST
- Content-Type: application/json

##### 参数

| 参数名 | 必选 | 类型 | 说明 |
|--------|------|------|------|
| virtual_id | 是 | string | 卡 id |
| amount | 是 | string | 充值的金额 |
| unique_order_id | 是 | string | 商户交易流水号 不可重复 |

##### 返回示例

```json
{
  "code": "200",
  "msg": "OK",
  "result": {
    "code": "0000",
    "message": "成功",
    "data": {
      "unique_id": "172448275152462122",
      "unique_order_id": "123123123123",
      "status": "Pending"
    }
  }
}
```

##### 返回参数说明

| 参数名 | 类型 | 说明 |
|--------|------|------|
| unique_id | string | 系统交易流水号 |
| unique_order_id | string | 商户请求流水号 |
| status | string | 充值状态（Failed, Success, Pending, ChannelProcessing, Processing） |

#### 4.3.5 卡退值

##### 简要描述

- 卡退值

##### 请求 URL

```
https://{host}/api/v1/vcc/card/refund
```

##### 请求方式

- POST
- Content-Type: application/json

##### 参数

| 参数名 | 必选 | 类型 | 说明 |
|--------|------|------|------|
| virtual_id | 是 | string | 卡 id |
| amount | 是 | string | 退值的金额 |
| unique_order_id | 是 | string | 商户交易流水号 不可重复 |

##### 返回示例

```json
{
  "code": "200",
  "msg": "OK",
  "result": {
    "code": "0000",
    "message": "成功",
    "data": {
      "unique_id": "172448275152462122",
      "unique_order_id": "123123123123",
      "status": "Pending"
    }
  }
}
```

##### 返回参数说明

| 参数名 | 类型 | 说明 |
|--------|------|------|
| unique_id | string | 系统交易流水号 |
| unique_order_id | string | 商户请求流水号 |
| status | string | 充值状态（Failed, Success, Pending, ChannelProcessing, Processing） |

#### 4.3.6 获取卡订单(充值/退值)信息

##### 简要描述

- 获取卡订单(充值/退值)信息

##### 请求 URL

```
https://{host}/api/v1/vcc/card/order/list
```

##### 请求方式

- GET

##### 参数

| 参数名 | 必选 | 类型 | 说明 |
|--------|------|------|------|
| virtual_id | 否 | string | 卡 id |
| unique_order_id | 否 | string | 商户交易流水号 |
| type | 否 | string | 类型 (充值/退值) 状态 Enums in (Recharge,ChargeOut) |
| status | 否 | string | 状态 Enums in (Failed,Success,Pending,ChannelProcessing,Processing) |
| start_page | 否 | string | 开始的页数 默认为 1 |
| row | 否 | string | 显示的条数 默认为 20 最大为 100 |

##### 返回示例

```json
{
  "code": "200",
  "msg": "OK",
  "result": {
    "code": "0000",
    "message": "成功",
    "data": {
      "total": 21,
      "list": [
        {
          "unique_id": "test2024061906",
          "unique_order_id": "test2024061906",
          "virtual_id": "1717730706303406",
          "cards_id": "C160103346595807009",
          "card_number": "515783XXXXXX5100",
          "amount": "10.00",
          "currency": "USD",
          "status": "Failed",
          "type": "ChargeOut"
        },
        {
          "unique_id": "171833679195003668",
          "unique_order_id": "",
          "virtual_id": "1717730116153051",
          "cards_id": "C160103346595807009",
          "card_number": "515783XXXXXX7840",
          "amount": "7.10",
          "currency": "USD",
          "status": "Success",
          "type": "Recharge"
        },
        {
          "unique_id": "171817417796124381",
          "unique_order_id": "",
          "virtual_id": "1718171112826308",
          "cards_id": "C160103346595807009",
          "card_number": "515783XXXXXX6183",
          "amount": "10.00",
          "currency": "USD",
          "status": "Success",
          "type": "ChargeOut"
        }
      ]
    }
  }
}
```

##### 返回参数说明

| 参数名 | 类型 | 说明 |
|--------|------|------|
| unique_id | string | 系统交易流水号 |
| unique_order_id | string | 商户交易流水号 |
| virtual_id | string | 当前查询卡 id |
| cards_id | string | cid |
| card_number | string | 卡号 |
| amount | string | 金额 |
| currency | string | 币种 |
| status | string | 充值/退值 状态 |
| type | string | 类型 充值/退值 |

#### 4.3.7 卡余额查询

##### 简要描述

- 卡余额查询

##### 请求 URL

```
https://{host}/api/v1/vcc/card/balance
```

##### 请求方式

- POST
- Content-Type: application/json

##### 参数

| 参数名 | 必选 | 类型 | 说明 |
|--------|------|------|------|
| virtual_id | 是 | string | 卡 id |

##### 返回示例

```json
{
  "code": "200",
  "msg": "OK",
  "result": {
    "code": "0000",
    "message": "成功",
    "data": {
      "card_number": "515783XXXXXX5714",
      "available_balance": "188.33"
    }
  }
}
```

##### 返回参数说明

| 参数名 | 类型 | 说明 |
|--------|------|------|
| card_number | string | 卡号 |
| available_balance | string | 卡可用余额 |

#### 4.3.8 卡额度限制

##### 简要描述

- 卡额度限制
- 注：共享卡可以进行额度限制

##### 请求 URL

```
https://{host}/api/v1/vcc/card/share/limit
```

##### 请求方式

- POST
- Content-Type: application/json

##### 参数

| 参数名 | 必选 | 类型 | 说明 |
|--------|------|------|------|
| virtual_id | 是 | string | 卡 id |
| single_amount | 是 | string | 单次交易限额 |
| day_amount | 是 | string | 单日交易限额 |
| week_amount | 是 | string | 周交易限额 |
| month_amount | 是 | string | 月交易限额 |
| total_amount | 是 | string | 总交易限额 |

##### 返回示例

```json
{
  "code": "200",
  "msg": "OK",
  "result": {
    "code": "0000",
    "message": "成功",
    "data": []
  }
}
```

#### 4.3.9 销卡

##### 简要描述

- 销卡
- 注意：此操作立即执行且销卡后不可恢复！！！

##### 请求 URL

```
https://{host}/api/v1/vcc/card/destroy
```

##### 请求方式

- POST
- Content-Type: application/json

##### 参数

| 参数名 | 必选 | 类型 | 说明 |
|--------|------|------|------|
| virtual_id | 是 | string | 卡 id |

##### 返回示例

```json
{
  "code": "200",
  "msg": "OK",
  "result": {
    "code": "0000",
    "message": "成功",
    "data": []
  }
}
```

#### 4.3.10 撤销销卡（已弃用）

##### 简要描述

- 撤销销卡（已弃用）

##### 请求 URL

```
https://{host}/api/v1/vcc/card/revoke
```

##### 请求方式

- GET

##### 参数

| 参数名 | 必选 | 类型 | 说明 |
|--------|------|------|------|
| virtual_id | 是 | string | 卡 id |

##### 返回示例

```json
{
  "code": "200",
  "msg": "OK",
  "result": {
    "code": "0000",
    "message": "成功",
    "data": []
  }
}
```

#### 4.3.11 卡冻结

##### 简要描述

- 卡冻结

##### 请求 URL

```
https://{host}/api/v1/vcc/card/block
```

##### 请求方式

- POST
- Content-Type: application/json

##### 参数

| 参数名 | 必选 | 类型 | 说明 |
|--------|------|------|------|
| virtual_id | 是 | string | 卡 id |

##### 返回示例

```json
{
  "code": "200",
  "msg": "OK",
  "result": {
    "code": "0000",
    "message": "成功",
    "data": []
  }
}
```

#### 4.3.12 卡解冻

##### 简要描述

- 卡解冻

##### 请求 URL

```
https://{host}/api/v1/vcc/card/unblock
```

##### 请求方式

- POST
- Content-Type: application/json

##### 参数

| 参数名 | 必选 | 类型 | 说明 |
|--------|------|------|------|
| virtual_id | 是 | string | 卡 id |

##### 返回示例

```json
{
  "code": "200",
  "msg": "OK",
  "result": {
    "code": "0000",
    "message": "成功",
    "data": []
  }
}
```

#### 4.3.13 获取卡交易信息

##### 简要描述

- 获取卡交易信息

##### 请求 URL

```
https://{host}/api/v1/vcc/card/authorizations
```

##### 请求方式

- GET

##### 参数

| 参数名 | 必选 | 类型 | 说明 |
|--------|------|------|------|
| virtual_id | 否 | string | 卡 id |
| start_date | 否 | string | 开始时间 秒级时间戳 (PRC 时间) |
| end_date | 否 | string | 结束时间 秒级时间戳 (PRC 时间) |
| start_page | 否 | string | 开始的页数 默认为 1 |
| row | 否 | string | 显示的条数 默认为 20 最大为 100 |

##### 返回示例

```json
{
  "code": "200",
  "msg": "OK",
  "result": {
    "code": "0000",
    "message": "成功",
    "data": {
      "total": 2,
      "list": [
        {
          "authorization_id": "163671118679796739",
          "virtual_id": "1717730712023561",
          "card_number": "532959XXXXXX7394",
          "amount": "100.00",
          "currency": "USD",
          "platform_amount": "100.00",
          "platform_currency": "USD",
          "transaction_type": "Auth",
          "transaction_status": "Success",
          "fail_reason": "",
          "auth_code": "12580",
          "merchant_mcc": "9871",
          "merchant_name": "test",
          "merchant_country": "CHN",
          "authorization_time": "2020-08-05 09:06:20"
        },
        {
          "authorization_id": "163669811952212854",
          "virtual_id": "1717730712073412",
          "card_number": "532959XXXXXX4132",
          "amount": "20.00",
          "currency": "USD",
          "platform_amount": "100.00",
          "platform_currency": "USD",
          "transaction_type": "Auth",
          "transaction_status": "Failed",
          "fail_reason": "INSUFFICIENT_FUNDS",
          "auth_code": "12580",
          "merchant_mcc": "9871",
          "merchant_name": "test",
          "merchant_country": "CHN",
          "authorization_time": "2020-08-05 09:06:20"
        }
      ]
    }
  }
}
```

##### 返回参数说明

| 参数名 | 类型 | 说明 |
|--------|------|------|
| authorization_id | string | 交易 id |
| virtual_id | string | 卡 id |
| card_number | string | 卡号 |
| amount | string | 卡交易金额 |
| currency | string | 卡交易币种 |
| platform_amount | string | 平台交易金额 |
| platform_currency | string | 平台交易币种 |
| transaction_type | string | 交易类型(Auth,Reversal,Return） |
| transaction_status | string | 交易状态 状态 Enums in (Failed,Success) |
| fail_reason | string | 失败原因 |
| auth_code | string | 授权码 |
| merchant_mcc | string | mcc |
| merchant_name | string | 卡账单 |
| merchant_country | string | 交易地址 |
| authorization_time | string | 授权时间 |

#### 4.3.14 获取卡清算交易信息

##### 简要描述

- 获取卡清算交易信息

##### 请求 URL

```
https://{host}/api/v1/vcc/card/entryTrans
```

##### 请求方式

- GET

##### 参数

| 参数名 | 必选 | 类型 | 说明 |
|--------|------|------|------|
| virtual_id | 否 | string | 卡 id |
| start_date | 否 | string | 开始时间 秒级时间戳 (PRC 时间) |
| end_date | 否 | string | 结束时间 秒级时间戳 (PRC 时间) |
| start_page | 否 | string | 开始的页数 默认为 1 |
| row | 否 | string | 显示的条数 默认为 20 最大为 100 |

##### 返回示例

```json
{
  "code": "200",
  "msg": "OK",
  "result": {
    "code": "0000",
    "message": "成功",
    "data": {
      "total": 2,
      "list": [
        {
          "unique_id": "172319980579595580",
          "authorization_id": "172319590497372158",
          "virtual_id": "1717730712023561",
          "card_number": "532959XXXXXX7394",
          "entry_amount": "100.00",
          "entry_currency": "USD",
          "trans_type": "debit",
          "trans_status": "Success",
          "entry_date": "2020-08-05 09:06:20"
        },
        {
          "unique_id": "172319934856008810",
          "authorization_id": "172319594204747146",
          "virtual_id": "1717730712023561",
          "card_number": "532959XXXXXX7394",
          "entry_amount": "200.00",
          "entry_currency": "USD",
          "trans_type": "credit",
          "trans_status": "Fail",
          "entry_date": "2020-08-05 09:06:20"
        }
      ]
    }
  }
}
```

##### 返回参数说明

| 参数名 | 类型 | 说明 |
|--------|------|------|
| unique_id | string | 清算交易 id |
| authorization_id | string | 授权交易 id |
| virtual_id | string | 卡 id |
| card_number | string | 卡号 |
| entry_amount | string | 清算金额 |
| entry_currency | string | 清算币种 |
| trans_type | string | 类型(debit,credit） |
| trans_status | string | 状态 状态 Enums in (Failed,Success) |
| entry_date | string | 清算时间 |

## 5. Webhooks

### 5.1 异步通知说明

#### 简要说明

- 当业务处理完成后，有部分数据或信息需要进行审核、业务本身数据需要异步处理后再下发等情况时，平台会通过异步通知的方式，将数据下发给商户，商户需要做好相关的通知对接和己方的业务处理

#### 注意事项

- 需要在商户端配置对应业务的通知地址（系统设置->api 信息->Webhooks）
- 通知最多 3 次，间隔为 5 分钟
- 商户接收通知并处理业务成功后，需正常响应 application/text 类型，值为 ok，来表明已经处理成功无需再下发通知，否则系统将按上一点的事件间隔继续通知，直到超过 3 次
- 实际通知的数据、内容请参考各项业务接口对应的通知文档
- 业务有可能重复通知，请对接时处理好自己内部的业务逻辑，防止数据重复通知导致的业务异常
- 消息通知类型用 Header 头中的 Message-Type 判断

#### Verify sign demo

##### PHP

```php
$sign = $content['result']['sign'] ?? '';
$verifySign = hash_hmac('sha256', json_encode($content['result']['data'] ?? []), $youApi->secret);
```

##### Golang

```go
type SignResult struct {
  Code    string                 `json:"code"`
  Message string                 `json:"message"`
  Data    map[string]interface{} `json:"data"`
  Sign    string                 `json:"sign"`
}

func main() {
  secret := "your_secret_here"
  // 将 SignResult 转换成 JSON 字符串
  jsonContent, err := json.Marshal(SignResult.Data)
  if err != nil {
    fmt.Println("Error encoding JSON: ", err)
    return
  }

  h := hmac.New(sha256.New, []byte(secret))
  // 写入消息内容
  h.Write(jsonContent)
  // 计算最终的哈希值，返回的是一个字节切片
  signature := h.Sum(nil)
  // 将签名结果转换为十六进制字符串
  verifySign := fmt.Sprintf("%x", signature)
}
```

### 5.2 Topics

#### 5.2.1 卡任务申请回调

##### 简要描述

- 卡任务申请回调
- Message-Type: VirtualBatch

##### 请求 URL

```
{{ baseUrl }}/webhook
```

##### 请求方式

- POST
- Content-Type: application/json

##### 参数

| 参数名 | 必选 | 类型 | 说明 |
|--------|------|------|------|
| batch_id | 是 | string | 回调通知批次 id |
| batch_status | 是 | string | 通知批次状态 Enums in (Pending,Process,Fail,Success) |

##### 返回示例

```json
{
  "code": "200",
  "msg": "OK",
  "result": {
    "code": "0000",
    "data": {
      "batch_id": "2024061708200240290",
      "batch_status": "Pending"
    },
    "message": "成功",
    "sign": "4b6d86f5eaf416af7c4d892ecbd7f86ad28c5fd16bd057c991cc07333602902a"
  }
}
```

#### 5.2.2 卡信息通知回调

##### 简要描述

- 卡信息通知回调
- Message-Type: VirtualInfo

##### 请求 URL

```
{{ baseUrl }}/webhook
```

##### 请求方式

- POST
- Content-Type: application/json

##### 参数

| 参数名 | 类型 | 说明 |
|--------|------|------|
| virtual_id | string | 卡 id |
| cards_id | string | 卡对应的 cid |
| bin | string | 卡 bin |
| batch_id | string | 卡批次 id |
| card_type | string | 卡类型 |
| card_number | string | 卡号 |
| cvv | string | cvv |
| expiration_year | string | 有效年份 |
| expiration_month | string | 有效月份 |
| allow_card_out | string | 是否可转出 |
| is_transaction | string | 是否统计交易 |
| apply_for_cancellation | string | 销卡申请时间 |
| status | string | 卡状态 状态 Enums in (Disable,Active,Pending,Processing,CardExpired,Opening) |
| created_at | string | 卡创建时间 |
| day_amount_limit | string | 卡日交易限额 |

##### 返回示例

```json
{
  "code": "200",
  "msg": "OK",
  "result": {
    "code": "0000",
    "data": {
      "total": 4,
      "list": [
        {
          "virtual_id": 1717730712076671,
          "cards_id": "C171867875808958002",
          "bin": 6602,
          "batch_id": "2024061809495727129",
          "card_type": "Shared",
          "card_number": "2234670327599365",
          "cvv": "319",
          "expiration_year": "2026",
          "expiration_month": "06",
          "allow_card_out": true,
          "is_transaction": true,
          "apply_for_cancellation": false,
          "status": "Active",
          "created_at": "2024-06-18 09:49:57",
          "day_amount_limit": "999.00"
        }
      ]
    },
    "message": "成功",
    "sign": "4b6d86f5eaf416af7c4d892ecbd7f86ad28c5fd16bd057c991cc07333602902a"
  }
}
```

#### 5.2.3 卡操作通知回调

##### 简要描述

- 卡操作通知回调
- Message-Type: VirtualOperate

##### 请求 URL

```
{{ baseUrl }}/webhook
```

##### 请求方式

- POST
- Content-Type: application/json

##### 参数

| 参数名 | 类型 | 说明 |
|--------|------|------|
| unique_id | string | 系统流水号 |
| unique_order_id | string | 商户请求流水号 |
| virtual_id | string | 当前查询卡 id |
| cards_id | string | cid |
| card_number | string | 卡号 |
| amount | string | 金额 |
| currency | string | 币种 |
| status | string | 充值/退值 状态 (Failed,Success,Pending,ChannelProcessing,Processing) |
| type | string | 类型 充值/退值(Recharge,ChargeOut) |

##### 返回示例

```json
{
  "code": "200",
  "msg": "OK",
  "result": {
    "code": "0000",
    "data": {
      "unique_id": "1717730116153051",
      "unique_order_id": "1717730116153051",
      "virtual_id": "1717730116153051",
      "cards_id": "C160103346595807009",
      "card_number": "515783XXXXXX7840",
      "amount": "10.00",
      "currency": "USD",
      "status": "Success",
      "type": "Recharge"
    },
    "message": "成功",
    "sign": "4b6d86f5eaf416af7c4d892ecbd7f86ad28c5fd16bd057c991cc07333602902a"
  }
}
```

#### 5.2.4 卡授权交易通知回调

##### 简要描述

- 卡授权交易通知回调
- Message-Type: VirtualAuthorizations

##### 请求 URL

```
{{ baseUrl }}/webhook
```

##### 请求方式

- POST
- Content-Type: application/json

##### 参数

| 参数名 | 类型 | 说明 |
|--------|------|------|
| authorization_id | string | 交易 id |
| virtual_id | string | 卡 id |
| card_number | string | 卡号 |
| amount | string | 金额 |
| currency | string | 币种 |
| transaction_type | string | 交易类型(Auth,Reversal,Return） |
| transaction_status | string | 交易状态 状态 Enums in (Failed,Success) |
| fail_reason | string | 失败原因 |
| authorization_time | string | 授权时间 |

##### 返回示例

```json
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
    "sign": "4b6d86f5eaf416af7c4d892ecbd7f86ad28c5fd16bd057c991cc07333602902a"
  }
}
```

#### 5.2.5 销卡通知回调

##### 简要描述

- 销卡通知回调
- Message-Type: VirtualCancelCard

##### 请求 URL

```
{{ baseUrl }}/webhook
```

##### 请求方式

- POST
- Content-Type: application/json

##### 参数

| 参数名 | 类型 | 说明 |
|--------|------|------|
| virtual_id | string | 卡 id |
| status | string | 状态 Enums in (Disable,Active,Pending,Processing,CardExpired,Opening) |

##### 返回示例

```json
{
  "code": "200",
  "msg": "OK",
  "result": {
    "code": "0000",
    "data": {
      "virtual_id": "1718854722753140",
      "status": "CardExpired",
      "transfer_info": {
        "unique_id": "172682522216891892",
        "amount": "33.00",
        "currency": "USD"
      }
    },
    "message": "成功",
    "sign": "4b6d86f5eaf416af7c4d892ecbd7f86ad28c5fd16bd057c991cc07333602902a"
  }
}
```

#### 5.2.6 子账户交易通知回调

##### 简要描述

- 子账户交易通知回调
- Message-Type: VirtualCidTransfer

##### 请求 URL

```
{{ baseUrl }}/webhook
```

##### 请求方式

- POST
- Content-Type: application/json

##### 参数

| 参数名 | 类型 | 说明 |
|--------|------|------|
| unique_order_id | string | 商户交易流水号 |
| status | string | 状态 Enums in (Pending,Success,Failed) |

##### 返回示例

```json
{
  "code": "200",
  "msg": "OK",
  "result": {
    "code": "0000",
    "data": {
      "unique_order_id": "test123",
      "status": "Pending"
    },
    "message": "成功",
    "sign": "4b6d86f5eaf416af7c4d892ecbd7f86ad28c5fd16bd057c991cc07333602902a"
  }
}
```

#### 5.2.7 子账户激活状态回调

##### 简要描述

- 子账户激活状态回调
- Message-Type: VirtualCidActivate

##### 请求 URL

```
{{ baseUrl }}/webhook
```

##### 请求方式

- POST
- Content-Type: application/json

##### 参数

| 参数名 | 类型 | 说明 |
|--------|------|------|
| cards_id | string | CID |
| status | string | 状态 Enums in (Inactive,Active,Pending,Canceled) |

##### 返回示例

```json
{
  "code": "200",
  "msg": "OK",
  "result": {
    "code": "0000",
    "data": {
      "cards_id": "C160103346595807002",
      "status": "Active"
    },
    "message": "成功",
    "sign": "4b6d86f5eaf416af7c4d892ecbd7f86ad28c5fd16bd057c991cc07333602902a"
  }
}
``` 