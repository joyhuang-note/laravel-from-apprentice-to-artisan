# Interface Segregation Principle #

## Introduction ##

> The Interface Segregation principle states that no implementation of an interface should be forced to depend on methods it does not use.

> (接口隔离原则规定在实现接口的时候，不能强迫去实现没有用处的方法。)

> this principle demands that interfaces be granular and focused.

> (该原则要求接口必须粒度很细，且专注于一个领域。)

> Instead of having a “fat” interface containing methods not needed by all implementations, it is preferable to have several smaller interfaces that may be implemented individually as needed.

## In Action ##

    interface SessionHandlerInterface {
        public function close();
        public function destroy($sessionId);
        public function gc($maxLifetime);
        public function open($savePath, $name);
        public function read($sesssionId);
        public function write($sessionId, $sessionData);
    }

> consider an implementation using Memcached. Would a Memcached implementation of this interface define functionality for each of these methods? Not only do we not need to implement all of these methods, we don't need half of them!

> we do not need to implement the `gc` method of the interface, nor do we need to implement the `open` or `close` methods of the interface.

> To correct this problem, let's start by defining a smaller, more focused interface for session garbage collection:

    interface GarbageCollectorInterface {
        public function gc($maxLifetime);
    }
