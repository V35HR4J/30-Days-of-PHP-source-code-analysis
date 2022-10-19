![SSRF](image/ssrf.png?raw=true "SSRF")

# Description
Server-side request forgery (also known as SSRF) is a web security vulnerability that allows an attacker to induce the server-side application to make requests to an unintended location. A successful SSRF attack can often result in unauthorized actions or access to data within the organization, either in the vulnerable application itself or on other back-end systems that the application can communicate with. In some situations, the SSRF vulnerability might allow an attacker to perform arbitrary command execution.

# Example
Let's have a look to following code which is vulnerable to SSRF.

```php
<?php
if (isset($_GET['image_url'])){
$url = $_GET['image_url'];
$file = fopen($url, 'rb');
header("Content-Type: image/png");
fpassthru($file);
}
echo 'Please enter image url'

?>
```

If you see the above code, the server tires to send the request to user controlled image_url and tires to dump the contents of the URL.

# Exploitation
Since the server is not validating the URL if it is image or not. neither checking schemas, attacker can exploit it to read internal files:

`http://localost/ssrf.php?image_url=file:///etc/passwd`

Reading AWS meta data:

`http://localost/ssrf.php?image_url=http://169.254.169.254/latest/meta-data/hostname`

# Prevention
- Disabled unused URL schemas like file:// , ftp:// , gopher://
- Internal services has to protected by authentication to add an extra layer of protection

References: [shieldfy](https://shieldfy.io/security-wiki/server-side-request-forgery/server-side-request-forgery/)