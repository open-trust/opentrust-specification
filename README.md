Open Trust Specifications 可信开放标准
====

## 背景
云计算、大数据、移动互联网、物联网等信息技术正在以加速的态势推进社会的数字化转型，为各行各业带来了创新的生产力，也带来了极大的复杂性。

集成、打通、安全、开放成为企业信息系统扩展能力边界、开拓市场的最大挑战。把别人的 API 能力集成进来，把自己的 API 能力开放出去，是企业 IT 服务基本的诉求。

## Open Trust 定义
Open Trust 致力于为云原生软件提供一种安全、标准的接口可信开放机制，让云原生软件彼此之间的连接打通变得简单可行，让“用 API 连接世界万物”成为可能。

与 Zero Trust 零信任架构不同，Zero Trust 的目标是打破内网边界，让企业的网络资源能从任何地方被安全的访问，Open Trust 的目标是让所有的网络资源都能以安全、标准的方式开放出来，相互连接，从而构建更强大的生态，派生更多的创新形态。

Open Trust 是云原生生态的基础设施，是数字时代的基本要素。

## Open Trust Standards

Open Trust 标准及其代码实现目前正在编写、迭代优化之中，不建议用于生产环境，它由以下五部分组成：

1. OTID: [Open Trust Identity](https://github.com/open-trust/specifications/blob/master/standards_otid_otvid.md#open-trust-standards-otid--otvid) ✓
2. OTVID: [Open Trust Verifiable Identity Document](https://github.com/open-trust/specifications/blob/master/standards_otid_otvid.md#open-trust-standards-otid--otvid) ✓
3. OT-Auth: [Open Trust Authority](https://github.com/open-trust/specifications/blob/master/standards_ot_auth.md#open-trust-standards-ot-auth) (WIP)
4. OT-AC: Open Trust Access Control (TODO)
5. OT-Login: Open Trust Login (TODO)

## Libraries, Products, and Tools

### Libraries

- [ot-go-lib](https://github.com/open-trust/ot-go-lib) Golang library for Open Trust support.
- ot-js-lib [TODO]

### Products

- ot-auth [TODO]
- ot-ac [TODO]

### Tools

#### ogto
`otgo` is a CLI tool for generating private and public key, signing or verifying OTVID.

Install & Usage:
```sh
go get github.com/open-trust/ot-go-lib/cmd/otgo
otgo help
```

## Users

- Teambition Urbs 灰度服务 [urbs-setting](https://github.com/teambition/urbs-setting), [urbs-console](https://github.com/teambition/urbs-console)
