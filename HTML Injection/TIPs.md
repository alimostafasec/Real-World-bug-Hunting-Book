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
***Note: A HTML comment can be closed using********`-->`********or ****`--!>`***

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

