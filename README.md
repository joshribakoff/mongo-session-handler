# Mongo Session Handler [![Build Status](https://travis-ci.org/altmetric/mongo-session-handler.svg?branch=master)](https://travis-ci.org/altmetric/mongo-session-handler)

A PHP session handler backed by MongoDB.

**Current version:** 2.0.0  
**Supported PHP versions:** 5.4, 5.5, 5.6, 7

## Installation

```shell
$ composer require altmetric/mongo-session-handler
```

## Usage

```php
<?php
use Monolog\Logger; // or any other PSR-3 compliant logger
use Altmetric\MongoSessionHandler;

$sessions = $mongoClient->someDB->sessions;
$logger = new Logger;
$handler = new MongoSessionHandler($sessions, $logger);

session_set_save_handler($handler);
session_set_cookie_params(0, '/', '.example.com', false, true);
session_name('my_session_name');
session_start();
```

## API Documentation

### `public MongoSessionHandler::__construct(MongoDB\Collection $collection, Psr\Log\LoggerInterface $logger)`

```php
$handler = new \Altmetric\MongoSessionHandler($client->db->sessions, $logger);
session_set_save_handler($handler, true);
session_start();
```

Instantiate a new MongoDB session handler with the following arguments:

* `$collection`: a [`MongoDB\Collection`](http://mongodb.github.io/mongo-php-library/classes/collection/) collection to use for session storage;
* `$logger`: [`Psr\Log\LoggerInterface`](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-3-logger-interface.md)-compliant logger.

This handler implements the [`SessionHandlerInterface`](http://php.net/manual/en/class.sessionhandlerinterface.php) meaning that it can be registered as a session handler with [`session_set_save_handler`](http://php.net/manual/en/function.session-set-save-handler.php).

## Concurrency

As MongoDB prior to 3.0 does not support [document level
locking](http://docs.mongodb.org/manual/core/storage/#document-level-locking),
this session handler operates on a principle of Last Write Wins.

If a user of a session causes two simultaneous writes then you may end up with
the following situation:

1. Window A reads session value of `['foo' => 'bar']`;
2. Window B reads session value of `['foo' => 'bar']`;
3. Window B writes session value of `['foo' => 'baz']`;
4. Window A writes session value of `['foo' => 'quux']`.

The session will now contain `['foo' => 'quux']` as it was the last successful
write. This may be surprising if you're trying to increment some value in a
session as it is not locked during reads and writes:

1. Window A reads session value of `['count' => 0]`;
2. Window B reads session value of `['count' => 0]`;
3. Window B writes session value of `['count' => 1]`;
4. Window A writes session value of `['count' => 1]`.

## Acknowledgements

* [Nick Ilyin's
  `php-mongo-session`](https://github.com/nicktacular/php-mongo-session)
  served as a valuable existing implementation of MongoDB-backed sessions in
  PHP.

## License

Copyright © 2015-2016 Altmetric LLP

Distributed under the MIT License.
