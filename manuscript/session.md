# Session

Provides session management functionality, including lazy session starting,
session segments, next-request-only ("flash") values, and CSRF tools.

## Installation

We are going to install `aura/session` version `2.0.*@dev` .

Add to your `composer.json`.

```json
{
    "require": {
        //
        "aura/session": "2.0.*@dev"
    }
}
```

and run

```bash
composer update
```

## Service

Aura.Session already have `aura/session:session` service which is an object of
_Aura\Session\Session_ . You can get inject the service to responder or
view helper and make use of the _Aura\Session\Session_ object.

### [Segments](https://github.com/auraphp/Aura.Session/#segments)

### [Lazy Session Starting](https://github.com/auraphp/Aura.Session/#lazy-session-starting)

### [Saving, Clearing, and Destroying Sessions](https://github.com/auraphp/Aura.Session/#saving-clearing-and-destroying-sessions)

### [Session Security](https://github.com/auraphp/Aura.Session/#session-security)

### [Session ID Regeneration](https://github.com/auraphp/Aura.Session/#session-id-regeneration)

#### [Cross-Site Request Forgery](https://github.com/auraphp/Aura.Session/#cross-site-request-forgery)

#### [CSRF Value Generation](https://github.com/auraphp/Aura.Session/#csrf-value-generation)

### [Flash Values](https://github.com/auraphp/Aura.Session/#flash-values)

### [Setting And Getting Flash Values](https://github.com/auraphp/Aura.Session/#setting-and-getting-flash-values)

### [Keeping and Clearing Flash Values](https://github.com/auraphp/Aura.Session/#keeping-and-clearing-flash-values)
