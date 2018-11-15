# oauth2-mdirector
[![Latest Stable Version](https://poser.pugx.org/antevenio/oauth2-mdirector/v/stable)](https://packagist.org/packages/antevenio/oauth2-mdirector)
[![Total Downloads](https://poser.pugx.org/antevenio/oauth2-mdirector/downloads)](https://packagist.org/packages/antevenio/oauth2-mdirector)
[![License](https://poser.pugx.org/antevenio/oauth2-mdirector/license)](https://packagist.org/packages/antevenio/oauth2-mdirector)
[![Travis build](https://api.travis-ci.org/Antevenio/oauth2-mdirector.svg?branch=master)](https://travis-ci.org/Antevenio/oauth2-mdirector)
[![Coverage Status](https://coveralls.io/repos/github/Antevenio/oauth2-mdirector/badge.svg?branch=master)](https://coveralls.io/github/Antevenio/oauth2-mdirector?branch=master)
[![Maintainability](https://api.codeclimate.com/v1/badges/f19e715eb520e7bd6a29/maintainability)](https://codeclimate.com/github/Antevenio/oauth2-mdirector/maintainability)

OAuth client library specific to access MDirector API services, written in PHP.

This package provides MDirector (http://www.mdirector.com) OAuth 2.0 support for the 
PHP League's [OAuth 2.0 Client](https://github.com/thephpleague/oauth2-client).

As of now, only an OAuth2 implementation for the MDirector email marketing application is provided. 
It is composed of an [oauth2-client](https://github.com/thephpleague/oauth2-client) 
provider and a wrapper around it to hide the burden of the required OAuth2 negotiations.

As a consumer you may choose to use just the provider or the client wrapper, as it suits you best.

There is also a command line script to help you test it from the shell.

## Requirements
The following versions of PHP are supported.

* PHP 5.6
* PHP 7.0
* PHP 7.1
* PHP 7.2

## Installation
```
composer require antevenio/oauth2-mdirector 
```

## Usage
As mentioned before, you can choose to use just the provider or the wrapper around it. 
Here you can find examples for each case: 

### 1. Oauth2-client provider
You can find the [oauth2-client](https://github.com/thephpleague/oauth2-client) provider under 
[OAuth2/Client/Provider](https://github.com/Antevenio/mdirector-oauth-client-php/tree/master/src/OAuth2/Client/Provider), 
for generic usage instructions please refer to generic usage in the
[oauth2-client github project](https://github.com/thephpleague/oauth2-client).

MDirector as of now is just providing the **Resource Owner Password Credentials Grant** 
having a generic clientId named **webapp**. Here is an example to get a valid accessToken:

```php
$provider = new \MDOAuth\OAuth2\Client\Provider\MDirector();

try {
    // Try to get an access token using the resource owner password credentials grant.
    $accessToken = $provider->getAccessToken('password', [
        'username' => '{yourCompanyId}',
        'password' => '{yourApiSecret}'
    ]);
} catch (\League\OAuth2\Client\Provider\Exception\IdentityProviderException $e) {
    // Failed to get the access token
    exit($e->getMessage());
}
```

When building your requests to the mdirector api, take into account that our api expects the
parameters to be on the query string for *GET* requests, or being 
application/x-www-form-urlencoded on the body of the request for any other method 
i.e. *POST*, *PUT*, *DELETE*... etc.

### 2. Wrapper client
Here you can find an example of use of our wrapper client.
Notice you just have to set parameters as an associative array.
```php
$companyId = 'yourCompanyId';
$secret = 'yourApiSecret';

$client = (new \MDOAuth\OAuth2\Client\MDirector\Factory())->create($companyId, $secret);
$response = $client->setUri('https://api.mdirector.com/api_contact')
    ->setMethod('get')
    ->setParameters([
        'email' => 'myemail@mydomain.org'    
    ])
    ->request();

echo $response->getBody()->getContents();
```

### 3. Shell script
The library also provides a [console](https://github.com/symfony/console) client so you can 
call the mdirector api from a shell.
To do so run:

```
$ ./bin/mdirector-oauth-client oauth2:mdirector --help    
```                                            
The command will display some self explanatory help about its usage and parameters.


### 4. Others

If you plan on using another client implementation, or have to call the API using a language other than PHP,
here you'll find the little bits of information you'll need to know:

You'll be asking for access tokens using a **password** grant type, 
being your *company id* your **username** and your *secret* your **password**. 

The endpoint for getting such tokens (or refresh them) would be:

https://app.mdirector.com/oauth2

And you'll be carrying your token on a Bearer header in your requests. 
(https://oauth.net/2/bearer-tokens/)
