Open Trust Standards: OT-Auth
====

## Status of this Memo
本标准目前处于编写阶段，已定义的内容可能会调整优化。

## Abstract
Open Trust 致力于为云原生软件提供一种安全、标准的接口可信开放机制，让云原生软件彼此之间的连接打通变得简单可行，让“用 API 连接世界万物”成为可能。

## Open Trust Authority
Open Trust Authority 服务简称 OT-Auth，是 Open Trust 的核心，它存储可信主体基本信息、分配 OTID、签发和验证 OTVID、维护跨信任域的联盟机制。

### Discovering Endpoint

```
GET https://trust-domain/.well-known/open-trust-configuration
```

OT-Auth 服务必须能基于 Web PKI 被公开发现和验证其信任域的有效性，它必须提供一个 endpoint 验证点，URI 为 `https://trust-domain/.well-known/open-trust-configuration`，返回结果为符合(TODO) [Verifiable Credentials Data Model 1.0](https://www.w3.org/TR/vc-data-model/) 标准的 JSON 文档。
该验证点参考了 [OpenID Connect Discovery](https://openid.net/specs/openid-connect-discovery-1_0.html) 的设计。


如上所示，该 URI 必须要使用 `https`，其域名必须是信任域的 `trust-domain`，`/.well-known/` 则在 [RFC5785](https://tools.ietf.org/html/rfc5785) 中定义，`open-trust-configuration` 的定义如下：

`otid` 必须为 OT-Auth 的 OTID，也就是 `otid:trust-domain`。

`serviceEndpoints` 必须为 OT-Auth 服务的 API 端点组，往这些 endpoint 直接发起 GET 请求应该要返回该服务的简要信息，可信主体从而可以选择其中最快的一个使用，注意该端点的域名可以与所属信任域不同。

`user_types` 和 `service_types` 必须为 OT-Auth 支持的可信主体类型列表数组，官方 OT-Auth 内置了五种类型：`user` 代表“用户”，`dev` 代表“设备”，`agent` 代表“代理机器人”，`app` 代表“应用”，`svc` 代表“服务”。可信主体类型可分为两大类，一类是承载访问主体职能的纯客户端侧的可信主体，可能会绑定其它非 OTID 的身份 ID，如 `user`、`dev`，简称用户类；第二类是承载访问客体职能的可信主体，为其它可信主体提供 API 能力，它也作为可信主体调用其它客体的 API，如 `agent`、`app`、`svc`，简称服务类。这里 `svc` 主要为服务类可信主体提供 API 能力，`app` 主要调用服务类 API，为用户类可信主体提供 API 能力，`agent` 则表现为用户类，调用 `app` 的 API，但同时为用户类可信主体提供代理 API 执行能力。

`keys` 也称为信任域公钥组，必须为 OT-Auth 当前有效的 [JWK](https://tools.ietf.org/html/rfc7517) Set 数组，该数组必须包含至少一个有效 JWK，这组 JWK 只包含公钥及算法信息，不能包含私钥，用于验证 OTVID。
Verifier 验证者应该缓存这组 JWK Set，并基于 `keysRefreshHint` 定期读取最新值，Verifier 应该用它们来验证可信主体请求携带的 OTVID，当验证通过并发现该 OTVID 存在 Release ID 值时，则有必要到 OT-Auth 的 `serviceEndpoints` 继续验证该 OTVID，这样可以确保被吊销的 OTVID 不再有效。

`keysRefreshHint` 必须为 OT-Auth 服务建议的 Verifier 验证者对 JWK Set 查询时间间隔。因为 JWK 会定期轮转更新，验证者有必要定时查询 JWK Set 从而确保获得最新有效公钥组，建议为 3600 秒。

`proof` (TODO，VC 标准还不够完善) 必须为符合 [Linked Data Proofs](https://w3c-ccg.github.io/ld-proofs/) 标准的数据。

假设信任域为 `ot.example.com`，则该信任域 OT-Auth 的 OTID 为 `otid:ot.example.com`，验证该信任域的有效性则请求 `https://ot.example.com/.well-known/open-trust-configuration`，如：

```
GET /.well-known/open-trust-configuration HTTP/2
Host: ot.example.com
```

应该获得如下响应结果：
```
HTTP/2 200 OK
Content-Type: application/json

{
  "otid": "otid:ot.example.com",
  "serviceEndpoints": ["https://api.example.com/ot"],
  "user_types": ["user", "dev"],
  "service_types": ["agent", "app", "svc"],
  "keysRefreshHint": 3600,
  "keys": [
    {
      "kty": "EC",
      "crv": "P-256",
      "x": "MKBCTNIcKUSDii11ySs3526iDZ8AiTo7Tu6KPAqv7D4",
      "y": "4Etl6SRW2YiLUrN5vfvVHuhp7x8PxltmWWlbbM4IFyM",
      "use": "sig",
      "kid": "1591110555"
    }, {
      "kty":"RSA",
      "n": "0vx7agoebGcQSuuPiLJXZptN9nndrQmbXEps2aiAFbWhM78LhWx
        4cbbfAAtVT86zwu1RK7aPFFxuhDR1L6tSoc_BJECPebWKRXjBZCiFV4n3oknjhMs
        tn64tZ_2W-5JsGY4Hc5n9yBXArwl93lqt7_RN5w6Cf0h4QyQ5v-65YGjQR0_FDW2
        QvzqY368QQMicAtaSqzs8KJZgnYb9c7d0zgdAZHzu6qMQvRL5hajrn1n91CbOpbI
        SD08qNLyrdkt-bFTWhAI4vMQFh6WeZu0fM4lFd2NcRwr3XPksINHaQ-G_xBniIqb
        w0Ls1jF44-csFCur-kEgU8awapJzKnqDKgw",
      "e":"AQAB",
      "alg":"RS256",
      "use": "sig",
      "kid":"1598110555"
    }
  ]
}
```

### OTID Registration

```
POST https://api.example.com/ot/register
```

```
POST /ot/register HTTP/2
Host: api.example.com
Content-Type: application/json
Accept: application/json
Authorization: Bearer eyJhbGciOiJSUzI1NiJ9.eyJ ...

{
  "subjectType": "user",
  "subjectId": "9eebccd2-12bf-40a6-b262-65fe0487d453",
  "description": "any description",
  "keys": [
    {
      "kty": "EC",
      "crv": "P-256",
      "x": "MKBCTNIcKUSDii11ySs3526iDZ8AiTo7Tu6KPAqv7D4",
      "y": "4Etl6SRW2YiLUrN5vfvVHuhp7x8PxltmWWlbbM4IFyM",
      "use": "sig",
      "kid": "1591110555"
    }]
}
```

### OTID Resolution

```
GET https://api.example.com/ot/resolve/{otid}
```

```
GET /ot/otid:ot.example.com:user:9eebccd2-12bf-40a6-b262-65fe0487d453 HTTP/2
Host: api.example.com
Accept: application/json
```

一类可信主体返回结果示意：
```
HTTP/2 200 OK
Date: Wed, 10 Apr 2020 08:26:52 GMT
Content-Type: application/json;charset=UTF-8

{
  "result": {
    "createdAt": "2020-10-21T11:12:13.065Z",
    "updatedAt": "2020-10-21T11:12:13.065Z",
    "subjectType": "user",
    "subjectId": "9eebccd2-12bf-40a6-b262-65fe0487d453",
    "description": "any description",
    "keys": [{
      "kty": "EC",
      "crv": "P-256",
      "x": "MKBCTNIcKUSDii11ySs3526iDZ8AiTo7Tu6KPAqv7D4",
      "y": "4Etl6SRW2YiLUrN5vfvVHuhp7x8PxltmWWlbbM4IFyM",
      "use": "sig",
      "kid": "1591110555"
    }],
    "keysUpdatedAt": "2020-10-21T11:12:13.065Z",
    "status": 0
  }
}
```

二类可信主体返回结果示意：
```
HTTP/2 200 OK
Date: Wed, 10 Apr 2020 08:26:52 GMT
Content-Type: application/json;charset=UTF-8

{
  "result": {
    "createdAt": "2020-10-21T11:12:13.065Z",
    "updatedAt": "2020-10-21T11:12:13.065Z",
    "subjectType": "service",
    "subjectId": "user-base",
    "description": "user service",
    "keys": [{
      "kty": "EC",
      "crv": "P-256",
      "x": "MKBCTNIcKUSDii11ySs3526iDZ8AiTo7Tu6KPAqv7D4",
      "y": "4Etl6SRW2YiLUrN5vfvVHuhp7x8PxltmWWlbbM4IFyM",
      "use": "sig",
      "kid": "1591110555"
    }],
    "keysUpdatedAt": "2020-10-21T11:12:13.065Z",
    "serviceEndpoints": ["https://svc.example.com/user-base"],
    "status": 0
  }
}
```

```
PUT https://api.example.com/ot/resolve/{otid}
```

```
DELETE https://api.example.com/ot/resolve/{otid}
```

### OTID Bundles

```
GET https://api.example.com/ot/resolve/{otid}/bundles
```

一类可信主体返回结果示意：
```
HTTP/2 200 OK
Date: Wed, 10 Apr 2020 08:26:52 GMT
Content-Type: application/json;charset=UTF-8

{
  "result": {
    "bundles": [{
      "provider": "otdi:ot.example.com:service:user-base",
      "bundleId": "5ee8c56c299ce809fb99fce9"
    }],
    "status": 0
  }
}
```

```
POST https://api.example.com/ot/resolve/{otid}/bundles
```

```
DELETE https://api.example.com/ot/resolve/{otid}/bundles
```

### OTVID Issue And Verify

```
POST https://api.example.com/ot/sign
```

```
POST https://api.example.com/ot/verify
```

### Trust Domain Federation

```
POST https://api.example.com/ot/federate
```
