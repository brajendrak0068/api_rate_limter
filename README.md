<h5>Before Starting</h5>

1.Please make sure you have Node.js installed on your development environment.

2.Please make sure you have a Redis server installed on your development environment or make sure you initialize the Redis client with appropriate host and port number later on when you run the application.

3.Once you clone this repository, run npm install on your command line to get all the dependencies.


<h5>Configurations and usage: </h5>

1.require module
	 var ratelimiter =  require('ratelimiter');

2. Create Rate limit configurations

	var limitsConfig = {
	    ECOM:{total: {week:[100, 604800], month:[600, 2628000], hour: [500, 3600], min:[20, 60]}, GET: { min: [20, 604800] }, '/status':{min: [20 ,60]} },
	    ABC: {total: {week:[10, 604800], hour:[5, 3600], min: [5, 60] }, POST: { week:[ 20, 604800 ]}, '/pay':{min:[ 30, 60] } }
	};

3. Create redis instance 

	var REDIS_PORT = 6379;
	var REDIS_HOST = "127.0.0.1";
	var redisClient = redis.createClient(REDIS_PORT, REDIS_HOST);

4. Finally pass the rate limits config and redisClient to rate limit middleware

       app.use(ratelimiter(limitsConfig, redisClient));



<h5> Design Decisions </h5> 

<h6>Following are the points which I have taken into considerations</h6>
1. Response time should very fast
  I chose an in-memory database, Redis, to ensure faster I/O.
  I minimized number of trips to Redis using a single pipelined call and getting increment value with the same call. So there is only one   trip made to Redis for each call to the rate limiter.

2. The amount of data that is stored should be minimum
  When a request is sent to rate limiter for the first time, it will create a key for it with an expiration time. Once the key expires,     there won't be any key stored for that user, unless she makes another call.

3. Request data should be saved atomically and be persistent
   I pipelined calls to Redis using multi/exec in order to execute only one atomic transaction per each call.

4. Be able to live in a distributed environment
     i)  I avoided race conditions by using atomic increment operations which also return the result of the increment after the same call.
    ii)  Redis supports partioning.
    iii) There can be different instances of the RateLimiter object running, but they will all have access to the same traffic info via Redis. 

5.Following the numbers and types of are covered    

