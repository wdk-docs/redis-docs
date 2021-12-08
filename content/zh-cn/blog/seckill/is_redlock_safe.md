---
title: "Is Redlock safe?"
linkTitle: ""
date: "2016-02-10"
---

> http://antirez.com/news/101

Martin Kleppmann, a distributed systems researcher, yesterday published an analysis of Redlock (http://redis.io/topics/distlock), that you can find here: http://martin.kleppmann.com/2016/02/08/how-to-do-distributed-locking.html

Redlock is a client side distributed locking algorithm I designed to be used with Redis, but the algorithm orchestrates, client side, a set of nodes that implement a data store with certain capabilities, in order to create a multi-master fault tolerant, and hopefully safe, distributed lock with auto release capabilities.
You can implement Redlock using MySQL instead of Redis, for example.

The algorithm's goal was to move away people that were using a single Redis instance, or a master-slave setup with failover, in order to implement distributed locks, to something much more reliable and safe, but having a very low complexity and good performance.

Since I published Redlock people implemented it in multiple languages and used it for different purposes.

Martin's analysis of the algorithm concludes that Redlock is not safe. It is great that Martin published an analysis, I asked for an analysis in the original Redlock specification here: http://redis.io/topics/distlock. So thank you Martin. However I don’t agree with the analysis. The good thing is that distributed systems are, unlike other fields of programming, pretty mathematically exact, or they are not, so a given set of properties can be guaranteed by an algorithm or the algorithm may fail to guarantee them under certain assumptions. In this analysis I’ll analyze Martin's analysis so that other experts in the field can check the two documents (the analysis and the counter-analysis), and eventually we can understand if Redlock can be considered safe or not.

## Why Martin thinks Redlock is unsafe

The arguments in the analysis are mainly two:

1. Distributed locks with an auto-release feature (the mutually exclusive lock property is only valid for a fixed amount of time after the lock is acquired) require a way to avoid issues when clients use a lock after the expire time, violating the mutual exclusion while accessing a shared resource. Martin says that Redlock does not have such a mechanism.

2. Martin says the algorithm is, regardless of problem “1”, inherently unsafe since it makes assumptions about the system model that cannot be guaranteed in practical systems.

I’ll address the two concerns separately for clarity, starting with the first “1”.

## Distributed locks, auto release, and tokens

A distributed lock without an auto release mechanism, where the lock owner will hold it indefinitely, is basically useless. If the client holding the lock crashes and does not recover with full state in a short amount of time, a deadlock is created where the shared resource that the distributed lock tried to protect remains forever unaccessible. This creates a liveness issue that is unacceptable in most situations, so a sane distributed lock must be able to auto release itself.

So practical locks are provided to clients with a maximum time to live. After the expire time, the mutual exclusion guarantee, which is the _main_ property of the lock, is gone: another client may already have the lock. What happens if two clients acquire the lock at two different times, but the first one is so slow, because of GC pauses or other scheduling issues, that will try to do work in the context of the shared resource at the same time with second client that acquired the lock?

Martin says that this problem is avoided by having the distributed lock server to provide, with every lock, a token, which is, in his example, just a number that is guaranteed to always increment. The rationale for Martin's usage of a token, is that this way, when two different clients access the locked resource at the same time, we can use the token in the database write transaction (that is assumed to materialize the work the client does): only the client with the greatest lock number will be able to write to the database.

In Martin's words:

“The fix for this problem is actually pretty simple: you need to include a fencing token with every write request to the storage service. In this context, a fencing token is simply a number that increases (e.g. incremented by the lock service) every time a client acquires the lock”

… snip …

“Note this requires the storage server to take an active role in checking tokens, and rejecting any writes on which the token has gone backwards”.

I think this argument has a number of issues:

1. Most of the times when you need a distributed lock system that can guarantee mutual exclusivity, when this property is violated you already lost. Distributed locks are very useful exactly when we have no other control in the shared resource. In his analysis, Martin assumes that you always have some other way to avoid race conditions when the mutual exclusivity of the lock is violated. I think this is a very strange way to reason about distributed locks with strong guarantees, it is not clear why you would use a lock with strong properties at all if you can resolve races in a different way. Yet I’ll continue with the other points below just for the sake of showing that Redlock can work well in this, very artificial, context.

2. If your data store can always accept the write only if your token is greater than all the past tokens, than it’s a linearizable store. If you have a linearizable store, you can just generate an incremental ID for each Redlock acquired, so this would make Redlock equivalent to another distributed lock system that provides an incremental token ID with every new lock. However in the next point I’ll show how this is not needed.

3. However “2” is not a sensible choice anyway: most of the times the result of working to a shared resource is not writing to a linearizable store, so what to do? Each Redlock is associated with a large random token (which is generated in a way that collisions can be ignored. The Redlock specification assumes textually “20 bytes from /dev/urandom”). What do you do with a unique token? For example you can implement Check and Set. When starting to work with a shared resource, we set its state to “`<token>`”, then we operate the read-modify-write only if the token is still the same when we write.

4. Note that in certain use cases, one could say, it’s useful anyway to have ordered tokens. While it’s hard to think at an use case, note that for the same GC pauses Martin mentions, the order in which the token was acquired, does not necessarily respects the order in which the clients will attempt to work on the shared resource, so the lock order may not be casually related to the effects of working to a shared resource.

5. Most of the times, locks are used to access resources that are updated in a way that is non transactional. Sometimes we use distributed locks to move physical objects, for example. Or to interact with another external API, and so forth.

I want to mention again that, what is strange about all this, is that it is assumed that you always must have a way to handle the fact that mutual exclusion is violated. Actually if you have such a system to avoid problems during race conditions, you probably don’t need a distributed lock at all, or at least you don’t need a lock with strong guarantees, but just a weak lock to avoid, most of the times, concurrent accesses for performances reasons.

However even if you happen to agree with Martin about the fact the above is very useful, the bottom line is that a unique identifier for each lock can be used for the same goals, but is much more practical in terms of not requiring strong guarantees from the store.

## Let’s talk about system models

The above criticism is basically common to everything which is a distributed lock with auto release, not providing a monotonically increasing counter with each lock. However the other critique of Martin is specific to Redlock. Here Martin really analyzes the algorithm, concluding it is broken.

Redlock assumes a semi synchronous system model where different processes can count time at more or less the same “speed”. The different processes don’t need in any way to have a bound error in the absolute time. What they need to do is just, for example, to be able to count 5 seconds with a maximum of 10% error. So one counts actual 4.5 seconds, another 5.5 seconds, and we are fine.

Martin also states that Redlock requires bound messages maximum delays, which is not correct as far as I can tell (I’ll explain later what’s the problem with his reasoning).

So let’s start with the issue of different processes being unable to count time at the same rate.

Martin says that the clock can randomly jump in a system because of two issues:

1. The system administrator manually alters the clock.

2. The ntpd daemon changes the clock a lot because it receives an update.

The above two problems can be avoided by “1” not doing this (otherwise even corrupting a Raft log with “echo foo > /my/raft/log.bin” is a problem), and “2” using an ntpd that does not change the time by jumping directly, but by distributing the change over the course of a larger time span.

However I think Martin is right that Redis and Redlock implementations should switch to the monotonic time API provided by most operating systems in order to make the above issues less of a problem. This was proposed several times in the past, adds a bit of complexity inside Redis, but is a good idea: I’ll implement this in the next weeks. However while we’ll switch to monotonic time API, because there are advantages, processes running in an operating system without a software (time server) or human (system administrator) elements altering the clock, _can_ count relative time with a bound error even with gettimeofday().

Note that there are past attempts to implement distributed systems even assuming a bound absolute time error (by using GPS units). Redlock does not require anything like that, just the ability of different processes to count 10 seconds as 9.5 or 11.2 (+/- 2 seconds max in the example), for instance.

So is Redlock safe or not? It depends on the above. Let’s assume we use the monotonically increasing time API, for the sake of simplicity to rule out implementation details (system administrators with a love for POKE and time servers). Can a process count relative time with a fixed percentage of maximum error? I think this is a sounding YES, and is simpler to reply yes to this than to: “can a process write a log without corrupting it”?

## Network delays & co

Martin says that Redlock does not just depend on the fact that processes can count time at approximately the same time, he says:

“However, Redlock is not like this. Its safety depends on a lot of timing assumptions: it assumes that all Redis nodes hold keys for approximately the right length of time before expiring; that that the network delay is small compared to the expiry duration; and that process pauses are much shorter than the expiry duration.”

So let’s split the above claims into different parts:

1. Redis nodes hold keys for approximately the right length of time.

2. Network delays are small compared to the expiry duration.

3. Process pauses are much shorter than the expiry duration.

All the time Martin says that “the system clock jumps” I assume that we covered this by not poking with the system time in a way that is a problem for the algorithm, or for the sake of simplicity by using the monotonic time API. So:

About claim 1: This is not an issue, we assumed that we can count time approximately at the same speed, unless there is any actual argument against it.

About claim 2: Things are a bit more complex. Martin says:

“Okay, so maybe you think that a clock jump is unrealistic, because you’re very confident in having correctly configured NTP to only ever slew the clock.” (Yep we agree here ;-) he continues and says…)

