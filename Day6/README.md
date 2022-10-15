![CSRF](image/csrf.png?raw=true "CSRF")

# Description
Cross-site request forgery (also known as CSRF) is a web security vulnerability that allows an attacker to induce users to perform actions that they do not intend to perform. It allows an attacker to partly circumvent the same origin policy, which is designed to prevent different websites from interfering with each other.

# Example
Following is the code snippet from Deny All Firewall (CVE-2019-14681) which is vulnerable to CSRF attack.

```php
if (isset($_GET['daf_remove']) && $_GET['daf_remove'] == 'true') {
 
	if ($this->daf_remove_rules()) {
  ```
With that code, if the GET input "daf_remove" is set to "true" the function daf_remove_rules() runs and the plugin’s functionality is disabled. What is missing there is a nonce check to prevent cross-site request forgery (CSRF) protection, so an attacker could cause a logged in Administrator to send a request that does that without the Administrator intending to disable the functionality.

Exploit:

`<img src= http://localhost/wp-admin/options-general.php?page=daf_settings&daf_remove=true>`

When the victim visits attacker's page, the plugin will be disabled.

# Prevention

A number of code patterns that prevent CSRF attacks exist, and more than one can be applied at the same time as part of a defence in depth security strategy.

- Developers should require anti-forgery tokens for any unsafe methods (POST, PUT, DELETE) and ensure that safe methods (GET, HEAD) do not have any side effects.

- Developers should consider implementing a Synchronizer Token Pattern

- If the origin header is present, developers should verify that its value matches the target origin.

- Importantly, developers should enforce User Interaction based CSRF Defense like the use of Captcha,OTP,Re-authentication,etc.

Reference: [Portswigger](https://portswigger.net/web-security/csrf), [Wordpress](https://plugins.trac.wordpress.org/changeset?reponame=&new=2110522%40deny-all-firewall&old=2110073%40deny-all-firewall)