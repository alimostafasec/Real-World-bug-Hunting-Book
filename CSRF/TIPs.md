# Cross-Site Request Forgery (CSRF) Explained
Cross-Site Request Forgery (CSRF) is a type of security vulnerability found in web applications. It enables attackers to perform actions on behalf of unsuspecting users by exploiting their authenticated sessions. The attack is executed when a user, who is logged into a victim’s platform, visits a malicious site. This site then triggers requests to the victim’s account through methods like executing JavaScript, submitting forms, or fetching images.
# Quick Check
You could capture the request in Burp and check CSRF protections, and to test from the browser you can click on Copy as fetch and check the request:


<img src="https://hacktricks.wiki/en/images/image%20(11)%20(1)%20(1).png" width="900">

# Defending Against CSRF
Several countermeasures can be implemented to protect against CSRF attacks:
• SameSite cookies: This attribute prevents the browser from sending cookies along with cross-site requests. More about SameSite cookies.
