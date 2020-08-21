# Open Trust Specifications

## 背景
云计算、大数据、移动互联网、物联网等信息技术正在以加速的态势推进社会的数字化转型，为各行各业带来了创新的生产力，也带来了极大的复杂性。
集成、打通、安全、开放成为企业信息系统扩展能力边界、开拓市场的最大挑战。把自己的 API 能力开放出去，把别人的 API 能力集成进来，是企业 IT 服务最大的刚需。

## Open Trust 定义
Open Trust 致力于为云原生软件提供一种安全、开放、标准的接口能力，让云原生软件彼此之间的连接打通变得简单可行，让“用 API 连接世界万物”成为可能。
与 Zero Trust 零信任架构不同，Zero Trust 的目标是让企业的网络资源访问变得开放、安全，Open Trust 的目标是让整个世界的网络资源访问变得开放、安全、标准。Open Trust 是云原生生态的基础设施，是数字经济时代的基本要求。

## Open Trust Standards
### Open Trust Identity
Open Trust Identity 简称为 OTID，参考了 [W3C DID](https://w3c.github.io/did-core/) 的设计，但没有像 DID 那样支持 Path、Query、Fragment 等，示例如下：

```otid:ot.example.com:user:9eebccd2-12bf-40a6-b262-65fe0487d453```

OTID 是符合 [RFC3986](https://tools.ietf.org/html/rfc3986) URI 规范的，它由 4 个部分组成：

```otid:trust-domain:subject-type:subject-id```

1. scheme，固定为 `otid`；
2. trust-domain，它必须是一个有公共 CA 授权证书的域名，并且能要提供 Open Trust Authentication 能力（详见 Open Trust Authentication 章节），Open Trust 信任域的联合信任是基于 Web PKI 来实现的；
3. subject-type，信任主体类型，它由 ALPHA / "." / "-" / "_" 组成，任何事物都可以是信任主题，如 person, group, organization, physical thing, digital thing, logical thing 等，但一个实际部署的 Open Trust 系统中，subject-type 是通过 Open Trust Authentication 服务配置控制的；
4. subject-id，在 trust-domain:subject-type 作用域下的唯一 ID，它由 ALPHA / DIGIT / "." / "-" / "_" 组成；
5. 整个 OTID 应该是持久的和不可变的，是小写的，是 URL 兼容的，长度不超过 1024 字节；
6. 信任域 Open Trust Authentication 服务的 OTID 为 `otid:trust-domain`，如 `otid:ot.example.com`，其它情况下 subject-type 和 subject-id 不能为空。

### Open Trust Verifiable Identity Document
Open Trust Verifiable Identity Document 简称为 OTVID，是主体用于证明自己身份的一份数据文档，它包含了 OTID、文档签名以及其它的一些属性或声明。OTVID 参考了 [SPIFFE](https://github.com/spiffe/spiffe) 的设计，但只使用 JWT 来封装。

当主体访问信任域的 Open Trust Authentication 服务时，需要提供由主体自己签发的 OTVID，Open Trust Authentication 服务保存了信任域内所有主体的公钥，从而可以验证其 OTVID。
当主体访问信任域的其它服务时，需要提供由信任域内的 Open Trust Authentication 签发的 OTVID，信任域内的服务只会通过信任域内的 Open Trust Authentication 来验证 OTVID。
当主体访问其它信任域的服务时，需要提供由目标信任域的 Open Trust Authentication 签发的 OTVID，目标信任域内的服务只会通过目标信任域内的 Open Trust Authentication 来验证 OTVID。跨信任域签发 OTVID 由 federation 联盟机制来实现，详见 Open Trust Authentication 章节。

访问主体的 OTVID 通过 HTTP 协议 `Authorization` header (“authorization” for HTTP/2) 和 `Bearer` scheme 传递给访问客体，详见 [RFC 6750 section 2.1](https://tools.ietf.org/html/rfc6750#section-2.1)。
如 HTTP/1.1 中为 `Authorization: Bearer <serialized_token>`，HTTP/2 中则为 `authorization: Bearer <serialized_token>`。

OTVID 是一个标准的 [JWT](https://tools.ietf.org/html/rfc7519)，其组成见下。

JOSE Header Algorithm `alg` 必须设置为如下值之一：

   `alg` Param Value | Digital Signature Algorithm
   ------------------|-----------------------------
   RS256 | RSASSA-PKCS1-v1_5 using SHA-256
   RS384 | RSASSA-PKCS1-v1_5 using SHA-384
   RS512 | RSASSA-PKCS1-v1_5 using SHA-512
   ES256 | ECDSA using P-256 and SHA-256
   ES384 | ECDSA using P-384 and SHA-384
   ES512 | ECDSA using P-521 and SHA-512
   PS256 | RSASSA-PSS using SHA-256 and MGF1 with SHA-256
   PS384 | RSASSA-PSS using SHA-384 and MGF1 with SHA-384
   PS512 | RSASSA-PSS using SHA-512 and MGF1 with SHA-512

JOSE Header Key ID `kid` 是可选的。

Subject Claim `sub` 必须被设置为访问主体的 OTID，如 `otid:ot.example.com:user:9eebccd2-12bf-40a6-b262-65fe0487d453`。

Issuer Claim `iss` 必须被设置为 OTVID 签发人的 OTID，如 `otid:ot.example.com`，一般是 OT-Auth（Open Trust Authentication）服务或访问主体自身的 OTID。

Audience Claim `aud` 必须被设置为访问客体的 OTID，如 `otid:ot.example.com:app:abc123`，并且只能设置一个（JWT 标准允许多个值）。
主体访问信任域的 OT-Auth 服务时，OTVID 的 `aud` 值是 `otid:ot.example.com`。访问客体必须要验证该 OTVID 的 `aud` 值是自己。

Expiration Time Claim `exp` 必须被设置，其值为 Unix 时间戳，1970-01-01 UTC 以来的秒数，一般建议设置为3~10分钟之内。

Issued At Claim `iat` 必须被设置为 OTVID 签发时间，Unix 时间戳。

Release Timestamp Claim `rts` 为可选值，它目前不是标准的 JWT Claim。当 `iss` 为 OT-Auth 服务并且 `exp` 比较长时（如超过10分钟，可以配置），应该被设置为 OT-Auth 服务存储的 `sub` 对应主体的 Release Timestamp 值。
`rts` 用于实现对已签发 OTVID 的吊销，当 OT-Auth 服务验证 OTVID 的 `rts` 与存储值不一致时，说明该 OTVID 已经无效。
当 `rts` 存在时，应该到 OT-Auth 的 `serviceEndpoints` 继续验证该 OTVID。

OT-Auth 服务签发 OTVID 时允许追加其它 Claims，具体见相关 API 文档，但扩展的 Claims 不作为验证 OTVID 是否有效的依据。

### Open Trust Authentication
Open Trust Authentication 服务简称 OT-Auth，是 Open Trust 的核心，它存储主体基本信息、分配 OTID、签发和验证 OTVID、维护跨信任域的联盟机制。

#### Discovery And Verify

OT-Auth 服务必须能基于 Web PKI 被公开发现和验证其信任域的有效性，参考了 [OpenID Connect Discovery](https://openid.net/specs/openid-connect-discovery-1_0.html) 的设计，
验证及发现 Open Trust 服务的 URI 为 `https://trust-domain/.well-known/open-trust-configuration`，返回结果为 application/json 类型的 JSON 文档。

如上所示，该 URI 必须要使用 `https`，其域名必须是信任域的 `trust-domain`，`/.well-known/` 则在 [RFC5785](https://tools.ietf.org/html/rfc5785) 中定义，`open-trust-configuration` 的定义如下：

`issuer` 必须为 OT-Auth 服务的 OTID。

`serviceEndpoints` 必须为 OT-Auth 服务的API 端点组，往这些 endpoint 直接发起 GET 请求应该要返回该服务的简要信息，消费者 SP 从而可以选择其中最快的一个调用，注意该端点的域名可以与所属信任域不同。

`subjectTypesSupported` 必须为 OT-Auth 服务支持的主体类型列表数组，建议用 `user` 代表“用户”类型， `robot` 代表“机器人”类型，用 `app` 代表“应用”类型，用 `service` 代表“服务”类型。

`algValuesSupported` 必须为 OT-Auth 服务支持的 OTVID 算法列表数组。

`keys` 必须为 OT-Auth 服务当前有效的 [JWK](https://tools.ietf.org/html/rfc7517) Set 数组，该数组必须包含至少一个有效 JWK，这组 JWK 只包含公钥及算法等信息，不能包含私钥，用于验证 OTVID。
消费方 SP 应该缓存这组 JWK Set，并基于 `keysRefreshHint` 定期读取最新值，SP 应该用它们来验证请求主体的 OTVID，其中任意一个 JWK 验证成功即可认定该 OTVID 是本信任域的 OT-Auth 服务签发的，是可信的。
当 SP 对 OTVID 有更严格的验证诉求时，可以继续到 OT-Auth 的 `serviceEndpoints` 继续验证该 OTVID，这样可以确保被吊销的 OTVID 不再有效。
当 OTVID 的 `exp` 足够短时（如签发 10 分钟内），则没必要到 OT-Auth 验证它是否被吊销，从而节约 IO 开销。

`keysRefreshHint` 必须为 OT-Auth 服务建议的消费方 SP 对 JWK Set 查询时间间隔。因为 JWK 会定期轮转更新，消费方 SP 有必要定时查询 JWK Set 从而确保获得最新有效值，建议为 3600 秒。

假设信任域为 `ot.example.com`，则该信任域 OT-Auth 服务的 OTID 为 `otid:ot.example.com`，验证该信任域的有效性则请求 `https://ot.example.com/.well-known/open-trust-configuration`，如：
```
GET /.well-known/open-trust-configuration HTTP/2
Host: ot.example.com
```

应该获得如下响应结果：
```
HTTP/2 200 OK
Content-Type: application/json

{
  "issuer": "otid:ot.example.com",
  "serviceEndpoints": ["https://server.example.com/ot"],
  "subjectTypesSupported": ["user", "robot", "app", "service"],
  "algValuesSupported": ["RS256", "ES256", "PS256"],
  "keysRefreshHint": 3600,
  "keys": [
    {
      "kty": "EC",
      "crv": "P-256",
      "x": "MKBCTNIcKUSDii11ySs3526iDZ8AiTo7Tu6KPAqv7D4",
      "y": "4Etl6SRW2YiLUrN5vfvVHuhp7x8PxltmWWlbbM4IFyM",
      "use": "sig",
      "kid": "1"
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
      "kid":"2011-04-29"
    }
  ]
}
```

#### OTID Registration

#### OTID Resolution

#### OTVID Issue And Verify

### Open Trust Access Control

### Open Trust Sign In