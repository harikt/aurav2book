# Dispatching

Aura web/framework projects can handle different variations of dispatching 
with the help of [Aura.Dispatcher](https://github.com/auraphp/Aura.Dispatcher).

* Microframework
* Modified Micro-Framework Style
* Full-Stack Style

So if you start the application as a small one and as it grows, 
it is easy to modify the application routes acting as a micro framework 
to a full stack style.

> N.b.: You can skip to your favourite usage.

## Microframework

The following is an example of a micro-framework style route, where the 
action logic is embedded in the route params. In the `modify()` 
config method, we retrieve the shared `web_request` and `web_response` 
services, along with the `web_router` service. We then add a route names 
`blog.read` and embed the action code as a closure.

```php
<?php
namespace Aura\Web_Project\_Config;

use Aura\Di\Config;
use Aura\Di\Container;

class Common extends Config
{
    // ...

    public function modify(Container $di)
    {
        $request = $di->get('web_request');
        $response = $di->get('web_response');

        $router = $di->get('web_router');
        $router
            ->add('blog.read', '/blog/read/{id}')
            ->addValues(array(
                'action' => function ($id) use ($request, $response) {
                    $content = "Reading blog post $id";
                    $response->content->set(htmlspecialchars(
                        $content, ENT_QUOTES|ENT_SUBSTITUTE, 'UTF-8'
                    ));
                }
            ));
    }

    // ...
}
```

## Modified Micro-Framework Style

We can modify the above example to put the action logic in the 
dispatcher instead of the route itself.

Extract the action closure to the dispatcher under the name `blog.read`. 
Then, in the route, use a `action` value that matches the name in 
the dispatcher.

```php
<?php
namespace Aura\Web_Project\_Config;

use Aura\Di\Config;
use Aura\Di\Container;

class Common extends Config
{
    // ...

    public function modify(Container $di)
    {
        $request = $di->get('web_request');
        $response = $di->get('web_response');

        $dispatcher = $di->get('web_dispatcher');
        $dispatcher->setObject(
            'blog.read',
            function ($id) use ($request, $response) {
                $content = "Reading blog post $id";
                $response->content->set(htmlspecialchars(
                    $content, ENT_QUOTES|ENT_SUBSTITUTE, 'UTF-8'
                ));
            }
        );

        $router = $di->get('web_router');
        $router
            ->add('blog.read', '/blog/read/{id}')
            ->addValues(array(
                'action' => 'blog.read',
            ));
    }

    // ...
}
```

## Full-Stack Style

You can migrate from a micro-framework style to a full-stack style (or start
with full-stack style in the first place).

First, define a action class and place it in the project `src/` directory.

```php
<?php
/**
 * {$PROJECT_PATH}/src/App/Actions/BlogRead.php
 */
namespace App\Actions;

use Aura\Web\Request;
use Aura\Web\Response;

class BlogRead
{
    public function __construct(Request $request, Response $response)
    {
        $this->request = $request;
        $this->response = $response;
    }

    public function __invoke($id)
    {
        $content = "Reading blog post $id";
        $this->response->content->set(htmlspecialchars(
            $content, ENT_QUOTES|ENT_SUBSTITUTE, 'UTF-8'
        ));
    }
}
```

Next, tell the project how to build the _BlogRead_ through the DI
_Container_. Edit the project `config/Common.php` file to configure the
_Container_ to pass the `web_request` and `web_response` service objects to 
the _BlogRead_ constructor.

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
        );
    }

    // ...
}
```

After that, put the _`App\Actions\BlogRead`_ object in the dispatcher
under the name `blog.read` as a lazy-loaded instantiation ...

```php
<?php
namespace Aura\Web_Project\_Config;

use Aura\Di\Config;
use Aura\Di\Container;

class Common extends Config
{
    // ...

    public function modify(Container $di)
    {
        // ...
        $dispatcher = $di->get('web_dispatcher');
        $dispatcher->setObject(
            'blog.read',
            $di->lazyNew('App\Actions\BlogRead')
        );
    }

    // ...
}
```

... and finally, point the router to the `blog.read` action object:

```php
<?php
namespace Aura\Web_Project\_Config;

use Aura\Di\Config;
use Aura\Di\Container;

class Common extends Config
{
    // ...

    public function modify(Container $di)
    {
        // ...
        $router = $di->get('web_router');
        $router
            ->add('blog.read', '/blog/read/{id}')
            ->addValues(array(
                'action' => 'blog.read',
            ));
    }

    // ...
}
```
