# The IoC Container #

## Basic Binding ##

> In a Laravel application, the IoC container can be accessed via the App facade.

> the Laravel Application class extends the Container class!

> 这个容器就是个用来存储各种绑定的地方

App::bind('BillerInterface', function()
{
    return new StripeBiller(App::make('BillingNotifierInterface'));
});

App::bind('BillingNotifierInterface', function()
{
    return new EmailBillingNotifier;
});

> Once we're using the container, we can switch interface implementations with a single line change. 

class UserController extends BaseController {

    public function __construct(BillerInterface $biller)
    {
        $this->biller = $biller;
    }
}

App::bind('BillingNotifierInterface', function()
{
    return new SmsBillingNotifier;
});


> Sometimes you may wish to resolve only one instance of a given class throughout your entire application. This can be achieved via the `singleton` method on the container class:

App::singleton('BillingNotifierInterface', function()
{
    return new SmsBillingNotifier;
});

> The `instance` method on the container is similar to `singleton`, however, you are able to pass an already existing object instance. The instance you give to the container will be used each time the container needs an instance of that class:

$notifier = new SmsBillingNotifier;
App::instance('BillingNotifierInterface', $notifier);

> Stand Alone Container:

> Working on a project that isn't built on Laravel? You can still utilize Laravel's IoC container by installing the `illuminate/container` package via Composer!

## Reflect Resolution ##

> One of the most powerful features of the Laravel container is its ability to automatically resolve dependencies via reflection.

> Reflection is the ability to inspect a classes and methods.

$reflection = new ReflectionClass('StripeBiller');
var_dump($reflection->getMethods());
var_dump($reflection->getConstants());

> When the Laravel container does not have a resolver for a class explictity bound, it will try to resolve the class via reflection.

class UserController extends BaseController
{
    public function __construct(StripBiller $biller)
    {
        $this->biller = $biller;
    }
}

> The flow looks like this:

1. Do I have a resolver for `StripBiller`?
2. No resolver? Reflect into `StripBiller` to determin if it has dependencies.
3. Resolve any dependencies needed by `StripBiller` (recursive).
4. Instantiate new `StripBiller` instance via `ReflectionClass->newInstanceArgs()`.

> interface can't be instantiated since they are just contracts. Without us giving the container any more information, it will be unable to instantiate this dependency.

class UserController extends BaseController
{
    public function __construct(BillerInterface $biller)
    {
        $this->biller = $biller;
    }
}

> We need to specify a class that should be used as the default implementation of this interface, and we may do so via the bind method:

App::bind('BillerInterface','StripBiller');

> we're gaining the ability to switch implementations of services with a simple one-line change to our container binding.

App::bind('BillerInterface', 'BalancedBiller');

> When binding implementations to interfaces, you may also use the singleton method so the container only instantiates one instance of the class per request cycle

App::singleton('BillerInterface', 'StripBiller');

> container is only one class: `Illuminate\Container\Container`.