“In that case, let’s look at an example of how a process pause may cause the algorithm to fail:
Client 1 requests lock on nodes A, B, C, D, E.
While the responses to client 1 are in flight, client 1 goes into stop-the-world GC.
Locks expire on all Redis nodes.
Client 2 acquires lock on nodes A, B, C, D, E.
Client 1 finishes GC, and receives the responses from Redis nodes indicating that it successfully acquired the lock (they were held in client 1’s kernel network buffers while the process was paused).
Clients 1 and 2 now both believe they hold the lock.”

If you read the Redlock specification, that I hadn't touched for months, you can see the steps to acquire the lock are:

1. Get the current time.

2. … All the steps needed to acquire the lock …

3. Get the current time, again.

4. Check if we are already out of time, or if we acquired the lock fast enough.

5. Do some work with your lock.

Note steps 1 and 3. Whatever delay happens in the network or in the processes involved, after acquiring the majority we _check again_ that we are not out of time. The delay can only happen after steps 3, resulting into the lock to be considered ok while actually expired, that is, we are back at the first problem Martin identified of distributed locks where the client fails to stop working to the shared resource before the lock validity expires. Let me tell again how this problem is common with _all the distributed locks implementations_, and how the token as a solution is both unrealistic and can be used with Redlock as well.

Note that whatever happens between 1 and 3, you can add the network delays you want, the lock will always be considered not valid if too much time elapsed, so Redlock looks completely immune from messages that have unbound delays between processes. It was designed with this goal in mind, and I cannot see how the above race condition could happen.

