---
title: Go
tags:
categories:
date: 2023-01-03
lastMod: 2023-01-03
---
Resources

[Go Internals](https://cmc.gitbook.io/go-internals/)

[Go Internals Resources](https://github.com/emluque/golang-internals-resources)

[Pointers](Pointer)

Interesting Links

[Type-Unsafe Pointers](https://go101.org/article/unsafe.html)

Concurrency

Goroutines

Can be in one of 3 states:

Waiting - stopped and waiting for something to continue

Runnable - The goroutine wants time on a thread (M)

Executing - The goroutine has been placed on a thread (M) and is executing its instructions

Channels

Goroutines: Execute tasks independently

Channels: Communicate and synchronize goroutines

Buffered Channels: `ch := make(chan int, 3)`

Unbuffered Channels: `ch := make(chan int)`

Creating a channel allocates a `hchan` struct on the Heap and returns a pointer to it (a channel is a pointer under the hood):


```plain text

type hchan struct {
	qcount   uint           // total data in the queue
	dataqsiz uint           // size of the circular queue
	buf      unsafe.Pointer // points to an array of dataqsiz elements
	elemsize uint16
	closed   uint32
	elemtype *_type // element type
	sendx    uint   // send index
	recvx    uint   // receive index
	recvq    waitq  // list of recv waiters
	sendq    waitq  // list of send waiters

	// lock protects all fields in hchan, as well as several
	// fields in sudogs blocked on this channel.
	//
	// Do not change another G's status while holding this lock
	// (in particular, do not ready a G), as this can deadlock
	// with stack shrinking.
	lock mutex
}
```

`buf` is a Circular Queue (a queue that wraps around itself). The send and receive positions are tracked using `sendx` (index of the send) and `recvx` (index of the receive)

When sending messages using a channel, the `hchan`...

Acquires a lock: Locks the `mutex`

Enqueues the message on `buf`

This uses `typedmemmove` to memory copy the message into the particular slot in the buffer

Releases the lock

a message is enqueued until the the `buf` is full. `sendx` and `recvx` are both still at 0. When the receiver receives a message, `recvx` is incremented.

When receiving messages using a channel, the `hchan`...

Acquires a lock: Locks the `mutex`

Dequeues the message in the `buf`, using `typedmemmove` to memcpy the item in the buffer slot into the variable used for the receive


```plain text
t := <-ch // memcpy the item from the hchan into t
```

Releases the lock

What happens when sending on a channel where the queue is full?

The senders execution is paused and is resumed once there is space in the queue again

Goroutines are userspace threads which means they are managed by the Go runtime, not the operating system

Userspace threads are more lightweight compared to OS threads

Userspace threads have to actually run on OS threads. The Go scheduler is responsible for managing this and uses an [M-N Scheduling Model](https://en.wikipedia.org/wiki/Thread_(computing)#Threading_models) in order to do so

You have N number of Goroutines and M number of OS threads - the scheduler multiplexes the Goroutines onto the OS threads

Go uses three structures for scheduling

[M: OS thread](https://github.com/golang/go/blob/5bd734839d9967f48184431b978c2dabb39c8953/src/runtime/runtime2.go#L513)

[G: goroutine](https://github.com/golang/go/blob/5bd734839d9967f48184431b978c2dabb39c8953/src/runtime/runtime2.go#L405)

[P: context for scheduling](https://github.com/golang/go/blob/5bd734839d9967f48184431b978c2dabb39c8953/src/runtime/runtime2.go#L606)

There are a fixed number of P's and each of them hold a list of goroutines that are ready to run. These are called `runqueues`.

**In order to run a goroutine (G) a thread (M) must hold a context (P)**

The channel send on a full channel calls into the scheduler with [gopark](https://github.com/golang/go/blob/5bd734839d9967f48184431b978c2dabb39c8953/src/runtime/proc.go#L349). 

gopark changes the status of the goroutine calling into the scheduler from "running" to "waiting" and removes the association between the goroutine and the OS thread (**M**) that it is running on. This frees the OS thread to run a different goroutine.

The scheduler pops a **G** from the `runqueue` and schedules it to run on the OS thread (**M**).

This means that **G** can be blocked but the **M** is not which is great for performance

Resuming a blocked goroutine

Sender

The goroutine sets up state for resumption before calling into the scheduler

The `hchan` struct stores waiting senders (`sendq`) and waiting receivers (`recvq`). Each of these queues is a [sudog](https://github.com/golang/go/blob/5bd734839d9967f48184431b978c2dabb39c8953/src/runtime/runtime2.go#L346), which contains information about the waiting goroutine.

A pointer to the waiting goroutine

A pointer to the element (`elem`) to send or receive

The goroutine creates a `sudog` and puts it on the `sendq` in the `hchan`. This sets up the state for a receiver in the future to use that information to resume the goroutine. 

Receiver

The `hchan` [dequeues](https://github.com/golang/go/blob/5bd734839d9967f48184431b978c2dabb39c8953/src/runtime/chan.go#L765) the element from the buffer

It pops off the waiting sender (`sudog`)

The `sudog` [enques](https://github.com/golang/go/blob/5bd734839d9967f48184431b978c2dabb39c8953/src/runtime/chan.go#L751) the message into the waiting buffer

Sets the goroutine to "runnable"

Calls [goready](https://github.com/golang/go/blob/5bd734839d9967f48184431b978c2dabb39c8953/src/runtime/proc.go#L375) to [set the goroutine as runnable](https://github.com/golang/go/blob/5bd734839d9967f48184431b978c2dabb39c8953/src/runtime/proc.go#L846)

Returns to the receiver

What happens when the receiver comes first (empty channel)?

The goroutines execution is paused and will be resumed after a send

It creates a `sudog` and sets up its state for resumption

It calls into the scheduler (`gopark`)

When the sender sends a message on the channel, there are 2 possibilities:

It could enqueue the message in the buffer and call `goready` on the receiver

We know [the memory location](https://github.com/golang/go/blob/5bd734839d9967f48184431b978c2dabb39c8953/src/runtime/runtime2.go#L355) for the receive so the sender could just write to the pointer directly

Each goroutine has [its own stack](https://github.com/golang/go/blob/5bd734839d9967f48184431b978c2dabb39c8953/src/runtime/runtime2.go#L413)

Goroutines never write to another goroutines stack, __except__ for in this scenario

This means that the receiver does not need to acquire a channel lock and manipulate the buffer because it already has the value

Unbuffered channels __always__ work like this

**Interesting functions / types**

[hchan](https://github.com/golang/go/blob/5bd734839d9967f48184431b978c2dabb39c8953/src/runtime/chan.go#L33)

[acquireSudog](https://github.com/golang/go/blob/5bd734839d9967f48184431b978c2dabb39c8953/src/runtime/proc.go#L382)

[chansend](https://github.com/golang/go/blob/5bd734839d9967f48184431b978c2dabb39c8953/src/runtime/chan.go#L159)

[typedmemmove](https://github.com/golang/go/blob/5bd734839d9967f48184431b978c2dabb39c8953/src/runtime/mbarrier.go#L157)

**Related Papers**

[Real-Time Garbage Collection on General-Purpose Machines](http://www.yuasa.kuis.kyoto-u.ac.jp/~yuasa/YuasasSnapshot.pdf)

[Write Barrier Elision for Concurrent Garbage Collectors](https://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.146.2784&rep=rep1&type=pdf)

This [article](https://www.sobyte.net/post/2021-12/golang-garbage-collector/) provides a high level overview

Scheduler


```plain text
// Goroutine scheduler
// The scheduler's job is to distribute ready-to-run goroutines over worker threads.
//
// The main concepts are:
// G - goroutine.
// M - worker thread, or machine.
// P - processor, a resource that is required to execute Go code.
//     M must have an associated P to execute Go code, however it can be
//     blocked or in a syscall w/o an associated P.
//
// Design doc at https://golang.org/s/go11sched.

```

[link](https://github.com/golang/go/blob/5bd734839d9967f48184431b978c2dabb39c8953/src/runtime/proc.go#L19)

The scheduler is described in 

Runs goroutines that are created

Pauses and resumes goroutines - blocking channel operations, mutex operations, etc

Coordinates blocking system calls, network I/O, runtime tasks like garbage collection, etc

Why have a scheduler?

It is needed because Go uses goroutines which are userspace threads

These are managed entirely by the Go runtime

Cheaper and lighter-weight than kernel threads

Have a smaller memory footprint

default goroutine stack = 2KB

default thread stack = 8KB (state tracking overhead)

Faster creation, destruction, and context switches

Goroutine switches = ~10ns

Thread switches = ~1µs

The OS only knows how to schedule OS threads that run on a particular CPU core

The Go scheduler creates goroutines which run on OS threads which run on the CPU

When does it schedule?

Anytime our program does something that does (or should) affect goroutine execution

Creation

Blocking / Pauses

Blocking System Calls - **this causes the underlying OS thread to block as well**

Goals

Use a small number of kernel threads

Support high concurrency

Leverage hardware parallelism. i.e. scale to N cores

On an N-core machine, Go programs should be able to run N goroutines in parallel

Create threads when needed and keep them around for reuse.

This is what `gopark` is - these are threads that are not being used yet and are "parked" waiting to be used

The Go scheduler uses distributed runqueues that are described in [this design doc](https://docs.google.com/document/d/1TTj4T2JO42uD5ID9e89oa0sLKhJYD0Y_kqxDv3I3XMw/edit)

Each thread gets its own runqueue and the Go scheduler can use [Work Stealing](http://supertech.csail.mit.edu/papers/steal.pdf) or a [handoff](https://medium.com/aosd-reading-assignment/scheduling-algorithms-for-shared-memory-multi-processor-systems-9c62208ce10c) to move goroutines from one `runqueue` on one processor to a `runqueue` on a different processor 

**The maximum number of runqueues is the number of CPU cores** (`GOMAXPROCS`)

Threads in syscalls won't count toward this limit

The limit applies to threads that are running goroutines

The threads get goroutines to run from a `runqueue`

We create a thread when all current threads are busy or we don't have one yet

Goroutines that are added to the `runqueue` will run at a future scheduling point (other goroutines hit a blocking call)

The Go scheduler implements preemption using a background thread called the [sysmon](https://medium.com/@blanchon.vincent/go-sysmon-runtime-monitoring-cff9395060b5) (also see [Go: Asynchronous Preemption](https://medium.com/@blanchon.vincent/go-asynchronous-preemption-b5194227371c))

Any goroutine running for more than 10ms will try to be preempted to leave running time to other goroutines

Once a goroutine meets this condition, a signal is [emitted](https://github.com/golang/go/blob/5bd734839d9967f48184431b978c2dabb39c8953/src/runtime/preempt.go#L207) to the [current thread](https://github.com/golang/go/blob/5bd734839d9967f48184431b978c2dabb39c8953/src/runtime/runtime2.go#L520) to initiate preemption (see [this article](https://medium.com/a-journey-with-go/go-gsignal-master-of-signals-329f7ff39391) on `gsignal`)

Once received, Go will push an instruction in the program counter which looks just like the goroutine called a function in the runtime. The function parks the goroutine, hands it to the scheduler, and the scheduler puts it on a __global__ `runqueue`

This is a low priority queue that is checked less frequently

Limitations

FIFO runqueues means that there is no notion of __priority__ in the queue

The scheduler is not aware of hardware topology which means that there is no real __locality__ 

This is solved by making the scheduler [NUMA Aware](https://docs.google.com/document/u/0/d/1d3iI2QWURgDIsSR6G2275vMeQ_X7w-qxM2Vp7iGwwuM/pub)

The GC marks live/reachable objects and cleans up dead/unreachable objects

Each Go process is allocated some virtual memory by the OS which is the total amount of memory that it has access to

The actual memory used within the virtual memory is called the Resident Set

Go stores dynamic data (size not known at compile time) in the Page Heap (`mheap`)

This is the biggest block of memory and where GC takes place

The Resident Set is divided into pages of 8KB each and is managed by a single, global `mheap`

Large Objects (>32KB) are allocated __directly__ from `mheap` which come at the expense of a central lock so only 1 P (processor) request at a time can be received

[mheap](https://github.com/golang/go/blob/5bd734839d9967f48184431b978c2dabb39c8953/src/runtime/mheap.go#L62) manages pages grouped into several different constructs

[mspan](https://github.com/golang/go/blob/5bd734839d9967f48184431b978c2dabb39c8953/src/runtime/mheap.go#L62): A [Doubly Linked List](Linked List) that holds the address of the start page, [span size class](https://github.com/golang/go/blob/5bd734839d9967f48184431b978c2dabb39c8953/src/runtime/mheap.go#L528), and the number of pages in a span. It manages the pages of memory in the `mheap`


```plain text
type mspan struct {
	next *mspan     // next span in list, or nil if none
	prev *mspan     // previous span in list, or nil if none
	startAddr uintptr // address of first byte of span aka s.base()
	npages    uintptr // number of pages in span
```

Go divides memory pages into a block of 67 different classes by size, starting at 8 bytes up to 32KB

Each span exists twice, one for objects with pointers (`scan` classes) and one for objects with no pointers (`noscan`)

[mcentral](https://github.com/golang/go/blob/5bd734839d9967f48184431b978c2dabb39c8953/src/runtime/mcentral.go#L20) groups together spans that have the same size

Each `mcentral` contains two [mspanSets](https://github.com/golang/go/blob/5bd734839d9967f48184431b978c2dabb39c8953/src/runtime/mspanset.go#L17)

empty: [Doubly Linked List](Linked List) of spans with no free objects or spans that are cached in a [mcache](https://github.com/golang/go/blob/5bd734839d9967f48184431b978c2dabb39c8953/src/runtime/mcache.go#L20). When a span is freed here it is moved to the nonempty list

non-empty: [Doubly Linked List](Linked List) of spans with a free object. When `mcentral` requests a new span it is taken from the non-empty list and  moved to the empty list.

When `mcentral` doesn't have any free span, it requests a new run of pages from `mheap`

[arena](https://github.com/golang/go/blob/5bd734839d9967f48184431b978c2dabb39c8953/src/runtime/mheap.go#L226): Heap memory grows and shrinks within the allotted virtual memory. When more memory is needed, `mheap` pulls them from the virtual memory as a chunk of 64MB (for 64-bit arch) that is called `arena`. Pages are mapped to spans here.

[mcache](https://github.com/golang/go/blob/5bd734839d9967f48184431b978c2dabb39c8953/src/runtime/mcache.go#L20): A cache of memory provided to a P (Logical Processor) to store small objects (<=32KB) which are used for dynamic data. `mcache` contains `scan` and `noscan` types of `mcache` for all class sizes. Goroutines can obtain memory from `mcache` without locks because a P can only have 1 G at a time. `mcache` requests new spans from `mcentral` as required.

Stack

There is one stack per G which is where static data is stored. This is not the same as `mcache` which is assigned to a P


```plain text
// Stack describes a Go execution stack.
// The bounds of the stack are exactly [lo, hi),
// with no implicit data structures on either side.
type stack struct {
	lo uintptr
	hi uintptr
}

type g struct {
	// Stack parameters.
	// stack describes the actual stack memory: [stack.lo, stack.hi).
	// stackguard0 is the stack pointer compared in the Go stack growth prologue.
	// It is stack.lo+StackGuard normally, but can be StackPreempt to trigger a preemption.
	// stackguard1 is the stack pointer compared in the C stack growth prologue.
	// It is stack.lo+StackGuard on g0 and gsignal stacks.
	// It is ~0 on other goroutine stacks, to trigger a call to morestackc (and crash).
	stack       stack   // offset known to runtime/cgo
	stackguard0 uintptr // offset known to liblink
	stackguard1 uintptr // offset known to liblink

```

Stack vs Heap

Stack

The stack is a LIFO [Data Structure](Data Structures) that is very fast due to the fact that there isn't a lookup required to access an item - you just pop it off the top

The data stored on the stack has to be of a fixed size and static (the data is known at compile time)

The data used for executing functions is stored on the stack as a Stack Frame

Every time a variable is declared in a function, it is pushed onto the topmost block in the stack

When the function exits, the topmost block is cleared

Multithreaded applications can have a stack per Thread

The memory on the stack is managed by the OS

Data stored on the stack include local variables, pointers, and function frames

Stack Overflow errors occur when the memory required for function execution or local variables exceeds the size of the stack (stack memory is limited)

Heap

The heap is used for __dynamic memory__

Unlike a Stack, the program needs to lookup data in the heap using [pointers](Pointer)  which makes it slower than using the stack

The only memory limit of data stored on the stack is the amount of memory given to the application in total

The Go compiler uses [Escape Analysis](https://www.ardanlabs.com/blog/2017/05/language-mechanics-on-escape-analysis.html) to find objects whose lifetime is known at compile time and allocates them on the stack rather than in GC'd heap memory

[This slide deck](https://speakerdeck.com/deepu105/golang-memory-usage-stack-vs-heap) helps visualize how the stack and heap are managed

Memory Allocation

Tiny (<16B): Allocated using the `mcache` [tiny allocator](https://github.com/golang/go/blob/5bd734839d9967f48184431b978c2dabb39c8953/src/runtime/mcache.go#L38). This is super efficient and multiple tiny allocations are done on a single 16-byte block

Small (16B-32KB): Allocated on the corresponding size class (`mspan`) on the `mcache` of the P where the G is running

In both Tiny and Small allocations, the allocator will obtain a run of pages from the `mheap` if the `mspan` list is empty which are used for the `mspan`. If the `mheap` is empty or doesn't have pages large enough then it allocates new group of pages (>=1MB) from the OS

Large (>32KB): Allocated directly on the corresponding size class of `mheap`

How is garbage collected

If an object has no references from the stack directly or indirectly (via a reference in another object) then Go frees the memory used by these orphaned objects

Go uses a [Tri-Color Mark and Sweep Collector](https://pusher.github.io/tricolor-gc-visualization/)

The process begins when a certain percentage (`GOGC`) of heap allocations are done

**Mark Setup** (Stop the World): The GC turns on [write barriers](https://www.memorymanagement.org/glossary/w.html#term-write-barrier) to maintain data integrity

**Marking** (Concurrent): Started in parallel to the application and uses 25% of available CPU. The corresponding Ps are reserved until marking is complete which is done using dedicated Goroutines. The GC marks objects that are alive in this stage. If this is taking longer, it will use a Goroutine from the active application to assist the process (Mark Assist).

**Mark Termination** (Stop the World): Every active Goroutine is paused, write barriers are turned off, and cleanup tasks are started. This is when the GC calculates the next GC goal and then the reserved Ps are released back to the application

**Sweeping** (Concurrent): Reclaims memory from from the heap that is not marked as alive. 

[This slideshow](https://speakerdeck.com/deepu105/go-gc-visualized) does a great job of showing how this process works

Objects are marked with colors to determine what action needs to be taken

White: Starting condition - all objects are white at first. These are possibly accessible from the roots and are candidates for collection.

Grey: Might have pointers to the white set

Black: Definitely no pointers to any objects in the white set

[Composition with Go](https://www.ardanlabs.com/blog/2015/09/composition-with-go.html)

[Loose Coupling](https://8thlight.com/blog/javier-saldana/2015/02/06/loose-coupling-in-go-lang.html)

[How to use interfaces](https://jordanorelli.com/post/32665860244/how-to-use-interfaces-in-go)

[Interfaces in Go](https://research.swtch.com/interfaces)

[Functional Iteration in Go](https://hackthology.com/functional-iteration-in-go.html)

[Go Memory Model](https://go.dev/ref/mem)

See [this series of articles](https://research.swtch.com/mm) by Russ Cox on Memory Models

Actually, like [all](https://research.swtch.com/) of the articles by Russ Cox are probably worth reading

Refers to the guarantees that Go gives you when accessing memory from shared goroutines

Reordering

Compiler and Processor can reorder ([Loop Fission / Fusion](https://en.wikipedia.org/wiki/Loop_fission_and_fusion)) looping mechanisms for performance. In the example below, the compiler can decide that updating a and b in separate loops is faster due to [data locality](https://en.wikipedia.org/wiki/Locality_of_reference)


```javascript
var a, b [100]int
for i := 0; i < 100; i++ {
  a[i] = 1
  b[i] = 2
}
```

is equivalent to


```javascript
var a, b [100]int
for i := 0; i < 100; i++ {
  a[i] = 1
}
for i := 0; i < 100; i++ {
  b[i] = 2
}
```

Memory Visibility

![](.//assets/ahr0chm6ly9maxjlymfzzxn0b3jhz2uuz29vz2xlyxbpcy5jb20vdjavyi9maxjlc2nyaxb0ltu3n2eylmfwchnwb3quy29tl28vaw1ncyuyrmfwccuyrmruanalmkyxdwdiuufsquqxlnbuzz9hbhq9bwvkawemdg9rzw49m2rhyzm0n2etnznjmy00ytlmltgzmwytowq5ztkwmzi4mte5.png)

[source](https://www.youtube.com/watch?v=NzhH0p32fMY)

Consider as an example that there is a thread running in Core 1 that updates a variable and Core 3 wants to access that variable. Core 1 will be updating that variable in its Registers. If Core 3 wants to access that variable, it likely will only be able to get stale values that have been written to one of Core 1's caches. This is because it would be too slow to update a variable in RAM every single time, so the program will just do it in the register.

Registers are extremely fast but hold only a small amount of data. As you progress to L1-L3 caches, they can hold more and more data but data access is slower. Writing to DRAM is the slowest of the options in memory.

"The Go memory model specifies the conditions under which reads of a variable in one goroutine can be guaranteed to observe values produced by writes to the same variable in a different goroutine." - [Go Memory Model](https://go.dev/ref/mem)

"Don't communicate by sharing memory; share memory by communicating"

Within a single goroutine

Reads and writes must behave as if they executed in the order specified by the program

Compiler can reorder as long as it doesn't change the behavior defined in the language specification

Order observed by one goroutine may differ from order perceived by another

[Happens-Before](https://en.wikipedia.org/wiki/Happened-before)

Partial ordering of operations

a -> b

Happens-Before is transitive

Within a single goroutine, the happens-before order is the order expressed by the program

**To guarantee that a read of a variable observes a write to the same variable, the write must happens-before the read**

Certain operations create happens-before relationships: locking

If two events don't have a happens before relationship, they are said to happen concurrently

When are actions visible to other goroutines

__Everything__ before unlocking a mutex is visible to another thread that accesses the mutex

What creates a happens-before relationship?

Goroutine Creation

The `go` statement that starts a new goroutine happens before the goroutine's execution begins


```plain text
var a string

func f() {
  print(a)
}

func hello() {
  a = "hello world"
  go f()
}
```

Channel Creation

A send on a channel happens before the corresponding receive from that channel completes


```plain text
var c = make(chan int, 10)
var a string

func f() {
  a = "hello world"
  c <- 0
}

func main() {
  go f()
  <-c
  print(a)
}
```

Channel Communication

A __receive__ from an __unbuffered channel__ happens __before the send__ on that channel completes


```plain text
var c = make(chan int)
var a string

func f() {
  a = "hello world"
  <-c
}

func main() {
  go f()
  c <- 0
  print(a)
}
```

Imagine a channel of size 3. Before you can send the 4th item on the channel, there must have been 1 receive. This receive happens before the send.


```plain text
var limit = make(chan int, 3)

func main() {
  for _, w := range work {
    go func(w func()) {
      limit <- 1
      w()
      <-limit
    }(w)
  }
  select {}
}
```

Once

A single call of f() from once.Do(f) happens (returns) before any call of once.Do(f) returns


```javascript
var a string
var once sync.Once

func setup() {
  a = "hello, world"
}

func main() {
  once.Do(setup)
  print(a)
}
```

Import initialization - i.e. an `init()` function

Functions in the sync package

WaitGroup

Atomics (not formally)

Example


```plain text
var a int
var done bool

func setup() {
  a = 42
  done = true
}

func main() {
  go setup()
  for !done {
  }
  fmt.Println(a)
}
```

This __does__ actually print 42 (go1.16), but it should not be depended upon
