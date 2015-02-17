# Action Domain Responder {#adr}

It is recommend to read [Action Domain Responder](https://pmjones.github.io/adr/) in short ADR.

Aura framework v2 promote the usage of one action per class.

In ADR there are 3 components.

1. Action is the logic that connects the Domain and Responder.
It uses the request input to interact with the Domain, and passes the
Domain output to the Responder.

1. Domain is the logic to manipulate the domain, session, application,
and environment data, modifying state and persistence as needed.

1. Responder is the logic to build an HTTP response or response description.
It deals with body content, templates and views, headers and cookies,
status codes, and so on.

Basically

1. The web handler receives a client request and dispatches it to an _Action_.

1. The _Action_ interacts with the _Domain_.

1. The _Action_ feeds data to the _Responder_. (N.b.: This may include
results from the _Domain_ interaction, data from the client request, and so on.)

1. The _Responder_ builds a response using the data fed to it by the _Action_.

1. The web handler sends the response back to the client.

## Responder Bundle {#responder-bundle}

We have [FOA.Responder_Bundle](https://github.com/friendsofaura/FOA.Responder_Bundle)
which helps to render different template engines like Aura.View, Twig, Mustache etc.
See [full list](https://github.com/friendsofaura/FOA.Responder_Bundle#integrated-templating-engines).

### Installation

```bash
composer require foa/responder-bundle
```

> Note : Current version 0.4

Let us modify our previous example of _BlogRead_ action class to
render the contents via _BlogRead_ responder.

Save at `{$PROJECT_PATH}/src/App/Responder/BlogRead.php`

```php
<?php
/**
 * {$PROJECT_PATH}/src/App/Responders/BlogRead.php
 */
namespace App\Responders;

use Aura\View\View;
use Aura\Web\Response;
use FOA\Responder_Bundle\AbstractResponder;

class BlogRead extends AbstractResponder
{
    protected $available = array(
        'text/html' => '',
        'application/json' => '.json',
    );

    protected function init()
    {
        $view_registry = $this->view->getViewRegistry();
        $view_registry->set('read', __DIR__ . '/views/read.php');

        $layout_registry = $this->view->getLayoutRegistry();
        $layout_registry->set('layout', __DIR__ . '/layouts/default.php');

        // the above four lines are only needed for Aura.View
        // for it could not locate the template by its own

        $this->payload_method['FOA\DomainPayload\Found'] = 'display';
    }

    protected function display()
    {
        $this->renderView('read', 'layout');
    }
}
```

Now modify the actions class `{$PROJECT_PATH}/src/App/Actions/BlogRead.php`
to inject the `BlogRead` responder. You also need to inject a
_Domain_ service which can fetch the details of the id.
We are skipping the service and assume you have some way to get the data.

Remove the _View_ and _Response_ objects from the action class because
the responder is responsible for rendering the view and set the response.

Now your modified action class will look like

```php
<?php
/**
 * {$PROJECT_PATH}/src/App/Actions/BlogRead.php
 */
namespace App\Actions;

use Aura\Web\Request;
use App\Responders\BlogRead as BlogReadResponder;
use FOA\DomainPayload\PayloadFactory;

class BlogRead
{
    protected $request;

    protected $responder;

    public function __construct(
        Request $request,
        BlogReadResponder $responder
    ) {
        // you may want to inject some service in-order to fetch the details
        $this->request = $request;
        $this->responder = $responder;
    }

    public function __invoke($id)
    {
        $blog = (object) array(
            'id' => $id, 'title' => 'Some awesome title', 'author' => 'Hari KT'
        );
        // In real life you want to do something like
        // $blog = $this->service->fetchId($id);

        $payload_factory = new PayloadFactory();
        $payload = $payload_factory->found($blog);
        $this->responder->setPayload($payload);
        return $this->responder;
    }
}
```

Modify our Closure as a view file and save in
`{$PROJECT_PATH}/src/App/Responders/views/read.php`.

```php
<?php echo "Reading '{$this->blog->title}' with post id: {$this->blog->id} !"; ?>
```

Time to edit your configuration file `{$PROJECT_PATH}/config/Common.php` .

Modify the class params for `App\Actions\BlogRead` to reflect
the changes made to the constructor.

```php
$di->params['App\Actions\BlogRead'] = array(
    'request' => $di->lazyGet('aura/web-kernel:request'),
    'responder' => $di->lazyNew('App\Responders\BlogRead'),
);

$di->params['FOA\Responder_Bundle\Renderer\AuraView']['engine'] = $di->lazyNew('Aura\View\View');
// responder
$di->params['FOA\Responder_Bundle\AbstractResponder']['response'] = $di->lazyGet('aura/web-kernel:response');
$di->params['FOA\Responder_Bundle\AbstractResponder']['renderer'] = $di->lazyNew('FOA\Responder_Bundle\Renderer\AuraView');
$di->params['FOA\Responder_Bundle\AbstractResponder']['accept'] = $di->lazyNew('Aura\Accept\Accept');
```

Browse the `http://localhost:8000/blog/read/1` .

### Questions {#adr-questions}

What have we achieved other than creating lots of classes ?

That is really a good question. We are moving the responsibility
to its own layers which will help us in testing the application.
Web applications get evolved even we start small, so testing each and
every part is always a great way to move forward.

This help us in to test the action classes, services etc.
