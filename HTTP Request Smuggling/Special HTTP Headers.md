# Special Http Headers :

#### Headers to Change Location 

* Rewrite **IP source**:

-   `X-Originating-IP: 127.0.0.1`
-   `X-Forwarded-For: 127.0.0.1`
-   `X-Forwarded: 127.0.0.1`
-   `Forwarded-For: 127.0.0.1`
-   `X-Forwarded-Host: 127.0.0.1`
-   `X-Remote-IP: 127.0.0.1`
-   `X-Remote-Addr: 127.0.0.1`
-   `X-ProxyUser-Ip: 127.0.0.1`
-   `X-Original-URL: 127.0.0.1`
-   `Client-IP: 127.0.0.1`
-   `X-Client-IP: 127.0.0.1`
-   `X-Host: 127.0.0.1`
-   `True-Client-IP: 127.0.0.1`
-   `Cluster-Client-IP: 127.0.0.1`
-   `Via: 1.0 fred, 1.1 127.0.0.1`
-   `Connection: close, X-Forwarded-For` (Check hop-by-hop headers)

Rewrite **location**:

-   `X-Original-URL: /admin/console`
-   `X-Rewrite-URL: /admin/console`

Hop-by-Hop headers

A hop-by-hop header is a header which is designed to be processed and consumed by the proxy currently handling the request, as opposed to an end-to-end header.

-   `Connection: close, X-Forwarded-For`

HTTP Request Smuggling

-   `Content-Length: 30`
-   `Transfer-Encoding: chunked`

------------------------------------------------------------

The Expect header: 


It's posible for the client to send the header `Expect: 100-continue` and then the server could respond with `HTTP/1.1 100 Continue` to allow the client to continue sending the body of the request. However, some proxies don't really llike this header.

Interesting results of `Expect: 100-continue`:

-   Sending a HEAD request with a body the server didn't took into account that HEAD requests don't have body and keep the connection open until it timed out.
-   Another servers sent extrange data: Random data read from the socket in the response, secret keys or even it allowed to prevent the front-end from removing header values.
-   It also caused a `0.CL` desync cause the backend responded with a 400 response isntead of a 100 response, but the proxy front-end was prepared to send the body of the initial request, so it sends it and the backend takes it as new request.
-   Sending an `Expect: y 100-continue` variation also caused the `0.CL` desync.
-   A similar error where the backend responded with a 404 generated a `CL.0` desync because the malicious request indicates a `Content-Length` so the backend sends the malicious request + the `Content-Length` bytes of the next request (of a victim), this desyncs the queue cause the backend sends the 404 request for the malicious request + the repsonse of the victim requests, but the front end thought that only 1 request was sent, so the second response is sent to a seond victim request and the the reponse of taht one is sent to the next one...

  ---------------------------------------------------------

Message body information


-   **`Content-Length`:** The size of the resource, in decimal number of bytes.
-   **`Content-Type`**: Indicates the media type of the resource
-   **`Content-Encoding`**: Used to specify the compression algorithm.
-   **`Content-Language`**: Describes the human language(s) intended for the audience, so that it allows a user to differentiate according to the users' own preferred language.
-   **`Content-Location`**: Indicates an alternate location for the returned data.

From a pentest point of view this information is usually "useless", but if the resource is **protected** by a 401 or 403 and you can find some **way** to **get** this **info**, this could be **interesting.**\
For example a combination of **`Range`** and **`Etag`** in a HEAD request can leak the content of the page via HEAD requests:

-   A request with the header `Range: bytes=20-20` and with a response containing `ETag: W/"1-eoGvPlkaxxP4HqHv6T3PNhV9g3Y"` is leaking that the SHA1 of the byte 20 is `ETag: eoGvPlkaxxP4HqHv6T3PNhV9g3Y`

Server Info


-   `Server: Apache/2.4.1 (Unix)`
-   `X-Powered-By: PHP/5.3.3`

Controls


-   **`Allow`**: This header is used to communicate the HTTP methods a resource can handle. For example, it might be specified as `Allow: GET, POST, HEAD`, indicating that the resource supports these methods.
-   **`Expect`**: Utilized by the client to convey expectations that the server needs to meet for the request to be processed successfully. A common use case involves the `Expect: 100-continue` header, which signals that the client intends to send a large data payload. The client looks for a `100 (Continue)` response before proceeding with the transmission. This mechanism helps in optimizing network usage by awaiting server confirmation.

### **X-Content-Type-Options**
  
This header prevents MIME type sniffing, a practice that could lead to XSS vulnerabilities. It ensures that browsers respect the MIME types specified by the server.

`X-Content-Type-Options: nosniff`

### **X-Frame-Options**

To combat clickjacking, this header restricts how documents can be embedded in `<frame>`, `<iframe>`, `<embed>`, or `<object>` tags, recommending all documents to specify their embedding permissions explicitly.

`X-Frame-Options: DENY`

### **Cross-Origin Resource Policy (CORP) and Cross-Origin Resource Sharing (CORS)**

CORP is crucial for specifying which resources can be loaded by websites, mitigating cross-site leaks. CORS, on the other hand, allows for a more flexible cross-origin resource sharing mechanism, relaxing the same-origin policy under certain conditions.

```js
Cross-Origin-Resource-Policy: same-origin
Access-Control-Allow-Origin: https://example.com
Access-Control-Allow-Credentials: true
```

