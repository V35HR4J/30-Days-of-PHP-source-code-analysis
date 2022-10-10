![SQLi](image/sqli.png?raw=true "SQL Injection")

# Description
A SQL injection attack consists of insertion or “injection” of a SQL query via the input data from the client to the application. A successful SQL injection exploit can read sensitive data from the database, modify database data (Insert/Update/Delete), execute administration operations on the database (such as shutdown the DBMS), recover the content of a given file present on the DBMS file system and in some cases issue commands to the operating system. SQL injection attacks are a type of injection attack, in which SQL commands are injected into data-plane input in order to affect the execution of predefined SQL commands.


An attacker can use SQL Injection to manipulate an SQL query via the input data from the client to the application, thus forcing the SQL server to execute an unintended operation constructed using untrusted input.

# Impact
A successful SQL Injection attack can result in a malicious user gaining complete access to all data in a database server with the ability to execute unauthorized SQL queries and compromise the confidentiality, integrity, and availability of the application. Depending on the backend DBMS used and the permissions granted to the user on the database, a SQL Injection could result in arbitrary file read/write and even remote code execution.

The severity of attacks that have leveraged SQL Injection should not be understated. Notorious breaches, including the devastating and internationally renowned hacks of Sony Pictures and LinkedIn, for example, are reported to have been executed using SQL Injection.

# Vulnerable example

Consider the following Symfony controller for the endpoint `/items?q=...:`

```php
class ItemsController extends Controller
{
    /**
     * @Route("/", name="items_index")
     * @Method("GET")
     */
    public function index(Request $request)
    {
        // fetch the query parameters
        $pattern = $request->query->get('q');

        // get matching list
        $manager = $this->getDoctrine()->getManager();
        $rsm = new ResultSetMapping();
        $rsm->addScalarResult('name', 'name');
        $query = $manager->createNativeQuery("SELECT * FROM items WHERE name LIKE '%$pattern%'", $rsm);
        $items = $query->getResult();

        // render the template
        return $this->render('items/index.html.twig', [
          'items' => $items
        ]);
    }
}
```
### Example exploit: 
injecting `/items?q=administrator'--` will tell the database to execute following query which result in login bypass for the user `administrator`

`SELECT * FROM users WHERE username = 'administrator'--' AND password = '`

## Prevention
The solution is always to avoid string concatenation; common solutions are (in order of preference, where possible):

* using higher level APIs provided by the framework at hand;

* using prepared statements;

* using appropriate escaping.

```
$stmt = $mysqli->prepare("SELECT * FROM items WHERE name LIKE ?");
$stmt->bind_param("s", "%$value%");
$stmt->execute();
$result = $stmt->get_result();
```

Reference: [OWASP](https://github.com/OWASP/CheatSheetSeries/blob/master/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.md)
