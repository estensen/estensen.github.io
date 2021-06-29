---
title: "Rate Limiting"
date: 2021-06-28T11:47:14+02:00
draft: true
---

Sometimes clients use your API in a way you didn't intend. Maybe they send requests more often than you'd like and degrade the experience for other users. Rate-limiting is a way of limiting how many requests a user can create.

Using a Go middleware we can intercept requests and check if they are allowed before passing it on to the handlers.

## Time Bucketed

The simplest way rate rate limiting is to store remaining limit that will expire after a certain amount of time.

Some pseudocode:

```bash
LIMIT <- 100
PERIOD <- 1 hour

if UserID is not in Store
	Store.UserID.limit <- LIMIT
	Store.UserID.limit--
	Store.UserID.reset <- now + PERIOD
	process_request()
else
	if now > Store.UserID.reset
		Store.UserID.limit <- LIMIT - 1
		Store.UserID.reset <- now + PERIOD
	else
		Store.UserID.limit--
		if Store.UserID.limit = 0
			rate_limit()
```

Using a store with TTL would simplify the code.

The major downside of this approach is that a client can exhaust their rate limit very fast. In this example, the client can do a 100 requests and then they have to wait an hour to do another requests. We would like something that is more evenly spread out.

## Leaky Bucket

Leaky bucket is an algorithm that uses the analogy of how a bucket with a constant leak will overflow if the average rate of which water is dripping into the bucket is greater that what leaks out. This creates a smoothing effect allowing client request rate to be specified as an average rate and to be reset faster. The bucket also allows bursts. We can set a strict request rate like 5 reqs/min, but also allow for 3 requests arriving simultaneously.

```txt
user       *       drip
reqs               into
           *       bucket
           *
         |   |
burst    |   |     bucket size
         | * |
          \*/

           *       constant
limit              drip
           *       out
```
