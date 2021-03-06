# ✨ Astrel Laravel

[![Latest Version on Packagist](https://img.shields.io/packagist/v/sustainable-hustle/astrel-laravel.svg)](https://packagist.org/packages/sustainable-hustle/astrel-laravel)
[![GitHub Tests Action Status](https://img.shields.io/github/workflow/status/sustainable-hustle/astrel-laravel/Tests?label=tests)](https://github.com/sustainable-hustle/astrel-laravel/actions?query=workflow%3ATests+branch%3Amain)
[![Total Downloads](https://img.shields.io/packagist/dt/sustainable-hustle/astrel-laravel.svg)](https://packagist.org/packages/sustainable-hustle/astrel-laravel)

Integrate [Astrel](https://astrel.io) to your Laravel applications.

[Astrel](https://astrel.io) is a remote config orchestration application that enables you to change anything in your apps without touching your code.

🍿 **Are you a visual learner?** [Here a quick video tutorial](https://youtu.be/p6s5ddh8hZI) to help you get started with Astrel and Laravel.

## Installation

Install this package via composer.

```bash
composer require sustainable-hustle/astrel-laravel
```

Add your Astrel API key in your `.env` file.

```bash
ASTREL_API_KEY="sxjTgBJAxkT2TNyKf9vabFI0L07AyItM5o3iiloS"
```

Finally, [listen for Astrel's webhooks](#clear-the-cache-when-receiving-a-webhook) to clear the cache.

```php
// routes/web.php
use SustainableHustle\Astrel\Facades\Astrel;
Astrel::webhookRoute('astrel/webhook');

// Make sure CSRF verification is disabled on that route.
class VerifyCsrfToken extends Middleware
{
    protected $except = [
        '/astrel/webhook',
    ];
}
```

Optionally, publish the `astrel` config file.

```bash
php artisan vendor:publish --tag="astrel-config"
```

## Basic usage

This package provides a facade you can use to retrieve one or many aspects. An aspect is a key/value pair that you configure and updates directly on your Astrel dashboard.

``` php
use SustainableHustle\Astrel\Facades\Astrel;

Astrel::all();                        // Returns all aspects.
Astrel::get('slug');                  // Returns the value of an aspect by giving its slug.
Astrel::get('slug', 'default value'); // Returns the default value if the given aspect has no value.
```

Alternatively, you may use the `astrel` helper method to access the Astrel manager or to access a value directly.

``` php
astrel()->all();                 // Equivalent to Astrel::all();
astrel('slug');                  // Equivalent to Astrel::get('slug');
astrel('slug', 'default value'); // Equivalent to Astrel::get('slug', 'default value');
```

Note that if you'd like to fallback to an environment variable you may also use the `astrel_env` helper function like so.

```php
astrel_env('my-slug');                           // Equivalent to astrel('my-slug', env('MY_SLUG'));
astrel_env('my-slug', 'MY_ENV_SLUG');            // Equivalent to astrel('my-slug', env('MY_ENV_SLUG'));
astrel_env('my-slug', 'MY_ENV_SLUG', 'default'); // Equivalent to astrel('my-slug', env('MY_ENV_SLUG', 'default'));
```

## Caching

This package automatically caches all retrieved aspects. This ensure your application does not make API calls every single time a value from Astrel is required.

You may use the `flush` and `refetch` methods from the `Astrel` facade to clear the cache as well as refetching its content immediately.

``` php
use SustainableHustle\Astrel\Facades\Astrel;

Astrel::flush()   // Flush the cache for all aspects.
Astrel::refetch() // Flush the cache and refetch all aspects immediately.
```

By default, the cache never expires meaning you will have to manually flush it when necessary. This is because Astrel will send you a webhook whenever something changes so you can flush the cache only when necessary (see section below).

However, if you'd like to customize the cache's lifetime, you may update the `cache_lifetime` variable in your `config/astrel.php` file.

## Clear the cache when receiving a webhook

As mentioned above, you can configure Astrel to send you a webhook whenever any value gets updated. That means you can use this webhook to clear and immediately refetch all of your aspects.

This package provides a helper method `webhookRoute` on the `Astrel` facade that does just that. Simply add this to your routes file and chain any route configuration you might like.

``` php
use SustainableHustle\Astrel\Facades\Astrel;

Astrel::webhookRoute();                 // Register a route that calls `Astrel::refetch()` when triggered.
Astrel::webhookRoute('astrel/webhook'); // Provide a custom path to that route.
Astrel::webhookRoute('astrel/webhook')  // This method returns a Route object so you can chain anything you want.
    ->name('webhooks.astrel')
    ->middleware('web');
```

Additionally, if you're using the default `web` middleware group, make sure to disable CSRF verification for that route in the `VerifyCsrfToken` middleware.

```diff
class VerifyCsrfToken extends Middleware
{
    protected $except = [
-       //
+      '/astrel/webhook',
    ];
}
```
