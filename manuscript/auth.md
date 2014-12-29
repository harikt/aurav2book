# Authentication

Authentication is made possible with the help of [aura/auth](https://packagist.org/packages/aura/auth).

> If you are not using PHP 5.5, it is recommended  to use [ircmaxell/password-compat](https://packagist.org/packages/ircmaxell/password-compat) to make use of [password_](http://php.net/password) functions.

```json
{
    "require": {
        // more packages
        "aura/auth": "2.0.*@dev",
        "ircmaxell/password-compat": "~1.0"
    }
}
```

Aura.Auth supports below adapters :

- Apache htpasswd files
- SQL tables via the [PDO](http://php.net/pdo) extension
- IMAP/POP/NNTP via the [imap](http://php.net/imap) extension
- LDAP and Active Directory via the [ldap](http://php.net/ldap) extension
- OAuth via customized adapters

We will concentrate on authentication via PDO adapter.

## Building Service class

To make things simple, we need an `AuthService` class. We don't need this, if [PR 59](https://github.com/auraphp/Aura.Auth/pull/59) is merged.

```php
<?php
namespace Vendor\Package;

use Aura\Auth\Auth;
use Aura\Auth\Service\LoginService;
use Aura\Auth\Service\LogoutService;
use Aura\Auth\Service\ResumeService;
use Aura\Auth\Status;

class AuthService
{
    protected $auth;

    protected $login_service;

    protected $logout_service;

    protected $resume_service;

    protected $resumed = false;

    public function __construct(
        Auth $auth,
        LoginService $login_service,
        LogoutService $logout_service,
        ResumeService $resume_service
    ) {
        $this->auth = $auth;
        $this->login_service  = $login_service;
        $this->logout_service = $logout_service;
        $this->resume_service = $resume_service;
    }

    public function login(array $input)
    {
        return $this->login_service->login($this->auth, $input);
    }

    public function forceLogin(
        $name,
        array $data = array(),
        $status = Status::VALID
    ) {
        return $this->login_service->forceLogin($this->auth, $name, $data, $status);
    }

    public function logout($status = Status::ANON)
    {
        return $this->logout_service->logout($this->auth, $status);
    }

    public function forceLogout($status = Status::ANON)
    {
        return $this->logout_service->forceLogout($this->auth, $status);
    }

    /**
     *
     * Magic call to all auth related methods
     *
     */
    public function __call($method, array $params)
    {
        $this->resume();
        return call_user_func_array(array($this->auth, $method), $params);
    }

    public function getAuth()
    {
        $this->resume();
        return $this->auth;
    }

    protected function resume()
    {
        if (! $this->resumed) {
            $this->resume_service->resume($this->auth);
            $this->resumed = true;
        }
    }
}
```

## Configuration

T> Set the adapter according to your choice : `$di->set('aura/auth:adapter', $di->lazyNew('Aura\Auth\Adapter\PdoAdapter'));`
T> If you need to call `AuthService` from action : `$di->params['Some\BlogAction']['auth'] = $di->lazyGet('aura/auth:auth_service');`

> Assuming you have already set your connection so is loadable via lazy call : `$di->lazyGet('default_connection')` . Make sure the  class `Vendor\Package\AuthService` is autoladable.

```php
<?php
// {PROJECT_PATH}/config/Common.php
namespace Aura\Framework_Project\_Config;

use Aura\Di\Config;
use Aura\Di\Container;

class Common extends Config
{
    public function define(Container $di)
    {
        // more code
        $di->set('aura/auth:auth_service', $di->lazyNew('Vendor\Package\AuthService'));

        /**
         * Auth service
         */
        $di->params['Vendor\Package\AuthService'] = array(
            'auth' => $di->lazyGet('aura/auth:auth'),
            'login_service' => $di->lazyGet('aura/auth:login_service'),
            'logout_service' => $di->lazyGet('aura/auth:logout_service'),
            'resume_service' => $di->lazyGet('aura/auth:resume_service')
        );

        $di->params['Aura\Auth\Verifier\PasswordVerifier'] = array(
            'algo' => PASSWORD_BCRYPT,
        );

        $di->set('aura/auth:adapter', $di->lazyNew('Aura\Auth\Adapter\PdoAdapter'));

        $di->params['Aura\Auth\Adapter\PdoAdapter'] = array(
            'pdo' => $di->lazyGet('default_connection'),
            'verifier' => $di->lazyNew('Aura\Auth\Verifier\PasswordVerifier'),
            'cols' => array(
                'username',
                'password',
                'roles',
            ),
            'from' => 'users',
            'where' => 'active=1'
        );
    }

    // more code
}
```

Consider [reading adapters](https://github.com/auraphp/Aura.Auth/blob/develop-2/README.md#adapters) if you need changes/improvements.

## Calling Auth Methods

You can retrieve authentication information using the following methods on the _AuthService_ instance :

- `getUserName()`: returns the authenticated username string

- `getUserData()`: returns the array of optional arbitrary user data

- `getFirstActive()`: returns the Unix time of first activity (login)

- `getLastActive()`: return the Unix time of most-recent activity (generally that of the current request)

- `getStatus()`: returns the current authentication status constant. These constants are:

    - `Status::ANON` -- anonymous/unauthenticated

    - `Status::IDLE` -- the authenticated session has been idle for too long

    - `Status::EXPIRED` -- the authenticated session has lasted for too long in total

    - `Status::VALID` -- authenticated and valid

- `isAnon()`, `isIdle()`, `isExpired()`, `isValid()`: these return true or false, based on the current authentication status.

You can also use the `set*()` variations of the `get*()` methods above to force the _Auth_ object to whatever values you like.

## Eg : Calling methods inside action

```php
<?php
namespace Vendor\Package;

class SomeAction
{
    protected $auth;

    public function __construct(AuthService $auth)
    {
        $this->auth = $auth;
    }

    public function __invoke()
    {
        $this->auth->isValid();
        $this->auth->login($input);
        $this->auth->logout();
        // etc
    }
}
```