Yet Martin's blog post was also reviewed by multiple DS experts, so I’m not sure if I’m missing something here or simply the way Redlock works was overlooked simultaneously by many. I’ll be happy to receive some clarification about this.

The above also addresses “process pauses” concern number 3. Pauses during the process of acquiring the lock don’t have effects on the algorithm's correctness. They can however, affect the ability of a client to make work within the specified lock time to live, as with any other distributed lock with auto release, as already covered above.

## Digression about network delays

Just a quick note. In server-side implementations of a distributed lock with auto-release, the client may ask to acquire a lock, the server may allow the client to do so, but the process can stop into a GC pause or the network may be slow or whatever, so the client may receive the "OK, the lock is your" too late, when the lock is already expired. However you can do a lot to avoid your process sleeping for a long time, and you can't do much to avoid network delays, so the steps to check the time before/after the lock is acquired, to see how much time is left, should actually be common practice even when using other systems implementing locks with an expiry.

## Fsync or not?

At some point Martin talks about the fact that Redlock uses delayed restarts of nodes. This requires, again, the ability to be able to wait more or less a specified amount of time, as covered above. Useless to repeat the same things again.

However what is important about this is that, this step is optional. You could configure each Redis node to fsync at every operation, so that when the client receives the reply, it knows the lock was already persisted on disk. This is how most other systems providing strong guarantees work. The very interesting thing about Redlock is that you can opt-out any disk involvement at all by implementing delayed restarts. This means it’s possible to process hundreds of thousands locks per second with a few Redis instances, which is something impossible to obtain with other systems.

## GPS units versus the local computer clock

Returning to the system model, one thing that makes Redlock system model practical is that you can assume a process to never be partitioned with the system clock. Note that this is different compared to other semi synchronous models where GPS units are used, because there are two non obvious partitions that may happen in this case:

1. The GPS is partitioned away from the GPS network, so it can’t acquire a fix.

2. The process and the GPS are not able to exchange messages or there are delays in the messages exchanged.

The above problems may result into a liveness or safety violation depending on how the system is orchestrated (safety issues only happen if there is a design error, for example if the GPS updates the system time asynchronously so that, when the GPS does not work, the absolute time error may go over the maximum bound).

The Redlock system model does not have these complexities nor requires additional hardware, just the computer clock, and even a very cheap clock with all the obvious biases due to the crystal temperature and other things influencing the precision.

## Conclusions

I think Martin has a point about the monotonic API, Redis and Redlock implementations should use it to avoid issues due to the system clock being altered. However I can’t identify other points of the analysis affecting Redlock safety, as explained above, nor do I find his final conclusions that people should not use Redlock when the mutual exclusion guarantee is needed, justified.

It would be great to both receive more feedbacks from experts and to test the algorithm with Jepsen, or similar tools, to accumulate more data.

A big thank you to the friends of mine that helped me to review this post.
blog comments powered by Disqus
