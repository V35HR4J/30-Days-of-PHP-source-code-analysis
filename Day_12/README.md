![Stored XSS](image/StoredXSS.png?raw=true "Stored XSS")
# Description
Stored cross-site scripting, also called persistent cross-site scripting, is a specific type of cross-site scripting (XSS) vulnerability. The terms stored or persistent mean that the attacker first sends the payload to the web application, then the application saves (i.e. stores/persists) the payload (for example, in a database or server-side text files), and finally, the application unintentionally executes the payload for every victim visiting of its web pages.

# Example
In this example, the developer wants to include a simple comment section on one of their pages (page.php) without deploying a full CMS such as WordPress. They include the following form on the page.php web page:

```html
<form action="/page.php" method="post" id="comment">
  <label for="name">Your name:</label>
  <input type="text" id="name" name="name">
  <label for="comment">Your comment:</label>
  <textarea id="comment" name="comment" rows="5" cols="30"></textarea>
  <button type="submit" form="comment" value="comment">Add a comment</button>
</form>
```

The page.php file includes the following code:

```php
// Add a new comment into the database 
(...)
$name=$_POST["name"];
$comment=$_POST["comment"];
$sql = "INSERT INTO comments (name, comment) VALUES (?,?)";
$statement = $pdo->prepare($sql);
$statement->execute([$name, $comment]);
(...)
// Display existing comments
$comments = $db->query('SELECT * FROM comments')->fetchAll();
foreach($comments as $comment) {
    echo "<tr><td>".$comment['name']."</td>";
    echo "<td>".$comment['comment']."</td></tr>";
}
(...)
```
As you can see, the application inserts the comment into the database without any validation or sanitization and later displays it on the same page for other users, again with no validation or sanitization.

# Exploit
The attacker enters the following comment into the form, leaving the name empty:

`<script>alert("LEAVE THIS PAGE! YOU ARE BEING HACKED!");</script>`

The comment is saved into the database. From now on, when any user visits the page, their browser interprets the following code:

`<tr><td></td><td><script>alert("LEAVE THIS PAGE! YOU ARE BEING HACKED!");</script></td></tr>`

The browser finds a `<script>` tag and executes the JavaScript code within it. As a result, it displays a pop-up window for the user urging them to leave the page.

The consequences in this fairly innocent example are that users, afraid for their safety, will stop visiting the page until the administrator is notified and removes malicious content from the database.


# Fix
```php
// Add a new comment into the database 
// HTMLPurifier with HTML escaping to avoid XSS
(...)
$name=$_POST["name"];
$comment=$_POST["comment"];
// Purify user data using HTMLPurifier
(...)
$purifier = new HTMLPurifier($config);
$purified_name = $purifier->purify($name);
$purified_comment = $purifier->purify($comment);
// Just to be sure, HTML-escape special characters
$safe_name = htmlspecialchars($purified_name, ENT_QUOTES);
$safe_comment = htmlspecialchars($purified_comment, ENT_QUOTES);
// Save safe data in the database
$sql = "INSERT INTO comments (name, comment) VALUES (?,?)";
$statement = $pdo->prepare($sql);
$statement->execute([$safe_name, $safe_comment]);
(...)
```
As you can see, the code contains HTMLPurifier and it escapes HTML characters which fixes the stored XSS issue.

# Preventions:
- Input Validation
- Output Encoding
- Content Security Policy (CSP)

References: [invicti](https://www.invicti.com/learn/stored-xss-persistent-cross-site-scripting/),[geeksforgeeks](https://www.geeksforgeeks.org/what-is-cross-site-scripting-xss/)