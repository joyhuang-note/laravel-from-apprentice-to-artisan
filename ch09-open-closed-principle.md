# Open Closed Principle #

## Introduction ##

> Open Closed Principle: The Open Closed principle of SOLID design states that code is open for extension but closed for modification.

> Consider the following section of the `process` method:

    $recent = $this->orders->getRecentOrderCount($order->account);

    if($recent > 0)
    {
        throw new Exception('Duplicate order likely.');
    }

> what if our business rules regarding order validation change? What if we get new rules? In fact, what if, as our business grows, we get many new rules?

> let's define a new interface: `OrderValidatorInterface`:

OrderValidatorInterface:

    interface OrderValidatorInterface {
        public function validate(Order $order);
    }


RecentOrderValidator:

    class RecentOrderValidator implements OrderValidatorInterface {
        public function __construct(OrderRepository $orders)
        {
            $this->orders = $orders;
        }
        public function validate(Order $order)
        {
            $recent = $this->orders->getRecentOrderCount($order->account);
            if($recent > 0)
            {
                throw new Exception('Duplicate order likely.');
            }
        }
    }

SuspendedAccountValidator:

    class SuspendedAccountValidator implements OrderValidatorInterface {
        public function validate(Order $order)
        {
            if($order->account->isSuspended())
            {
                throw new Exception("Suspended accounts may not order.");
            }
        }
    }

> let's use them within our `OrderProcessor`. We'll simply inject an array of validators into the processor instance

    class OrderProcessor {
        public function __construct(BillerInterface $biller, OrderRepository $orders, array $validators = array())
        {
            $this->biller = $bller;
            $this->orders = $orders;
            $this->validators = $validators;
        }
    }

> Next, we can simply loop through the validators in the process method:

    public function process(Order $order)
    {
        foreach($this->validators as $validator)
        {
            $validator->validate($order);
        }

        // Process valid order...
    }

> Finally, we will register our `OrderProcessor` class in the application IoC container:

    App::bind('OrderProcessor', function()
    {
        return new OrderProcessor(
            App::make('BillerInterface'),
            App::make('OrderRepository'),
            array(
                App::make('RecentOrderValidator'),
                App::make('SuspendedAccountValidator')
            )
        );
    });

> we can now add and remove new validation rules without modifying a single line of existing code.
