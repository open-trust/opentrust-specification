Open Trust Standards: OTID & OTVID
====

## Status of this Memo
本标准目前已完成编写，但已定义的内容可能会调整优化。

## Abstract
Open Trust 致力于为云原生软件提供一种安全、标准的接口可信开放机制，让云原生软件彼此之间的连接打通变得简单可行，让“用 API 连接世界万物”成为可能。

## Open Trust Identity
Open Trust Identity 简称为 OTID，参考了 [W3C DID](https://w3c.github.io/did-core/) 的设计，但没有像 DID 那样支持 Path、Query、Fragment 等，示例如下：

```otid:ot.example.com:user:9eebccd2-12bf-40a6-b262-65fe0487d453```

OTID 是符合 [RFC3986](https://tools.ietf.org/html/rfc3986) URI 规范的字符串，它由 4 个部分组成：

```otid:trust-domain:subject-type:subject-id```

1. scheme，固定为 `otid`；
2. trust-domain，信任域，它由小写 ALPHA / DIGIT / "." / "-" / "_" 组成，它必须是由受信任公共 CA 颁发的证书签名的域名，并且要提供 [Open Trust Authority](#open-trust-authority) 服务能力，基于公共 PKI 体系，是 Open Trust 信任域的联合信任机制的基础；
3. subject-type，可信主体类型，它由小写 ALPHA / DIGIT / "." / "-" / "_" 组成，任何事物都可以是信任主体，如 person, group, organization, physical thing, digital thing, logical thing 等，但一个实际部署的 Open Trust 系统中，subject-type 可通过 Open Trust Authority 服务配置来限定支持的类型；
4. subject-id，可信主体身份，它由小写 ALPHA / DIGIT / "." / "-" / "_" 组成，在 trust-domain:subject-type 作用域下唯一；
5. OTID 是持久的、不可变的、小写的、URL 兼容的，长度不能超过 512 字节，建议长度控制在 128 字节以内；
6. 信任域 Open Trust Authority 服务的 OTID 是一个特殊值 `otid:trust-domain`，其 subject-type 和 subject-id 都为空，如 `otid:ot.example.com`，其它 OTID 的 subject-type 和 subject-id 都不能为空。

OTID 相关实现可参考 [ot-go-lib](https://github.com/open-trust/ot-go-lib/blob/master/otid.go)

## Open Trust Verifiable Identity Document
Open Trust Verifiable Identity Document 简称为 OTVID，是可信主体用于证明自己身份的一份数据文档，它包含了 OTID、文档签名以及其它的一些属性或声明。OTVID 参考了 [SPIFFE](https://github.com/spiffe/spiffe) 的设计，但只基于 JWT 标准封装。

当可信主体访问信任域的 Open Trust Authority 服务（以下简称 OT-Auth）时，需要提供由可信主体自己签发的 OTVID，OT-Auth 保存了信任域内所有可信主体的公钥，从而可以验证其 OTVID。

当可信主体访问信任域内的其它服务时，需要提供由信任域的 OT-Auth 签发的 OTVID，信任域内的服务作为 Verifier 验证者，应该且只能通过本信任域 OT-Auth 提供的公钥或 OT-Auth 的 API 来验证 OTVID。

当可信主体访问其它信任域的服务（包括其它信任域的 OT-Auth）时，需要提供由目标信任域的 OT-Auth 签发的 OTVID，因为目标信任域内的服务只会通过目标信任域的 OT-Auth 来验证 OTVID。跨信任域签发 OTVID 由 OT-Auth 的 federation 联盟机制来实现。

可信主体的 OTVID 通过 HTTP 协议 `Authorization` header ("authorization" for HTTP/2) 和 `Bearer` scheme 传递给访问客体，详见 [RFC 6750 section 2.1](https://tools.ietf.org/html/rfc6750#section-2.1)。如 HTTP/1.1 中为 `Authorization: Bearer <serialized_token>`，HTTP/2 中则为 `authorization: Bearer <serialized_token>`。

OTVID 是一个标准的 [JWT](https://tools.ietf.org/html/rfc7519)，其组成见下。

1. JOSE Header Algorithm `alg` 必须设置为如下值之一，建议使用 ES512：

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

2. JOSE Header Key ID `kid` 必须被设置为签发该 OTVID 的 JWK 的 `kid`。
3. Subject Claim `sub` 必须被设置为可信主体的 OTID，如 `otid:ot.example.com:user:9eebccd2-12bf-40a6-b262-65fe0487d453`。
4. Issuer Claim `iss` 必须被设置为 OTVID 签发者的 OTID，如 `otid:ot.example.com`，签发者一般是 OT-Auth 或可信主体自身。
5. Audience Claim `aud` 必须被设置为访问客体的 OTID，如 `otid:ot.example.com:app:abc123`，并且只能设置一个（JWT 标准允许多个值）。
可信主体访问信任域的 OT-Auth 时，其 OTVID 的 `aud` 值是  OT-Auth 的 OTID，如 `otid:ot.example.com`。访问客体必须要验证该 OTVID 的 `aud` 值是自己的 OTID。
6. Expiration Time Claim `exp` 必须被设置，其值为 Unix 时间戳，1970-01-01 UTC 以来的秒数，一般建议设置为3~10分钟之内。
7. Issued At Claim `iat` 必须被设置为 OTVID 签发时间，Unix 时间戳。
8. Release ID Claim `rid` 为可选值，字符串类型，它不是 [JWT](https://tools.ietf.org/html/rfc7519) 标准 Claim。Release ID 定义为可信主体的可信发布标识，存储在 OT-Auth 中，并且不应该被外部访问到。目前用于实现 OIVID 的 revoke 机制。当可信主体的可信信息发生变动时，如可信主体的私钥泄露需要更新，OT-Auth 应该同时更新其 Release ID 值，并且之前的 Release ID 值签发出去的 OIVIDs 不再有效。当 OTVID 的 `rid` 值存在时，访问客体 Verifier 用信任域公钥验证 OTVID 之后，还应该到 OT-Auth 实时验证该 OTVID 是否有效。当 OTVID 的 `exp` 足够短（如 10 分钟）时，可以不必设置 `rid`。
9. OT-Auth 服务签发 OTVID 时允许追加其它 Claims，具体见相关 API 文档，但扩展的 Claims 仅用于实际业务便利，不作为验证 OTVID 是否有效的依据。
10. OTVID 序列化后的长度不能超过 2048 字节，建议长度控制在 1024 字节以内；

OTVID 相关实现可参考 [ot-go-lib](https://github.com/open-trust/ot-go-lib/blob/master/otvid.go)
