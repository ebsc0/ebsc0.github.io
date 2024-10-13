---
layout: post
title: "Reference: Concurrency"
date: 2023-12-14 12:28:11 -0400
category: project
link: https://github.com/ebsc0/reference
---

A computer may have mutliple processes or threads executing concurrently (two events are concurrent if we cannot tell by looking at the program which will happen first). However, this may present a challenge when dealing with shared variables where two writers are writing to a variable or a reader and a writer access the same variable - The outcomes of every execution is non-deterministic.

To solve this issue, we use semaphores and mutex's to restrict access of a critical section to a number of processes or threads.

## Semaphores

A semaphore is an integer that can be incremented or decremented only:

1. When a thread decrements a semaphore, if the result is negative, the thread blocks itself and cannot continue until another thread increments the semaphore.
2. When a thread increments the semaphore, if there are other threads waiting, one of the waiting threads gets unblocked.

```c
#include <semaphore.h>
int sem_init(sem_t *semaphore, int shared, int init_value); /* shared = 0 (threads); shared = 1 (processes) */
int sem_destroy(sem_t *semaphore);
int sem_wait(sem_t *semaphore);
int sem_post(sem_t *semaphore);
```

## Mutex

A mutex is a binary semaphore that only allows one thread to be in the critical section at a time.

```c
#include <pthread.h>
pthread_mutex_t lock;
pthread_mutex_init(&lock, NULL); /* NULL mutex attribute */
pthread_mutex_lock(&lock);
pthread_mutex_unlock(&lock);
pthread_mutex_destroy(&lock);
```

## Synchronization Patterns

Using semaphores and mutex's, we can create common synchronization patterns to fit our application.

### Rendevouz

Multiple threads arrive at a point of execution and no thread is allowed to continue until all threads have arrived.

### Consumer-Producer Problem

Classic synchronization problem involving Producers producing data to a bounded-buffer, and Consumers consuming the data.

We must ensure that:

1. Only 1 Consumer or Producer can access the buffer at a time.
2. Consumers cannot consume when the buffer is empty.
3. Producers cannot produce when the buffer is full.

#### General Solution

```c
/* producer */
while(event == NULL) {;} /* wait for event */

sem_wait(&spaces); /* wait for spaces in buffer */
pthread_mutex_lock(&mutex); /* start critical section */
buffer_add(event); /* produce */
pthread_mutex_unlock(&mutex); /* end critical section */
sem_post(&items); /* signal there are items in buffer */
```

```c
/* consumer */
sem_wait(&items); /* wait for items to be in buffer */
pthread_mutex_lock(&mutex); /* start critical section */
event = buffer_get(); /* consume */
pthread_mutex_unlock(&mutex); /* end critical section */
sem_post(&spaces); /* signal there are spaces in buffer */

process(event); /* process event */
```

### Reader-Writer Problem

Classic synchronization problem involving Readers reading and Writers modifying data in a data structure.

We must ensure that:

1. any number of Readers can be in the critical section.
2. only 1 Writer can be in the critical section.
3. Readers and Writers cannot be in the critical section at the same time.

#### Basic Solution (Lightswitch)

This basic pattern is called the lightswitch by the analogy where the first person entering a room will turn on the lights and the last one to leave the room turns the lights off.

```c
/* writer */
sem_wait(&room_empty); /* wait for room to be empty */
write(); /* write */
sem_post(&room_empty); /* signal room is empty */
```

```c
/* reader */
/* entering threads */
pthread_mutex_lock(&mutex);
n_readers += 1; /* increment readers */
if(n_readers == 1) sem_wait(&room_empty); /* wait for room to be empty if first reader */
pthread_mutex_unlock(&mutex);

/* critical section */

/* exiting threads */
pthread_mutex_lock(&mutex);
n_readers -= 1; /* decrement readers */
if(n_readers == 0) sem_post(&room_empty); /* signal room is empty if last reader */
pthread_mutex_unlock(&mutex);
```

#### No-Starvation Solution (uses Turnstiles)

In the previous solution, deadlocks do not occur but writers can starve if readers come and go such that a reader arrives before the last reader leaves.

In order to solve this problem, we must use a turnstile to prevent new readers from entering the room once a writer has arrived in queue.

```c
/* writer */
sem_wait(&turnstile); /* acquire turnstile to reserve place in queue and prevent any readers from entering */
sem_wait(&room_empty); /* wait until room is empty */

... /* same critical section as before */

sem_post(&turnstile); /* release turnstile when done */
sem_post(&room_empty); /* signal room is empty */
```

```c
/* reader */
sem_wait(&turnstile); /* acquire turnstile */
sem_post(&turnstile); /* release turnstile */

... /* same reader solution as before */
```

## Deadlock

Deadlock occurs when a thread holding a resource prevents another thread from executing while the thread waits for the other thread to finish. This creates a cycle of dependencies that may result in the program halting execution.

### Dining Philosophers Problem

There are 5 philosophers and 5 chopsticks placed in alternating order around a circular table.

The following constraints must be met:

1. A chopstick cannot be shared.
2. Philosophers should not starve (i.e. no deadlock or starvation).
3. The maximum number of philosophers must be eating without causing deadlock.

The problem arises when all philosophers attempt to hold their chopstick on their left at the same time. Due to their only being 5 chopsticks while needing 2 in order to eat, the circular nature of the table forbids any 1 philosopher from acquiring 2 chopsticks causing deadlock.

#### Solution 1 (Bouncer)

We can enforce a number of philosophers at the table to be strictly less than the number of chopsticks. In this case, the bouncer only lets in 4 philosophers at the table. Because 1 chopstick remains, we are guaranteed to have at least 1 philosopher eating. When that philosopher is done eating, it will release its chopsticks and allow another philosopher to eat and so on...

#### Solution 2 (Asymmetric)

If one of the philosophers is a leftie, it will try to grab the left chopstick that another philosopher already has. Fortunately, the next philosopher can grab the lefties right chopstick and proceed to eat. Everyone will have a chance to eat as each philosopher will release their chopsticks once they are finished.

Through this analogy, we can design asymmetric processes that will requests resources in a way that does not create a deadlock.
