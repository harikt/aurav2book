# Command line / cli / console

In this chapter we assume you have installed either `aura/framework-project`
or `aura/cli-project`.

Both `aura/framework-project` and `aura/cli-project` make use of the `aura/cli`
library. Aura.Cli can be used as standalone library. Please
[refer getting started](https://github.com/auraphp/Aura.Cli#getting-started)
if you looking for standalone usage.

## Features of Aura.Cli

Below is a list of few examples taken from the `Aura.Cli` readme.

### Context Discovery

The Context object provides information about the command line environment,
including any option flags passed via the command line.

```php
// get copies of superglobals
$env    = $context->env->get();
$server = $context->server->get();
$argv   = $context->argv->get();

// equivalent to:
// $value = isset($_ENV['key']) ? $_ENV['key'] : 'other_value';
$value = $context->env->get('key', 'other_value');
```

[Read more](https://github.com/auraphp/Aura.Cli#context-discovery)

##  Getopt Support

The Context object provides support for retrieving command-line options
and params, along with positional arguments.

```php
$options = array(
    'a',        // short flag -a, parameter is not allowed
    'b:',       // short flag -b, parameter is required
    'c::',      // short flag -c, parameter is optional
    'foo',      // long option --foo, parameter is not allowed
    'bar:',     // long option --bar, parameter is required
    'baz::',    // long option --baz, parameter is optional
    'g*::',     // short flag -g, parameter is optional, multi-pass
);

$getopt = $context->getopt($options);
$a   = $getopt->get('-a', false); // true if -a was passed, false if not
$b   = $getopt->get('-b');
$c   = $getopt->get('-c', 'default value');
```

[Read more](https://github.com/auraphp/Aura.Cli#getopt-support)

### Positional Arguments

```php
<?php
$getopt = $context->getopt();

// if the script was invoked with:
// php script.php arg1 arg2 arg3 arg4

$val0 = $getopt->get(0); // script.php
$val1 = $getopt->get(1); // arg1
$val2 = $getopt->get(2); // arg2
$val3 = $getopt->get(3); // arg3
$val4 = $getopt->get(4); // arg4
```

[Read more](https://github.com/auraphp/Aura.Cli#positional-arguments)

### Standard Input/Output Streams

The Stdio object allows you to work with standard input/output streams.

```php
<?php
// print to stdout
$stdio->outln('This is normal text.');

// print to stderr
$stdio->errln('<<red>>This is an error in red.');
$stdio->errln('Output will stay red until a formatting change.<<reset>>');
```

[Read more](https://github.com/auraphp/Aura.Cli#standard-inputoutput-streams)

### Writing Command Help

```php
<?php
use Aura\Cli\Help;

class MyCommandHelp extends Help
{
    protected function init()
    {
        $this->setSummary('A single-line summary.');
        $this->setUsage('<arg1> <arg2>');
        $this->setOptions(array(
            'f,foo' => "The -f/--foo option description",
            'bar::' => "The --bar option description",
        ));
        $this->setDescr("A multi-line description of the command.");
    }
}
```

[Read more](https://github.com/auraphp/Aura.Cli#standard-inputoutput-streams)

## Services

Aura.Cli_Kernel defines the following service objects in the _Container_:

- `aura/cli-kernel:dispatcher`: an instance of _Aura\Dispatcher\Dispatcher_
- `aura/cli-kernel:context`: an instance of _Aura\Cli\Context_
- `aura/cli-kernel:stdio`: an instance of _Aura\Cli\Stdio_
- `aura/cli-kernel:help_service`: an instance of _Aura\Cli_Kernel\HelpService_
- `aura/project-kernel:logger`: an instance of `Monolog\Logger`

## Quick Start

The dependency injection _Container_ is absolutely central to the operation
of an Aura project. Please be familiar with
[the DI docs](#leanpub-auto-dependency-injection) before continuing.

You should also familiarize yourself with [Aura.Dispatcher](#leanpub-auto-dispatching),
as well as the [Aura.Cli](https://github.com/auraphp/Aura.Cli) _Context_, _Stdio_, and _Status_ objects.

## Project Configuration

Every Aura project is configured the same way. Please see the [shared configuration docs](#leanpub-auto-configuration) for more information.

## Logging

The project automatically logs to `{$PROJECT_PATH}/tmp/log/{$mode}.log`.
If you want to change the logging behaviors for a particular config mode,
edit the related config file (e.g., `config/Dev.php`) file to modify
the `aura/project-kernel:logger` service.

## Commands

We configure commands via the project-level `config/` class files.
If a command needs to be available in every config mode, edit the
project-level `config/Common.php` class file. If it only needs to
be available in a specific mode, e.g. `dev`, then edit the config
file for that mode.

Here are two different styles of command definition.

## Micro-Framework Style

The following is an example of a command where the logic is embedded in the dispatcher, using the `aura/cli-kernel:context` and `aura/cli-kernel:stdio` services along with standard exit codes. (The dispatcher object name doubles as the command name.)

```php
<?php
namespace Aura\Cli_Project\_Config;

use Aura\Di\Config;
use Aura\Di\Container;

class Common extends Config
{
    // ...

    public function modifyCliDispatcher(Container $di)
    {
        $context = $di->get('aura/cli-kernel:context');
        $stdio = $di->get('aura/cli-kernel:stdio');
        $dispatcher = $di->get('aura/cli-kernel:dispatcher');
        $dispatcher->setObject(
            'foo',
            function ($id = null) use ($context, $stdio) {
                if (! $id) {
                    $stdio->errln("Please pass an ID.");
                    return \Aura\Cli\Status::USAGE;
                }

                $id = (int) $id;
                $stdio->outln("You passed " . $id . " as the ID.");
            }
        );
    }
?>
```

You can now run the command to see its output.

    cd {$PROJECT_PATH}
    php cli/console.php foo 88

(If you do not pass an ID argument, you will see an error message.)

## Full-Stack Style

You can migrate from a micro-controller style to a full-stack style (or start
with full-stack style in the first place).

First, define a command class and place it in the project `src/` directory.

```php
<?php
/**
 * {$PROJECT_PATH}/src/App/Command/FooCommand.php
 */
namespace App\Command;

use Aura\Cli\Stdio;
use Aura\Cli\Context;
use Aura\Cli\Status;

class FooCommand
{
    public function __construct(Context $context, Stdio $stdio)
    {
        $this->context = $context;
        $this->stdio = $stdio;
    }

    public function __invoke($id = null)
    {
        if (! $id) {
            $this->stdio->errln("Please pass an ID.");
            return Status::USAGE;
        }

        $id = (int) $id;
        $this->stdio->outln("You passed " . $id . " as the ID.");
    }
}
?>
```

Next, tell the project how to build the _FooCommand_ through the DI
_Container_. Edit the project `config/Common.php` file to configure the
_Container_ to pass the `aura/cli-kernel:context` and `aura/cli-kernel:stdio` service objects to
the _FooCommand_ constructor. Then put the _App\Command\FooCommand_ object in the dispatcher under the name `foo` as a lazy-loaded instantiation.

```php
<?php
namespace Aura\Cli_Project\_Config;

use Aura\Di\Config;
use Aura\Di\Container;

class Common extends Config
{
    public function define(Container $di)
    {
        $di->set('aura/project-kernel:logger', $di->newInstance('Monolog\Logger'));

        $di->params['App\Command\FooCommand'] = array(
            'context' => $di->lazyGet('aura/cli-kernel:context'),
            'stdio' => $di->lazyGet('aura/cli-kernel:stdio'),
        );
    }

    // ...

    public function modifyCliDispatcher(Container $di)
    {
        $dispatcher = $di->get('aura/cli-kernel:dispatcher');

        $dispatcher->setObject(
            'foo',
            $di->lazyNew('App\Command\FooCommand')
        );
    }
?>
```

You can now run the command to see its output.

    cd {$PROJECT_PATH}
    php cli/console.php foo 88

(If you do not pass an ID argument, you will see an error message.)
