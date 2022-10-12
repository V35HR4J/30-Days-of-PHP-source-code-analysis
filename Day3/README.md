![Type Juggling](image/Traversal.png?raw=true "Type Juggling")

# Description

Type Juggling (also known as Type Confusion) vulnerabilities are a class of vulnerability wherein an object is initialized or accessed as the incorrect type, allowing an attacker to potentially bypass authentication or undermine the type safety of an application, possibly leading to arbitrary code execution.

# Impact

A successful Type Juggling attack can result in the complete compromise of the confidentiality, integrity, and availability of the target system. For example, the type confusion vulnerability CVE-2015-0336 in Adobe Flash Player allows an attacker to execute arbitrary code, which could lead to unauthorized access or the modification of data.


# Vulnerable Example

```php
if($_POST['password'] == "secret") {
  ...
}
```
Here, by giving the password as 0, the statement will evaluate to true. This is because PHP attempts to convert "secret" to an integer, which it cannot, and thus, converts the string to 0.



# Prevention

Always use strict comparison where possible, using `===` instead of `==` and similarly `!==` in place of `!=`.

References: [OWASP](https://owasp.org/www-pdf-archive/PHPMagicTricks-TypeJuggling.pdf)