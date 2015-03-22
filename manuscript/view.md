# View {#view}

Aura web framework doesn't come packaged with any templating. The reason is love for
templating differs from person to person.

With the help of [foa/responder-bundle](https://github.com/friendsofaura/FOA.Responder_Bundle),
we can integrate

* [Aura.View](https://github.com/auraphp/Aura.View)
* [Twig](http://twig.sensiolabs.org)
* [Mustache](https://github.com/bobthecow/mustache.php)
* [Plates](https://github.com/thephpleague/Plates)
* [Smarty](https://github.com/smarty-php/smarty)
* [and the one you like](https://github.com/friendsofaura/FOA.Responder_Bundle#integrating-other-templating-engines)

The advantage of using `foa/responder-bundle` is you have a common method `render`, which helps you to switch between template engine with less overhead.

It also helps integrating the [Action Domain Responder](http://pmjones.github.io/adr/) which we will cover on a different chapter.

## Installation

```bash
composer require foa/responder-bundle
```

Choose your templating engine and install the same. In this example we are going to make use of `foa/html-view-bundle` which integrates [aura/view](https://github.com/auraphp/Aura.View/) and [aura/html](https://github.com/auraphp/Aura.Html/).

```bash
composer require foa/html-view-bundle
```

## Configuration

Add the below lines in `{PROJECT_PATH}/config/Common.php` in the `define` method.

```php
<?php
$di->params['Aura\View\TemplateRegistry']['paths'] = array(dirname(__DIR__) '/templates');
$di->params['FOA\Responder_Bundle\Renderer\AuraView']['engine'] = $di->lazyNew('Aura\View\View');
$di->set('renderer', $di->lazyNew('FOA\Responder_Bundle\Renderer\AuraView'));
```

> Setting the paths to `Aura\View\TemplateRegistry` will only work for version >= 2.1

## Integration with actions {#integration-with-action}

Let us integrate the above renderer to the full stack framework example shown in previous chapter.

Edit the `{$PROJECT_PATH}/src/App/Actions/BlogRead.php` to inject an instance of `FOA\Responder_Bundle\Renderer\RendererInterface`.

```php
<?php
/**
 * {$PROJECT_PATH}/src/App/Actions/BlogRead.php
 */
namespace App\Actions;

use Aura\Web\Request;
use Aura\Web\Response;
use FOA\Responder_Bundle\Renderer\RendererInterface;

class BlogRead
{
    // ...

    public function __construct(
        Request $request,
        Response $response,
        RendererInterface $renderer
    ) {
        $this->request = $request;
        $this->response = $response;
        $this->renderer = $renderer;
    }

    public function __invoke($id)
    {
        // set the content rendered from the renderer
        $this->response->content->set($this->renderer->render(array('id' => $id), 'read'));
    }
}
```

Modify the `config/Common.php` for the DI container to
pass an instance that satisfies `FOA\Responder_Bundle\Renderer\RendererInterface` to `App\Actions\BlogRead`.

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
            'request' => $di->lazyGet('aura/web-kernel:request'),
            'response' => $di->lazyGet('aura/web-kernel:response'),
            'renderer' => $di->lazyGet('renderer'),
        );
    }

    // ...
}
```

Create the file `templates/read.php` with contents

```php
echo "Reading blog post {$this->id}!";
```

Please refer respective library documentation on usage inside template file.
