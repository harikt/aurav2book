# Validation

[Aura.Filter](https://github.com/auraphp/Aura.Filter) is a tool to validate 
and sanitize data.

We are going to look into version 2 of Aura.Filter.

## Installation

Add `"aura/filter": "2.0.*@dev"` into the require section of `composer.json`.

```json
{
    "require": {
        // ...
        "aura/filter": "2.0.*@dev"
    }
}
```

and run

```bash
composer update
```

## Instantiation

The easiest way to instantiate a new filter (i.e., a new `Aura\Filter\RuleCollection`) 
with all the available rules is to use the FilterFactory class.

```php
$filter = (new FilterFactory())->newInstance();
```

We may be mostly using it with DI configuration.

We are not going to cover 
[soft, hard, and stop rules](https://github.com/auraphp/Aura.Filter/#soft-hard-and-stop-rules), 
[validation and sanitization](https://github.com/auraphp/Aura.Filter/#validating-and-sanitizing),
[available rules](https://github.com/auraphp/Aura.Filter/#available-rules),
[writing your rule class](https://github.com/auraphp/Aura.Filter/#creating-and-using-custom-rules).

We don't want to duplicate the efforts of the Aura.Filter documentation
and feel there is nothing more to add here. So please refer to the
[documentation on Aura.Filter](https://github.com/auraphp/Aura.Filter/blob/develop-2/README.md#getting-started).
If you find something missing or hard to follow
[open an issue](https://github.com/harikt/aurav2book/issues) happy to help.

## Messages and Internationalization

The error messages you will be getting on failures will be string constants. So you
may need to convert them to proper string. We will cover this in the
internationalization section.
