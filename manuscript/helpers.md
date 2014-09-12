# View Helpers

The [Aura.Html](https://github.com/auraphp/Aura.Html) package has been
extracted from Aura.View (v1), and can now be used in any
template, view, or presentation system.

With the added flexibility in Aura.View (v2), one can use the Aura.Html
package to make use of the various HTML helpers.

Aura.Html provides HTML escapers and helpers, including form input
helpers.

## Installing Aura.Html

Edit your `composer.json` file and add `"aura/html": "2.0.*"` in
the require section.

```json
{
    "require": {
        // ... other require libraries
        "aura/html": "2.0.*"
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

Edit the `{$PROJECT_PATH}/config/Common.php` file and add a line
`$di->params['Aura\View\View']['helpers'] = $di->lazyGet('html_helper');`
in `define()` method.

W> Note : The service name definitions are getting changed from `html_helper`
to `aura/<library>:<service>`. If you are using `2.0.0` then the service name
is `html_helper`. If you are using `dev` version it is `aura/html:helper`.
And also not to spend more time on the config for different projects you
have [https://github.com/friendsofaura/FOA.Html_View_Bundle](https://github.com/friendsofaura/FOA.Html_View_Bundle)

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

## Aura.Html Tag Helpers

Use a helper by calling it as a method on the _HelperLocator_. The available helpers are:

- [a/anchor](#a)
- [base](#base)
- [img/image](#img)
- [label](#label)
- [links](#links)
- [metas](#metas)
- [ol](#ol)
- [scripts / scriptsfoot](#scripts)
- [ul](#ul)
- [styles](#styles)
- [tag](#tag)
- [title](#title)

There is also a series of [helpers for forms](#aurahtml-form-helpers).

### a {#a}

Helper for `<a>` tags.

```html+php
<?php
echo $this->a(
    'http://auraphp.com',       // (string) href
    'Aura Project',             // (string) text
    array('id' => 'aura-link')  // (array) optional attributes
);
?>
<a href="http://auraphp.com" id="aura-link">Aura Project</a>
```

### base {#base}

Helper for `<base>` tags.

```html+php
<?php
echo $this->base(
    '/base' // (string) href
);
?>
<base href="/base" />
```

### img {#img}

Helper for `<img>` tags.

```html+php
<?php
echo $this->img(
    '/images/hello.jpg',            // (string) image href src
    array('id' => 'image-id');      // (array) optional attributes
?>
<!-- if alt is not specified, uses the basename of the image href -->
<img src="/images/hello.jpg" alt="hello" id="image-id">

```

### label {#label}

Helper for `<label>` tags.

```html+php
<?php
echo $this->label(
    'Label For Field',          // (string) label text
    array('for' => 'field'));   // (array) optional attributes
?>
<label for="field">Label For Field</label>

<?php
// wrap html with the label before the html
echo $this->label('Foo: ')
            ->before($this->input(array(
                'type' => 'text',
                'name' => 'foo',
            )));
?>
<label>Foo: <input type="text" name="foo" value="" /></label>

<?php
// wrap html with the label after the html
echo $this->label(' (Foo)')
            ->after($this->input(array(
                'type' => 'text',
                'name' => 'foo',
            )));
?>
<label><input type="text" name="foo" value="" /> (Foo)</label>
```

### links {#links}

Helper for a set of generic `<link>` tags. Build a set of links with `add()` then output them all at once.

```html+php
<?php
// build the array of links with add()
$this->links()->add(array(
    'rel' => 'prev',                // (array) link attributes
    'href' => '/path/to/prev',
));

$this->links()->add(array(        // (array) link attributes
    'rel' => 'next',
    'href' => '/path/to/next',
));

// output the links
echo $this->links();
?>
<link rel="prev" href="/path/to/prev" />
<link ref="next" href="/path/to/next" />

<?php
// alternatively, echo a chain of add() calls
echo $this->links()
    ->add(array(                    // (array) link attributes
        'rel' => 'prev',
        'href' => '/path/to/prev',
    ))
    ->add(array(                    // (array) link attributes
        'rel' => 'next',
        'href' => '/path/to/next',
    ));
?>
```
<link rel="prev" href="/path/to/prev" />
<link ref="next" href="/path/to/next" />

### metas {#metas}

Helper for a set of `<meta>` tags. Build a set of metas with `add*()` then output them all at once.

```html+php
<?php
// add an http-equivalent meta
$this->metas()->addHttp(
    'Location',         // (string) header label
    '/redirect/to/here' // (string) header value
);

// add a name meta
$this->metas()->addName(
    'foo',              // the meta name
    'bar'               // the meta content
);

// output the metas
echo $this->meta();
?>
<meta http-equiv="Location" content="/redirect/to/here">
<meta name="foo" content="bar">

<?php
// alternatively, echo a chain of add calls
echo $this->metas()
    ->addHttp(
        'Location',         // (string) header label
        '/redirect/to/here' // (string) header value
    )
    ->addName(
        'foo',              // the meta name
        'bar'               // the meta content
    );
?>
<meta http-equiv="Location" content="/redirect/to/here">
<meta name="foo" content="bar">
```

### ol {#ol}

Helper for `<ol>` tags with `<li>` items.  Build the set of items (both raw and escaped) then output them all at once.

```html+php
<?php
// start the list of items
$this->ol(array(                  // (array) optional attributes
    'id' => 'test',
));

// add a single item to be escaped
$this->ol()->item(
    'foo',                          // (string) the item text
    array('id' => 'foo')            // (array) optional attributes
);

// add several items to be escaped
$this->ol()->items(array(         // (array) the items to add
    'bar',                          // the item text, no item attributes
    'baz' => array('id' => 'baz'),  // item text with item attributes
));

// add a single raw item not to be escaped
$this->ol()->rawItem(
    '<a href="/first">First</a>',   // (string) the raw item html
    array('id' => 'first')          // (array) optional attributes
);

// add several raw items not to be escaped
$this->ol()->rawItems(array(         // (array) the raw items to add
    '<a href="/prev">Prev</a>',     // the item text, no item attributes
    '<a href="/next">Next</a>',
    '<a href="/last">Last</a>' => array('id' => 'last') // text and attributes
));

// output the list
echo $this->ol();
?>
<ol id="test">
    <li id="foo">foo</li>
    <li>bar</li>
    <li id="baz">baz</li>
    <li><a href="/first">First</a></li>
    <li><a href="/pref">First</a></li>
    <li><a href="/next">First</a></li>
    <li><a href="/last">First</a></li>
</ol>
```

### scripts {#scripts}

Helper for a set of `<script>` tags. Build a set of script links, then output them all at once.

```html+php
<?php
// add a single script
$this->scripts()->add('/js/middle.js');

// add another script after that one
$this->add('/js/last.js');

// add another at a specific priority order
echo $this->scripts(
    '/js/first.js',     // (string) the script src
    50                  // (int) optional priority order (default 100)
);

// add a conditional script
$this->scripts->addCond(
    'ie6',              // (string) the condition
    '/js/ie6.js',       // (string) the script src
    25                  // (int) optional priority order (default 100)
));

?>
<!--[if ie6]><script src="/js/ie6.js" type="text/javascript"></script><![endif]-->
<script src="/js/first.js" type="text/javascript"></script>
<script src="/js/middle.js" type="text/javascript"></script>
<script src="/js/last.js" type="text/javascript"></script>
```

The `scriptsFoot()` helper works the same way, but is intended for placing a separate set of scripts at the end of the HTML body.

### ul {#ul}

Helper for `<ul>` tags with `<li>` items.  Build the set of items (both raw and escaped) then output them all at once.

```html+php
<?php
// start the list of items
$this->ul(array(                  // (array) optional attributes
    'id' => 'test',
));

// add a single item to be escaped
$this->ul()->item(
    'foo',                          // (string) the item text
    array('id' => 'foo')            // (array) optional attributes
);

// add several items to be escaped
$this->ul()->items(array(         // (array) the items to add
    'bar',                          // the item text, no item attributes
    'baz' => array('id' => 'baz'),  // item text with item attributes
));

// add a single raw item not to be escaped
$this->ul()->rawItem(
    '<a href="/first">First</a>',   // (string) the raw item html
    array('id' => 'first')          // (array) optional attributes
);

// add several raw items not to be escaped
$this->ul()->rawItems(array(      // (array) the raw items to add
    '<a href="/prev">Prev</a>',     // the item text, no item attributes
    '<a href="/next">Next</a>',
    '<a href="/last">Last</a>' => array('id' => 'last') // text and attributes
));

// output the list
echo $this->ul();
?>
<ul id="test">
    <li id="foo">foo</li>
    <li>bar</li>
    <li id="baz">baz</li>
    <li><a href="/first">First</a></li>
    <li><a href="/prev">Prev</a></li>
    <li><a href="/next">Next</a></li>
    <li><a href="/last">Last</a></li>
</ul>
```

### styles {#styles}

Helper for a set of `<link>` tags for stylesheets. Build a set of style links, then output them all at once. As with the `script` helper, you can optionally set the priority order for each stylesheet.

```html+php
<?php
// add a stylesheet link
$this->styles()->add(
    '/css/middle.css',          // (string) the stylesheet href
    array('media' => 'print')   // (array) optional attributes
);

// add another one after that
$this->styles()->add('/css/last.css');

// add one at a specific priority order
$this->styles()->add(
    '/css/first.css',           // (string) the stylesheet href
    null,                       // (array) optional attributes
    50                          // (int) optional priority order (default 100)
);

// add a conditional stylesheet
$this->styles()->addCond(
    'ie6',                      // (string) the condition
    '/css/ie6.css',             // (string) the stylesheet href
    array('media' => 'print'),  // (array) optional attributes
    25                          // (int) optional priority order (default 100)
);

// output the stylesheet links
echo $this->styles();
?>
<!--[if ie6]><link rel="stylesheet" href="/css/ie6.css" type="text/css" media="print" /><![endif]-->
<link rel="stylesheet" href="/css/first.css" type="text/css" media="screen" />
<link rel="stylesheet" href="/css/middle.css" type="text/css" media="print" />
<link rel="stylesheet" href="/css/last.css" type="text/css" media="screen" />
?>
```

### tag {#tag}

A generic tag helper.

```html+php
<?php
echo $this->tag(
    'div',                  // (string) the tag name
    array('id' => 'foo')    // (array) optional array of attributes
);
echo $this->tag('/div');
?>
<div id="foo"></div>
```

### title {#title}

Helper for the `<title>` tag.

```html+php
<?php
// escaped variations (can be intermixed with raw variations)

// set the title
$this->title()->set('This & That');

// append the title
$this->title()->append(' > Suf1');
$this->title()->append(' > Suf2');

// prepend the title
$this->title()->prepend('Pre1 > ');
$this->title()->prepend('Pre2 > ');

echo $this->title();
?>
<title>Pre2 &gt; Pre1 &gt; This &amp; That &gt; Suf1 &gt; Suf2</title>

<?php
// raw variations (can be intermixed with escaped variations):

// set the title
$this->title()->set('This & That');

// append the title
$this->title()->append(' > Suf1');
$this->title()->append(' > Suf2');

// prepend the title
$this->title()->prepend('Pre1 > ');
$this->title()->prepend('Pre2 > ');

echo $this->title();
?>
<title>Pre2 > Pre1 > This & That > Suf1 > Suf2</title>
```

## Aura.Html Form Helpers {#aurahtml-form-helpers}

## The Form Element

Open and close a form element like so:

```html+php
<?php
echo $this->form(array(
    'id' => 'my-form',
    'method' => 'put',
    'action' => '/hello-action',
));

echo $this->tag('/form');
?>
<form id="my-form" method="put" action="/hello-action" enctype="multipart/form-data"></form>
```

## HTML 5 Input Elements

All of the HTML 5 input helpers use the same method signature: a single descriptor array that formats the input element.

```html+php
<?php
echo $this->input(array(
    'type'    => $type,     // (string) the element type
    'name'    => $name,     // (string) the element name
    'value'   => $value,    // (string) the current value of the element
    'attribs' => array(),   // (array) element attributes
    'options' => array(),   // (array) options for select and radios
));
?>
```

The array is used so that other libraries can generate form element descriptions without needing to depend on Aura.Html for a particular object.

The available input element `type` values are:

- [button](#button)
- [checkbox](#checkbox)
- [color](#color)
- [date](#date)
- [datetime](#datetime)
- [datetime-local](#datetime-local)
- [email](#email)
- [file](#file)
- [hidden](#hidden)
- [image](#image)
- [month](#month)
- [number](#number)
- [password](#password)
- [radio](#radio)
- [range](#range)
- [reset](#reset)
- [search](#search)
- [select](#select) (including options)
- [submit](#submit)
- [tel](#tel)
- [text](#text)
- [textarea](#textarea)
- [time](#time)
- [url](#url)
- [week](#week)

### button {#button}

```html+php
<?php
echo $this->input(array(
    'type'    => 'button',
    'name'    => 'foo',
    'value'   => 'bar',
    'attribs' => array()
));
?>
<input type="button" name="foo" value="bar" />
```

### checkbox {#checkbox}

The `checkbox` type honors the `value_unchecked` pseudo-attribute as a way to specify a `hidden` element for the (you guessed it) unchecked value. It also honors the pseudo-element `label` to place a label after the checkbox.

```html+php
<?php
echo $this->input(array(
    'type'    => 'checkbox',
    'name'    => 'foo',
    'value'   => 'y',               // the current value
    'attribs' => array(
        'label' => 'Check me',      // the checkbox label
        'value' => 'y',             // the value when checked
        'value_unchecked' => '0',   // the value when unchecked
    ),
));
?>
<input type="hidden" name="foo" value="n" />
<label><input type="checkbox" name="foo" value="y" checked /> Check me</label>
```

### color {#color}

```html+php
<?php
echo $this->input(array(
    'type'    => 'color',
    'name'    => 'foo',
    'value'   => 'bar',
    'attribs' => array()
));
?>
<input type="color" name="foo" value="bar" />
```

### date {#date}

```html+php
<?php
echo $this->input(array(
    'type'    => 'date',
    'name'    => 'foo',
    'value'   => 'bar',
    'attribs' => array()
));
?>
<input type="date" name="foo" value="bar" />
```

### datetime {#datetime}

```html+php
<?php
echo $this->input(array(
    'type'    => 'datetime',
    'name'    => 'foo',
    'value'   => 'bar',
    'attribs' => array()
));
?>
<input type="datetime" name="foo" value="bar" />
```

### datetime-local {#datetime-local}

```html+php
<?php
echo $this->input(array(
    'type'    => 'datetime-local',
    'name'    => 'foo',
    'value'   => 'bar',
    'attribs' => array()
));
?>
<input type="datetime-local" name="foo" value="bar" />
```

### email {#email}

```html+php
<?php
echo $this->input(array(
    'type'    => 'email',
    'name'    => 'foo',
    'value'   => 'bar',
    'attribs' => array()
));
?>
<input type="email" name="foo" value="bar" />
```

### file {#file}

```html+php
<?php
echo $this->input(array(
    'type'    => 'file',
    'name'    => 'foo',
    'value'   => 'bar',
    'attribs' => array()
));
?>
<input type="file" name="foo" value="bar" />
```

### hidden {#hidden}

```html+php
<?php
echo $this->input(array(
    'type'    => 'hidden',
    'name'    => 'foo',
    'value'   => 'bar',
    'attribs' => array()
));
?>
<input type="hidden" name="foo" value="bar" />
```

### image {#image}

```html+php
<?php
echo $this->input(array(
    'type'    => 'image',
    'name'    => 'foo',
    'value'   => 'bar',
    'attribs' => array()
));
?>
<input type="image" name="foo" value="bar" />
```

### month {#month}

```html+php
<?php
echo $this->input(array(
    'type'    => 'month',
    'name'    => 'foo',
    'value'   => 'bar',
    'attribs' => array()
));
?>
<input type="month" name="foo" value="bar" />
```

### number {#number}

```html+php
<?php
echo $this->input(array(
    'type'    => 'number',
    'name'    => 'foo',
    'value'   => 'bar',
    'attribs' => array()
));
?>
<input type="number" name="foo" value="bar" />
```

### password {#password}

```html+php
<?php
echo $this->input(array(
    'type'    => 'password',
    'name'    => 'foo',
    'value'   => 'bar',
    'attribs' => array()
));
?>
<input type="password" name="foo" value="bar" />
```

### radio {#radio}

This element type allows you to generate a single radio input, or multiple radio inputs if you pass an `options` element.

```html+php
<?php
echo $this->input(array(
    'type'    => 'radio',
    'name'    => 'foo',
    'value'   => 'bar',     // (string) the currently selected radio
    'attribs' => array(),
    'options' => array(     // (array) `value => label` pairs
        'bar' => 'baz',
        'dib' => 'zim',
        'gir' => 'irk',
    ),
));
?>
<label><input type="radio" name="foo" value="bar" checked /> baz</label>
<label><input type="radio" name="foo" value="dib" /> zim</label>
<label><input type="radio" name="foo" value="gir" /> irk</label>
```

### range {#range}

```html+php
<?php
echo $this->input(array(
    'type'    => 'range',
    'name'    => 'foo',
    'value'   => 'bar',
    'attribs' => array()
));
?>
<input type="range" name="foo" value="bar" />
```

### reset {#reset}

```html+php
<?php
echo $this->input(array(
    'type'    => 'reset',
    'name'    => 'foo',
    'value'   => 'bar',
    'attribs' => array()
));
?>
<input type="reset" name="foo" value="bar" />
```

### search {#search}

```html+php
<?php
echo $this->input(array(
    'type'    => 'search',
    'name'    => 'foo',
    'value'   => 'bar',
    'attribs' => array()
));
?>
<input type="search" name="foo" value="bar" />
```

### select {#select}

Helper for a `<select>` tag with `<option>` tags. The pseudo-attribute `placeholder` is honored as a placeholder label when no option is selected. Using the attribute `'multiple' => true` will set up a multiple select, and automatically add `[]` to the name if it is not already there.

```html+php
<?php
echo $this->input(array(
    'type'    => 'select',
    'name'    => 'foo',
    'value'   => 'bar',
    'attribs' => array(
        'placeholder' => 'Please pick one',
    ),
    'options' => array(
        'baz' => 'Baz Label',
        'dib' => 'Dib Label',
        'bar' => 'Bar Label',
        'zim' => 'Zim Label',
    ),
));
?>
<select name="foo">
    <option disabled value="">Please pick one</option>
    <option value="baz">Baz Label</option>
    <option value="dib">Dib Label</option>
    <option value="bar" selected>Bar Label</option>
    <option value="zim">Zim Label</option>
</select>

<?php
// programatically build option-by-option
$select = $this->input(array(
    'type'    => 'select',
    'name'    => 'foo',
));

// set the currently selected value(s)
$select->selected('bar');   // (string|array) the currently selected value(s)

// set attributes for the select tag
$select->attribs(array(
    'placeholder' => 'Please pick one',
));

// add a single option
$select->option(
    'baz',                  // (string) the option value
    'Baz Label',            // (string) the option label
    array()                 // (array) optional attributes for the option tag
);

// add several options
$select->options(array(
    'dib' => 'Dib Label',
    'bar' => 'Bar Label',
    'zim' => 'Zim Label',
));

// output the select
echo $select;
?>
<select name="foo">
    <option disabled value="">Please pick one</option>
    <option value="baz">Baz Label</option>
    <option value="dib">Dib Label</option>
    <option value="bar" selected>Bar Label</option>
    <option value="zim">Zim Label</option>
</select>
```

The helper also supports option groups. If an `options` array value is itself an array, the key for that element will be used as an `<optgroup>` label and the array of values will be options under that group.

```html+php
<?php
echo $this->input(array(
    'type'    => 'select',
    'name'    => 'foo',
    'value'   => 'bar',
    'attribs' => array(),
    'options' => array(
        'Group A' => array(
            'baz' => 'Baz Label',
            'dib' => 'Dib Label',
        ),
        'Group B' => array(
            'bar' => 'Bar Label',
            'zim' => 'Zim Label',
        ),
    ),
));
?>
<select name="foo">
    <optgroup label="Group A">
        <option value="baz">Baz Label</option>
        <option value="dib">Dib Label</option>
    </optgroup>
    <optgroup label="Group B">
        <option value="bar" selected>Bar Label</option>
        <option value="zim">Zim Label</option>
    </optgroup>
</select>

<?php
// or do so programmatically
$select = $this->input(array(
    'type'    => 'select',
    'name'    => 'foo',
));

// set the currently selected value(s)
$select->selected('bar');   // (string|array) the currently selected value(s)

// start an option group
$select->optgroup('Group A');

// add several options
$select->options(array(
    'baz' => 'Baz Label',
    'dib' => 'Dib Label',
));

// start another option group (sub-groups are not allowed by HTML spec)
$select->optgroup('Group B');

// add a single option
$select->option(
    'bar',                  // (string) the option value
    'Bar Label',            // (string) the option label
    array()                 // (array) optional attributes for the option tag
);

// add a single option
$select->option(
    'zim',                  // (string) the option value
    'Zim Label',            // (string) the option label
    array()                 // (array) optional attributes for the option tag
);

// output the select
echo $select;
?>
<select name="foo">
    <optgroup label="Group A">
        <option value="baz">Baz Label</option>
        <option value="dib">Dib Label</option>
    </optgroup>
    <optgroup label="Group B">
        <option value="bar" selected>Bar Label</option>
        <option value="zim">Zim Label</option>
    </optgroup>
</select>
```

### submit {#submit}

```html+php
<?php
echo $this->input(array(
    'type'    => 'submit',
    'name'    => 'foo',
    'value'   => 'bar',
    'attribs' => array()
));
?>
<input type="submit" name="foo" value="bar" />
```

### tel {#tel}

```html+php
<?php
echo $this->irnput(array(
    'type'    => 'tel',
    'name'    => 'foo',
    'value'   => 'bar',
    'attribs' => array()
));
?>
<input type="tel" name="foo" value="bar" />
```

### text {#text}

```html+php
<?php
echo $this->input(array(
    'type'    => 'text',
    'name'    => 'foo',
    'value'   => 'bar',
    'attribs' => array()
));
?>
<input type="text" name="foo" value="bar" />
```

### textarea {#textarea}

```html+php
<?php
echo $this->input(array(
    'type'    => 'textarea',
    'name'    => 'foo',
    'value'   => 'bar',
    'attribs' => array()
));
?>
<textarea name="foo">bar</textarea>
```

### time {#time}

```html+php
<?php
echo $this->input(array(
    'type'    => 'time',
    'name'    => 'foo',
    'value'   => 'bar',
    'attribs' => array()
));
?>
<input type="time" name="foo" value="bar" />
```

### url {#url}

```html+php
<?php
echo $this->input(array(
    'type'    => 'url',
    'name'    => 'foo',
    'value'   => 'bar',
    'attribs' => array()
));
?>
<input type="url" name="foo" value="bar" />
```

### week {#week}

```html+php
<?php
echo $this->input(array(
    'type'    => 'week',
    'name'    => 'foo',
    'value'   => 'bar',
    'attribs' => array()
));
?>
<input type="week" name="foo" value="bar" />
```

## Custom Helpers {#custom-helpers}

There are two steps to adding your own custom helpers:

1. Write a helper class.

2. Set a factory for that class into the _HelperLocator_ under a service name.

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
        $di->params['App\Html\Helper\Router']['router'] = $di->lazyGet('aura/web-kernel:router');
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
