+++
date = '2026-10-26T11:36:15+09:00'
draft = false
title = 'Iptime Ddns Caa'
+++

## 증상

블로그를 공개하기 위해 cert-manager를 설치하고 Let's Encrypt 인증서를 신청했지만 발급되지 않았다.
포트포워딩은 정상이었고 외부에서 80 포트로 접근도 가능한 상태였다.

## 로그
'''
kubectl describe challenge <challenge-name>

Reason: Error accepting authorization: acme: authorization error
for savom.iptime.org: 403 urn:ietf:params:acme:error:caa:
While processing CAA for savom.iptime.org:
CAA record for iptime.org prevents issuance
'''


## 원인

CAA는 "이 도메인의 인증서는 어느 기관이 어떤 조건으로 발급할 수 있다"를 선언하는 DNS 레코드다.
도메인 소유자가 의도하지 않은 발급을 막는 보안 장치다.

```bash
dig CAA iptime.org +short
0 issuewild ";"
0 issue "letsencrypt.org; accounturi=...; validationmethods=dns-01"
```

Let's Encrypt 자체는 허용돼 있다. 다만 조건이 붙어 있다.

- `accounturi` — EFM의 특정 ACME 계정으로만 발급 가능
- `validationmethods=dns-01` — DNS-01 챌린지만 허용
- `issuewild ";"` — 와일드카드는 완전 차단

내가 쓴 건 HTTP-01이었으니 두 번째 조건에서 걸렸다.
DNS-01로 바꿔도 소용없다. `_acme-challenge` TXT 레코드를 넣을 권한이 없기 때문이다.

결국 서브도메인을 빌려 쓰는 입장에서는 어떤 방법으로도 발급받을 수 없다.
## 해결

DuckDNS로 전환했다.

```bash
dig CAA duckdns.org +short
0 issue "letsencrypt.org"
0 issuewild "letsencrypt.org"
0 iodef "mailto:abuse@duckdns.org"
```

Let's Encrypt를 명시적으로 허용하고 있고, 와일드카드 인증서까지 가능하다.

## 교훈

무료 DDNS를 고를 때는 CAA 레코드를 먼저 확인하자.
포트포워딩과 DNS 응답만 보고 진행하면 인증서 단계에서 막히는데,
그때는 이미 Ingress와 cert-manager 설정을 다 해놓은 뒤라 되돌리는 비용이 크다.
