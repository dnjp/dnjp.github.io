---
title: C++
tags:
categories:
date: 2023-01-03
lastMod: 2023-01-03
---
Memory Management

Memory Types

Von Neumann vs Harvard Architecture

Von Neumann Architecture (VN)

John Von Neumann created this architecture

For VN Architecture you need:

a Store to hold your instructions and data (Memory)

a control unit (CPU)

arithmetic capability (ALU)

ALU is generally combined with CPU now

This setup takes input, puts it through the components described above, and

emits output

VN has linear flow of data

Input -> CPU -> Output

Harvard Architecture (H)

H is like VN except that it separates how data and instructions are stored in

memory

In H, there are different buses used for accessing data and instructions in

memory, because they are stored separately

The main advantage of doing this is that the CPU can fetch the data and

instructions at the same time instead of sequentially

The disadvantage of this is that it is more expensive because it requires

additional buses

Caching

Caches act as a buffer because the CPU is actually faster than RAM

L1 Cache

The L1 cache is normally included on the CPU

Data that needs to be accesses ASAP goes on the L1 cache

The L1 cache is the fastest and smallest memory type in the cache hierarchy

Typically the capacity is between 16 - 64 kBytes

In the Harvard architecture, instructions and data are separated on the

cache

This is denoted by L1i (instructions) and L1d (data)

L2 Cache

The L2 cache is located close to the CPU and has a direct connection to it

Information exchange is managed by the L2 controller on the main board

Size is normally <= 2mb

The L2 or L3 caches are slower than L1 and are used for information that

isn't needed as quickly as L1

L3 Cache

Shared between all cores of a multicore processor

