![Reflected XSS](image/reflectedXSS.png?raw=true "Reflected XSS")
# Description
Reflected cross-site scripting (or XSS) arises when an application receives data in an HTTP request and includes that data within the immediate response in an unsafe way.

# Impact 
- Perform any action within the application that the user can perform.
- View any information that the user is able to view.
- Modify any information that the user is able to modify.
- Initiate interactions with other application users, including malicious attacks, that will appear to originate from the initial victim user.

# Example

Let's review following source code of Raygun4WP wordpress plugin which was vulnerable to Reflected XSS. 
CVE-2017-9288 which was fixed on version 1.8.1

```php
<?php
	$previousUrl = "javascript:window.history.back();";

	if( isset($_GET['backurl']) ) {
		$previousUrl =$_GET["backurl"];
	}
?>

			<a class="rg4wp-button" href="<?php echo $previousUrl; ?>">Back</a>
```

If you see here, the variable `previousUrl` is taken from user controlled parameter `backurl` and is displayed without any sanitization which makes an attacker to execute malicious js code on the behalf of innocient victim.

# Exploit:

`https://localhost/sendtesterror.php?backurl="><script>alert(1)</script>`

When victim visits this url, as discussed above, the javascript code `<script>alert(1)</script>` triggers on victim's side and an alert pops up. similarly an attacker can steal the cookie with malicious javascript code and perfrom many other unintended actions from victim's side.


# Fix:

```php
<?php
	$previousUrl = "javascript:window.history.back();";

	if( isset($_GET['backurl']) ) {
		$previousUrl = htmlentities($_GET["backurl"]);
	}
?>

			<a class="rg4wp-button" href="<?php echo $previousUrl; ?>">Back</a>


```
Now, the variable `previousUrl` is sanitized using the function `htmlentities()` before displaying it, which converts characters to HTML entities. View [full commit](https://github.com/MindscapeHQ/raygun4wordpress/pull/17/files).

# Preventions:
- Input Validation
- Output Encoding
- Content Security Policy (CSP)