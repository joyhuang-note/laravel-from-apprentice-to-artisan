# Application Structure #

## MVC Is Killing You ##

> The biggest roadblock towards developers achieving good design is a simple acronym: M-V-C. Models, views, and controllers have dominated web framework thinking for years

> Supposedly, the model is the database. It's where all your database stuff goes, whatever that means. But, as you quickly learn, your application needs a lot more logic than just a simple database access class. It needs to do validation, call external services, send e-mails, and more.

> What Is A Model?: it will be easier to separate our application into smaller, cleaner classes with a clearly defined responsiblity.

> [** Important **] So, what is the solution to this dilemma? Many developers start packing logic into their controllers. Once the controllers get large enough, they need to re-use business logic that is in other controllers. Instead of extracting the logic into another class, most developers mistakenly assume they need to call controllers from within other controllers. This pattern is typically called “HMVC”. Unfortunately, this pattern often indicates poor application design, and controllers that are much too complicated.

> HMVC (Usually) Indicates Poor Design: Feel the need to call controllers from other controllers? This is often indicative of poor application design and too much business logic in your controllers. Extract the logic into a third class that can be injected into any controller.

> let's just delete the model directory and start fresh!

// app
    // QuickBill
        // Repositories
            -> UserRepository.php
            -> PaymentRepository.php
        // Billing
            -> BillerInterface.php
            -> StripeBiller.php
        // Notifications
            -> BillingNotifierInterface.php
            -> SmsBillingNotifier.php
        User.php
        Payment.php

(application-without-model)

> What About Validation?: Where to perform validation often stumps developers. Consider placing validation methods on your “entity” classes (like `User.php` and `Payment.php`). Possible method name might be: `validForCreation` or `hasValidDomain`. Alternatively, you could create a `UserValidator` class within a Validation namespace and inject that validator into your repository. Experiment with both approaches and see what you like best!

## It's All About The Layers ##

> As you may have noticed, a key to solid application design is simply separating responsibilities, or creating layers of responsibility.

> Controllers are responsible for receiving an HTTP request and calling the proper business layer classes.

> business / domain layer is your application. It contains the classes that retrieve data, validate data, process payments, send e-mail, and any other function of your application.

> Separation of responsibilities is one of the keys to writing maintainable applications.

> You should frequently ask yourself: “Should this class care about X?” If answer is “no”, extract the logic into another class that can be injected as a dependency.

> Single Reason To Change: should we need to change code within a Biller implementation when tweaking our notification logic? Of course not. The Biller implementations are concerned with billing, and should only work with notification logic via a contract.

## Where To Put “Stuff” ##

### Helper Functions ###

> Laravel ships with a file full of helpers functions (`support/helpers.php`). 

> A great place to include these functions are the “start” files.

> your `start/global.php` file, which is included on every request to the application, you may simply require your own `helpers.php` file:

    // Within app/start/global.php

    require_once __DIR__.'/../helpers.php';

### Event Listeners ###

A great option (where stores Event Listeners) is a service provider.

> By grouping related event registrations inside of a service provider, the code stays neatly tucked away behind the scenes of your main application code.

> View composers, which are a type of event, may also be neatly grouped within service providers.

    <?php namespace QuickBill\Providers;
        use Illuminate\Support\ServiceProvider;

        class BillingEventsProvider extends ServiceProvider{
            public function boot()
            {
                Event::listen('billing.failed', function($bill)
                {
                    // Handle failed billing event...
                });
            }
        }

> After creating the provider, we would simply add it our `providers` array within the `app/config/app.php` configuration file.

Wear The Boot: Remember, in the example above, we are using the `boot` method for a reason. The `register` method in a service provider is only intended for binding classes into container.

### Error Handlers ###

> these are best moved into a service provider.

> The service provider might be named something like `QuickBillErrorProvider`.

> Within the `boot` method of this provider you may register all of your custom error handlers.

    <?php namespace QuickBill\Providers;

    use App, Illuminate\Support\ServiceProvider;

    class QuickBillErrorProvider extends ServiceProvider {
        public function register()
        {    
            //
        }

        public function boot()
        {
            App::error(function(BillingFailedException $e)
            {
                // Handle failed billing exceptions ...
            });
        }

### The Rest ###

>  you might create a Providers namespace for all of your application's custom service providers, creating a directory structure like so:

// app
    // QuickBill
        // Billing
        // Extensions
            //Pagination
                -> Environment.php
        // Providers
            -> EventPusherServiceProvider.php
        // Repositories
        User.php
        Payment.php


