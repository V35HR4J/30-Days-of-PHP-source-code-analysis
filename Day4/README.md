![XXE](image/xxe.png?raw=true "XXE")

# Description

XML external entity injection (also known as XXE) is a web security vulnerability that allows an attacker to interfere with an application's processing of XML data. It often allows an attacker to view files on the application server filesystem, and to interact with any back-end or external systems that the application itself can access.

# Vulnerable example

```php
<?php
  $xml = file_get_contents($_FILES["file_to_upload"]["tmp_name"]);

  libxml_disable_entity_loader(false);
  $dom = new DOMDocument();
  $dom->loadXML($xml, LIBXML_NOENT);
  echo($dom->textContent);
?>
```

The script parses XML data in an unsafe way and can be exploited to inject a specially forged XML to disclose the `/etc/passwd` system file, referring its path as an external entity.

`<!DOCTYPE d [<!ENTITY e SYSTEM "/etc/passwd">]><t>&e;</t>`

# Prevention

The libxml2 parser resolves the entities when the LIBXML_NOENT option is used, which may be set at system level or used in the code. 

The fixed code parses the XML code without loading the malicious entities.

```php
<?php
  $xml = file_get_contents($_FILES["file_to_upload"]["tmp_name"]);

  libxml_disable_entity_loader(true);
  $dom = new DOMDocument();
  $dom->loadXML($xml);
  echo($dom->textContent);
?>
```

Reference: [OWASP](https://owasp.org/www-community/vulnerabilities/XML_External_Entity_(XXE)_Processing)
