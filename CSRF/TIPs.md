# Cross-Site Request Forgery (CSRF) Explained
Cross-Site Request Forgery (CSRF) is a type of security vulnerability found in web applications. It enables attackers to perform actions on behalf of unsuspecting users by exploiting their authenticated sessions. The attack is executed when a user, who is logged into a victim’s platform, visits a malicious site. This site then triggers requests to the victim’s account through methods like executing JavaScript, submitting forms, or fetching images.
# Quick Check
You could capture the request in Burp and check CSRF protections, and to test from the browser you can click on Copy as fetch and check the request:


<img src="https://hacktricks.wiki/en/images/image%20(11)%20(1)%20(1).png" width="900">

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

# Common pitfalls of defenses

• SameSite pitfalls: SameSite=Lax still allows top-level cross-site navigations like links and form GETs, so many GET-based CSRFs remain possible. See cookie matrix in Hacking with Cookies > SameSite.

• Header checks: Validate Origin when present; if both Origin and Referer are absent, fail closed. Don’t rely on substring/regex matches of Referer that can be bypassed with lookalike domains or crafted URLs, and note the meta name="referrer" content="never" suppression trick.

• Method overrides: Treat overridden methods (_method or override headers) as state-changing and enforce CSRF on the effective method, not just on POST.

• Login flows: Apply CSRF protections to login as well; otherwise, login CSRF enables forced re-authentication into attacker-controlled accounts, which can be chained with stored XSS.

# Lack of token
Applications might implement a mechanism to validate tokens when they are present. However, a vulnerability arises if the validation is skipped altogether when the token is absent. Attackers can exploit this by removing the parameter that carries the token, not just its value. This allows them to circumvent the validation process and conduct a Cross-Site Request Forgery (CSRF) attack effectively.

Moreover, some implementations only check that the parameter exists but don’t validate its content, so an empty token value is accepted. In that case, simply submitting the request with csrf= is enough:
```js
POST /admin/users/role HTTP/2
Host: example.com
Content-Type: application/x-www-form-urlencoded

username=guest&role=admin&csrf=
```

