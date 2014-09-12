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
            'request' => $di->lazyGet('aura/web-kernel:request'),
            'response' => $di->lazyGet('aura/web-kernel:response'),
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

## Escaping Output

Security-minded observers will note that all the examples in this document use manually-escaped output. Because this package is not specific to any particular media type, it **does not** come with escaping functionality.

When you generate output via templates, you **must** escape it appropriately for security purposes. This means that HTML templates should use HTML escaping, CSS templates should use CSS escaping, XML templates should use XML escaping, PDF templates should use PDF escaping, RTF templates should use RTF escaping, and so on.

For a good set of HTML escapers, we will use [Aura.Html](#view-helpers).

## Registering View Templates

Now that we have a _View_, we need to add named templates to its view template registry. These are typically PHP file paths, but [templates can also be closures](#closures-as-templates).  For example:

```php
<?php
$view_registry = $view->getViewRegistry();
$view_registry->set('browse', '/path/to/views/browse.php');
?>
```

The `browse.php` file may look something like this:

```php
<?php
foreach ($this->items as $item) {
    $id = htmlspecialchars($item['id'], ENT_QUOTES, 'UTF-8');
    $name = htmlspecialchars($item['name'], ENT_QUOTES, 'UTF-8')
    echo "Item ID #{$id} is '{$name}'." . PHP_EOL;
?>
```

Note that we use `echo`, and not `return`, in templates.

I> The template logic will be executed inside the _View_ object scope,
I> which means that `$this` in the template code will refer to the _View_
I> object. The same is true for closure-based templates.

## Setting Data

We will almost always want to use dynamic data in our templates. To assign a data collection to the _View_, use the `setData()` method and either an array or an object. We can then use data elements as if they are properties on the
_View_ object.

```php
<?php
$view->setData(array(
    'items' => array(
        array(
            'id' => '1',
            'name' => 'Foo',
        ),
        array(
            'id' => '2',
            'name' => 'Bar',
        ),
        array(
            'id' => '3',
            'name' => 'Baz',
        ),
    )
));
?>
```

I> Recall that `$this` in the template logic refers to the _View_ object,
I> so that data assigned to the _View_ can be accessed as properties on `$this`.

The `setData()` method will overwrite all existing data in the _View_ object. The `addData()` method, on the other hand, will merge with existing data in the _View_ object.

## Invoking A One-Step View

Now that we have registered a template and assigned some data to the _View_, we tell the _View_ which template to use, and then invoke the _View_:

```php
<?php
$view->setView('browse');
$output = $view->__invoke(); // or just $view()
?>
```

The `$output` in this case will be something like this:

```
Item #1 is 'Foo'.
Item #2 is 'Bar'.
Item #3 is 'Baz'.
```


## Using Sub-Templates (aka "Partials")

Sometimes we will want to split a template up into multiple pieces. We can
render these "partial" template pieces using the `render()` method in our main template code.

First, we place the sub-template in the view registry (or in the layout registry if it for use in layouts). Then we `render()` it from inside the main template code. Sub-templates can use any naming scheme we like. Some systems use the convention of prefixing partial templates with an underscore, and the following example will use that convention.

Second, we can pass an array of variables to be extracted into the local scope of the partial template. (The `$this` variable will always be available regardless.)

For example, let's split up our `browse.php` template file so that it uses a sub-template for displaying items.

```php
<?php
// add templates to the view registry
$view_registry = $view->getViewRegistry();

// the "main" template
$view_registry->set('browse', '/path/to/views/browse.php');

// the "sub" template
$view_registry->set('_item', '/path/to/views/_item.php');
?>
```

We extract the item-display code from `browse.php` into `_item.php`:

```php
<?php
$id = htmlspecialchars($item['id'], ENT_QUOTES, 'UTF-8');
$name = htmlspecialchars($item['name'], ENT_QUOTES, 'UTF-8')
echo "Item ID #{$id} is '{$name}'." . PHP_EOL;
?>
```

Then we modify `browse.php` to use the sub-template:

```php
<?php
foreach ($this->items as $item) {
    echo $this->render('_item', array(
        'item' => $item,
    ));
?>
```

The output will be the same as earlier when we invoke the view.

T> Alternatively, we can use `include` or `require` to execute a 
T> PHP file directly in the current template scope.


## Using Sections

Sections are similar to sub-templates (aka "partials") except that they are captured inline for later use. In general, they are used by view templates to capture output for layout templates.

For example, we can capture output in the view template to a named section ...

```php
<?php
// begin buffering output for a named section
$this->beginSection('local-nav');

echo "<div>";
// ... echo the local navigation output ...
echo "</div>";

// end buffering and capture the output
$this->endSection();
?>
```

... and then use that output in a layout template:

```php
<?php
if ($this->hasSection('local-nav')) {
    echo $this->getSection('local-nav');
} else {
    echo "<div>No local navigation.</div>";
}
?>
```

In addition, the `setSection()` method can be used to set the section body directly, instead of capturing it:

```php
<?php
$this->setSection('local-nav', $this->render('_local-nav.php'));
?>
```

## Using Helpers

We can register closures or other invokable objects as helpers through
the _HelperRegistry_. We can then call these helpers as if they are methods on the _View_.

```php
<?php
$helpers = $view->getHelpers();
$helpers->set('hello', function ($name) {
    return "Hello {$name}!";
});

$view_registry = $view->getViewRegistry();
$view_registry->set('index', function () {
    echo $this->hello('World');
});

$view->setView('index');
$output = $view();
?>
```

This library does not come with any view helpers. You will need to add your own
helpers to the registry as closures or invokable objects.

## Custom Helper Managers

The _View_ is not type-hinted to any particular class for its helper manager. This means you may inject an arbitrary object of your own at _View_ construction time to manage helpers. To do so, pass a helper manager of your own to the _ViewFactory_.

```php
<?php
class OtherHelperManager
{
    public function __call($helper_name, $args)
    {
        // logic to call $helper_name with
        // $args and return the result
    }
}

$helpers = new OtherHelperManager;
$view = $view_factory->newInstance($helpers);
?>
```

In the next chapter we will cover the comprehensive set of HTML helpers,
including form and input helpers.

## Rendering a Two-Step View

To wrap the main content in a layout as part of a two-step view, we register
layout templates with the _View_ and then call `setLayout()` to pick one of
them for the second step. (If no layout is set, the second step will not be
executed.)

Let's say we have already set the `browse` template above into our view registry. We then set a layout template called `default` into the layout registry:

```php
<?php
$layout_registry = $view->getLayoutRegistry();
$layout_registry->set('default', '/path/to/layouts/default.php');
?>
```

The `default.php` layout template might look like this:

```html+php
<html>
<head>
    <title>My Site</title>
</head>
<body>
<?= $this->getContent(); ?>
</body>
</html>
```

We can then set the view and layout templates on the _View_ object and then invoke it:

```php
<?php
$view->setView('browse');
$view->setLayout('default');
$output = $view->__invoke(); // or just $view()
?>
```

The output from the inner view template is automatically retained and becomes available via the `getContent()` method on the _View_ object. The layout template then calls `getContent()` to place the inner view results in the outer layout template.

I> We can also call `setLayout()` from inside the view template, allowing 
I> us to pick a layout as part of the view logic.

The view template and the layout template both execute inside the same _View_ object. This means:

- All data values are shared between the view and the layout. Any data assigned to the view, or modified by the view, is used as-is by the layout.

- All helpers are shared between the view and the layout. This sharing situation allows the view to modify data and helpers before the layout is executed.

- All section bodies are shared between the view and the layout. A section that is captured from the view template can therefore be used by the layout template.

## Closures As Templates {#closures-as-templates}

The view and layout registries accept closures as templates. For example, these are closure-based equivlents of the `browse.php` and `_item.php` template files above:

```php
<?php
$view_registry->set('browse', function () {
    foreach ($this->items as $item) {
        echo $this->render('_item', array(
            'item' => $item,
        ));
    }
);

$view_registry->set('_item', function (array $vars) {
    extract($vars, EXTR_SKIP);
    $id = htmlspecialchars($item['id'], ENT_QUOTES, 'UTF-8');
    $name = htmlspecialchars($item['name'], ENT_QUOTES, 'UTF-8')
    echo "Item ID #{$id} is '{$name}'." . PHP_EOL;
);
?>
```

When registering a closure-based template, continue to use `echo` instead of `return` when generating output. The closure is rebound to the _View_ object, so `$this` in the closure will refer to the _View_ just as it does in a file-based template.

A bit of extra effort is required with closure-based sub-templates (aka "partials"). Whereas file-based templates automatically extract the passed array of variables into the local scope, a closure-based template must:

1. Define a function parameter to receive the injected variables (the `$vars` param in the `_item` template); and,

2. Extract the injected variables using `extract()`. Alternatively, the closure may use the injected variables parameter directly.

Aside from that, closure-based templates work exactly like file-based templates.