[Cache Coherence protocol](https://en.wikipedia.org/wiki/Cache_coherence)

can work much faster

Locality

Temporal Locality

The address ranges that are accessed are likely to be used again in the near

future.

This can be used at all levels of the memory hierarchy to keep memory areas

accessible as quickly as possible

Spatial Locality

After an access to an address range, the next access to an address in the

immediate vicinity is highly probable

This is how arrays work

This can be exploited by moving the adjacent address areas upwards into the

next hierarchy level during memory access

Cache Aware Programming

C++ is uses

[Row-major](https://eli.thegreenplace.net/2015/memory-layout-of-multi-dimensional-arrays/)

memory layout for multi-dimensional arrays

Row Major

Data is layed out linearly. i.e


```c
int a[3][4];
a = [a00, a01, a02, a03, a10, a11, a12, a13, a20, a21, a22, a23]
```

Virtual Memory

When the available physical memory is below the upper bound imposed by the

architecture, virtual memory is used.

With virtual memory, a mapping is performed between the virtual address space

a proram sees and the physical addresses of various storage devices. This can

be RAM but also the hard disk.

This allows the OS to use physical memory for the parts of a process that are

currently being used and backup the rest of the virtual memory to a secondary

storage location.

With virtual memory, RAM is no longer the limit.

Memory Page: is a number of directly successive memory locations in virtual

memory defined by the computer architecture and by the operating system.The

computer memory is divided into memory pages of equal size. The use of memory

pages enables the operating system to perform virtual memory management. The

entire working memory is divided into tiles and each address in this computer

architecture is interpreted by the Memory Management Unit (MMU) as a logical

address and converted into a physical address.

A typical page size if 4kb

Kind of like a block of memory

If two processes need to share memory, the pages are mapped to the same

frame

Memory Frame: mostly identical to the concept of a memory page with the key

difference being its location in the physical main memory instead of the

virtual memory.

Physical memory is divided into slots that can hold pages, called frames.

The physical memory basically behaves like a cache for virtual memory.

The operating system creates a Page Table that holds the mappings of pages to

frames

Some pages are mapped to frames in the physical memory and others are not

Variables and Memory

The Process Memory Model

Each program is assigned its own virtual memory by the operating system.

The address space is arranged in a linear fashion with one block of data being

stored at each address.

![Stack and Heap](.//assets/ahr0chm6ly92awrlby51zgfjaxr5lwrhdgeuy29tl3rvcghlci8ymde5l1nlchrlbwjlci81zdg5mmexn19jmjetzmlnms9jmjetzmlnms5wbmcgiln0ywnrigfuzcbizwfwig==.png "stack and heap")

Stack: A contiguous memory block with a fixed maximum size. The stack is used

for storing automatically allocated variables (local vars or fn params).

If there are multiple threads, then each thread has its own stack memory.

New memory on the stack is allocated when the path of execution enters a

scope and freed again once the scope is left.

The stack is managed automatically by the compiler.

Each time a function is called, the stack grows (from top to bottom)

Each time a function returns, the stack contracts

When using multiple threads, each thread has its own stack memory

When the maximum size of the stack is exceeded, a program will crash

Allocating/deallocating memory is _fast_ on the stack because it only

requires moving the stack pointer to a new position

Heap ("free store" in C++): is where data with dynamic storage lives.

Shared among multiple threads in a program.

Managing memory on the heap is more computationally expensive and slower

than stack memory.

The heap is managed by the programmer, not the OS.

The programmer can allocate memory by issueing commands such as `maloc` or

`new`

This block of memory remains allocated until the programmer explicitly

issues a command such as `free` or `delete`

Memory can be allocated in an arbitrary scope without it being deleted when

the scope is left

As long as the address to an allocated block of memory is returned by a

function, the caller can freely use it.

Variables are allocated at run time

If allocated memory is not explicitly deallocated, it results in a _memory

leak_

The heap is shared among multiple threads, which means memory management

needs to take concurrency into account

Allocating/deallocating memory can occur arbitrarily, depending on the

lifetime of the variables which can result in fragmented memory over time.

This makes it much more difficult to manage.

BBS (Block Started by Symbol): Used in many compilers and linkers for a

segment that contains global and static variables that are initialized with

zero values.

Used for arrays that are not initialized with predefined values.

Data: serves the same purpose as the BSS segment with the difference bing that

variables in te Data segment have been initialized with a value other than 0.

Memory for variables in the Data segment and BSS is allocated once when a

program is run and persists throughout its lifetime.

Dynamic Memory Allocation

C-style allocation/deallocation

To allocate memory on the heap means to make a contiguous memory area

accessible to the program at runtime and to mark this memory as occupied so

that no one else can write there by mistake.

In C, you allocate a block of memory with `malloc` like this:


```c
#include <stdio.h>
#include <stdlib.h>
int main(){
    int *p = (int*)malloc(sizeof(int));
    printf("address=%p, value=%d\n", p, *p);return 0;
}

```

malloc is used to dynammically allocate a single large block of memory with

the specified size. It returns a pointer of type `void` which can be cast

into a pointer of any form.

You can also allocate memory with `calloc` which is used to dynamically

allocate the specified number of blocks of memory of the specified type. It

initializes each block with a default value `0`.


```c
#include <stdio.h>
#include <stdlib.h>
struct MyStruct {
    int i;
    double d;
    char a[5];
};

int main()
{
    MyStruct *p = (MyStruct*)calloc(4, sizeof(MyStruct));
    p[0].i = 1;
    p[1].d = 3.14159;
    p[2].a[0] = 'a';
    printf("address=%p, value=%d\n", p, p[0].i);
    return 0;
}

```

You can reallocate memory like this:


```c
#include <stdio.h>
#include <stdlib.h>
int main()
{
    // reserve memory for two integers
    int *p = (int*)malloc(2*sizeof(int));
    p[0] = 1; p[1] = 2;
    // resize memory to hold four integers
    p = (int*)realloc(p,4*sizeof(int));
    p[2] = 3; p[3] = 4;
    // resize memory again to hold two integers
    p = (int*)realloc(p,2*sizeof(int));
    return 0;
}

```

If memory has been reserved, it should also be released as soon as it is

no longer needed

The `free` function releases the reserved memory so it can be used again, like this:


```c
#include <stdio.h>
#include <stdlib.h>
int main()
{
    void *p = malloc(100);
    free(p);
    return 0;
}

```

`free` can only free memory that was reserved with `malloc` or `calloc`

`free` can only release memory that has not been released before.

Releasing the same block of memory twice will result in an error.

a _danging pointer_ happens when you do something like this:


```c
int main()
{
    void *p = malloc(100);
    void *p2 = p;
    free(p); // OK
    free(p2); // ERROR - dangling pointer
    return 0;
}

```

New/Delete

`malloc` and `free` are the default way of allocating and deallocating memory in C

In C++, you should use `new`/`delete` as they can handle the more complex memory management tasks associated with OOP

When nusing `new` in C++, it calls the constructor of the class and `delete`

calls the destructor of the class

`malloc` does not do this which can cause serious issues when using `malloc`

to allocate memory for a C++ class

Additionally, `new`/`delete` in C++ are type safe where `malloc`/`free` are

not

You can overload the `new`/`delete` functions in C++ to fit your use case

Any call to `new` must always be followed by a call to `delete` so as not to create a _memory leak_

In some cases, you might want to separate memory allocation from object

construction (eg. reconstructing an object several times).

This can be done with `placement new` like this:


```c++
void *memory = malloc(sizeof(MyClass));
MyClass *object = new (memory) MyClass; // referred to as "placement new"
```

There is no delete equivalent to this function, so you have to deallocate manually like this:


```c++
object->~MyClass(); // calls destructor
free(memory); // frees the memory
```

You can allocate an array of objects like this:


```c++
void* operator new[](size_t size);
void operator delete[](void*);
```

Overloading Example


```c++
  #include <iostream>
  #include <stdlib.h>

  class MyClass
  {
      int _mymember;

  public:
      MyClass()
      {
  std::cout << "Constructor is called\n";
      }

      ~MyClass()
      {
  std::cout << "Destructor is called\n";
      }

      void *operator new(size_t size)
      {
  std::cout << "new: Allocating " << size << " bytes of memory" << std::endl;
  void *p = malloc(size);

  return p;
      }

      void operator delete(void *p)
      {
  std::cout << "delete: Memory is freed again " << std::endl;
  free(p);
      }
  };

  int main()
  {
      MyClass *p = new MyClass();
      delete p;
  }

```

Possible Errors with Manual Memory Management

1. Memory Leaks

Occurs when data is allocated on the heap at runtime, but not properly deallocated

2. Buffer Overruns

Occurs when memory outside the allocated limits is overwritten and thus corrupted.

This can be very difficult to identify and debug.

It makes it possible to inject malicious code into programs

Example


```c++
char str[5]; // allocated stack memory is too small to hold the string
strcpy(str,"BufferOverrun");
printf("%s",str);
```

3. Uninitialized Memory

Depending on the compiler, data structures are sometimes initialized (most

often to zero) and sometimes not. When allocating memory on the heap without

proper initialization, it might sometimes contain garbage that can cause

problems.

Generally, a variable will be automatically initialized in these cases:

1. It is a class instance where the default constructor initializes all

primitive types

2. Array initializer syntax is used, such as `int a[10] = {}`

3. It is a global or extern variable

4. It is defined `static`

Example


```c++
int a;
int b=a*42;
printf("%d",b);
```

4. Incorrect pairing of allocatoin and deallocation

Freeing a block of memory more than once will cause a program to crash

This can happen when a bloc kof memory is freed that has never been allocated

or has been freed before

Can also be caused by improper pairings of allocation/deallocation are used such as using malloc with delete or new with free.

Example 1


```c++
double *pDbl = new double[5]; // prong new and delete are paired
delete pDbl;

```

Example 2


```c++
char *pChr=new char[5];
delete[] pChr;
delete[] pChr; // double deletion
```

5. Invalid memory access

Occurs when trying to access a bloc kof heap memory that has not yet or has already been deallocated

Example


```c++
char *pStr=new char[25];
delete[] pStr;
strcpy(pStr, "Invalid Access"); // pStr already deleted
```

Copying/Moving

RAII

A common way of safely accessing resources is by wrapping a manager class

around the handle, which is initialized when the resource is acquired (in the

class constructor) and released when it is deleted (in the class destructor).

This is referred to as _Resource Acquisition is Initialization(RAII)_.

One problem with this is that copying the manager object will also copy the handle of the resource. This allows two objects access to the same resource.

In C++, the copying process can be controlled by defining a tailored copy

constructor as well as a copy assignment operator. The copying process must be

closely linked to the respective resource release mechanism and is often referred to as _copy-ownership policy_.

Tailoring the copy constructor policy according to your memory management

policy is an important choice you often need to mak ewhen designing a class.

No copying policy: The simplest policy of all is to forbid copying and

assigning class instances all together.

This can be achieved by declaring, but not defining a private copy constructor and assignment operator or by making both public and assigning the `delete` operator.

Example:


```c++
  class NoCopyClass1
  {
  private:
      NoCopyClass1(const NoCopyClass1 &);
      NoCopyClass1 &operator=(const NoCopyClass1 &);

  public:
      NoCopyClass1(){};
  };

  // this one is more explicit
  class NoCopyClass2
  {
  public:
      NoCopyClass2(){}
      NoCopyClass2(const NoCopyClass2 &) = delete;
      NoCopyClass2 &operator=(const NoCopyClass2 &) = delete;
  };

  int main()
  {
    NoCopyClass1 original1;
    NoCopyClass1 copy1a(original1); // copy c’tor
    NoCopyClass1 copy1b = original1; // assigment operator

    NoCopyClass2 original2;
    NoCopyClass2 copy2a(original2); // copy c’tor
    NoCopyClass2 copy2b = original2; // assigment operator

    return 0;
  }

```

Exclusive ownership policy: states that whenever a resource management object

is copied, the resource handle is transferred from the source pointer to the destination pointer.

In the process, the source pointer is set to `nullptr` to make ownership exclusive.

At any time, the resource handle belongs only to a single object, which is responsible for its deletion when it is no longer needed.

Example: take a look at [this version](https://github.com/dnjp/C/blob/master/c%2B%2B/memory/src/exclusiveownership.cc) for a more detailed version


```c++
  #include <iostream>

  class ExclusiveCopy
  {
  private:
      int *_myInt;

  public:
      ExclusiveCopy()
      {
 _myInt = (int *)malloc(sizeof(int));
 std::cout << "resource allocated" << std::endl;
      }
      ~ExclusiveCopy()
      {
 if (_myInt != nullptr)
 {
     free(_myInt);
     std::cout << "resource freed" << std::endl;
 }

      }
      ExclusiveCopy(ExclusiveCopy &source)
      {
 _myInt = source._myInt;
 source._myInt = nullptr;
      }
      ExclusiveCopy &operator=(ExclusiveCopy &source)
      {
 _myInt = source._myInt;
 source._myInt = nullptr;
 return *this;
      }
  };

  int main()
  {
    ExclusiveCopy source;
    ExclusiveCopy destination(source);

    return 0;
  }
```

Deep copying policy: Allocate proprietary memory in the destination object and then copy the content to which the source object handle is pointing into the newly allocated block of memory. This way, the content is preserved during copy or assignment. The problem with this approach is that this increases the memory demands and the uniqueness of the data is lost. After the deep copy, two versions of the same resource exist in memory. Example:


```c++
    #include <iostream>

    class DeepCopy
    {
    private:
int *_myInt;

    public:
DeepCopy(int val)
{
    _myInt = (int *)malloc(sizeof(int));
    *_myInt = val;
    std::cout << "resource allocated at address " << _myInt << std::endl;
}
~DeepCopy()
{
    free(_myInt);
    std::cout << "resource freed at address " << _myInt << std::endl;
}
DeepCopy(DeepCopy &source)
{
    _myInt = (int *)malloc(sizeof(int));
    *_myInt = *source._myInt;
    std::cout << "resource allocated at address " << _myInt << " with _myInt = " << *_myInt << std::endl;
}
DeepCopy &operator=(DeepCopy &source)
{
    _myInt = (int *)malloc(sizeof(int));
    std::cout << "resource allocated at address " << _myInt << " with _myInt=" << *_myInt << std::endl;
    *_myInt = *source._myInt;
    return *this;
}
    };

    int main()
    {
      DeepCopy source(42);
      DeepCopy dest1(source);
      DeepCopy dest2 = dest1;

      return 0;
    }

```

Shared ownership policy: performs a copy or assignment similar to default behavior (copying the handle instead of the content) while at the same time keeping track of the number of instances that also point to the same resource. Each time an instance goes out of scope, the counter is decremented. Once the last object is about to be deleted, it can safely deallocate its resources

This is the central idea behind `unique_ptr` which is in the smart pointers

group

Does not overload the assignment operator

Example:


```c++
  #include <iostream>

  class SharedCopy
  {
  private:
      int *_myInt;
      static int _cnt;

  public:
      SharedCopy(int val);
      ~SharedCopy();
      SharedCopy(SharedCopy &source);
  };

  int SharedCopy::_cnt = 0;

  SharedCopy::SharedCopy(int val)
  {
      _myInt = (int *)malloc(sizeof(int));
      *_myInt = val;
      ++_cnt;
      std::cout << "resource allocated at address " << _myInt << std::endl;
  }

  SharedCopy::~SharedCopy()
  {
      --_cnt;
      if (_cnt == 0)
      {
  free(_myInt);
  std::cout << "resource freed at address " << _myInt << std::endl;
      }
      else
      {
  std::cout << "instance at address " << this << " goes out of scope with _cnt = " << _cnt << std::endl;
      }
  }

  SharedCopy::SharedCopy(SharedCopy &source)
  {
      _myInt = source._myInt;
      ++_cnt;
      std::cout << _cnt << " instances with handles to address " << _myInt << " with _myInt = " << *_myInt << std::endl;
  }

  int main()
  {
    SharedCopy source(42);
    SharedCopy destination1(source);
    SharedCopy destination2(source);
    SharedCopy destination3(source);

    return 0;
  }

```

Rule of Three: states that if a class needs to have an overloaded copy

constructor, copy assignment operator, ~or~ destructor, then it must also

implement the other two as well to ensure that memory is managed consistently.

Lvalues and Rvalues

What are lvalues and rvalues?

Every expression in C++ has a type and belongs to a value category. When

objects are created, copied, or moved during evaluation of an expression, the

compiler uses these value expressions to decide which method to call or which

operator to use.

[This image](https://r859981c931117xjupyterlqnspax0s.udacity-student-workspaces.com/files/images/C42-FIG1.png?_xsrf=2%7C87555f10%7C5948e4163c438afeb794781e26ed750b%7C1578317008&1578614041017)

diagrams the different value categories

Lvalues: have an address that can be accessed. They are expressions whose

evaluation by the compiler determines the identity of objects or functions.

Prvalues: do not have an address that is accessible directly. They are

temporary expressions used to initialize objects or compute the value of the

operand of an operator.

We will refer to prvalues as rvalues from here

"l" and "r" are originally derived from the perspective of the assignment

operator `=`.

It expected an rvalue on the right which it assigns to a lvalue on the

left.

An lvalue is an entity thta points to a specific memory location. An rvalue

is usually a short-lived object which is only needed in a narrow local

scope.

You can think of lvalues as named containers for rvalues

Example:


```c++
int i = 42; // i is an lvalue and 42 is an rvalue
int *j = &i;
/*
    &i generates the address of i as an rvalue and assigns it to j,
    which is an lvalue now holding the memory location of i
*/

```

One of the primary use-cases for lvalue references is the

pass-by-reference semantics in function calls as in the example here:


```javascript
    #include <iostream>

    // has an lvalue reference as a parameter which establishes an
    // alias to the integer i which is passed to it in main
    void myFunction(int &val) {
      ++val;
    }

    int main() {
      int i = 1;
      myFunction(i);

      std::cout << "i = " << i << std::endl;

      return 0;
    }

```

rvalue reference: identified by the && after a type name.

With this operator, it is possible to store and even modify an rvalue

(i.e. a temporary object which would otherwise be lost quickly)

Example:


```c++
    #include <iostream>

    int main()
    {
      int i = 1;
      int j = 2;
      int k = i + j;

      /*
       * creates rvalue reference, to which the address of the temporary object is assigned
       * that holds the result of the addition. Instead of first creating the r value i+j, then
       * copying it and deleting it, we can now hold the temporary object in memory. This can be
       * much more efficient than the first approach.
       */
      int &&l = i + j;

      std::cout << "k = " << k << ", l = " << l << std::endl; // prints: k=3, l=3

      return 0;
    }

```

This allows you to do things like this:


```c++
    #include <iostream>
    void myFunction(int &&val) {
      ++val;
      std::cout << "val = " << val << std::endl; // prints val=4
    }

    int main() {
      int i = 1;
      int j = 2;
      myFunction(i + j);

      return 0;
    }

```

Move Semantics

The last example shows us a few things:

The object that binds to the rvaue reference (&&val) is yours, it is not

needed anymore within the scope of the caller (which is main)

Passing values like this improves performance as no temporary copy needs to

be made anymore

Ownership changes, since the object the reference binds to has been

abandoned by the caller and now binds to a handle which is available only to

the receiver. This could not have been achieved with lvalue references as

any change to the object that binds to the lvalue reference would also be

visible on the caller side

rvalue references are themselves lvalues.

A reference is always defined in a certain context. Even though the object

it refers to may be disposable in the context it has been created, it is not

disposable in the context of the reference. Within the scope of myFunction

(from the example above), `val` is an lvalue as it gives access to the

memory location where the numbers 42 is stored.

However, in the example above we couldn't pass an lvalue to `myFunction`

because an rvalue reference cannot bind to an lvalue.

We couldn't do smething like:


```c++
  int i = 23;
  myFunction(i);
```

The solution to this problem is the function `std::move` which vonerts an

to use the lvalue as an argument for the function:

lvalue into an rvalue (technically it's an xvalue), which makes it possible


```javascript
  int i = 23;
  myFunction(std::move(i));
```

This says that in the scope of `main` we will not use i anymore, which now

exists only in the scope of `myFunction`.

Rule of Five: states that if you have to write one of the functions listed

below, then you sohuld consider implementing all of them with a proper

resource management policy in place. If you forget to implement one or more,

the compiler will usually generate the missing ones (without a warning) but

the default versions might not be suitable for the purpose you have in mind.

1. destructor: Responsible for freeing the resource once the object it belongs

to goes out of scope.

2. Assignment operator: The default assignment operation performs a

member-wise shallow copy, which does not copy the content behind the

resource handle. If a deep copy is needed, it has to be implemented by the

programmer.

3. Copy constructor: As with the assignment operator, the deault copy

constructor performs a shallow copy of the data members. If something else

is needed, the programmer has to implement it accordingly.

4. Move constructor: Because copying objects can be an expensive operation

which involves creating, copying, and destroying temporary objects, rvalue

references are used to bind to an rvaluee. Using this mechanism, the move

constructor transfers the ownership of a resource from a (temporary) rvalue

object to a permenant lvalue object.

5. Move assignment operator: With this operator, ownership of a resource can

be transferred from oneobject to another.

Examples of this can be seen [here](https://github.com/dnjp/C/blob/master/c%2B%2B/memory/src/rule3.cc)

RAII (Resource Acquisition is Initialization)

Problems that can arrise with `new`/`delete`:

1. Proper pairing of new and delete: Every dynamically allocated object that

is created with new must be followed by a manual deallocation at a "proper"

place in the program. If the programmer forgets to call delete or if it is

done at an inappropriate position, memory leaks will occur which might clog

up a large portion of memory.

2. Correct operator pairing: C++ offers a variety of new/delete operators,

especially when dealing with arrays on the heap. A dynamically allocated

array initialized with `new[]` may only be deleted with the operator

`delete[]`. If the wrong operator is used, program behavior will be

undefined.

3. Memory ownership: If a third-party function returns a pointer to a data

structure, the only way of knowing who will be responsible for resource

deallocation is by looking into either the code or the documentation. If

both are not available, there is no way to infer the ownership from the

return type.

Smart pointers

Smart pointers were introduced in C++ to solve these issues. When a smart

pointer is no longer needed, the memory to which it points is automatically

deallocated.

Smart pointers are classes wrapped around raw pointers. By overloading the

`->` and `*` operators, smart pointer objects make sure that the memory to

which their internal raw pointer refers to is properly deallocated. This

method of wrapping a management class around a resource has been conceived

by Bjarne Stroustroup and is called _Resource Acquisition Is Initialization

(RAII)_.

Resource Acquisition Is Initialization

In most programs, there will be many situations where a certain action at

some point will necessitate a proper reaction at another point, such as:

1. Allocating memory with new or malloc, which must be matched with a call

to delete or free.

2. Opening a file or network connection, which must be closed again after

the content has been read or written.

3. Protecting synchronization primitives such as atomic operations, memory

barriers, monitors or critical sectoins, which must be released to allow

other threads to obtain them.

[This table](https://r859981c931307xjupyterltf6w5vg3.udacity-student-workspaces.com/files/C5-1%20%20Resource%20Acquisition%20Is%20Initialization/C51-FIG1.png?_xsrf=2%7Cb1d5d029%7C9577be636285e5471ec58a13bbe34632%7C1578710630&1578760333465)

gives an overview of some resources and their respective

allocation/deallocation calls in C++.

The problem of reliable resource release:

The general usagepattern for the above examples is like this:

1. Obtain Resource

2. Use Resource

3. Release Resource

However, there are problems with this simple pattern:

1. The program might throw an exception during resource use and thus the

point of release might never be reached.

2. There might be several points where the resource could potentially be

relased, making it hard for a programmer to keep track of all

eventualities.

3. We might simply forget to release the resource again.

RAII

Th major idea of RAII revolves around object ownership and information

hiding.

Allocation/Deallocation are hidden within the management class, so a

programmer using the class does not have to worry about memory management.

If he has not directly allocated a resource, he will not need to directly

deallocate it. Whoever owns a resource deals with it. In RAII, this is the

management class around the protected resource.

Advantages:

1. Use class destructors to perform resource clean-up tasks such as proper

memory deallocation when the RAII obbject gets out of scope

2. Manage ownership and lifetime of dynamically allocated objects

3. Implement encapsulation and information hiding due to resource

acquisition and release being performed within the same object.

There are three major parts to an RAII class:

1. A resource is allocated in the constructor of the RAII class

2. The resource is deallocated in the destructor

3. All instances of the RAII class rae allocated on the stack to reliably

control the lifetime via the object scope.

Example [here](https://github.com/dnjp/C/blob/master/c%2B%2B/memory/src/raii3.cc)

RAII and Smart Pointers

With smart pointers, resource acquisition occurs at the same time that the

object is initialized (when instantiated with `make_shared` or

`make_unique`), so that all resources for the object are created and

initialized in a single line of code.

In modern C++, raw pointers managed with `new` and `delete` should only be

used in small blocks of code with limited scope, where performance is

critical, and ownership rights of the memory resources are clear.

Smart Pointers

C++11 has introduced three types of smart pointers:

1. The _unique pointer_: `std::unique_ptr` is a smart pointer which

exclusively wons a dynamically allocated resource on the heap. There must

not be a second unique pointer to the same resource.

A unique pointer is constructed like `std::unique_ptr<Type> p(new Type);`

[Example](https://github.com/dnjp/C/blob/master/c%2B%2B/memory/src/unique.cc)

[Example 2](https://github.com/dnjp/C/blob/master/c%2B%2B/memory/src/unique2.cc)

[Diagram](https://r859981c931308xjupyterlqe3a4k79.udacity-student-workspaces.com/files/images/C52-FIG1.png?_xsrf=2%7C000b5ba0%7Cbb1300869e8d71218b311094cee47c62%7C1578765415&1578765424241)

2. The _shared pointer_: `std::shared_ptr` points to a heap resource but

does not explicitly own it. There may even be several shared pointers to

the same resource, each of which will increase an internal reference

count. As soon as this count reaches zero, the resource will

automatically be deallocated.

A shared pointer owns the resource it points to. The main difference

between the a shared pointer and a unique pointer, is that shared pointers

keep a reference counter on how many of them pointto the same memory

resource.

Each time a shared pointer goes out of scope, the counter is decreased.

When it reaches 0, the memory is properly deallocated.

The shared pointer type is useful for cases where you require access to a

memory location on the heap in multiple patrs of your program and you want

to make sure that whoever owns a rheared pointer to the memory can rely on

the fact that it will be accessible throughout the lifetime of that

pointer.

[Diagram](https://r859981c931308xjupyterlqe3a4k79.udacity-student-workspaces.com/files/images/C52-FIG2.png?_xsrf=2%7C000b5ba0%7Cbb1300869e8d71218b311094cee47c62%7C1578765415&1578768273467)

[Example](https://github.com/dnjp/C/blob/master/c%2B%2B/memory/src/shared.cc)

[Example 2](https://github.com/dnjp/C/blob/master/c%2B%2B/memory/src/shared2.cc)

[Problems](https://github.com/dnjp/C/blob/master/c%2B%2B/memory/src/sharedproblem.cc)

3. The _weak pointer_: `std::weak_ptr` behaves similar to the shared pointer

but does not increase the reference counter.

Prior to C++11, there was a concept called `std::auto_ptr` which tried to

accomplish something similar to the above. However, usage of auto_ptr can

now be considered as deprecated and shoult not be used.

Similar to shared pointers, there can be multiple weak pointers to the

same resource.

Weak pointers do not increase the reference count.

Weak pointers hold a non-owning reference to an object that is managed by

another shared pointer.

You can only create weak pointers out of shared pointers or out of another

weak pointer.

With a weak pointer, even though this type does not prevent an object from

being deleted, the validity of its resource can be checked.

With smart pointers there will always be a managing instance which is

responsible for the proper allocation and deallocation of a resource.

[Example](https://github.com/dnjp/C/blob/master/c%2B%2B/memory/src/weak.cc)

[Example 2](https://github.com/dnjp/C/blob/master/c%2B%2B/memory/src/weakexpired.cc)

Converting between smart pointers

[Example](https://github.com/dnjp/C/blob/master/c%2B%2B/memory/src/smartconvert.cc)

Rules for smart pointers

The C++ core guidelines contain a total of

[13 rules](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#rsmart-smart-pointers)

for the recommended use of smart pointers.

R.20: Use unique_ptr or shared_ptr to represent ownership

R.21: Prefer unique_ptr of std::shared_ptr unless you need to share

ownership

R.22: Use make_shared() ot make shared_ptr

R.23: Use make_unique() to make std::unique_ptr

R.24: Use weak_ptr to break cycles of shared_ptr

Using make\_... factory functions allows you to create a smart pointer in

a single step which removes the risk of a memory leak.

Transferring Ownership

R.30: Take smart pointers as parameters only to explicitly express lifetime

semantics

Functions that only manipulate objects without affecting its lifetime in any

way should not be concerned with a particular kind of smart pointer.

A function that does not manipulate the lifetime or ownership should use raw

pointers or references instead.

A function shuold only take smart pointers as parameters only if it examines

or manipulates the smart pointer itself.

The following examples are pass-by-value types that lend ownership of the

underlying object:

`void f(std::unique_ptr<MyObject> ptr)`

`void f(std::shared_ptr<MyObject> ptr)`

`void f(std::weak_ptr<MyObject> ptr)`

Passing smart pointers by value means to lend their ownership to a

particular function `f`. In the above examples, all pointers are passed by

value. The function `f` has a private copy of it which it can (and should)

modify.

R.32: Take a unique_ptr parameter to express that a function assumes ownership

of a widget

[Example](https://github.com/dnjp/C/blob/master/c%2B%2B/memory/src/ownership.cc)

R.34: Take a shared_ptr parameter to express that a function is part owner.

[Example](https://github.com/dnjp/C/blob/master/c%2B%2B/memory/src/ownership2.cc)

R.33: Take a unique_ptr& parameter to express that a function reseats the

widget

R.35: Take a shared_ptr& parameter to express that a function might reseat the

shared pointer

Both rules recommend passing-by-reference, when the function is supposed to

modify the ownership of an existing smart pointer and not a copy.

We pass a non-const reference to a unique_ptr to a function if it might

modify it in any way.

Passing a unique_ptr as const is not useful as the function will not be able

to do anything with it: Unique pointers are all about proprietary ownership

and as soon as the pointer is passed, the function will assume ownership.

But, without the right to modify the pointer, the options are very limited.

A shared_ptr can either be passed as const or non-const reference. The const

should be used when you want to express that the function will only read

from the pointer or it might create a local copy and share ownership.

The general rule of thumb is that we can use a simple raw pointer or a plain

reference when the function we are passing will only inspect the managed

object without modifying the smart pointer.

The internal raw pointer to the object can be retrieved using the get()

member function.

By providing access to the raw pointer, you can use the smart pointer to

manage memory in your own code and pass ther aw pointer to code that does

not support smart pointers.

When using raw pointers retrieved from get(), DO NOT DELETE THEM OR CREATE

RAW POINTERS FROM THEM. If you do so, the ownership rules applying to the

resource would be severely violated.

Returning smart pointers from functions

With return values, the same logic that we have used for passing smart

pointers to functions apply: return a smart pointer, both unique or shared,

if the called needs to manipulate or access the pointer properties. In case

the caller just needs the underlying object, a raw pointer should be

returned.

Smart pointers should always be returned by value due to the following

advantages:

1. The overhead associated with return-by-value due to the expensive copying

process is significantly mitigated by the built-in move semantics of

smart pointers.

They only hold a reference to the managed object, which is quickly

switched from destination to source during the move process.

2. Since C++17, the compiler used _Return Value Optimization(RVO)_ to avoid

the copy usually associated with return-by-value. This technique is able

to optimize even move semantics and smart pointers.

3. When returning a shared_ptr by value, the internal reference counter is

guaranteed to be properly incremented. Thi sis not the case when

returning by pointer or by reference.

The following list contains all the variations (omitting const) of passing

an object to a function:


```cpp

					 - void f( object* );  // (a)

					 - void f( object& ); // (b)

					 - void f( unique_ptr<object> ); // (c)

					 - void f( unique_ptr<object>& ); // (d)

					 - void f( shared_ptr<object> ); // (e)

					 - void f( shared_ptr<object>& ); // (f)

					 - 
```

The preferred way to pass object parameters is by using a or be.

To decide whether a or b is more appropriate, you should think about

whether you need to express that there is no object. This ca nonly be done

with pointers by passing (e.g. nullptr). In most other cases, you should

use a reference instead.

The preferred way of passing an object to a function so that the function

takes ownership of the object is by using method c.

In this case, we are passing a unique pointer by value from caller to

function, which then takes ownership of the pointer and the underlying

object.

This is only possible using move semantics as there may be only a single

reference to the object managed by the unique pointer.

After the object has been passed int his way, the caller will have an

invalid unique pointer and the function to which the object now belongs

may destroy it or move it somewhere else.

In the case where you want to modify a unique pointer and re-use it in the

context of the caller, method d is the most suitable.

Using this call structure, the function states that it might modify the

smart pointer. It is not recommended to use it for accepting an object

only because we should avoid restricting ourselves unnecessarily to a

praticular object lifetime strategy on the caller side.

In the case where we want to express that a function will store and share

ownership of an object on the heap, method e is most suitable.

Here, we are making a copy of the shared pointer passed to the function.

In doing so, the internal reference counter wihtin all shared pointers

referring to the same heap object is incremented by one.

this strategy is recommended for cases where the function needs to retain

a copy of the shared_ptr and thus share ownership of the object. This is

useful when you need the reference count or you must make sure that the

object is not prematurely deallocated.

If the local scope of the function is not the final destination, a shared

pointer can also be moved, which does not increase the reference count and

is more efficient.

When you need to modify shared pointers and re=use them in the context of

the caller, method f is the right choice.

This expresses that the function f will modify the pointer itself. As with

method e, we will be limiting the usability of the function to cases where

the object is managed by a shared_ptr and nothing else.

Concurrency

Definition:: a single system is completing multiple independent tasks in parallel. Such a system is called a multitasking system. Under the hood, this uses very fast switching of tasks which is not Parallelism

Writing programs that have multiple paths of execution with the ability to run in parallel

One such path is called a thread and using several threads is called multi-threading

Programs using multiple threads are called concurrent

True Concurrency where a tasks are running at the same time require a parallel architecture which requires multiple processors/cores. This is called hardware concurrency.

Processes and Threads

The first way to take advantage of concurrency is to split your application into multiple single threaded processes that are run at the same time.

These processes can pass messages to one another using Inter-Process Communication(IPC).

One disadvantage is that the communication processes can be slow and everything has to be managed through the operating system.

The OS has to provide protection between processes so that one process doesn't modify data that another process is using.

One advantage of running separate processes is that it is easy to have them run in a distributed architecture.

The second way to take advantage of concurrency is to run multiple threads in a single process.

Threads are often referred to as lightweight processes that run independent of each other and each with its own set of instructions.

Threads in a process share the same address space which reduces overhead.

The problem is that because the threads share the same address space, you have to make sure that they aren't trying to access the same data at the same time.

[Example](https://video.udacity-data.com/topher/2019/June/5d07fe1e_c2-2-a2a/c2-2-a2a.png)

A process (also called a task) is a program at runtime. It is comprised of the runtime environment provided by the OS, as well as of the embedded binary code of the program during execution. A process is controlled by the OS through certain actions with which it sets the process into one of several carefully defined states:

__Ready__: After its creation, a process enters the ready state and is loaded into main memory. The process now is ready to run and is waiting for CPU time to be executed. Processes that are ready for execution by the CPU are stored in a queue managed by the OS.

__Running__: The OS has selected the process for execution and the instructions within the process are executed on one or more of the available CPU cores.

__Blocked__: A process that is blocked is one that is waiting for an event (such as a system resource becoming available) or the completion of an I/O operation.

__Terminated__: When a process completes its execution or when it is being explicitly killed, it changes to the "terminated" state. The underlying program is no longer executing, but the process remains in the process table as a "zombie process". When it is finally removed from the process table, its lifetime ends.

__Ready suspended__: A process that was initially in ready state but has been swapped out of main memory and placed onto external storage is said to be in suspend ready state. The process will transition back to ready state whenever it is moved to main memory again.

__Blocked suspended__: A process that is blocked may also be swapped out of main memory. It may be swapped back in again under the same conditions as a "ready suspended" process. In such a case, the process will move to the blocked state, and may still be waiting for a resource to become available.

Processes are managed by the scheduler of the OS. The scheduler can either let a process run until it ends or blocks (non-interrupting scheduler), or it can ensure that the currently running process is interrupted after a short period of time. The scheduler can switch back and forth between different active processes (interrupting scheduler), alternatively assigning them CPU time. The latter is the typical scheduling strategy of any modern operating system. This is pretty resource intensive, so the OS supports a more resource-friendly way of handling concurrent operations: threads.

A thread represents a concurrent execution unit within a process. Threads are characterized as light-weight processes (LWP). These are significantly easier to create and destroy. In many systems the creation of a thread is up to 100 times faster than the creation of a process. This is especially advantageous in situations when the need for concurrent operations changes dynamically.

Threads exist within processes and share their resources. As illustrated in this [Example](https://video.udacity-data.com/topher/2019/June/5d07fe20_c2-2-a2c/c2-2-a2c.png), a process can contain several threads or, if no parallel processing is provided for in the program flow, only a single thread.

A major difference between a process and a thread is that each process has its own address space, while a thread does not require a new address space to be created. 

All threads in a process can access its shared memory. Threads also share other OS dependent resources such as processors, files, and network connections. As a result, the management overhead for threads is typically less than for processes. Threads, however, are not protected against each other and must carefully synchronize when accessing the shared process resources to avoid conflicts.

Similar to Processes, Threads exist in different states which is illustrated [here](https://video.udacity-data.com/topher/2019/June/5d07fe21_c2-2-a2d/c2-2-a2d.png):

__New__: A thread is in this state once it has been created. Until it is actually running, it will not take any CPU resources.

__Runnable__: In this state, a thread might actually be running or it might be ready to run at any instant of time. It is the responsibility of the thread scheduler to assign CPU time to the thread.

__Blocked__: A thread might be in this state, when it is waiting for I/O operations to complete. When blocked, a thread cannot continue its execution any further until it is moved to the runnable state again. It will not consume any CPU time in this state. The thread scheduler is responsible for reactivating the thread.

Running a Single Thread

Compiling multi-threaded applications with g++ requires the `-pthread` flag

[Get Cores](https://github.com/dnjp/C/blob/master/c%2B%2B/concurrency/src/1/single_thread/getcores.cpp)

[Single Thread](https://github.com/dnjp/C/blob/master/c%2B%2B/concurrency/src/1/single_thread/singlethread.cpp)

[Randomness](https://github.com/dnjp/C/blob/master/c%2B%2B/concurrency/src/1/single_thread/randomness.cpp)

[Threads](https://github.com/dnjp/C/blob/master/c%2B%2B/concurrency/src/1/single_thread/threads.cpp)

[Barrier](https://github.com/dnjp/C/blob/master/c%2B%2B/concurrency/src/1/single_thread/barrier.cpp)

[Detach](https://github.com/dnjp/C/blob/master/c%2B%2B/concurrency/src/1/single_thread/detach.cpp)

[Example](https://github.com/dnjp/C/blob/master/c%2B%2B/concurrency/src/1/single_thread/example1.cpp)

Starting a thread with a function object

__Callable Object__: Passing functions to other functions. Callable objects are objects that can appear as the left-hand operand of the call operator. These can be pointers to functions, objects of a class that defines an overloaded function call operator and _lambdas_. In the context of concurrency, we can use callable objects to attach a function to a thread. The `std::thread` constructor can also be called with instances of classes that implement the function-call operator. We can do this by overloading the `()` operator.

[Lambdas](https://github.com/dnjp/C/blob/master/c%2B%2B/concurrency/src/1/thread_function/lambdas.cpp)

[Lambda Example](https://github.com/dnjp/C/blob/master/c%2B%2B/concurrency/src/1/thread_function/lambda2.cpp)

[Lambda Thread](https://github.com/dnjp/C/blob/master/c%2B%2B/concurrency/src/1/thread_function/lambdathread.cpp)

[Vehicle Example](https://github.com/dnjp/C/blob/master/c%2B%2B/concurrency/src/1/thread_function/vehicle.cpp)

[Vehicle Example 2](https://github.com/dnjp/C/blob/master/c%2B%2B/concurrency/src/1/thread_function/vehicle2.cpp)

Starting a Thread with Variadic Templates & Member Functions

Variadic Templates allow the definition of functions that take a variable number of arguments

[Variadic](https://github.com/dnjp/C/blob/master/c%2B%2B/concurrency/src/1/variadic/variadic.cpp)

[Variadic 2](https://github.com/dnjp/C/blob/master/c%2B%2B/concurrency/src/1/variadic/variadic2.cpp)

[Variadic 3](https://github.com/dnjp/C/blob/master/c%2B%2B/concurrency/src/1/variadic/variadic3.cpp)

[Variadic 4](https://github.com/dnjp/C/blob/master/c%2B%2B/concurrency/src/1/variadic/variadic4.cpp)

[Variadic 5](https://github.com/dnjp/C/blob/master/c%2B%2B/concurrency/src/1/variadic/variadic5.cpp)

[Variadic 6](https://github.com/dnjp/C/blob/master/c%2B%2B/concurrency/src/1/variadic/variadic6.cpp)

Running Multiple Threads

Using threads follows a basic concept called fork-join-parallelism. The basic mechanism of this concept follows a simple three-step pattern:

1. Split the flow of execution into a parallel thread (fork)

2. Perform some work in both the main thread and the parallel thread

3. Wait for the parallel thread to finish and unite the split flow of execution again (join)

[Diagram](https://r845225c859889xjupyterllynna106.udacity-student-workspaces.com/files/images/C2-6-A2%20multithreading.jpg?_xsrf=2%7Cd57b2125%7Ca31542aaf127187a82a1240ed1628148%7C1580059594&1580059600233)

In the main thread, the program flow is forked into three parallel branches. In both worker branches, some work is performed - which is why threads are often referred to as "worker threads". Once the work is completed, the flow of execution is united again in the main function using the join() command. 

In [this multi-thread example](https://github.com/dnjp/C/blob/master/c%2B%2B/concurrency/src/1/multiple_thread/multthread.cpp), join acts as a barrier where all threads are united. The execution of main is in fact halted, until both worker threads have successfully completed their respective work.

[Multi-thread Example](https://github.com/dnjp/C/blob/master/c%2B%2B/concurrency/src/1/multiple_thread/multthread.cpp)

[Concurrency Bug](https://github.com/dnjp/C/blob/master/c%2B%2B/concurrency/src/1/multiple_thread/concbug.cpp)

Promises and Futures

With the methods we've used so far we can pass data to a thread when we construct it: either by passing arguments to the thread function using variadic templates, or we can use a Lambda to capture arguments by value or by reference.

A drawback of these approaches is that the information flows from the parent thread (main) to the worker threads. We will now focus on methods to pass data in the opposite direction - from the worker back to the parent.

In order to do this, the threads need to adhere to a strict synchronization protocol. The mechanism for doing this acts as a single-use channel between the threads. The sending fend of the channel is called promise(`std::promise`) while the receiving end is called future(`std::future`). Each `std::promise` object is meant to be used only a single time.

[Promise/Future Example 1](https://github.com/dnjp/C/blob/master/c%2B%2B/concurrency/src/2/promise_future/pf1.cpp)

[Promise/Future Example 2](https://github.com/dnjp/C/blob/master/c%2B%2B/concurrency/src/2/promise_future/pf2.cpp)

[Promise/Future Example 3](https://github.com/dnjp/C/blob/master/c%2B%2B/concurrency/src/2/promise_future/pf3.cpp)

Threads vs Tasks

Determining the optimal number of threads to use is a hard problem. It usually depends on the number of available cores whether it makes sense to execute code as a thread or in a sequential manner. The use of `std::async` (and thus tasks) take the burden away from the user and let the system decide whether to execute the code sequentially or as a thread.

With tasks the programmer decides what CAN be run in parallel in principle and the system then decides at runtime what WILL be run in parallel. Internally, this achieved by using thread-pools which represent the number of available threads based on the cores/processors as well as by using work-stealing queues, where tasks are redustributed among the available processors dynamicaly. The [following diagram](https://r845225c860122xjupyterlej7g1gt6.udacity-student-workspaces.com/files/images/C3-3-A3a.png?_xsrf=2%7C7b8175b5%7C5cccdaed98c4363aaafaa732895fe0ca%7C1580085578&1580170733832) shows the principal of task distribution on a multi-core system using work stealing queues.

As can be seen, the first core is heavily oversubscribed with several tasks that are waiting to be executed. The other cores are running idle. The idea o a work stealing queue is to have a watchdog program running in the background that regularly monitors the amount of work performed by each processor and redistributes it as needed.

For the above example this would mean that tasks waiting for execution on the first core would be shifted (or "stolen") from busy cores and added to available free cores such that idle time is reduced. After this rearranging procedure, the task distribution in our example could look as shown [here](https://r845225c860122xjupyterlej7g1gt6.udacity-student-workspaces.com/files/images/C3-3-A3b.png?_xsrf=2%7C7b8175b5%7C5cccdaed98c4363aaafaa732895fe0ca%7C1580085578&1580170733832).

A work distribution in this manner can only work when parallelism is explicitly described in the program by the programmer. If this is not the case, work-stealing will not perform effectively.

With tasks, the system takes care of many details (e.g. join). With threads, the programmer is responsible for many details. As far as resources go, threads are usually more heavy-weight as they are generated by the operating system (OS). It takes time for the OS to be called and to allocate memory / stack / kernel data structures for the thread. Also, destroying the thread is expensive. Tasks on the other hand are a more light-weight as they will be using a pool of already created threads (the "thread pool").

Threads and tasks are used for different problems. Threads have more to do with latency. When you have functions that can block, threads can avoid the program to be blocked. Tasks on the other hand focus on throughput, where many operations are executed in parallel.

Data Races

Definition:: Occurs when two or more threads are trying to access the same block of memory and at least one of them is attempting to modify it. The first thread might still be writing while a second thread might be reading. In this scenario, the value at the memory location is completely undefined. Depending on the system scheduler, the second thread will be executed at an unknown point in time and thus see different data at the memory location with each execution.

Depending on the type of program, the result might be anything from a crash to a security breach when data is read by a thread that was not meant to be read, such as a user password or other sensitive information. Such an error is called a _data race_ because two threads are racing to get access to a memory location first, with the content at the memory location depending on the result of the race.

In [this example](https://r845225c860123xjupyterltnlmk1ob.udacity-student-workspaces.com/files/images/C3-4-A2.png?_xsrf=2%7Ccf7f1ad4%7C765dc089702fe18621483c893d950fc6%7C1580174352&1580213360934) one thread wants to increment a variable X, whereas the other thread wants to print the same variable. Depending on the timing of the program and thus the order of execution, the printed result might change each time the program is executed.

In this example, one safe way of passing data to a thread would be to carefully synchronize the two threads using either `join()` or the promise-future concept that can guarantee the availability of a result. Data races are always to be avoided. Even if nothing bad seems to happen, they are a bug and should always be treated as such.

Another possible solution for this example would be to make a copy of the original argument and pass the copy to the thread, thereby preventing the data race.

Examples:

[Data Race Example 1](https://github.com/dnjp/C/blob/master/c%2B%2B/concurrency/src/2/data_race/race1.cpp)

[Data Race Example 2](https://github.com/dnjp/C/tree/master/c%2B%2B/concurrency/src/2/data_race/race2.cpp)

[Data Race Example 3](https://github.com/dnjp/C/tree/master/c%2B%2B/concurrency/src/2/data_race/race3.cpp)

[Data Race Example 4](https://github.com/dnjp/C/tree/master/c%2B%2B/concurrency/src/2/data_race/race4.cpp)

[Data Race Example 5](https://github.com/dnjp/C/tree/master/c%2B%2B/concurrency/src/2/data_race/race5.cpp)

Mutexes and Locks

Promises/Futures are great for one time communication, but ideally we want the ability to share a communication channel between threads so they can work together.

Ideally, we would like to have a communication protocol that corresponds to voice communication over a radio channel, where the transmitter uses the expression "over" to indicate the end of the transmission to the receiver. By using such a protocol, sender and receiver can take turns in transmitting their data. In C++, this concept of taking turns can be constructed by an entity called a "mutex" which stands for MUtual EXclusion.

Data Races require simultaneous access from two threads. If we can guarantee that only a single thread at a time can access a particular memory location, data races would not occur. In order for this to work, we would need to establish a communication protocol. It is important to note that a mutex is not the solution to the data race problem per se but merely an enabler for a thread-safe communication protocol that has to be implemented and adhered to by the programmer. [Example](https://r845225c860408xjupyterlj66yb3ee.udacity-student-workspaces.com/files/images/C4-2-A2.png?_xsrf=2%7C2cf46bc9%7C918065bd084c660547ec7e0a31eec123%7C1580433563&1580433905823)

How does this work?

Assuming we have a piece of memory (e.g. a shared variable) that we want to protect from simultaneous access, we can assign a mutex to be the guardian of this particular memory. It is important to understand that a mutex is bound to the memory it protects. A thread 1 who wants to access the protected memory must first "lock" the mutex. After thread 1 is "under the lock", a thread 2 is blocked from access to the shared variable, it cannot acquire the lock on the mutex and is temporarily suspended by the system.

Once the reading or writing operation of thread 1 is complete, it must "unlock" the mutex so that thread 2 can access the memory location. Often, the code which is executed "under the lock" is referred to as a "critical section". It is important to note that also read-only access to the shared memory has to lock the mutex to prevent a data race - which would happen when another thread, who might be under the lock at that time, were to modify the data.

When several threads were to try to acquire and lock the mutex, only one of them would be successful. All other threads would automatically be put on hold - just as cars waiting at an intersection. Once the thread who has succeeded in acquiring the lock had finished its job and unlocked the mutex, a queued thread waiting for access would be woken up and allowed to lock the mutex to proceed with his read/write operation. If all threads were to follow this protocol, a data race would effectively be avoided.

Using a mutex consists of 4 straight-forward steps:

1. Include the `<mutex>` header

2. Create a `std::mutex`

3. Lock the mutex using `lock()` before read/write is called

4. Unlock the mutex after the read/write operation is finished using `unlock()`

Using a timed_mutex

`mutex`: provides the core functions lock() and unlock() and the non-blocking try_lock() method that returns if the mutex is not available.

`recursive mutex` allows multiple acquisitions of the mutex from the same thread.

`timed_mutex`: similar to mutex, but it comes with two more methods try_lock_for() and try_lock_until() that try to acquire the mutex for a period of time or until a moment in time is reached.

`recursive_timed_mutex`: is a combination of timed_mutex and recursive_mutex

Examples:

[Example 1](./src/3/mutex/example_1.cpp)

[Example 2](./src/3/mutex/example_2.cpp)

Using mutexes can significantly reduce the risk of data races as seen in the example above. But imagine what would happen if an exception was thrown while executing code in the critical section (between lock and unlock). In such a case, the mutex would remain locked indefinitely and no other thread could unlock it - the program would most likely freeze.

A second type of deadlock is a state in which two or more threads are blocked because each thread waits for the resource of the other thread to be released before releasing its resource. The result of the deadlock is complete standstill. The thread and therefore usually the whole program is blocked forever.

Up until now ew have directly called `lock()` and `unlock()` functions of a mutex. The idea of "working under the lock" it to block unwanted access by othre threads to the same resource. Only the thread which acquired the lock can unlock the mutex and give all remaining threads the chance to acquire the lock. In practice however, direct calls to `lock()` should be avoided at all cost. Imagine that while working under the lock, a thread would throw an exception and exit the critical section without calling the unlock function on the mutex. In such a situation, the program would most likely freeze as no ther thread could acquire the mutex anymore. This is exactly what we have seen in the function `divideByNumber` from [this example](./src/3/mutex/example_4.cpp).

We can avoid this problem by creating a `std::lock_guard` object, which keeps an associated mutex locked during the entire object life time. The lock is acquired on construction and released automatically on destruction. This makes it impossible to forget unlocking a critical section. Also, `std::lock_guard` guarantees exception safety because any critical section is automatically unlocked when an exception is thrown. In our examples so far, we can simply replace `_mutex.lock()` and `_mutex.unlock()` like the code in [this example](./src/3/mutex/example_6.cpp).

Note that there is no direct call to lock or unlock the mutex anymore. We now have a `std::lock_guard` object that takes the mutex as an argument and locks it at creation. When the method divideByNumber exits, the mutex is automatically unlocked by the `std::lock_guard` object as soon as it is destroyed - which happens when the local variable gets out of scope.

The problem with the previous example is that we can only lock the mutex once and the only way to control lock and unlock is by invalidating the scope of the `std::lock_guard` object. But what if we wanted (or needed) a finer control over the locking mechanism?

A more flexible alternative to `std::lock_guard` is `unique_lock` that also provides support for more advanced mechanisms, such as deferred locking, time locking, recursive locking, transfer of lock ownership and use of condition variables (which we will discuss later). It behaves similar to lock_guard but provides much more flexibility, especially with regard to the timing behavior of the locking mechanism. Here's an [updated example](./src/3/mutex/example_7.cpp)

The main advantages of using `std::unique_lock` over `std::lock_guard` are briefly summarized here. Using `std::unique_lock` allows you to:

construct an instance without an associated mutex using the default constructor

construct an instance with an associated mutex while leaving the mutex unlocked at first using the deferred-locking constructor

construct an instance that tries to lock a mutex, but leaves it unlocked if the lock failed using the try-lock constructor

construct an instance that tries to acquire a lock for either a specified time period or until a specified point in time

Despite the advantages of `std::unique_lock` and `std::lock_guard` over accessing the mutex directly, the deadlock situation where two mutexes are accessed simultaneously will still occur.

In most cases, your code should only hold one lock on a mutex at a time. Occasionally you can nest your locks, for example by calling a subsystem that protects its internal data with a mutex while holding a lock on another mutex, but it is generally better to avoid locks on multiple mutexes at the same time, if possible. Sometimes, however, it is necessary to hold a lock on more than one mutex because you need to perform an operation on two different data elements, each protected by its own mutex.

In the last section, we have seen that using several mutexes at once can lead to a deadlock, if the order of locking them is not carefully managed. To avoid this problem, the system must be told that both mutexes should be locked at the same time, so that one of the threads takes over both locks and blocking is avoided. That's what the `std::lock()` function is for - you provide a set of `lock_guard` or `unique_lock` objects and the system ensures that they are all locked when the function returns.

In the [following example](./src/3/mutex/example_8.cpp), `std::mutex` has been replaced with `std::lock_guard`.

In the following [deadlock-free code](./src/3/mutex/example_9.cpp), `std::lock` is used to ensure that the mutexes are always locked in the same order, regardless of the order of arguments. Note that `std::adopt_lock` option allows us to use `std::lock_guard` on a already locked mutex.

As a rule of thumb, programmers should try to avoid using several mutexes at once. Practice shows that this can be achieved in the majority of cases. For the remaining cases though, using `std::lock` is a safe way to avoid a deadlock situation.

The Monitor Object Pattern

In the previous section we have learned that data protection is a critical element in concurrent programming. After looking at several ways to achieve this, we now want to build on these concepts to devise a method for a controlled and finely grained data exchange between threads - a concurrent message queue. One important step towards such a construct is to implement a monitor object, which is a design pattern that synchronizes concurrent method execution to ensure that only one method at a time runs within an object. It also allows an object's methods to cooperatively schedule their execution sequences.

The problem solved by this pattern is based on the observation that many applications contain objects whose methods are invoked concurrently by multiple client threads. These methods often modify the state of their objects, for example by adding data to an internal vector. For such concurrent programs to execute correctly, it is necessary to synchronize and schedule access to the objects very carefully. The idea of a monitor object is to synchronize the access to an object's methods so that only one method can execute at any one time.

In a previous section, we have looked at a code example which came pretty close to the functionality of a monitor object: the class `WaitingVehicles`. Let us modify and partially re-implement this class, which we want to use as a shared place where concurrent threads may store data, in our case instances of the class `Vehicle`. As we will be using the same `WaitingVehicles` object for all threads, we have to pass it to them by reference - and as all threads will be writing to the object at the same time we will pass it as a shared pointer. Keep in mind that there will be many threads that will try to pass data to the `WaitingVehicles` object simultaneously and thus there is the danger of a data race.

The polling loop we used in [this example](./src/4/example_2.cpp) has not been programmed optimally. As long as the program is running, the wile-loop will keep the processor busy, constantly asking whether new data is available. In the [next example](./src/4/example_3.cpp) we will look at a better way to solve this problem without putting too much load on the processor.

The alternative to a polling loop is for the main thread to block and wait for a signal that new data is available. This would prevent the infinite loop from keeping the processor busy. We have already discussed a mechanism that would fulfill this purpose - the promise-future construct. The problem with futures is that they can only be used a single time. Once a future is ready and `get()` has been called, it cannot be used any more. For our purpose, we need a signaling mechanism that can be ere-used. The C++ standard offers such a construct in the form of "condition variables".

A `std::condition_variable` has a method `wait()` which blocks when it is called by a thread. The condition variable is kept blocked until it is released by another thread. The release works via the method `notify_one()` or the `notify_all` method. The key difference between the two methods it that `notify_one` will only wake up a single waiting thread while `notify_all` will wake up all the waiting threads at once.

A condition variable is a low-level building block for more advanced communication protocols. It neither has a memory of its own nordoes it remember notifications. Imagine that one thread calls `wait()` before another thread calls `notify()`, the condition variable works as expected and the first thread will wake up. Imagine the case however where the call order is reversed such that `notify()` is called before `wait()`, the notification will be lost and the thread will block indefinitely. So in a more sophisticated communicating protocol, a condition variable should always be used in conjunction with another shared state that can be checked independently. Notifying a condition variable in this case would then only mean to proceed and check this other shared state.

Let us pretend our shared variable was a boolean called `dataIsAvailable`. Now let's discuss two scenarios for the protocol depending on who acts first, the producer or the consumer thread:

[Scenario 1](https://r845225c860413xjupyterlimbc3b2e.udacity-student-workspaces.com/files/images/C4-4-A5a.png?_xsrf=2%7Ce4a6dfeb%7C4eca8063c7ed594533fa61149fb53a33%7C1580581606&1580581612457)

The consumer thread checks `dataIsAvailable()` and since it is false, the consumer thread blocks and waits on the condition variable. Later in time, the producer thread sets `dataIsAvailable` to true and calls `notify_one` on the condition variable. At this point the consumer wakes up and proceeds with its work.

[Scenario 2](https://r845225c860413xjupyterlimbc3b2e.udacity-student-workspaces.com/files/images/C4-4-A5b.png?_xsrf=2%7Ce4a6dfeb%7C4eca8063c7ed594533fa61149fb53a33%7C1580581606&1580581612457)

Here, the producer thread comes first, sets `dataIsAvailable()` to true and calls `notify_one`. Then, the consumer thread comes and checks `dataIsAvailable()` and finds it to be true - so it does not call wait and proceeds directly with its work. Even though the notification is lost, it does not cause a problem in this construct - the message has been passed successfully through `dataIsAvailable` and the wait-lock has been avoided.

In an ideal (non-concurrent) world, these two scenarios would most probably be sufficient to describe two possible combinations. But in concurrent programming, things are not so easy. As seen in the diagrams, there are four atomic operations, two for each thread. So when executed often enough, all possible inter-leavings will show themselves - and we have to find the ones that still cause a problem.

Here is [one combination](https://r845225c860413xjupyterlimbc3b2e.udacity-student-workspaces.com/files/images/C4-4-A5c.png?_xsrf=2%7Ce4a6dfeb%7C4eca8063c7ed594533fa61149fb53a33%7C1580581606&1580581612457) that would cause the program to lock.

The consumer thread reads `dataIsAvailable()`, which is false in the example. Then, the producer sets `dataIsAvailable()` to true and calls notify. Due to this unlucky interleaving of actions, the consumer thread calls wait because it has seen `dataIsAvailable()` as false. This is possible because the consumer thread tasks are not a joint atomic operation but may be separated by the scheduler and interleaved with some other tasks - in this case the two actions performed by the producer thread. The problem here is that after calling wait, the consumer thread will never wake up again. Also, the shared variable `dataReady` is not protected by a mutex here - which makes it even more likely that something will go wrong.

One quick idea for a solution which might come to mind would be to perform the two operations `dataIsAvailable` and wait under a locked mutex. While this would effectively prevent the interleaving of tasks between different threads, it would also prevent another thread from ever modifying `dataIsAvailable` again.

One reason for discussing these failed scenarios in such depth is to make you aware of the complexity of concurrent behavior - even with a simple protocol like the one we are discussing right now.

Let us take a look at the final solution to the above problems and thus a [working version](https://r845225c860413xjupyterlimbc3b2e.udacity-student-workspaces.com/files/images/C4-4-A5d.png?_xsrf=2%7Ce4a6dfeb%7C4eca8063c7ed594533fa61149fb53a33%7C1580581606&1580581612457) of our communication protocol:

As can be seen here, we are closing the gap between reading the state and entering the wait. We are reading the state under the lock (red bar) and we call wait still under the lock. Then we let wait release the lock and enter the wait state in one atomic step. This is only possible because the `wait()` method is able to take a lock as an argument. The lock that we can pass to wait however, is not the `lock_guard` we have been using until now, but instead it has to be a lock that can temporarily unlocked inside wait - a suitable lock for this purpose would be the `unique_lock` type which we have already discussed.

One thing to note is that once in a while, the system will, for no obvious reason, wake up a thread. These are called "spurious wake-ups". If such a spurious wake-up happened with taking proper precautions, we would issue wait without new data being available (because the wake-up has not been caused by the condition variable but by the system in this case). To prevent the call to `wait` in this case, we have to modify the code slightly:


```c++
std::unique_lock<sd::mutex> uLock(_mutex);
while (_vehicles.empty())
_cond.wait(uLock); // pass unique lock to condition variable
```

In this code, even after a spurious wake-up, we are now checking wether data really is available. If so, we would be issueing the call to wait on the condition variable. And onyl if we are inside wait, may other threads modify and access `dataIsAvailable`. If the vector is empty, `wait` is called. When the thread wakes up again, the condition is immediately re-checked and, in case it has not been a spurious wake-up, we can continue with our job and retrieve the vector.

We can further simplify this code by letting the `wait()` function do the testing as well as the looping for us. Instead of the while loop, we can just pass a Lambda to `wait()` which repeatedly checks whether the vector contains elements (thus the inverted logical expression):


```c++
std::unique_lock<sd::mutex> uLock(_mutex);
while (_vehicles.empty())
_cond.wait(uLock); // pass unique lock to condition variable
```

When `wait()` finishes, we are guaranteed to find a new element in the vector this time. Also, we are still holding the lock and thus no other thread is able to access the vector - so there is no danger of a data race in this situation. As soon as we are out of scope, the lock will be automatically released.

In the `main()` function, there is still the polling loop that infinitely queries the availability of new Vehicle objects. But contrary to the example before, a call to `popBack` now puts the `main()` thread into a wait state and only resumes when new data is available - thus significantly reducing the load to the processor.


