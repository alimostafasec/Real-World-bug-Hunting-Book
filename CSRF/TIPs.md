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

