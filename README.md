[![PMAPI logo](https://sapp.s3.amazonaws.com/github/sutpmapi.png)](https://dev.sign-up.to/)

# PMAPIClient

A PHP client library for [Sign-Up.to's Permission Marketing API (PMAPI)](https://dev.sign-up.to/)

## Composer installation

```composer require sut/pmapi```

## Examples

For full documentation on Sign-Up.to's Permission Marketing API please refer to the [dev site](https://dev.sign-up.to/)

### Example 1: Add a new subscriber and send an opt-in email 

The following example demonstrates use of the Subscriber helper class to create a new subscriber and send an opt-in email.

```php
require_once __DIR__ . '/vendor/autoload.php';

define('CID', 1); // Company ID
define('UID', 1); // User ID
define('HASH', 'xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx'); // API access hash

$request = new SuT\PMAPI\Client\Core\Request(new SuT\PMAPI\Client\Core\AuthHash(UID, CID, HASH));

// Define the id of the list we want to add the subscriber to
$listID = 12345;

// Use the Subscriber helper class to create a new subscriber
$subscriber            = new SuT\PMAPI\Client\Helpers\Subscriber($request);
$subscriber->firstname = "John";
$subscriber->lastname  = "Smith";
$subscriber->email     = "example@example.com";
$subscriber->confirmed = false;
$subscriber->list_id   = $listID;
$subscriber->save();
$subscriber->sendOptInEmail($listID);
// Or you can pass a confirmation URL
$subscriber->sendOptInEmail($listID, 'https://your-domain.com/thanks-for-signing-up');
```

### Example 2: Returning the latest list

The following example demonstrates working with the Request class to retrieve the latest list.

```php
require_once __DIR__ . '/vendor/autoload.php';

// Define your authentication details, if you don't have any then follow the instructions here: 
// https://dev.sign-up.to/documentation/reference/latest/guides/get-started/
define('CID', 1); // Company ID
define('UID', 1); // User ID
define('HASH', 'xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx'); // API access hash

// Create a request object based on the hash authentication method.
// Note: authentication will not occur until an API call is made using the PMAPIRequest object.
$request = new SuT\PMAPI\Client\Core\Request(new SuT\PMAPI\Client\Core\AuthHash(UID, CID,HASH));

// Get the most recently created list.
$args = array(
    'sort'    => 'cdate',
    'reverse' => true,
    'count'   => 1,
);

$response = $request->list->get($args);

if ($response->isError)
{
    die("Failed to obtain a collection of lists: {$response->error}\n");
}

$listID           = $response->data[0]['id'];
$listName         = $response->data[0]['name'];
$createdTimestamp = $response->data[0]['cdate']; // All dates are returned as timestamps
$createdDate      = date('r', $createdTimestamp); 

echo "List '$listName' has id '$listID' and was created on $createdDate\n";
```

### Example 3: Token authentication

The following example demonstrates token authentication.

```php
require_once __DIR__ . '/vendor/autoload.php';

// Define your authentication details, if you don't have any then follow the instructions here: 
// https://dev.sign-up.to/documentation/reference/latest/guides/get-started/
define("USERNAME", "xxxxxxxxxxx"); // Your Sign-Up.to username
define("PASSWORD", "xxxxxxxxxxx"); // Your Sign-Up.to password

// Create a token
$request = new SuT\PMAPI\Client\Core\Request(new SuT\PMAPI\Client\Core\AuthNone());
$response = $request->token->post(array('username' => USERNAME,
                                        'password' => PASSWORD));

if ($response->isError)
{
    die ("ERROR: Authentication failed: {$response->error}\n");
}

// The authentication token
$token = $response->data['token'];

// The token expiry 
$expiry = $response->data['expiry'];

// You can then use this token to make requests
$request = new SuT\PMAPI\Client\Core\Request(new SuT\PMAPI\Client\Core\AuthToken($token));

// Get the most recently created list.
$args = array(
    'sort'    => 'cdate',
    'reverse' => true,
    'count'   => 1,
);

$response = $request->list->get($args);

if ($response->isError)
{
    die("Failed to obtain a collection of lists: {$response->error}\n");
}
```
