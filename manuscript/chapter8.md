# View Helpers

[Aura.Html](https://github.com/auraphp/Aura.Html) have been 
extracted from Aura.View (v1), that can be used in any 
template, view, or presentation system.

With the flexibility coming Aura.View (v2) need to integrate the Aura.Html
helpers to make use of the various html helpers.

Aura.Html provides HTML escapers and helpers, including form input 
helpers.

## Installing Aura.Html

Edit your `composer.json` file and add `"aura/html": "2.0.*@dev"` in
the require section.

```json
{    
    "require": {
        // ... other require libraries
        "aura/html": "2.0.*@dev"        
    }
}
```

Save the file, and run

```bash
composer update
```

## DI Configuration

The DI configuration for Aura.Html is already in 
[config/Common.php](https://github.com/auraphp/Aura.Html/blob/develop-2/config/Common.php)

Now we need to set the object `Aura\Html\HelperLocator` as the `helpers` 
argument 

```php
$di->params['Aura\View\View'] = array(
    'view_registry' => $di->lazyNew('Aura\View\TemplateRegistry'),
    'layout_registry' => $di->lazyNew('Aura\View\TemplateRegistry'),
    'helpers' => $di->lazyNew('Aura\View\HelperRegistry'),
);
```

as in [aura/view/config/Common.php](https://github.com/auraphp/Aura.View/blob/develop-2/config/Common.php)

Edit `{$PROJECT_PATH}/config/Common.php` file and add a line 
`$di->params['Aura\View\View']['helpers'] = $di->lazyGet('html_helper');`
in `define()` method.

```php
<?php
namespace Aura\Web_Project\_Config;
 
use Aura\Di\Config;
use Aura\Di\Container;

class Common extends Config
{
    public function define(Container $di)
    {
        // ...
        $di->params['Aura\View\View']['helpers'] = $di->lazyGet('html_helper');
    }
    // ...
}
```

Now you can use [tag helpers](https://github.com/auraphp/Aura.Html/blob/develop-2/README-HELPERS.md), 
[form helpers](https://github.com/auraphp/Aura.Html/blob/develop-2/README-FORMS.md) 
and [escaping](https://github.com/auraphp/Aura.Html#escaping) functionalities.

## Custom Helpers

There are two steps to adding your own custom helpers:

1. Write a helper class

2. Set a factory for that class into the _HelperLocator_ under a service name

A helper class needs only to implement the `__invoke()` method.  
We suggest extending from _AbstractHelper_ to get access to indenting, 
escaping, etc., but it's not required.

We are going to create a router helper which can return 
the router object, and from which we can generate
routes from the already defined routes.

```php
<?php
<?php
// {$PROJECT_PATH}/src/App/Html/Helper/Router.php
namespace App\Html\Helper;

use Aura\Html\Helper\AbstractHelper;
use Aura\Router\Router as AuraRouter;

class Router
{
    protected $router;

    public function __construct(AuraRouter $router)
    {
        $this->router = $router;
    }

    public function __invoke()
    {
        return $this->router;
    }
}
```

Now that we have a helper class, we set a factory for it into the 
_HelperLocator_ under a service name. 
Therein, we create **and return** the helper class.

Edit `{$PROJECT_PATH}/config/Common.php`

```php
<?php
namespace Aura\Web_Project\_Config;
 
use Aura\Di\Config;
use Aura\Di\Container;

class Common extends Config
{
    public function define(Container $di)
    {
        // ...
        $di->params['App\Html\Helper\Router']['router'] = $di->lazyGet('web_router');
        $di->params['Aura\Html\HelperLocator']['map']['router'] = $di->lazyNew('App\Html\Helper\Router');
    }
    // ...
}
```
    
The service name in the _HelperLocator_ doubles as a method name. 
This means we can call the helper via `$this->router()`:

```php
<?php echo $this->router()->generate('blog.read', array('id', 2)); ?>
```

Note that we can use any service name for the helper, although it is generally
useful to name the service for the helper class, and for a word that 
can be called as a method.
