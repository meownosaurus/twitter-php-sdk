Twitter PHP (v.1.0)
------------

The [Twitter Platform](https://dev.twitter.com/) is
a set of APIs that make your app more social.


Get started
-----

[Register your application](https://dev.twitter.com/apps/) with Twitter, and receive your OAuth `consumer_key` and `consumer_secret`.  

Usage
-----

Users start out on connect.php which displays the "Sign in with Twitter" image hyperlinked
to redirect.php. This button should be displayed on your homepage in your login section. The
client credentials are saved in config.php as `CONSUMER_KEY` and `CONSUMER_SECRET`. You can
save a static callback URL in the app settings page, in the config file or use a dynamic
callback URL later in step 2. In example use https://example.com/callback.php.

1) When a user lands on redirect.php we build a new Twitter object using the client credentials.
If you have your own configuration method feel free to use it instead of config.php.

```php
<?php

    i$twitter = new Twitter(CONSUMER_KEY, CONSUMER_SECRET); // Use config.php client credentials

?>
```

2) Using the built $twitter object you will ask Twitter for temporary credentials. The `oauth_callback` value is required.

    $temporary_credentials = $twitter->getRequestToken(OAUTH_CALLBACK); // Use config.php callback URL.

```php
<?php

    i$twitter = new Twitter(CONSUMER_KEY, CONSUMER_SECRET); // Use config.php client credentials

?>
```

3) Now that we have temporary credentials the user has to go to Twitter and authorize the app
to access and updates their data. You can also pass a second parameter of FALSE to not use [Sign
in with Twitter](https://dev.twitter.com/docs/auth/sign-twitter).

```php
<?php

    $redirect_url = $twitter->getAuthorizeURL($temporary_credentials); // Use Sign in with Twitter
    $redirect_url = $twitter->getAuthorizeURL($temporary_credentials, FALSE);

?>
```

4) You will now have a Twitter URL that you must send the user to.

    https://api.twitter.com/oauth/authenticate?oauth_token=xyz123

5) The user is now on twitter.com and may have to login. Once authenticated with Twitter they will
will either have to click on allow/deny, or will be automatically redirected back to the callback.

6) Now that the user has returned to callback.php and allowed access we need to build a new
Twitter object using the temporary credentials.

```php
<?php

    $twitter = new Twitter(CONSUMER_KEY, CONSUMER_SECRET, $_SESSION['oauth_token'],$_SESSION['oauth_token_secret']);

?>
```

7) Now we ask Twitter for long lasting token credentials. These are specific to the application
and user and will act like password to make future requests. Normally the token credentials would
get saved in your database but for this example we are just using sessions.

```php
<?php

    $token_credentials = $twitter->getAccessToken($_REQUEST['oauth_verifier']);

?>
```

8) With the token credentials we build a new Twitter object.


```php
<?php

    $twitter = new Twitter(CONSUMER_KEY, CONSUMER_SECRET, $token_credentials['oauth_token'],$token_credentials['oauth_token_secret']);

?>
```

9) And finally we can make requests authenticated as the user. You can GET, POST, and DELETE API
methods. Directly copy the path from the API documentation and add an array of any parameter
you wish to include for the API method such as curser or in_reply_to_status_id.

```php
<?php

    $me = $twitter->get('account/verify_credentials');
    $status = $twitter->post('statuses/update', array('status' => 'Text of status here', 'in_reply_to_status_id' => 123456));
    $status = $twitter->delete('statuses/destroy/12345');

?>
```

Examples
-----

The examples are a good place to start. The minimal you'll need to
have is:

```php
<?php
    require_once 'twitter-php/twitter.php';

    define('CONSUMER_KEY', 'YOUR_CONSUMER_KEY');
    define('CONSUMER_SECRET', 'YOUR_CONSUMER_SECRET');
    define('OAUTH_CALLBACK', 'YOUR_OAUTH_CALLBACK');

    $twitter = new Twitter(CONSUMER_KEY, CONSUMER_SECRET);
    $request_token = $twitter->getRequestToken(OAUTH_CALLBACK); //get Request Token

?>
```

Authenticate user (OAuth)
-----

```php
<?php
    require_once 'twitter-php/twitter.php';

    define('CONSUMER_KEY', 'YOUR_CONSUMER_KEY');
    define('CONSUMER_SECRET', 'YOUR_CONSUMER_SECRET');
    define('OAUTH_CALLBACK', 'YOUR_OAUTH_CALLBACK');

    if(isset($_GET['oauth_token'])){
	$twitter = new Twitter(CONSUMER_KEY, CONSUMER_SECRET, $_SESSION['request_token'], $_SESSION['request_token_secret']);
	$token_credentials = $twitter->getAccessToken($_REQUEST['oauth_verifier']);
	if($token_credentials){
		$twitter = new Twitter(CONSUMER_KEY, CONSUMER_SECRET, $token_credentials['oauth_token'], $token_credentials['oauth_token_secret']);
		// Save it in a session var 
		$_SESSION['access_token'] = $token_credentials; 
		// Let's get the user's info
		$params =array();
		$params['include_entities']='false';
		$me = $twitter->get('account/verify_credentials',$params);
		echo print_r($me);
	} else {
		exit("Error.");
	}
    }
?>
```