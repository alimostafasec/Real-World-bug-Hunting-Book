# What is

>This vulnerability occurs when a **desyncronization** between **front-end proxies** and the **back-end** server allows an **attacker** to **send** an HTTP **request** that will be **interpreted** as a **single request** by the **front-end** proxies (load balance/reverse-proxy) and **as 2 request** by the **back-end** server.\
This allows a user to **modify the next request that arrives to the back-end server after his**.

* **Content-Length**

> The Content-Length entity header indicates the size of the entity-body, in bytes, sent to the recipient.

* **Transfer-Encoding: chunked**

> The Transfer-Encoding header specifies the form of encoding used to safely transfer the payload body to the user.\
> Chunked means that large data is sent in a series of chunks


>[!TIP]
>* NOTE :
>
>When trying to exploit this with Burp Suite **disable `Update Content-Length` and `Normalize HTTP/1 line endings`** in the repeater because >some gadgets abuse newlines, carriage returns and malformed content-lengths.

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

        
