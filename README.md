Design Decisions

Following are the points which I have taken into considerations
1. Response time should very fast
  I chose an in-memory database, Redis, to ensure faster I/O.
  I minimized number of trips to Redis using a single pipelined call and getting increment value with the same call. So there is only one   trip made to Redis for each call to the rate limiter.

2. The amount of data that is stored should be minimum
  This was especially important considering we're using an in-memory database.
  When a request is sent to rate limiter for the first time, it will create a key for it with an expiration time. Once the key expires,     there won't be any key stored for that user, unless she makes another call.

3. Request data should be saved atomically and be persistent

  Redis provides different ranges of persistence options.
  I pipelined calls to Redis using multi/exec in order to execute only one atomic transaction per each call.

4. Be able to live in a distributed environment
  Decision above (#3) also influenced this one.
  I avoided race conditions by using atomic increment operations which also return the result of the increment after the same call.
  Redis supports partioning.
  There can be different instances of the RateLimiter object running, but they will all have access to the same traffic info via Redis.

5.Following the numbers and types of are covered    
  Config for individual client
	I:q: <CLIENT-ID>
	limit::Question:q!
		SEC -> int (Optional)
		MIN -> int (Optional)
		HOUR -> int (Optional)
		WEEK -> int (Optional)
		MONTH  -> int (Optional)
	specialization:
	    type: METHOD
			- <METHOD 1> : 
				limit:
					SEC -> int (Optional)
					MIN -> int (Optional)
					HOUR -> int (Optional)
					WEEK -> int (Optional)
					MONTH  -> int (Optional)
			- <METHOD 2> :
				limit:
					SEC -> int (Optional)
					MIN -> int (Optional)
					HOUR -> int (Optional)
					WEEK -> int (Optional)
					MONTH  -> int (Optional)
	    type: API
			- <API 1>:
				limit:
					SEC -> int (Optional)
					MIN -> int (Optional)
					HOUR -> int (Optional)
					WEEK -> int (Optional)
					MONTH  -> int (Optional)
			- <API 2>:
				limit:
					SEC -> int (Optional)
					MIN -> int (Optional)
					HOUR -> int (Optional)
					WEEK -> int (Optional)
					MONTH  -> int (Optional)


Exmaple:
client: ABC_ECOM
	limit:
		HOUR -> 100
		WEEK -> 900
		MONTH  -> 10000
	specialization:
	    type: METHOD
			- GET : 
				limit:
					SEC -> 10
					MIN -> 50
					WEEK -> 700
			- POST :
				limit:
					SEC -> 20
					HOUR -> 40
					WEEK -> 900
					MONTH -> 1000
	    type: API
			- /status :
				limit:
					SEC -> 20
					HOUR -> 40
					WEEK -> 900
					MONTH -> 1000
			- /pay :
				limit:
					SEC -> 10
					MIN -> 50
					WEEK -> 700
