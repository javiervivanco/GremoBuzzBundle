# GremoBuzzBundle
[![Build status](https://img.shields.io/travis/gremo/GremoBuzzBundle.svg?style=flat-square)](https://travis-ci.org/gremo/GremoBuzzBundle) [![GitHub issues](https://img.shields.io/github/issues/gremo/GremoBuzzBundle.svg?style=flat-square)](https://github.com/gremo/GremoBuzzBundle/issues)  [![Latest stable](https://img.shields.io/packagist/v/gremo/buzz-bundle.svg?style=flat-square)](https://packagist.org/packages/gremo/buzz-bundle) [![Downloads total](https://img.shields.io/packagist/dt/gremo/buzz-bundle.svg?style=flat-square)](https://packagist.org/packages/gremo/buzz-bundle)

Symfony 2 Bundle for using the lightweight Buzz HTTP client.

- [Installation](#installation)
- [Configuration](#configuration)
- [Usage](#usage)
- [Adding listeners](#adding-listeners)

## Installation

Add the following to your `deps` file (for Symfony 2.0.*):

```
[buzz]
    git=https://github.com/kriswallsmith/Buzz.git

[GremoBuzzBundle]
    git=https://github.com/gremo/GremoBuzzBundle.git
    target=bundles/Gremo/BuzzBundle
```

Then register the namespaces with the autoloader (`app/autoload.php`):

```php
$loader->registerNamespaces(array(
    // ...
    'Buzz'  => __DIR__.'/../vendor/buzz/lib',
    'Gremo' => __DIR__.'/../vendor/bundles',
    // ...
));
```

If you are using [Composer](http://getcomposer.org/) and Symfony >= 2.1.*, add the following to your `composer.json` file:

```javascript
{
    "require": {
        "gremo/buzz-bundle": "*"
    }
}
```

Finally register the bundle with your kernel in `app/appKernel.php`:

```php
public function registerBundles()
{
    $bundles = array(
        // ...
        new Gremo\BuzzBundle\GremoBuzzBundle(),
        // ...
    );

    // ...
}
```

## Configuration
Configuration is not needed. Reference, along with default values:
```yml
# GremoBuzzBundle Configuration
gremo_buzz:
    client: "native" # allowed "curl", "multi_curl" or "native"
    options:
        ignore_errors: true
        max_redirects: 5
        timeout: 5
        verify_peer: true
```

## Usage
Get `gremo_buzz` service from the service container and start using the browser:

```php
/** @var $browser \Buzz\Browser */
$browser = $this->get('gremo_buzz');
```

Refer to [Kris Wallsmith Buzz library](https://github.com/kriswallsmith/Buzz) for sending HTTP requests.

## Adding listeners
You can register a listener creating a service that implements `Buzz\Listener\ListenerInterface` and tagging it as `gremo_buzz.listener` (optionally defining a `priority` attribute).

Higher priority means that the corresponding listener is executed first. Same priority would lead to unexpected behaviours, as well as not numerical ones.

An example listener that logs outgoing requests:

```php
use Buzz\Listener\ListenerInterface;
use Buzz\Message\MessageInterface;
use Buzz\Message\RequestInterface;
use JMS\DiExtraBundle\Annotation as DI;
use Psr\Log\LoggerInterface;

/**
 * @DI\Service("buzz.listener.logger")
 * @DI\Tag("gremo_buzz.listener", attributes={"priority"=10})
 */
class BuzzLoggerListener implements ListenerInterface
{
    /**
     * @var \Psr\Log\LoggerInterface
     */
    private $logger;

    /**
     * @var float
     */
    private $startTime;

    /**
     * @DI\InjectParams({"logger" = @DI\Inject("logger")})
     */
    public function __construct(LoggerInterface $logger)
    {
        $this->logger = $logger;
    }

    public function preSend(RequestInterface $request)
    {
        $this->startTime = microtime(true);
    }

    public function postSend(RequestInterface $request, MessageInterface $response)
    {
        $seconds = microtime(true) - $this->startTime;

        $logger->info(sprintf(
            'Sent "%s %s%s" in %dms',
            $request->getMethod(),
            $request->getHost(),
            $request->getResource(),
            round($seconds * 1000)
        ));
    }
}
```

Note that this example uses the new `Psr\Log\LoggerInterface` and may not work for old versions of Symfony.
