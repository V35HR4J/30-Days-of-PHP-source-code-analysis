![Command Injection](image/commandi.png?raw=true "Command Injection")

# Description
Command injection is an attack in which the goal is execution of arbitrary commands on the host operating system via a vulnerable application. Command injection attacks are possible when an application passes unsafe user supplied data (forms, cookies, HTTP headers etc.) to a system shell. In this attack, the attacker-supplied operating system commands are usually executed with the privileges of the vulnerable application. Command injection attacks are possible largely due to insufficient input validation.


# Example
Let's analyze following code which does nslookup of a given domain.

```php
<?php
    if(isset($_POST["target"]))
    {
        $target = $_POST["target"];
        if($target == "")
        {
            echo "<font color=\"red\">Enter a domain name...</font>";
        }
        else
        {
            echo "<p align=\"left\">" . shell_exec("nslookup  " . commandi($target)) . "</p>";
        }
    }
?>
```
If you see here, target is taken through post request which is fully user controlled. So, if we passed our command via target, it will go inside shell_exec() function then the command is passed to the shell. So, we can simply break the nslookup command and execute our own new command.

# Exploit:
```html
<html>
  <body>
    <form action="https://localhost/injection.php" method="POST">
      <input type="hidden" name="target" value="example.com;id" />
      <input type="hidden" name="form" value="submit" />
      <input type="submit" value="Submit request" />
    </form>
  </body>
</html>

```
As you can see here, we have simply breaked the nslookup command with `;` which means command seperation in linux. Then, we have passed our own command `id` which will give us the current user id. So, in this way we can execute any command we want just like we did `id` command in above POC.


# Prevention:
Developers must use structured mechanisms that automatically enforce the separation between data and code.

Importantly, OS Command Injection vulnerabilities can be entirely prevented if developers stringently avoid OS Command call outs from the application layer. Alternative methods of implementing necessary levels of functionality almost always exist.

If invoking the command shell is unavoidable, endeavor not to use functions that call out using a single string; rather, opt for functions that require a list of individual arguments. These functions typically perform appropriate quoting and filtering of arguments.

References: [OWASP](https://www.owasp.org/index.php/Command_Injection)