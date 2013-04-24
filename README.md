# Transloadit PHP SDK

# Important

If you are on v0.10.0 or below, just pull-updating to v1.0.0 will break the SDK for you.
v1.0.0 makes PHP 5.3.0 a requirement. For development use `composer install --dev` to get phpunit version and run `vendor/bin/phpunit test` to run tests.

# Purpose

This is the official PHP SDK for the
[transloadit.com](http://transloadit.com/) API.

You can use it to setup file upload forms for your users, or to upload
existing files from your server.

## Examples

To get started, each of our examples requires that you create a new
Transloadit instance like this

```php
<?php
use transloadit\Transloadit;
$Transloadit = new Transloadit(array(
  'key' => 'your-key',
  'secret' => 'your-secret',
));

// Example code goes here!
```

**Note:** All of the examples below can be found and run from within the
`example/` folder.

<!-- This section is generated by: make docs -->

### 1. Upload and resize an image from your server

This example demonstrates how you can use the sdk to create an assembly
on your server.

It takes a sample image file, uploads it to transloadit, and starts a
resizing job on it.

```php
<?php

$response = $transloadit->createAssembly(array(
  'files' => array(dirname(__FILE__).'/fixture/straw-apple.jpg'),
  'params' => array(
    'steps' => array(
      'resize' => array(
        'robot' => '/image/resize',
        'width' => 200,
        'height' => 100,
      )
    )
  ),
));

// Show the results of the assembly we spawned
echo '<pre>';
print_r($response);
echo '</pre>';

```

### 2. Create a simple end-user upload form

This example shows you how to create a simple transloadit upload form
that redirects back to your site after the upload is done.

Once the script receives the redirect request, the current status for
this assembly is shown using Transloadit::response().

Note: There is no guarantee that the assembly has already finished
executing by the time the `$response` is fetched. You should use
the `notify_url` parameter for this.

```php
<?php

// Check if this request is a transloadit redirect_url notification.
// If so fetch the response and output the current assembly status:
$response = Transloadit::response();
if ($response) {
  echo '<h1>Assembly status:</h1>';
  echo '<pre>';
  print_r($response);
  echo '</pre>';
  exit;
}

// This should work on most environments, but you might have to modify
// this for your particular setup.
$redirectUrl = sprintf(
  'http://%s%s',
  $_SERVER['HTTP_HOST'],
  $_SERVER['REQUEST_URI']
);

// Setup a simple file upload form that resizes an image to 200x100px
echo $transloadit->createAssemblyForm(array(
  'params' => array(
    'steps' => array(
      'resize' => array(
        'robot' => '/image/resize',
        'width' => 200,
        'height' => 100,
      )
    ),
    'redirect_url' => $redirectUrl
  )
));
?>
<h1>Pick an image to resize</h1>
<input name="example_upload" type="file">
<input type="submit" value="Upload">
</form>

```

### 3. Integrate the jQuery plugin into the previous example

Integrating the jQuery plugin simply means adding a few lines of JavaScript
to the previous example. Check the HTML comments below to see what changed.

```php
<?php

$response = Transloadit::response();
if ($response) {
  echo '<h1>Assembly status:</h1>';
  echo '<pre>';
  print_r($response);
  echo '</pre>';
  exit;
}

$redirectUrl = sprintf(
  'http://%s%s',
  $_SERVER['HTTP_HOST'],
  $_SERVER['REQUEST_URI']
);

echo $transloadit->createAssemblyForm(array(
  'params' => array(
    'steps' => array(
      'resize' => array(
        'robot' => '/image/resize',
        'width' => 200,
        'height' => 100,
      )
    ),
    'redirect_url' => $redirectUrl
  )
));
?>
<!--
Including the jQuery plugin is as simple as adding jQuery and including the
JS snippet for the plugin. See http://transloadit.com/docs/jquery-plugin
-->
<script type="text/javascript" src="http://ajax.googleapis.com/ajax/libs/jquery/1.4.4/jquery.min.js"></script>
<script type="text/javascript">
var tlProtocol = (('https:' == document.location.protocol) ? 'https://' : 'http://');
document.write(unescape("%3Cscript src='" + tlProtocol + "assets.transloadit.com/js/jquery.transloadit2.js' type='text/javascript'%3E%3C/script%3E"));
</script>
<script type="text/javascript">
 $(document).ready(function() {
   // Tell the transloadit plugin to bind itself to our form
   $('form').transloadit();
 });
</script>
<!-- Nothing changed below here -->
<h1>Pick an image to resize</h1>
<input name="example_upload" type="file">
<input type="submit" value="Upload">
</form>

```
<!-- End of generated doc section -->

## API

### $Transloadit = new Transloadit($properties = array());

Creates a new Transloadit instance and applies the given $properties.

#### $Transloadit->key = null;

The auth key of your Transloadit account.

#### $Transloadit->secret = null;

The auth secret of your Transloadit account.

#### $Transloadit->request($options = array(), $execute = true);

Creates a new `TransloaditRequest` using the `$Transloadit->key` and
`$Transloadit->secret` properties.

If `$execute` is set to `true`, `$TransloaditRequest->execute()` will be
called and used as the return value.

Otherwise the new `TransloaditRequest` instance is being returned.

#### $Transloadit->createAssemblyForm($options = array());

Creates a new Transloadit assembly form including the hidden 'params' and
'signature' fields. A closing form tag is not included.

`$options` is an array of `TransloaditRequest` properties to be used.
For example: `"params"`, `"expires"`, `"protocol"`, etc..

In addition to that, you can also pass an `"attributes"` key, which allows
you to set custom form attributes. For example:

```php
$Transloadit->createAssemblyForm(array(
  'attributes' => array(
    'id' => 'my_great_upload_form',
    'class' => 'transloadit_form',
  ),
));
```

#### $Transloadit->createAssembly($options);

Sends a new assembly request to Transloadit. This is the preferred way of
uploading files from your server.

`$options` is an array of `TransloaditRequest` properties to be used.

Check example #1 above for more information.

#### Transloadit::response()

This static method is used to parse the notifications Transloadit sends to
your server.

There are two kinds of notifications this method handles:

* When using the `"redirect_url"` parameter, and Transloadit redirects
  back to your site, a `$_GET['assembly_url']` query parameter gets added.
  This method detects the presence of this parameter and fetches the current
  assembly status from that url and returns it as a `TransloaditResponse`.
* When using the `"notify_url"` parameter, Transloadit sends a
  `$_POST['transloadit']` parameter. This method detects this, and parses
  the notification JSON into a `TransloaditResponse` object for you.

If the current request does not seem to be invoked by Transloadit, this
method returns `false`.


### $TransloaditRequest = new TransloaditRequest($properties = array());

Creates a new TransloaditRequest instance and applies the given $properties.

#### $TransloaditRequest->key = null;

The auth key of your Transloadit account.

#### $TransloaditRequest->secret = null;

The auth secret of your Transloadit account.

#### $TransloaditRequest->protocol = 'http';

The protocol to use when making this request. Valid values are `'http'` and
`'https'`.

#### $TransloaditRequest->method = 'GET';

Inherited from `CurlRequest`. Can be used to set the type of request to be
made.

#### $TransloaditRequest->host = 'api2.transloadit.com';

The host to send this request to.

#### $TransloaditRequest->path = null;

The url path to request.

#### $TransloaditRequest->url = null;

Inherited from `CurlRequest`. Lets you overwrite the above host / path
properties with a fully custom url alltogether.

#### $TransloaditRequest->fields = array();

A list of additional fields to send along with your request. Transloadit
will include those in all assembly related notifications.

#### $TransloaditRequest->files = array();

An array of paths to local files you would like to upload. For example:

```php
$TransloaditRequest->files = array('/my/file.jpg');
```

or

```php
$TransloaditRequest->files = array('my_upload' => '/my/file.jpg');
```

The first example would automatically give your file a field name of
`'file_1'` when executing the request.

#### $TransloaditRequest->params = array();

An array representing the JSON params to be send to Transloadit. You
do not have to include an `'auth'` key here, as this class handles that
for you as part of `$TransloaditRequest->prepare()`.

#### $TransloaditRequest->expires = '+2 hours';

If you have configured a '`$TransloaditRequest->secret`', this class will
automatically sign your request. The expires property lets you configure
the duration for which the signature is valid.

#### $TransloaditRequest->headers = array();

Lets you send additional headers along with your request. You should not
have to change this property.

#### $TransloaditRequest->execute()

Sends this request to Transloadit and returns a `TransloaditResponse`
instance.

### $TransloaditResponse = new TransloaditResponse($properties = array());

Creates a new TransloaditResponse instance and applies the given $properties.

#### $TransloaditResponse->data = null;

Inherited from `CurlResponse`. Contains an array of the parsed JSON
response from Transloadit.

You should generally only access this property after having checked for
errors using `$TransloaditResponse->error()`.

#### $TransloaditResponse->error();

Returns `false` or a string containing an explanation of what went wrong.

All of the following will cause an error string to be returned:

* Network issues of any kind
* The Transloadit response JSON contains an `{"error": "..."}` key
* A malformed response was received

## Contributing

Feel free to fork this project. We will happily merge bug fixes or other small
improvements. For bigger changes you should probably get in touch with us
before you start to avoid not seeing them merged.

## Versioning

This project implements the Semantic Versioning guidelines.

Releases will be numbered with the following format:

`<major>.<minor>.<patch>`

And constructed with the following guidelines:

* Breaking backward compatibility bumps the major (and resets the minor and patch)
* New additions without breaking backward compatibility bumps the minor (and resets the patch)
* Bug fixes and misc changes bumps the patch

For more information on SemVer, please visit http://semver.org/.

## Versions

### [master](https://github.com/transloadit/php-sdk/tree/master)

 - Fix broken examples
 - Improve documentation (version changelogs)

diff: https://github.com/transloadit/php-sdk/compare/v1.0.0...master

### [v1.0.0](https://github.com/transloadit/php-sdk/tree/v1.0.0)

A big thanks to [@nervetattoo](https://github.com/nervetattoo) for making this version happen!

 - Add support for Composer
 - Make phpunit run through Composer
 - Change to namespaced PHP

diff: https://github.com/transloadit/php-sdk/compare/v0.10.0...v1.0.0

### [v0.10.0](https://github.com/transloadit/php-sdk/tree/v0.10.0)

 - Add support for Strict mode
 - Add support for more auth params
 - Improve documentation
 - Bug fixes
 - Refactoring

diff: https://github.com/transloadit/php-sdk/compare/v0.9.1...v0.10.0

### [v0.9.1](https://github.com/transloadit/php-sdk/tree/v0.9.1)

 - Improve documentation
 - Better handling of errors & non-json responses
 - Change directory layout

diff: https://github.com/transloadit/php-sdk/compare/v0.9...v0.9.1

### [v0.9](https://github.com/transloadit/php-sdk/tree/v0.9)

 - Use markdown for docs
 - Add support for signed GET requests
 - Add support for HTTPS
 - Simplified API
 - Improve handling of magic quotes
 
diff: https://github.com/transloadit/php-sdk/compare/v0.2...v0.9

### [v0.2](https://github.com/transloadit/php-sdk/tree/v0.2)

- Add error handling

diff: https://github.com/transloadit/php-sdk/compare/v0.1...v0.2

### [v0.1](https://github.com/transloadit/php-sdk/tree/v0.1)

The very first version





