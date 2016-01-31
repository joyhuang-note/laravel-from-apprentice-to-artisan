# Service Providers #

- Service Provider
  - As Bootstrapper
  - As Organizer
  - Booting Providers
  - Providing The Core

## As Bootstrapper ##

> A Laravel service provider is a class that registers IoC container bindings.

> A service provider must have at least one method: `register`.

> When a request enters your application and the framework is booting up, the register method is called on the providers listed in your configuration file. 

> Register Vs. Boot: Never attemp to use services in the `register` method. This method is only for binding objects into the IoC container.

> All resolutions and interaction with the bound classes must be done inside the `boot` method.

> The (third-party) package's installation instructions will typically tell you to register their provider with your application by adding it to the providers array in your configuration file.

> Package Providers: Not all third-party packages require a service provider. They (service providers) are simply a convenient place to organize bootstrap code and container bindings.

## Deferred Providers 延迟加载的服务提供者 ##

> Not every provider that is listed in your providers configuration array will be instantiated on every request.

> In order to only instantiate the providers that are needed for a given request, Laravel generates a “service manifest” that is stored in the `app/storage/meta` directory.

> Manifest Generation: When you add a service provider to the providers array, Laravel will automatically regenerate the service manifest file during the next request to the application.

## As Organizer ##

> use the provider as organizational tools

> Get It Started: Your application's “start” files are all stored in the `app/start`directory.

> Instead of doing container registerations in those files, create service providers that register related services.

> Start files named after an environment are loaded after the `global.php` file. Finally, the `artisan.php` start file is loaded when any console command is executed.

interface EventPusherInterface{
    public function push($message, array $data = array());
}

class PusherEventPusher implements EventPusherInterface{
    public function __construct(PusherSdk $pusher)
    {
        $this->pusher = $pusher;
    }
    public function push($message, array $data = array())
    {
        // Push message via the Pusher SDK...
    }
}

> Next, let's create an `EventPusherServiceProvider`:

use Illuminate\Support\ServiceProvider;

class EventPusherServiceProvider extends ServiceProvider {
    public function register()
    {
        $this->app->singleton('PusherSdk', function()
        {
            return new PusherSdk('app-key', 'secret-key');
        }

        $this->app->singleton('EventPusherInterface', 'PusherEventPusher');
    }
}

> Finally, we just need to add the `EventPusherServiceProvider` to our providers array in the `app/config/app.php` configuration file. Now we are ready to inject the `EventPusherInterface` into any controller or class within our application.

> Should You Singleton?: If you only want one instance of the class to be created per request cycle, use `singleton`. Otherwise, use `bind`.

> Note that a service provider has an `$app` instance available via the base `ServiceProvider` class. This is a full `Illuminate\Foundation\Application` instance, which inherits from the `Container` class, so we can call all of the IoC container methods we are used to.

>  If you preffer to use the `App` facade inside the service provider, you may do that as well:

App::singleton('EventPusherInterface', 'PusherEventPusher');

## Booting Providers ##

> After all providers have been registered, they are “booted”. This will fire the `boot`method on each provider.

> within the `register` method, we have no gurantee all other providers have been loaded, the service you are trying to use may not be available yet. So, service provider code that uses other services should always live in the `boot` method

> Within the boot method, you may do whatever you like: register event listeners, include a route file, register filters, or anything else you can imagine.

public function boot()
{
    require_once __DIR__.'/events.php';
    require_once __DIR__.'/routes.php';
}

> Remember, don't assume that service providers are something that only packages use. Create your own to help organize your application's services.

## Providing The Core ##





