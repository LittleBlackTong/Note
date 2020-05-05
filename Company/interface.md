# ESS 角标接口

### 接口地址

> https://uat-sunpeopleprx-2.scgdomain.com/api/teams/all/assistant/v2/badge

### 接口参数

```
{
    "token": "4f022cc8c8b0cb221f4082b7cb2d750a",
    "timestamp": "2019-10-12 12:12:13.1250213",
    "affectedUsers": [
        {
            "adAccount": "cfruit.syc01",
            "function": "EPIDEMIC",
            "value": 1
        },
        {
            "adAccount": "cfruit.wcg01",
            "function": "EPIDEMIC",
            "value": 1
        }
       
    ]
}

```

> token: 为接口固定 token 不需要更改
> timestamp: 为接口调用时间 注意: 手动调用该接口时注意调用时间一定要大于上一次调用时间,不然对应用户的角标不会更新
> affectedusers: 为推送用户的数组
> adAccount: 推送角标的用户名称
> function: 推送角标的类型 EPIDEMIC 为 ess 抗疫援助角标类型
> value: 为推送用户拥有对应的角标数量
