# Forms

Forms are an integral part of web application.
[Aura.Input](https://github.com/auraphp/Aura.Input) is a tool to
describe HTML fields and values.

## Installation

Even though Aura.Input has a base filter implementation,
it is good to integrate a powerful filter system like Aura.Filter.

The `foa/filter-input-bundle`, and `foa/filter-intl-bundle` already
have done the heavy lifting integrating the `aura/input`, `aura/filter`,
`aura/intl` and having the necessary DI configuration.

Add those bundles to your `composer.json`.

```json
{
    "require": {
        "foa/filter-input-bundle": "1.1.*",
        "foa/filter-intl-bundle": "1.1.*"
    }
}
```

and run

```bash
composer update
```

## Usage

Inorder to create a form, we need to extend the `Aura\Input\Form` class
and override the `init()` method.

An example is shown below.

```php
<?php
/**
 * {$PROJECT_PATH}/src/App/Input/ContactForm.php
 */
namespace App\Input;

use Aura\Input\Form;

class ContactForm extends Form
{
    public function init()
    {
        $states = array(
            'AL' => 'Alabama',
            'AK' => 'Alaska',
            'AZ' => 'Arizona',
            'AR' => 'Arkansas',
            // ...
        );

        // set input fields
        // hint the view layer to treat the first_name field as a text input,
        // with size and maxlength attributes
        $this->setField('first_name', 'text')
             ->setAttribs(array(
                 'name' => "contact[first_name]",
                 'id' => 'first_name',
                'size' => 20,
                'maxlength' => 20,
             ));

        // hint the view layer to treat the state field as a select, with a
        // particular set of options (the keys are the option values,
        // and the values are the displayed text)
        $this->setField('state', 'select')
             ->setAttribs(array(
                 'name' => "contact[state]",
                 'id' => 'state',
             ))
             ->setOptions($states);

        $this->setField('message', 'textarea')
            ->setAttribs([
                'name' => "contact[message]",
                'id' => 'message',
                'cols' => 40,
                'rows' => 5,
            ]);
        // etc.

        // get filter object
        $filter = $this->getFilter();
        // set your filters.
        $filter->addSoftRule('first_name', $filter::IS, 'string');
        $filter->addSoftRule('first_name', $filter::IS, 'strlenMin', 4);
        $filter->addSoftRule('state', $filter::IS, 'inKeys', array_keys($states));
        $filter->addSoftRule('message', $filter::IS, 'string');
        $filter->addSoftRule('message', $filter::IS, 'strlenMin', 6);
    }
}
```

We will talk about [Aura.Filter](https://github.com/auraphp/Aura.Filter/tree/develop) in next chapter.

> Note : We are using v1 components of input, intl, filter.

## Configuration

If we create `App\Input\ContactForm` object via the new operator
we need to pass the dependencies manually as

```php
use Aura\Input\Form;
use Aura\Input\Builder;
use FOA\Filter_Input_Bundle\Filter;

$filter = new Filter();

// add rules to the filter

$contact_form = new ContactForm(new Builder, $filter);
```

Creating object via DI configuration helps us not to add the dependencies,
add the necessary rules to the filter.

We only need to figure out where we need the form, and use params or
setter injection.

```php
$di->params['Vendor\Package\SomeDomain']['contact_form'] = $di->lazyNew('App\Input\ContactForm');
```

## Populating

Form can be populated using `fill()` method.

```php
$this->contact_form->fill($_POST);
```

> In aura term it will be [$this->request->post->get()](https://github.com/auraphp/Aura.Web/blob/develop-2/README-REQUEST.md#superglobals)

## Validating User Input

You can validate the form via the `filter()` method.

```php
// apply the filters
$pass = $this->contact_form->filter();

// did all the filters pass?
if ($pass) {
    // yes input is valid.
} else {
    // no; user input is not valid.
}
```

## Rendering

Inorder to render the form, we need to pass the ContactForm object and use the
`Aura.Html` helpers.

Assuming you have passed the `ContactForm` object, and the variable assigned
is `contact_form` you can use the `get` method on the form object to
get the hints of field, and pass to input helper.

An example is given below :

```php
echo $this->input($this->contact_form->get('first_name'));
```

Read more [on form helpers here](#leanpub-auto-aurahtml-form-helpers).

In the session chapter we will learn how to set flash message when the
form submission was success.
