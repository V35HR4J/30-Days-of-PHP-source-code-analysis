
![IDOR](image/idor.png?raw=true "IDOR")

# Description
Insecure direct object references (IDOR) are a type of access control vulnerability that arises when an application uses user-supplied input to access objects directly. The term IDOR was popularized by its appearance in the OWASP 2007 Top Ten. However, it is just one example of many access control implementation mistakes that can lead to access controls being circumvented. IDOR vulnerabilities are most commonly associated with horizontal privilege escalation, but they can also arise in relation to vertical privilege escalation.

# Example
Following is a small code snippet of Ultimate Member 2.1.2 which was vulnerable to IDOR. (CVE-2020-6859)
Lets try to analyze the it:

```php
function ajax_file_upload() {
			$ret['error'] = null;
			$ret = array();

			$nonce = $_POST['_wpnonce'];
			$id = $_POST['key'];
			$timestamp = $_POST['timestamp'];
			$user_id = empty( $_POST['user_id'] ) ? get_current_user_id() : $_POST['user_id'];

			UM()->fields()->set_id = $_POST['set_id'];
			UM()->fields()->set_mode = $_POST['set_mode'];
			.
			.
			.
			fileupload()->upload( $id, $nonce );
```

If you see here, the current logged in user is not verified and the file is uploaded just with the help of the id and nonce. So, if we can get the id of any other user, we can upload a file to their account.


# Fix

```php
function ajax_file_upload() {
			$ret['error'] = null;
			$ret = array();

			$nonce = $_POST['_wpnonce'];
			$id = $_POST['key'];
			$timestamp = $_POST['timestamp'];
			$user_id = empty( $_POST['user_id'] ) ? get_current_user_id() : $_POST['user_id'];

			UM()->fields()->set_id = $_POST['set_id'];
			UM()->fields()->set_mode = $_POST['set_mode'];

			// Checks if the current user is the owner of the user_id
			if ( ! UM()->roles()->um_current_user_can( 'edit', $user_id ) ) {
				$ret['error'] = __( 'You haven\'t ability to edit this user', 'ultimate-member' );
				wp_send_json_error( $ret );
			}
			else {
				fileupload()->upload( $id, $nonce );
			}
			.
			.
			.
			fileupload()->upload( $id, $nonce );
```

If you see here, the current logged in user is verified before executing the upload function. So, only the current logged in user can upload a file with their respective id. [Full Commit](https://github.com/ultimatemember/ultimatemember/commit/249682559012734a4f7d71f52609b2f301ea55b1)
# Preventions

- Access Control Checks
- Using Indirect References
- Syntactic and Logical Validation
