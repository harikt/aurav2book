# View

Aura web framework doesn't come packaged with any templating. The reason is love for 
templating differs from person to person. And integrating any templating
library in Aura is not hard as long as it can be installed and loaded 
via Composer.

In this chapter we are going to integrate [Aura.View][] an implementation 
of the [TemplateView](http://martinfowler.com/eaaCatalog/templateView.html) and 
[TwoStepView](http://martinfowler.com/eaaCatalog/twoStepView.html) 
patterns, with support for helpers and for closures as templates, 
using PHP itself as the templating language.

## Installing Aura.View

Edit your `composer.json` file and add `"aura/view": "2.0.*"` in
the require section.

```json
{
    "require": {
        // ... other require libraries
        "aura/view": "2.0.*"
    }
}
```

Save the file, and run

```bash
composer update
```

The DI configuration for every aura library is already in 
[config/Common.php](https://github.com/auraphp/Aura.View/blob/develop-2/config/Common.php)

So we don't need to do anything for now.

## Integration with actions

Let us integrate Aura.View to the full stack framework example shown in 
previous chapter.

Edit the `{$PROJECT_PATH}/src/App/Actions/BlogRead.php` to accept 
`Aura\View\View` object in constructor.

```php
<?php
/**
 * {$PROJECT_PATH}/src/App/Actions/BlogRead.php
 */
namespace App\Actions;

use Aura\Web\Request;
use Aura\Web\Response;
use Aura\View\View;

class BlogRead
{
    // ...

    public function __construct(
        Request $request, 
        Response $response, 
        View $view
    ) {
        $this->request = $request;
        $this->response = $response;
        $this->view = $view;
    }

    public function __invoke($id)
    {
        // ...
    }
}
```

Now, we need to modify the `config/Common.php` for the DI container to
pass the `Aura\View\View` object to `App\Actions\BlogRead`.

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

        $di->params['App\Actions\BlogRead'] = array(
            'request' => $di->lazyGet('web_request'),
            'response' => $di->lazyGet('web_response'),
            'view' => $di->lazyNew('Aura\View\View'),
        );
    }

    // ...
}
```

Now move the template to a file or use a Closure which can be 
rendered by [Aura.View][].
Let us once again edit the file `BlogRead.php` file `__invoke` method.

```php
<?php
/**
 * {$PROJECT_PATH}/src/App/Actions/BlogRead.php
 */
namespace App\Actions;

// ...

class BlogRead
{
    // ...

    public function __invoke($id)
    {
        $view_registry = $this->view->getViewRegistry();
        $view_registry->set('read', function () {
            echo "Reading blog post {$this->id}!";
        });
        // or set a php template
        // $view_registry->set('read', __DIR__ . '/views/read.php');

        $this->view->setData(array('id' => $id));
        $this->view->setView('read');
        $this->response->content->set($this->view->__invoke());
    }
}
```

This documentation intentionally has not shown the 
[ADR](https://github.com/pmjones/mvc-refinement) way. 
We will be showing shortly in the upcoming chapters how to use
templates, logics etc. not in the same action class.

Consider reading 
[Aura.View](https://github.com/auraphp/Aura.View/#escaping-output) 
documentation on how to use 
[partials](https://github.com/auraphp/Aura.View/#using-sub-templates-aka-partials),
[sections](https://github.com/auraphp/Aura.View/#using-sections) , 
[helpers](https://github.com/auraphp/Aura.View/#using-helpers) , 
[two step view](https://github.com/auraphp/Aura.View/#rendering-a-two-step-view)
etc.

In the next chapter we will show how to integrate 
[Aura.Html](https://github.com/auraphp/Aura.Html) helpers in Aura.View.

[Aura.View]: https://github.com/auraphp/Aura.View/
