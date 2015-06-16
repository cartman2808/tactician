---
layout: default
permalink: /plugins/locking-middleware/
title: Locking Middleware
---

# Locking Middleware

[![Author](http://img.shields.io/badge/author-@rosstuck-blue.svg?style=flat-square)](https://twitter.com/rosstuck)
[![Source](http://img.shields.io/badge/source-league/tactician-blue.svg?style=flat-square)](https://github.com/thephpleague/tactician)
[![Packagist](http://img.shields.io/packagist/v/league/tactician.svg?style=flat-square)](https://packagist.org/packages/league/tactician)

This plugin actually ships in the core Tactician "CommandBus" package, so it's available without installing any separate composer packages.

The LockingMiddleware blocks any Commands from running inside Commands. If a Command is already being executed and another Command comes in, this plugin will queue it in-memory until the first Command completes.

It's very simple to setup:

~~~ php
use League\Tactician\Plugins\LockingMiddleware;
use League\Tactician\CommandBus;

$lockingMiddleware = new LockingMiddleware();

$commandBus = new CommandBus(
    [
        $lockingMiddleware,
        // ... your other middleware...
    ]
);
~~~

The LockingMiddleware is very useful for controlling your transactional boundaries. Depending on where you stack it with your other middleware, you can decide when they trigger: when the command was added or when it's actually executed. Both are useful.

For example, imagine this setup:

~~~ php
new CommandBus(
    [
        $loggingMiddleware,
        $lockingMiddleware,
        $transactionMiddleware,
        $commandHandlerMiddleware
    ]
);
~~~

By placing the TransactionMiddleware after the Locking, we've ensured that each Command executes in a completely separate database transaction. However, if we wanted all commands to execute inside one big transaction, we could just move the TransactionMiddleware above the Locking.

Likewise, by putting the Logging _before_ the Locking, we get the log entry at the initial point where the command is first added. This might be better for our debugging purposes then logging it at execution when several other commands have already run and presumably written to the log.

However, the important point is that by making the Locking behavior a separate decorator, you can tweak this behavior in anyway you want.