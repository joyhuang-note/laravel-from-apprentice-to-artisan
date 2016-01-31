# Dependency Inversion Principle #

## Introduction ##

Low-level code:

> implements basic operations like reading files from a disk, or interaction with a database.

High-level code:

> encapsulates complex logic and relies on the low-level code to function, but should not be directly coupled to it.

> high-level code should not depend on low-level code.

> high-level code should depend on an abstraction layer that serves as a “middle-man” between the high and low-level code.

> A second aspect to the principle is that abstractions should not depend upon details, but rather details should depend upon abstractions.

## In Action ##

    class Authenticator {
        public function __construct(DatabaseConnection $db)
        {
            $this->db = $db;
        }
        public function findUser($id)
        {
            return $this->db->exec('select * from users where id = ?', array($id));
        }
        public function authenticate($credentials)
        {
            // Authenticate the user...
        }
    }

> the `Authenticator` class is responsible for finding and authenticating users.

> we are type-hinting a `DatabaseConnection` instance.

> So, we're tightly coupling our authenticator to the database, and essentially saying that users will always only be provided out of a relational SQL database. 

> our high-level code (the `Authenticator`) is directly depending on low-level code (the `DatabaseConnection`).

> the high-level code should depend on an abstraction that sits on top of the low-level code, such as an interface. Not only that, but the low-level code should also depend upon an abstraction. So, let's write an interface that we can use within our `Authenticator`:

    interface UserProviderInterface {
        public function find($id);
        public function findByUsername($username);
    }

> inject an implementation of this interface into our `Authenticator`:

    class Authenticator {
        public function __construct(UserProviderInterface $users, HasherInterface $hash)
        {
            $this->hash = $hash;
            $this->users = $users;
        }
        public function findUser($id)
        {
            return $this->users->find($id);
        }
        public function authenticate($credentials)
        {
            $user = $this->users->findByUsername($credentials['username']);
            return $this->hash->make($credentials['password']) == $user->password;
        }
    }

> our low-level code now depends on the high-level `UserProviderInterface` abstraction, since it implements the interface itself

    class RedisUserProvider implements UserProviderInterface {
        public function __construct(RedisConnection $redis)
        {
            $this->redis = $redis;
        }
        public function find($id)
        {
            $this->redis->get('users:'.$id);
        }
        public function findByUsername($username)
        {
            $id = $this->redis->get('user:id:'.$username);
            return $this->find($id);
        }
    }

> Inverted Thinking: this principle states that both high and low-level code depend upon a high-level abstraction.


