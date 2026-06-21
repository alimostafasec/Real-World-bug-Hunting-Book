# Recent Vulnerabilities (2023 -- 2025)

>The last few years have produced several high-impact CRLF/HTTP header-injection bugs in widely-used server- and client-side components. Reproducing and studying them locally is an excellent way of understanding real-world exploitation patterns.

| Year | Component | CVE / Advisory | Root cause | PoC highlight |
| --- | --- | --- | --- | --- |
| 2024 | RestSharp (≥110.0.0 <110.2.0) | **CVE-2024-45302** | The `AddHeader()` helper did not sanitize CR/LF, allowing construction of multiple request headers when RestSharp is used as an HTTP client inside back-end services. Down-stream systems could be coerced into SSRF or request smuggling. | `client.AddHeader("X-Foo","bar%0d%0aHost:evil")` |
| 2024 | Refit (≤ 7.2.101) | **CVE-2024-51501** | Header attributes on interface methods were copied verbatim into the request. By embedding `%0d%0a`, attackers could add arbitrary headers or even a second request when Refit was used by server-side worker jobs. | `[Headers("X: a%0d%0aContent-Length:0%0d%0a%0d%0aGET /admin HTTP/1.1")]` |
| 2023 | Apache APISIX Dashboard | **GHSA-4h3j-f5x9-r6x3** | User-supplied `redirect` parameter was echoed into a `Location:` header without encoding, enabling open redirect + cache poisoning. | `/login?redirect=%0d%0aContent-Type:text/html%0d%0a%0d%0a<script>alert(1)</script>` |
