# Liskov Substitution Principle #

## Introduction ##

> if a class uses an implementation of an interface, it must be able to use any implementation of that interface without requiring any modifications

> (如果一个类使用了一个接口的一个实现类，那么该接口的任何其他实现类也可以被这里直接使用，不用做出任何修改。)

> continue to use our `OrderProcessor` example from the previous chapter:

    public function process(Order $order)
    {
        // Validate order...
        $this->orders->logOrder($order);
    }

>  we log the order using the OrderReporsitoryInterface implementation.

> Our only `OrderRepositoryInterface` implementation was a `CsvOrderRepository`. Now, as our order rate grows, we want to use a relational database to store them.

    class DatabaseOrderRepository implements OrderRepositoryInterface {
        protected $connection;
        public function connect($username, $password)
        {
            $this->connection = new DatabaseConnection($username, $password);
        }

        public function logOrder(Order $order)
        {
            $this->connection->run('insert into orders values (?, ?)', array(
                $order->id, $order->amount
            ));
        }
    }

> Now, let's examine how we would have to consume this implementation:

    public function process(Order $order)
    {
        // Validate order...

        if($this->repository instanceof DatabaseOrderRepository)
        {
            $this->repository->connect('root', 'password');
        }
        $this->repository->logOrder($order);
    }

> Notice that we are forced to check if our `OrderRepositoryInterface` is a database implementation from within our consuming processor class.

> if the `OrderRepositoryInterface` is consumed by dozens of other classes?

> The example above clearly breaks the Liskov Substitution Principle.

> Here is our new DatabaseOrderRepository implementation:

    class DatabaseOrderRepository implements OrderRepositoryInterface {
        protected $connector;
        public function __construct(DatabaseConnector $connector)
        {
            $this->connector = $connector;
        }
        public function connect()
        {
            return $this->connector->bootConnection();
        }
        public function logOrder(Order $order)
        {
            $connection = $this->connect();
            $connection->run('insert into orders values (?, ?)', array(
                $order->id, $order->amount
            ));
        }
    }

> Now our `DatabaseOrderRepository` is managing the connection to the database, and we can remove our “bootstrap” code from the consuming `OrderProcessor`:

    public function process(Order $order)
    {
        // Validate order...

        $this->repository->logOrder($order);
    }

> With this modification, we can now use our `CsvOrderRepository` or `DatabaseOrderRepository` without modifying the `OrderProcessor` consumer.

> Watch For Leaks: You have probably noticed that this principle is closely related to the avoidance of “leaky abstractions”

> Our database repository's leaky abstraction was our first clue that the Liskov Substitution Principle was being broken.
