![DOM based XSS](image/dom.png?raw=true "DOM based XSS")
# Description
DOM Based XSS (or as it is called in some texts, “type-0 XSS”) is an XSS attack wherein the attack payload is executed as a result of modifying the DOM “environment” in the victim’s browser used by the original client side script, so that the client side code runs in an “unexpected” manner. That is, the page itself (the HTTP response that is) does not change, but the client side code contained in the page executes differently due to the malicious modifications that have occurred in the DOM environment.

# Example
Suppose the following code is used to create a form to let the user choose their preferred language. A default language is also provided in the query string, as the parameter “default”.

```html
…
Select your language:
<select><script>
document.write("<OPTION value=1>"+decodeURIComponent(document.location.href.substring(document.location.href.indexOf("default=")+8))+"</OPTION>");
document.write("<OPTION value=2>English</OPTION>");
</script></select>
…
```
The page is invoked with a URL such as:

`http://www.some.site/page.html?default=French`

A DOM Based XSS attack against this page can be accomplished by sending the following URL to a victim:

`http://www.some.site/page.html?default=<script>alert(document.cookie)</script>`

When the victim clicks on this link, the browser sends a request for:

`/page.html?default=<script>alert(document.cookie)</script>`

to www.some.site. The server responds with the page containing the above Javascript code. The browser creates a DOM object for the page, in which the document.location object contains the string:

`http://www.some.site/page.html?default=<script>alert(document.cookie)</script>`

The original Javascript code in the page does not expect the default parameter to contain HTML markup, and as such it simply decodes and echoes it into the page (DOM) at runtime. The browser then renders the resulting page and executes the attacker’s script:

`alert(document.cookie)`


# Preventions

The primary rule that you must follow to prevent DOM XSS is: sanitize all untrusted data, even if it is only used in client-side scripts. If you have to use user input on your page, always use it in the text context, never as HTML tags or any other potential code.

Avoid methods such as document.innerHTML and instead use safer functions, for example, document.innerText and document.textContent. If you can, entirely avoid using user input, especially if it affects DOM elements such as the document.url, the document.location, or the document.referrer.

Also, keep in mind that DOM XSS and other types of XSS are not mutually exclusive. Your application can be vulnerable to both reflected/stored XSS and DOM XSS. The good news is that if user input is handled properly at the foundation level (e.g. your framework), you should be able to mitigate all XSS vulnerabilities.

References: [OWASP](https://owasp.org/www-community/attacks/DOM_Based_XSS), [Acunetix](https://www.acunetix.com/blog/web-security-zone/how-to-prevent-dom-based-cross-site-scripting/#:~:text=How%20To%20Prevent%20DOM%20XSS,Avoid%20methods%20such%20as%20document.)