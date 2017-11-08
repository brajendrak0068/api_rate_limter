Design Goals and Decisions

I had to take a few important things into account when building this application:

1. Response time should be as fast as possible

I chose an in-memory database, Redis, to ensure faster I/O.
I minimized number of trips to Redis using a single pipelined call and getting increment value with the same call. So there is only one trip made to Redis for each call to the rate limiter.
2. The amount of data that is stored should be minimum

This was especially important considering we're using an in-memory database.
When a request is sent to rate limiter for the first time, it will create a key for it with an expiration time. Once the key expires, there won't be any key stored for that user, unless she makes another call.
3. Request data should be saved atomically and be persistent

Redis provides different ranges of persistence options.
I pipelined calls to Redis using multi/exec in order to execute only one atomic transaction per each call.
4. Be able to live in a distributed environment

Decision above (#3) also influenced this one.
I avoided race conditions by using atomic increment operations which also return the result of the increment after the same call.
Redis supports partioning.
There can be different instances of the RateLimiter object running, but they will all have access to the same traffic info via Redis.
5. Number and type of limits should be flexible

Users can create limits as they wish providing different time ranges and maximum calls.
So there is no fixed hierarchy: Limits per X seconds, minutes, hours, days, all are possible.
