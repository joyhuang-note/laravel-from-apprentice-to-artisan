# Dependency Injection #

## The Problem ##
> The foundation of the Laravel framework is its powerful IoC container.

> IoC container is simply a convenience mechanism for achieving a software design pattern: dependency injectionl.

> Separation Of Concerns: Every class should have a single responsibility, and that responsibility should be entirely encapsulated by the class.（译者注：我认为就是不要让多个类负责同样的职责）

> Think of the Internet as just a cable into your application. The bulk of a monitor's functionality is independent of the cable. The cable is just a transport mechanism just like HTTP is a transport mechanism for your application. So, we don't want to clutter up our transport mechanism (the controller) with application logic. This will allow any transport layer, such as an API or mobile application, to access our application logic.

## Build A Contract ##

> Respect Boundaries: Remember to respect responsibility boundaries. Controllers and routes serve as a mediator between HTTP and your application. When writing large applications, don't clutter them up with your domain logic.

interface UserRepositoryInterface
{
    public function all();
}
   
class DbUserRepository implements UserRepositoryInterface
{
    public function all()
    {
        return User::all()->toArray();
    }
}

class UserController extends BaseController
{
    public function __construct(UserRepositoryInterface $users)
    {
        $this->users = $users;
    }
   
    public function getIndex()
    {
        $users=$this->users->all();
        return View::make('users.index', compact('users'));
    }
}

public function testIndexActionBindsUsersFromRepository()
{    
    // Arrange...
    $repository = Mockery::mock('UserRepositoryInterface');
    $repository->shouldReceive('all')->once()->andReturn(array('foo'));
    App::instance('UserRepositoryInterface', $repository);
    // Act...
    $response  = $this->action('GET', 'UserController@getIndex');
         
    // Assert...
    $this->assertResponseOk();
    $this->assertViewHas('users', array('foo'));
}

## Taking It Further ##

> But what about IoC containers? Aren't they necessary to do dependency injection? Absolutely not! As we'll see in the following chapters, containers make dependency injection easier to manage, but they are not a requirement.

interface BillerInterface {
    public function bill(array $user, $amount);
}

interface BillingNotifierInterface {
    public function notify(array $user, $amount);
}

class StripeBiller implements BillerInterface {
    public function __construct(BillingNotifierInterface $notifier)
    {
        $this->notifier = $notifier;
    }

    public function bill(array $user, $amount)
    {
        // Bill the user via Stripe...
        $this->notifier->notify($user, $amount);
    }
}

> By separating the responsibilities of each class, we're now able to easily inject various notifier implementations into our billing class. For example, we could inject a SmsNotifier or an EmailNotifier.

> how do we do dependency injection?

$biller = new StripeBiller(new SmsNotifier);
