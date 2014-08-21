# Getting Started

[Composer](http://getcomposer.org) has become the de facto standard 
for installing libraries in the php world. Aura does the same.


## Installation

There are 3 types of skeletons

* aura/web-project : only web application, no cli support built in.
* aura/cli-project : only for command line applications.
* aura/framework-project : supports both web and cli

We are going to install `aura/framework-project`, so we can show command line 
examples.

```php
composer create-project --stability=dev aura/framework-project {$PROJECT_PATH}
```
    
It will create the `{$PROJECT_PATH}` directory and install the dependencies
in vendor folder.

### Structure

The directory structure looks something similar to this. The list is not 
complete for we have removed some of the files and directories.

```bash
├── cli
│   └── console.php
├── composer.json
├── composer.lock
├── config
│   ├── Common.php
│   ├── Dev.php
│   ├── _env.php
│   ├── Prod.php
│   └── Test.php
├── src
├── tmp
│   ├── cache
│   └── log
├── vendor
│   ├── aura
│   │   ├── cli
│   │   ├── cli-kernel
│   │   ├── di
│   │   ├── dispatcher
│   │   ├── project-kernel
│   │   ├── router
│   │   ├── web
│   │   └── web-kernel
│   ├── autoload.php
│   ├── monolog
│   │   └── monolog
│   └── psr
│       └── log
└── web
    └── index.php
```

The `web/index.php` is where you need to point your virtual host. Check the 
chapter Setting up your virtual host for more information.

Alternatively you can start the built-in PHP server.


```bash
php -S localhost:8000 -t web/
```

If you point your web browser to `http://localhost:8000` you can see 
the message `Hello World!`.

Great! Everything is working fine.

## Exploring the Hello World!

Open the file `config/Common.php`. Look into the `modifyWebRouter()` and 
`modifyWebDispatcher()` methods.

```php
public function modifyWebRouter(Container $di)
{
    $router = $di->get('web_router');
    $router->add('hello', '/')
           ->setValues(array('action' => 'hello'));
}
```

The `modifyWebRouter()` gets the shared router service and adds a route 
named `hello` which points to `/` . So any request to `http://localhost:8000` 
is satisfied by route named `hello`.

Now we have the route, the router don't know what to do when a request come.
The dispatcher is what helps to dispatch things.

```php
public function modifyWebDispatcher($di)
{
    $dispatcher = $di->get('web_dispatcher');

    $dispatcher->setObject('hello', function () use ($di) {
        $response = $di->get('web_response');
        $response->content->set('Hello World!');
    });
}
```

We get the shared dispatcher service, set the same name as in the 
controller of route in `setObject`, and use a Closure or Callable.

In this example we are using a Closure, which get the di container and use 
it as a service, get the shared web response and set the content.

Don't worry too much about dependency injection and dependency injection 
container. We will be talking more details in the next Chapter.
