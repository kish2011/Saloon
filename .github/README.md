![1752](https://user-images.githubusercontent.com/29132017/149842636-e9964b27-7ace-4af9-a6db-23c325505295.jpg)

<div align="center">

# 🚪 Saloon 🚪

*A Laravel & PHP package that allows you to write your API integrations in a beautiful, standardised syntax.*

![Build Status](https://github.com/sammyjo20/saloon/actions/workflows/tests.yml/badge.svg)

</div>

## Introduction

Saloon is a PHP package which introduces a class-based/OOP approach to building connections to APIs. Saloon introduces an easy to understand pattern to help you standardise the way you interact with
third-party APIs, reduce repeated code (DRY) and lets you mock API requests for your tests. It's perfect for writing your next SDK, or implementing directly into an existing project to work with your
APIs. It has a great package for Laravel for a tight integration with the fantastic ecosystem.

```php
<?php

use App\Http\Saloon\Requests\GetForgeServerRequest;

$request = new GetForgeServerRequest(serverId: '123456');

$response = $request->send();
$data = $response->json();
```

## Features

- Simple syntax, standardises the way you interact with APIs
- You don't have to worry about Guzzle/Http Facade/cURL
- Organise all your API integrations in one place
- Easily add on your own functionality with plugins
- Powerful interceptor logic to customise the response
- Supports Guzzle Handlers for unlimited customisation
- Mocking requests for testing
- Framework agnostic
- Can be used for building SDKs

## Getting Started

### Using Laravel?

There is a version of Saloon for Laravel, to install it use Composer.

```bash
composer require sammyjo20/saloon-laravel
```

Otherwise, to install Saloon, use Composer to install it into your PHP app.

```bash
composer require sammyjo20/saloon
```

> Saloon requires PHP 8

## Connectors

Once you have installed Saloon, the first thing you want to create is a "Connector". Connectors are classes where you define an APIs basic requirements. Within a connector, you can define the URL of
the API, default headers and even pass in your own functionality which is shared across all a connection's requests. You should have a separate connector for each API integration.
> If you are using Laravel, you can use the **php artisan saloon:connector** command.

Let's have a look at our ForgeConnector. As you can see, the bare minimum you must define is a base url.

```php
<?php

use Sammyjo20\Saloon\Http\SaloonConnector;

class ForgeConnector extends SaloonConnector
{
    /**
     * Define the base url for the connector.
     *
     * @return string
     */
    public function defineBaseUrl(): string
    {
        return 'https://forge.laravel.com/api/v1';
    }
}
```

You can also specify default headers and configuration options which will be applied to every request.

```php
<?php

use Sammyjo20\Saloon\Http\SaloonConnector;

class ForgeConnector extends SaloonConnector
{
    /**
     * Define the base url for the connector.
     *
     * @return string
     */
    public function defineBaseUrl(): string
    {
        return 'https://forge.laravel.com/api/v1';
    }
    
    /**
     * Define the base headers for the connector.
     *
     * @return string[]
     */
    public function defaultHeaders(): array
    {
        return [
            'Authorization' => 'Bearer ' . config('services.forge.key') // "config" is a built in Laravel function.
        ];
    }
    
    /**
     * Define the default Guzzle configuration for the connector.
     *
     * @return string[]
     */
    public function defaultConfig(): array
    {
        // You can specify any of the Guzzle configuration options here.
        // See https://docs.guzzlephp.org/en/stable/request-options.html for more.
    
        return [
            'timeout' => 5,
        ];
    }
}
```

## Requests

The second most important file in Saloon is your request. Requests are where you define each method of the API you want to call. The minimum requirements are `$method`. `$connector`
and `defineMethod()`.

> If you are using Laravel, you can use the **php artisan saloon:request** command.

Let's have a look at our GetForgeServerRequest.

```php
<?php

use App\Http\Saloon\Connectors\ForgeConnector;
use Sammyjo20\Saloon\Constants\Saloon;
use Sammyjo20\Saloon\Http\SaloonRequest;

class GetForgeServerRequest extends SaloonRequest
{
    /**
     * Define the method that the request will use.
     *
     * @var string|null
     */
    protected ?string $method = Saloon::GET;

    /**
     * The connector.
     *
     * @var string|null
     */
    protected ?string $connector = ForgeConnector::class;

    /**
     * Define the endpoint for the request.
     *
     * @return string
     */
    public function defineEndpoint(): string
    {
        return '/servers/' . $this->serverId;
    }
    
    /**
     * Constructor, you can pass in your own properties.
     *
     */
    public function __construct(
        public string $serverId
    ){}
}
```

Requests can also have their own default headers and configuration which are merged in with the connector's default headers and configuration. These will take priority over connector's default values.

```php
<?php

use App\Http\Saloon\Connectors\ForgeConnector;
use Sammyjo20\Saloon\Constants\Saloon;
use Sammyjo20\Saloon\Http\SaloonRequest;

class GetForgeServerRequest extends SaloonRequest
{
    /**
     * Define the method that the request will use.
     *
     * @var string|null
     */
    protected ?string $method = Saloon::GET;

    /**
     * The connector.
     *
     * @var string|null
     */
    protected ?string $connector = ForgeConnector::class;

    /**
     * Define the endpoint for the request.
     *
     * @return string
     */
    public function defineEndpoint(): string
    {
        return '/servers/' . $this->serverId;
    }
    
    /**
     * Define the base headers for the connector.
     *
     * @return string[]
     */
    public function defaultHeaders(): array
    {
        return [
            'X-Custom-Header' => 'Hello-World',
        ];
    }
    
    /**
     * Define the default Guzzle configuration for the connector.
     *
     * @return string[]
     */
    public function defaultConfig(): array
    {
        // You can specify any of the Guzzle configuration options here.
        // See https://docs.guzzlephp.org/en/stable/request-options.html for more.
    
        return [
            'query' => [
                'filter' => 'onlyActive',
            ],
        ];
    }
    
    public function __construct(
        public string $serverId
    ){}
}
```

## Making your request

Once you have created your Saloon Connector and Request, you are ready to make your request!

Here's a simple example of making a request.

```php
<?php

use App\Http\Saloon\Requests\GetForgeServerRequest;

$request = new GetForgeServerRequest(serverId: '123456');

$response = $request->send();
```

You can also set/overwrite any configuration at this stage too! Any headers/configuration added at this stage are merged in with the default values from the connector and request, but will take the
highest priority.

```php
<?php

use App\Http\Saloon\Requests\GetForgeServerRequest;

$request = new GetForgeServerRequest(serverId: '123456');

$request->addHeader('Accept', 'application/json');
$request->addConfig('debug', true);

$request->setHeaders($array); // This will overwrite all default headers.
$request->setConfig($array); // This will overwrite all default configration options.

$response = $request->send();
```

### Post Requests

[Click here to read more about POST/PUT/PATCH requests](#form-data)

## API responses

Once Saloon has sent the request, you will be given an instance of `SaloonResponse` to easily interact with the response from the server.

```php
$response = $request->send();
```

### Available methods

The Saloon response has a lot of handy methods for you. A lot of these are taken from `Illuminate/Http`.

```php
$response->getSaloonRequestOptions(): array
$response->toPsrResponse(): ResponseInterface
$response->body(): string
$response->json(): array
$response->object(): object
$response->collect(): Collection
$response->header(): string
$response->headers(): array
$response->getStatusFromResponse(): int
$response->status(): int
$response->effectiveUri(): UriInterface
$response->successful(): bool
$response->ok(): bool
$response->redirect(): bool
$response->failed(): bool
$response->clientError(): bool
$response->serverError(): bool
$response->onError(callable $callback)
$response->cookies(): CookieJar
$response->handlerStats(): array
$response->toException()
$response->throw()
```

## Saloon Plugins

Saloon also comes with a library of useful "plugins" in the form of traits. These plugins can be added to either the Connector or a Request, depending on if you want the plugin to be used on all
requests within a connection or just one request.

Plugins can add their own headers/Guzzle configuration. These are especially useful if you have headers that you frequently want to add to a specific connection or request. For example
the `Content-Type: application/json` header.

Let's take a look at the `AcceptsJson` feature plugin. We will add it to our ForgeConnector.

```php
<?php

use Sammyjo20\Saloon\Http\SaloonConnector;
use Sammyjo20\Saloon\Traits\Features\AcceptsJson;

class ForgeConnector extends SaloonConnector
{
    use AcceptsJson;

    /**
     * Define the base url for the connector.
     *
     * @return string
     */
    public function defineBaseUrl(): string
    {
        return 'https://forge.laravel.com/api/v1';
    }
}
```

Now let's take a look inside the `AcceptsJson` feature plugin. As you can see, we are using the `mergeHeaders($headers)` so it adds its own headers.

```php
<?php

namespace Sammyjo20\Saloon\Traits\Features;

trait AcceptsJson
{
    public function bootAcceptsJsonFeature()
    {
        $this->mergeHeaders([
            'Accept' => 'application/json'
        ]);
        
        // You can also do:
        // mergeQuery
        // addHandler
        // addResponseInterceptor
        // addHeader
        // addConfig
        // addData
    }
}
```

This plugin will add the header `Accept: application/json` to all requests made by the connector. If you add the `AcceptsJson` to just one request, the header will only be applied to that request.
Plugin headers take a lower priority than the `defaultHeaders` defined in the Request/Connector and can be overwritten at runtime with `$request->setHeaders()`.

## Form Data

Most API integrations you will write will often require sending data using a POST/PUT/PATCH request. Saloon makes this easy for you with the `HasJsonBody`, `HasFormParams` and `HasMultipartBody`
plugin traits. Let’s look at how you use the `HasJsonBody` plugin.

Firstly, add the trait to the class. After that, use the `defaultData()` method to define data that will be sent in the request. You don't have to define a `defaultData()` method for it to work,
however it's recommended that you do so you can update your request payload in the Saloon request. See below for setting the form data at runtime.

```php
<?php

namespace App\Http\Saloon\Requests;

use App\Http\Saloon\Connectors\ForgeConnector;
use Sammyjo20\Saloon\Constants\Saloon;
use Sammyjo20\Saloon\Http\SaloonRequest;
use Sammyjo20\Saloon\Traits\Features\HasJsonBody;

class CreateForgeSiteRequest extends SaloonRequest
{
    use HasJsonBody;

    /**
     * Define the method that the request will use.
     *
     * @var string|null
     */
    protected ?string $method = Saloon::POST;

    /**
     * The connector.
     *
     * @var string|null
     */
    protected ?string $connector = ForgeConnector::class;

    /**
     * Define the endpoint
     *
     * @return string
     * @throws \Sammyjo20\Saloon\Exceptions\SaloonMissingAttributeException
     */
    public function defineEndpoint(): string
    {
        return '/servers/' . $this->serverId . '/sites';
    }
    
    /**
     * Define data on the request
     * 
     * @return array
     */
    public function defaultData(): array
    {
        return [
            'domain' => $this->domain,
            'type' => 'php',
        ];
    }
    
    /**
     * Constructor, you can pass in your own properties.
     *
     */
    public function __construct(
        public string $serverId,
        public string $domain,
    ){}
}
```

Similar to headers and config, post data can have a hierarchy, and can also be added to, over overwritten at runtime.

```php
<?php

$request = new CreateForgeSiteRequest($serverId, $domain);

// Add an individual key

$request->addData('key', 'value');

// Merge in your own data to the existing data array

$request->mergeData([
    'database' => 'test123'
]);

// Overwrite the request data entirely.

$request->setData([
    'domain' => $customDomain,
]);
```

> Tip: Instead of using `setData` you could add a constructor to your request with arguments for the data.

## Form Body

Sometimes you will need to send custom form data like XML or a stream. In this case, you should use the `HasBody` plugin trait. To get started,
add the `HasBody` trait to your request class. After that, add a `defineBody` method on your request class. This method should return the raw body that you intend to send to the server. You should
also specify headers so the receiving server can understand what Content-Type you are sending.

[If you are sending XML, consider using the HasXMLBody trait.](#xml-data)

```php
<?php

namespace App\Http\Saloon\Requests;

use App\Http\Saloon\Connectors\ForgeConnector;
use Sammyjo20\Saloon\Constants\Saloon;
use Sammyjo20\Saloon\Http\SaloonRequest;
use Sammyjo20\Saloon\Traits\Features\HasBody;

class XmlServiceRequest extends SaloonRequest
{
    use HasBody;

    // ...
    
    public function defaultHeaders(): array
    {
        return [
            'Content-Type' => 'application/xml',
        ];
    }
    
    public function defineBody(): mixed
    {
        return '<?xml version="1.0" encoding="UTF-8"?>';
    }   
}
```
> Due to the unpredictability of the body content, there are no methods to modify the body at runtime. Consider using a constructor in your request class if you need to pass in custom data.

## Query Parameters

Saloon offers an easy way to add query parameters to your connectors and requests too. Let’s take a look at a request with the `HasQueryParams` trait.

Firstly, add the trait to the class. After that, use the `defaultQuery()` method to define query parameters that will be sent in the request. You don't have to define a `defaultQuery()` method for it
to work, however it's recommended that you do so you can update your request payload in the Saloon request. See below for setting the query parameters at runtime.

```php
<?php

namespace App\Http\Saloon\Requests;

use App\Http\Saloon\Connectors\ForgeConnector;
use Sammyjo20\Saloon\Constants\Saloon;
use Sammyjo20\Saloon\Http\SaloonRequest;
use Sammyjo20\Saloon\Traits\Features\HasQueryParams;

class GetForgeServersRequest extends SaloonRequest
{
    use HasQueryParams;

    // ...
    
    /**
     * Define query parameters on the request
     * 
     * @return array
     */
    public function defaultQuery(): array
    {
        return [
            'sort' => 'updated_at',
        ];
    }
}
```

Similar to headers, config and form data - query parameters have a hierarchy and can also be added to or overwritten at run time

```php
<?php

$request = new GetForgeSerersRequest();

// Add an individual key

$request->addQuery('key', 'value');

// Merge in your own query parameters to the existing array

$request->mergeQuery([
    'include' => 'user'
]);

// Overwrite the query parameters entirely.

$request->setQuery([
    'sort' => $sort,
]);
```

## XML Data
If you are sending XML data to an XML service, you can use the `HasXMLBody` trait on your request which will automatically add the headers that you need to send XML. Once you add the trait, 
make sure to add the `defineXMLBody` method where you can return your XML as a string.

```php
<?php

namespace App\Http\Saloon\Requests;

use App\Http\Saloon\Connectors\ForgeConnector;
use Sammyjo20\Saloon\Constants\Saloon;
use Sammyjo20\Saloon\Http\SaloonRequest;
use Sammyjo20\Saloon\Traits\Features\HasXMLBody;

class XMLRequest extends SaloonRequest
{
    use HasXMLBody;

    // ...
    
    public function defineXmlBody(): string
    {
        return '<?xml version="1.0" encoding="UTF-8"?>';
    }
}
```

### All Plugins Available

- AcceptsJson
- AlwaysThrowsOnErrors
- HasJsonBody
- HasBody
- HasXMLBody
- HasFormParams
- HasMultipartBody
- HasQueryParams
- HasTimeout
- WithDebugData
- DisablesSSLVerification (Please be careful with this)

### Write your own plugins

Feel free to write your own plugins, just make sure to include a "boot" method inside of the plugin. The format is: `boot{YOUR CLASS NAME}Feature`. For example if I had a class called WithAWSHeader
your boot method would be called `bootWithAWSHeaderFeature`.

## Advanced

### Interceptors

Saloon already allows you to add functionality to your requests in the form of plugins, but if you would like to intercept the response before it is passed back to you, you can add a response
interceptor. These can be added in the `boot` method of your plugin, or they can be added to the generic `boot` method on the Connector/Request.

Let's have a look at an interceptor that will automatically throw if the response returns an unsuccessful error.

```php
class CreateForgeServerRequest extends SaloonRequest
{
    //...

    public function boot(): void
    {
        $this->addResponseInterceptor(function (SaloonRequest $request, SaloonResponse $response) {
            $response->throw();
    
            return $response;
        });
    }
}
```

> You can have as many response interceptors as you like.

### Guzzle Handlers

If you need to modify the underlying Guzzle request/response right before its sent, you can use handlers. These are an incredibly useful feature in Guzzle to add functionality to request/responses. To
add a handler, simply use the `addHandler` method in your plugin or `boot` method on your connector/request.

[Learn more about Guzzle handlers](https://docs.guzzlephp.org/en/stable/handlers-and-middleware.html)

```php
class CreateForgeServerRequest extends SaloonRequest
{
    //...

    public function boot(): void
    {
        $this->addHandler('myCustomHandler', function (callable $handler) {
            return function (RequestInterface $request, array $options) use ($handler) {
                $request->withHeader('X-Custom-Header', 'Hello');
                
                return $handler($request, $options);             
            };
        });
    }
}
```

> Saloon will not know about the changes you make to your request/responses in handlers.

## Mocking / Faking

It's important when building API integrations or SDKs to test your application's response to particular API requests without actually making a request to the API. Saloon has a fluent, and easy to understand workflow to build testing. There are some small differences between testing with Laravel and testing your SDK, which is why I have separated them below.

### Mocking Saloon Requests With Laravel
If you are using the `sammyjo20/saloon-laravel` Laravel package, the method of testing is slightly different. This is because we make use of Laravel's service container to store fake responses for later.

When testing in Laravel, use the Saloon facade and call `Saloon::fake()` before testing your application logic that makes a Saloon request. Saloon will automatically detect when an API request is about to be made, and will respond with the fake response, stopping the real API request being sent out. This powerful feature allows you to test **any** feature in your application that uses Saloon requests.

#### Basic Usage (Sequence Tests)
Basic sequence testing allows you to define any number of fake responses. When your application uses Saloon, it will pull out the next response in the sequence, removing it too.

```php
use Sammyjo20\SaloonLaravel\Facades\Saloon;

Saloon::fake([
    new MockResponse(['name' => 'Sam'], 200),
    new MockResponse(['name' => 'Alex'], 200),
    new MockResponse(['error' => 'Server Unavailable'], 500),
]);

(new GetForgeServerRequest)->send() // Will return with `['name' => 'Sam']` and status `200`
(new GetForgeServerRequest)->send() // Will return with `['name' => 'Alex']` and status `200`
(new GetForgeServerRequest)->send() // Will return with `['error' => 'Server Unavailable']` and status `500`
```

#### Connector Mocking
You may also explicitly define mock responses for a particular connector that is used. Unlike sequence tests, these are kept even after the response has been sent.

```php
use Sammyjo20\SaloonLaravel\Facades\Saloon;

Saloon::fake([
    ForgeConnector::class => new MockResponse(['name' => 'Sam'], 200),
    OtherServiceConnector::class => new MockResponse(['name' => 'Alex'], 200),
]);

(new GetForgeServerRequest)->send() // Will return with `['name' => 'Sam']` and status `200`
(new GetForgeServerRequest)->send() // Will return with `['name' => 'Sam']` and status `200`
(new OtherServiceRequest)->send() // Will return with `['name' => 'Alex']` and status `200`
```

#### Request Mocking
You may also explicitly define mock responses for a particular request that is used. Unlike sequence tests, these are kept even after the response has been sent.

```php
use Sammyjo20\SaloonLaravel\Facades\Saloon;

Saloon::fake([
    GetForgeServerRequest::class => new MockResponse(['name' => 'Sam'], 200),
    OtherServiceRequest::class => new MockResponse(['name' => 'Alex'], 200),
]);

(new GetForgeServerRequest)->send() // Will return with `['name' => 'Sam']` and status `200`
(new OtherServiceRequest)->send() // Will return with `['name' => 'Alex']` and status `200`
```

#### URL Mocking
You can also define fake responses for particular URL patterns. Whenever a request is made for a particular pattern, Saloon will respond to that request.

```php
use Sammyjo20\SaloonLaravel\Facades\Saloon;

Saloon::fake([
    'forge.laravel.com/api/*' => new MockResponse(['name' => 'Sam'], 200),
    'samcarre.dev/*' => new MockResponse(['name' => 'Alex'], 200),
    'samcarre.dev/exact' => new MockResponse(['name' => 'Taylor'], 200), // Exact requests
    '*' => new MockResponse(['name' => 'Wildcard'], 200), // Any other requests
]);

(new GetForgeServerRequest)->send() // Will return with `['name' => 'Sam']` and status `200`
(new OtherServiceRequest)->send() // Will return with `['name' => 'Alex']` and status `200`
(new ExactRequest)->send() // Will return with `['name' => 'Taylor']` and status `200`
(new WildcardServiceRequest)->send() // Will return with `['name' => 'Wildcard']` and status `200`
```

### Testing Without Laravel (SDKs, non-laravel applications)

Saloon provides a way to test your SDKs really easily. You will need to create an instance of `MockClient` and then pass the `MockClient` as an argument to the `send` method on your request. There are multiple ways you can write tests.

#### Basic Usage (Sequence Tests)
Basic sequence testing allows you to define any number of fake responses. When your application uses Saloon, it will pull out the next response in the sequence, removing it too.

```php
use Sammyjo20\Saloon\Clients\MockClient;

$mockClient = new MockClient([
    new MockResponse(['name' => 'Sam'], 200),
    new MockResponse(['name' => 'Alex'], 200),
    new MockResponse(['error' => 'Server Unavailable'], 500),
]);

(new GetForgeServerRequest)->send($mockClient) // Will return with `['name' => 'Sam']` and status `200`
(new GetForgeServerRequest)->send($mockClient) // Will return with `['name' => 'Alex']` and status `200`
(new GetForgeServerRequest)->send($mockClient) // Will return with `['error' => 'Server Unavailable']` and status `500`
```

#### Connector Mocking
You may also explicitly define mock responses for a particular connector that is used. Unlike sequence tests, these are kept even after the response has been sent.

```php
use Sammyjo20\Saloon\Clients\MockClient;

$mockClient = new MockClient([
    ForgeConnector::class => new MockResponse(['name' => 'Sam'], 200),
    OtherServiceConnector::class => new MockResponse(['name' => 'Alex'], 200),
]);

(new GetForgeServerRequest)->send($mockClient) // Will return with `['name' => 'Sam']` and status `200`
(new GetForgeServerRequest)->send($mockClient) // Will return with `['name' => 'Sam']` and status `200`
(new OtherServiceRequest)->send($mockClient) // Will return with `['name' => 'Alex']` and status `200`
```

#### Request Mocking
You may also explicitly define mock responses for a particular request that is used. Unlike sequence tests, these are kept even after the response has been sent.

```php
use Sammyjo20\Saloon\Clients\MockClient;

$mockClient = new MockClient([
    GetForgeServerRequest::class => new MockResponse(['name' => 'Sam'], 200),
    OtherServiceRequest::class => new MockResponse(['name' => 'Alex'], 200),
]);

(new GetForgeServerRequest)->send($mockClient) // Will return with `['name' => 'Sam']` and status `200`
(new OtherServiceRequest)->send($mockClient) // Will return with `['name' => 'Alex']` and status `200`
```

#### URL Mocking
You can also define fake responses for particular URL patterns. Whenever a request is made for a particular pattern, Saloon will respond to that request.

```php
use Sammyjo20\Saloon\Clients\MockClient;

$mockClient = new MockClient([
    'forge.laravel.com/api/*' => new MockResponse(['name' => 'Sam'], 200),
    'samcarre.dev/*' => new MockResponse(['name' => 'Alex'], 200),
    'samcarre.dev/exact' => new MockResponse(['name' => 'Taylor'], 200), // Exact requests
    '*' => new MockResponse(['name' => 'Wildcard'], 200), // Any other requests
]);

(new GetForgeServerRequest)->send($mockClient) // Will return with `['name' => 'Sam']` and status `200`
(new OtherServiceRequest)->send($mockClient) // Will return with `['name' => 'Alex']` and status `200`
(new ExactRequest)->send($mockClient) // Will return with `['name' => 'Taylor']` and status `200`
(new WildcardServiceRequest)->send($mockClient) // Will return with `['name' => 'Wildcard']` and status `200`
```

## And that's it! ✨

I really hope this package has been useful to you, if you like my work and want to show some love, consider buying me some coding fuel (Coffee) ❤

[Donate Java (the drink not the language)](https://ko-fi.com/sammyjo20)

## Banner Image Credit

- Freepik.com
