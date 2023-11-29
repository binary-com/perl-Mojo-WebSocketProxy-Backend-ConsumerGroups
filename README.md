# NAME

Mojo::WebSocketProxy::Backend::ConsumerGroups - Class for communication with backend by sending messaging through redis streams.

# VERSION

version 0.05

# DESCRIPTION

Class for communication with backend by sending messaging through redis streams.

- `Redis streams` is used as channel for sending request to backend servers.
- `Redis subscriptions` is used as channel for receiving responses from backend servers.

# NAME

Mojo::WebSocketProxy::Backend::ConsumerGroup

# METHODS

## new

Creates object instance of the class

- `redis_uri` - URI for Redis connection. Ignored if the `redis` argument is also given.
- `redis` - Redis client object (must be compatible with [Mojo::Redis2](https://metacpan.org/pod/Mojo%3A%3ARedis2)). This argument will override the `redis_uri` argument.
- `timeout` - Request timeout, in seconds. If not set, uses the environment variable `RPC_QUEUE_RESPONSE_TIMEOUT`, or defaults to 30
- `queue_separation_enabled` - Boolean to specify if messages should be assigned to different queus based on their `category` or only `general` queue.
- `category_timeout_config` - A hash containing the timeout value for each request category.

        { general => 5, other => 120 }

## loop

## pending\_requests

Returns `hashref` which is used as a storage for keeping requests which were sent.
Stucture of the hash should be like:

- `key` - request id, which we'll get from redis after successful adding request to the stream
- `value` - future object, which will be done in case of getting response, of cancelled in case of timeout

## redis

## timeout

## category\_timeout\_config

Hash containing the timeout value for each rpc call category.

    { general => 5, other => 120 }

## queue\_separation\_enabled

Boolean specifying if category separation should be enabled.

## whoami

Return unique ID of Redis which will be used by backend server to send response.
Id is persistent for the object.

## call\_rpc

Makes a remote call to a process  returning the result to the client in JSON format.
Before, After and error actions can be specified using call backs.
It takes the following arguments

- `$c`  : [Mojolicious::Controller](https://metacpan.org/pod/Mojolicious%3A%3AController)
- `$req_storage` A hashref of attributes stored with the request.  This routine uses some of the following named arguments:
    - `method` The name of the method at the remote end.
    - `msg_type` a name for this method; if not supplied `method` is used.
    - `call_params` a hashref of arguments on top of `req_storage` to send to remote method. This will be suplemented with `$req_storage->{args}`
    added as an `args` key and be merged with `$req_storage->{stash_params}` with stash\_params overwriting any matching
    keys in `call_params`.
    - `rpc_response_callback`  If supplied this will be run with args: `Mojolicious::Controller` instance, the rpc\_response and `$req_storage`.
    **Note:** if `rpc_response_callback` is supplied the success and error callbacks are not used.
    - `before_get_rpc_response`  arrayref of subroutines to be run before the remote response is received, with args: `$c` and `req_storage`
    - `after_get_rpc_response` arrayref of subroutines to be run after the remote response is received; called with args: `$c` and `req_storage`
    called only when there is an response from the remote call.
    - `before_call` arrayref of subroutines called before the request to the remote service is made.
    - `rpc_failure_cb` a subroutine reference to call if the remote call fails. Called with `Mojolicious::Controller`, the rpc\_response and `$req_storage`
    - `category` - if supplied, the message will be assigned to the Redis channel with the corresponding name. The _general_ channel will be used by default if either `$msg_type` is not provided or `queue_separation_enabled` is 0.

Returns undef.

## request

Sends request to backend service. The method accepts following arguments:

- `request_data` - an `arrayref` containing data for the item which is going to be put into redis stream.
- `$category_name` - this will be passed to `_send_request` to specify which redis category this message belongs to.
- `category_timeout` - timeout value for this specific call (differs based on category)

Returns future.
Which will be marked as done in case getting response from backend server.
And it'll be marked as failed in case of request timeout or in case of error putting request to redis stream.

## wait\_for\_messages

By using redis subscription, we subscribe on channel for receiving responses from backend server.
We'll use uniq id generated by [whoami](https://metacpan.org/pod/whoami) as subscription channel.
Subscription will be done only once within first request to backend server.

# SEE ALSO

[Mojolicious::Plugin::WebSocketProxy](https://metacpan.org/pod/Mojolicious%3A%3APlugin%3A%3AWebSocketProxy),
[Mojo::WebSocketProxy](https://metacpan.org/pod/Mojo%3A%3AWebSocketProxy)
[Mojo::WebSocketProxy::Backend](https://metacpan.org/pod/Mojo%3A%3AWebSocketProxy%3A%3ABackend),
[Mojo::WebSocketProxy::Dispatcher](https://metacpan.org/pod/Mojo%3A%3AWebSocketProxy%3A%3ADispatcher),
[Mojo::WebSocketProxy::Config](https://metacpan.org/pod/Mojo%3A%3AWebSocketProxy%3A%3AConfig)
[Mojo::WebSocketProxy::Parser](https://metacpan.org/pod/Mojo%3A%3AWebSocketProxy%3A%3AParser)

# COPYRIGHT AND LICENSE

Copyright (C) 2022 deriv.com

# AUTHOR

DERIV <DERIV@cpan.org>

# COPYRIGHT AND LICENSE

This software is copyright (c) 2021 by Deriv Services Ltd.

This is free software; you can redistribute it and/or modify it under
the same terms as the Perl 5 programming language system itself.
