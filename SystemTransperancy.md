
## What to monitor

#### Memory
Minimum heap size, maximum heap size, generation sizes

#### Garbage collection
Type, frequency, memory reclaimed, size of request

#### Worker threads
Number of threads, threads busy, threads busy more than five
seconds, high-water mark (maximum concurrent threads in use),
low-water mark, number of times a thread was not available,
request backlog

#### Database connection pools
Number of connections, connections in use, high-water mark, low-
water mark, number of times a connection was not available,
request backlog

#### Traffic statistics, for each request channel
Total requests processed, average response time, requests
aborted, requests per second, time of last request, accepting traffic
or not
The application itself should reveal plenty of information about its own
metrics.

#### Business transaction, for each type
Number processed, number aborted, dollar value, transaction
aging, conversion rate, completion rate

#### Users
Demographics or classification, technographics, percentage of
users who are registered, number of users, usage patterns, errors
encountered

#### Integration points
Current state, manual override applied, number of times used,
average response time from remote system, number of failures

#### Circuit breakers
Current state, manual override applied, number of failed calls,
time of last successful call, number of state transitions

## What to expose

#### Traffic indicators
Page requests total, page requests, transaction counts, concurrent
sessions

#### Resource pool health
Enabled state, total resources, resources checked out, high-
water mark, number of resources created, number of resources
destroyed, number of times checked out, number of threads
blocked waiting for a resource, number of times a thread has
blocked waiting

#### Database connection health
Number of SQLException s thrown, number of queries, average
response time to queries

#### Integration point health
State of circuit breaker, number of timeouts, number of requests,
average response time, number of good responses, number of net-
work errors, number of protocol errors, number of application
errors, actual IP address of the remote endpoint, current number
of concurrent requests, concurrent request high-water mark

#### Cache health
Items in cache, memory used by cache, cache hit rate, items
flushed by garbage collector, configured upper limit, time spent
creating items