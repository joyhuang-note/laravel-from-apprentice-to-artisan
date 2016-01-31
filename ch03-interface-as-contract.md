# Interface As Contract #

## Strong Typing & Water Fowl ##

> This is an example of duck typing. If the object looks like a duck, quacks like a duck, it must be a duck. Or, in this case, if the object looks like a user, and acts like a user, it must be one.

public function billUser($user)
{
    $this->biller->bill($user->getId(), $this->amount);
}

> PHP has a mixture of both strong and duck typed constructs.

public function billUser(User $user)
{
    $this->biller->bill($user->getId(), $this->amount);
}

## A Contract Example ##

> Interfaces do not contain any code, but simply define a set of methods that an object must implement.

> Polymorphism: we mean that an interface can have multiple implementations. For example, a UserRepositoryInterface could have a MySQL and a Redis implementation, and both implementations would qualify as a UserRepositoryInterface instance.

## Interfaces & Team Development ##

> Interface As Schematic: Interfaces are helpful for developing a “skeleton” of defined functionality provided by your application.
