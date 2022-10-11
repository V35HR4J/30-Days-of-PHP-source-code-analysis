![Directory Traversal](image/Traversal.png?raw=true "Directory Traversal")

# Description

Directory traversal (also known as file path traversal) is a web security vulnerability that allows an attacker to read arbitrary files on the server that is running an application. This might include application code and data, credentials for back-end systems, and sensitive operating system files. In some cases, an attacker might be able to write to arbitrary files on the server, allowing them to modify application data or behavior, and ultimately take full control of the server.

# Impact

The damage an attacker can cause by employing this type of attack is really only limited by the value of the exposed information. If a developer has structured their web root folder to include sensitive configuration files, for example, the fallout will, of course, be highly damaging. Furthermore, as with many other attacks that are a part of the attacker's toolkit, the vulnerability can be used by an attacker as a stepping stone, leading to the full compromise of the system.

# Vulnerable Example

The example below shows a vulnerable Symfony controller that serves files from a directory in an unsecure way:

```php
class FetchFileController extends Controller
{
    const ROOT = '/path/to/root/';

    /**
     * @Route("/", name="fetch_file_index")
     * @Method("GET")
     */
    public function indexAction(Request $request)
    {
        $file_name = $request->query->get('filename');
        $file_data = @file_get_contents(self::ROOT . "/$file_name");
        return $this->render('fetch_file/index.html.twig', ['file_data' => $file_data]);
    }
}
```

The file_name query parameter is controlled by the user; it is possible to retrieve arbitrary files in the filesystem by injecting `../` in the file_name parameter as shown below:

`http://example.com/fetch_file/?file_name=../../../../../etc/passwd`
This results in the absolute path `/path/to/root/../../../../../etc/passwd`, and its canonicalized form: `/etc/passwd`.

# Prevention

Make sure to avoid using the user-provided value to escape the designed directory by means of ../. The basename function can be used to extract the base component of a path.

To remediate the vulnerability in the previous example, the $file_name variable should be fixed as follows:

`$file_name = basename($file_name);`
The previous attack would result in reading the `/path/to/root/passwd` file (which does not exist).

References: [OWASP](https://owasp.org/www-community/attacks/Path_Traversal) [PORTSWIGGER](https://portswigger.net/web-security/file-path-traversal) 