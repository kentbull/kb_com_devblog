+++
title = "KERI Internals Part 1: Concurrency and Async Programming with the HIO Async Framework and I/O Library"
slug = "keri-internals-hio-async"
date = "2024-05-11"

[taxonomies]
tags=["keri", "acdc", "hio", "async", "python", "asyncio", "concurrency", "programming"]

[extra]
comment = true
+++

# KERI Internals Part 1: Concurrency and Async Programming with the HIO Async Framework and I/O Library

{% img_right(path="/images/posts/13-hio-header.webp", 
    width=300, height=300,
    alt="Python and HIO", op="fit_width") 
    %}
Python and HIO
{% end %}

Welcome to the inner workings of the Python implementation of KERI! HIO stands for **Hierarchical IO**.

**Disclaimer**: this post is for a technical audience who have a need to read and understand the WebOfTrust Python implementation of the KERI, ACDC, and CESR Trust over IP (ToIP) specifications.

Have you ever wanted to contribute to the KERI ecosystem and been curious about the way the Python implementations are written? Or have you wanted to build on top of or modify the Python KERI codebase?

Not to worry, this technical series on KERI internals has your back. You will be ready to read through, understand, and build on top of the Python KERI code base once you understand the HIO async runtime, the focus of this article.

You are wanted as a contributor to the KERI ecosystem! The goal of this article is to assist you in becoming either a contributor to the Python implementation of KERI & ACDC or an advanced user of said implementation.

## HIO Introduction

HIO is an asynchronous runtime and input/output (IO) framework written by Dr. Samuel Smith that supports cooperative multitasking. It is used throughout the Python implementation of the KERI suite of protocols.

This article serves as an introduction to the three primary classes composing the basis for HIO's asynchronous runtime and as the lifecycle context functions for the main task class, the `Doer`. Additionally, you will have an idea of how these concepts relate to similar concepts in Python's AsyncIO runtime. The three HIO classes include:

1. the `Doist`, the root scheduler,
2. the `DoDoer`, the hierarchical container of `Doer` and `DoDoer` instances
3. `Doer`, the core task concept in HIO.

Due to its nature as the asynchronous runtime engine, HIO is found at the heart of the core Python libraries in the WebOfTrust ecosystem including the core library KERIpy, the agent server KERIA, and the SignifyPy client companion to KERIA.

In order to understand the purpose of three classes mentioned above and how they compare to Python's AsyncIO it is important to clarify terminology around concurrent and asynchronous programming in Python. As Python's `async/await` is much more common and familiar than HIO this article starts there to introduce the concepts.

### Why is HIO used in KERIpy, KERIA, and SignifyPy?

Performance, control, and features, at a high level, are the reason why HIO was used for KERIpy. HIO's use of what are called "classic coroutines" and asynchronous buffers for I/O provide a level of control and performance that is difficult to achieve with Python's AsyncIO implementation. An API into the timing system used for the event loop and scheduler provide tight, deterministic control over scheduling order of tasks.

A future article will go deeper than this short overview into the argument for using HIO and what specifically sets it apart from other async frameworks like AsyncIO, Curio, and Trio.

## Async Framework Short Comparison

An asynchronous framework typically consists of a number of major abstractions including an event loop, task or coroutine, scheduler, queues for communicating between tasks, futures, callbacks, non-blocking I/O, synchronization primitives (locks, semaphores), timeouts and cancellation, and some notion of lifecycle for tasks. This article focuses specifically on the event loop, scheduler, and task abstractions in HIO and Python's AsyncIO.

### Cooperative Multitasking

Both HIO and AsyncIO allow you to accomplish what is called "cooperative multitasking" which is where each coroutine yields control to a central scheduler so that other coroutines can be activated for their next execution. In AsyncIO the scheduler is the `asyncio` event loop and a coroutine is any function declared with the `async def` syntax. In HIO the scheduler is the `Doist` class and the coroutine is the `Doer` class.

### Concurrency and parallelism in Python

When discussing concurrency or asynchronous programming it is important to distinguish between what is typically meant by concurrency and parallelism.

> Concurrency is about dealing with lots of things at once.
>
> Parallelism is about doing lots of things at once.
>
> Not the same, but related. One is about structure, one is about execution.
>
> Concurrency provides a way to structure a solution to solve a problem that may (but not necessarily) be parallelizable.
>
> — Rob Pike, co-inventor of the Go language

Parallelism is a special case of concurrency. In Python `threading`, `multiprocessing`, and `asyncio` are the core packages for concurrent programming. In this post we only address the `asyncio` package, which supports what are called native coroutines.

## Python's AsyncIO package

### Native coroutines – async/await

A native coroutine is any function that, as mentioned earlier, uses the `async def` syntax to define a function, introduced with PEP-492 in Python 3.5 (2015). Calling an `async def` function does not automatically execute the code in the function. To execute the code the `await` keyword must be used when calling the function. This instructs the `asyncio` event loop to schedule execution of the function.

```python
import asyncio

# Native coroutine - uses the "async def" syntax to define a function
async def print_after(delay, what):
    await asyncio.sleep(delay)
    print(what)

# An asyncio-compatible main function
async def main():
    print(f"started at {time.strftime('%X')}")
    await print_after(1, 'hello')
    await print_after(2, 'world')
    print(f"finished at {time.strftime('%X')}")

# The asyncio task scheduler - uses the default asyncio event loop
asyncio.run(main())
```

In Python the `asyncio` package provides the `run` function where you can run the default event loop and task scheduler with `asyncio.run(my_main_coroutine())`.

The image below illustrates how coroutines, the core task concept in asyncio, are run in the AsyncIO framework.

{% image(path="/images/posts/13-hio-asyncio-task-mgt.png", class="diagram",
    op="fit_width",
    alt="AsyncIO Task Management") 
    %}
AsyncIO Task Management
{% end %}

You have your program, the Python process, that sends tasks to the asyncio event loop with either an explicit call to `asyncio.create_task()` or use the `await` keyword to schedule a task in the `asyncio` event loop and wait for its completion within the body of the function that used the `await` keyword.

AsyncIO can be difficult to use correctly though it is usually easy to recognize due to most library authors targeting `asyncio` mark their async functions with `async def`. There is also the `@types.coroutine` annotation to make an existing generator function compatible with native coroutines. In order to use AsyncIO and get the performance benefits of using asyncio your whole program has to embrace the AsyncIO paradigm, meaning you use `async def` native coroutines for anything that does I/O or long-running tasks and you use `await` to schedule all coroutines.

### Summary of AsyncIO

AsyncIO has a few main concepts for the async runtime, the asyncio event loop and an `async def` function as a coroutine. These basic concepts map nicely onto the HIO concepts of a `Doist`, the root scheduler in HIO, and the `Doer`, the coroutine or task concept in HIO. The main point where AsyncIO and HIO differ are that HIO has an explicit construct for creating hierarchies of tasks, the `DoDoer`. While there is no explicit construct in AsyncIO for a `DoDoer` any `async/await` coroutine could be considered to be a container for other coroutines.

### Combining AsyncIO and HIO

Though `asyncio` native coroutines are not used at all in HIO the two approaches are compatible and composable. You can schedule AsyncIO tasks from a HIO task (a Doer) and you can also schedule a HIO task from an asyncio coroutine.

Yet first we must describe what HIO is. The subject of combining AsyncIO and HIO will be covered in a future article. This article is a short introduction to the three main classes of HIO's async runtime implementation.

## What is HIO?

{% img_left(path="/images/posts/13-ioflo-logo.png", 
    width=100, height=100,
    alt="ioflo logo", op="fit_width") 
    %}
ioflo
{% end %}

HIO stands for Hierarchical IO. The [README](https://github.com/ioflo/hio/blob/main/README.md) describes it as weightless, hierarchical asynchronous coroutines and I/O in Python. This means that the task abstractions in HIO allow for nesting subtasks within tasks. HIO has three primary classes that make up its asynchronous runtime:

1. the `Doist`, or the root scheduler,
2. the `DoDoer`, a container holding either other `DoDoer` instances or `Doer` instances allowing you to create task hierarchies,
3. the `Doer`, the basic task or coroutine construct in HIO.

HIO makes heavy use of what are now known as "[classic coroutines](https://www.fluentpython.com/extra/classic-coroutines)" where the scheduler uses the `my_coro.send(data)` function to send data into a Python generator function. This generator function is the classic coroutine.

A few keywords distinguish classical coroutines including:

- `yield`: used to pause execution of a coroutine (generator function), send a value out of a coroutine, and receive a value into a coroutine.
- `yield from`: used when nesting generators (inner generators) to pause execution of the outer generator and pass, or delegate, control to a sub-generator. Once the sub-generator completes then control is passed back to the outer generator.
  - The `yield from` keyword is very similar to the `await` keyword from AsyncIO. Both drive sub-generators and both allow consumption of values returned by sub-generators.
  - `await` does not completely replace `yield from` because `await` must be used inside a native coroutine and must be used with an awaitable object.
  - `yield from` can be used in any function and with any iterable.

The `yield` keyword used in the body of a Python generator function allows it to receive values from the `my_coro.send()` function, similar to how Erlang/Elixir use the OTP to pass messages between processes with `send` and `receive`. The Python `my_coro.send(data)` is the **"send"** and the `myvar = yield from` invocation is the **"receive."** And the `yield from` keyword used in the body of a classic coroutine allows delegating to, or transferring execution to, a nested or sub-generator.

This classic coroutine approach HIO uses is grounded in structured concurrency where there are clear entry and exit points to tasks, errors in concurrently executing tasks propagate up the task chain, and clear expression of control flow within the structure of source code despite the presence of concurrency. The context methods of a HIO `Doer` task provide the clear entry and exit points as well as a clear exception handling mechanism.

### Overview

The root scheduler, the Doist, processes an array of `Doer` and `DoDoer` tasks. The `DoDoer` is the hierarchical task concept, and the `Doer` is the core task concept as shown below in the diagram.

Your program, the Python process, runs the Doist and the Doist runs the list of tasks until they finish or the program is terminated.

{% image(path="/images/posts/13-hio-task-mgt.png", class="diagram",
    op="fit_width",
    alt="HIO Task Management") 
    %}
HIO Task Management
{% end %}

```python
# from github.com/WebOfTrust/keripy/src/keri/app/cli/directing.py
# module: keri.app.cli.directing

# receives a list of tasks for the scheduler to run
def runController(doers, expire=0.0): 
    """
    Utility Function to create doist to run doers
    """
    tock = 0.03125

    # creates the Doist, the root scheduler
    doist = doing.Doist(limit=expire, tock=tock, real=True)  
   
    # adds tasks to the Doist to run. Calling "do" runs the Doist
    doist.do(doers=doers)
```

Here is a code example of creating an array of `doers` to pass to the root scheduler, the Doist, from KERIpy. This `runWitness` function shows the set of tasks that must be created in order to run a KERIpy witness.

```python
# from github.com/WebOfTrust/keripy/src/keri/app/cli/commands/witness/start.py
# module: keri.app.cli.commands.witness

# Function used by the CLI to run a single basic witness
def runWitness(name="witness", base="", alias="witness", bran="", tcp=5631, http=5632, expire=0.0):
    """
    Setup and run one witness
    """

    ks = keeping.Keeper(name=name, base=base, temp=False, reopen=True)

    aeid = ks.gbls.get('aeid')
    if aeid is None:
        hby = habbing.Habery(name=name, base=base, bran=bran)
    else:
        hby = existing.setupHby(name=name, base=base, bran=bran)

    hbyDoer = habbing.HaberyDoer(habery=hby)  # setup doer
    
    doers = [hbyDoer]  # list of tasks

    # extends the task list with the tasks from indirecting.setupWitness
    doers.extend(indirecting.setupWitness(alias=alias, hby=hby, tcpPort=tcp, httpPort=http)) 

    # calls the Doist root scheduler with a list of tasks
    directing.runController(doers=doers, expire=expire)
```

This function creates a few tasks to be run and hands them off to the `Doist` scheduler with `directing.runController`. The scheduler then runs the tasks to completion, or infinitely, depending on the contents of the `recur` function shown below in the `Doer`.

### HIO Task – a Doer

The core task concept in HIO is expressed as the `Doer` class shown in the UML diagram below. The HIO scheduler, a `Doist`, runs the `Doer` task until the `.done` attribute becomes `True`. There are six context functions five of which are executed over the lifecycle of the task including `enter`, `recur`, `clean`, `close`, and `exit`. The `abort` function is only called when a task is cancelled or an exception is raised.

{% image(path="/images/posts/13-hio-doer-uml.png", class="diagram",
    op="fit_width",
    alt="Doer UML Diagram") 
    %}
Doer UML Diagram
{% end %}

### HIO Scheduler – the Doist

At the top of the execution hierarchy in the HIO library you find the `Doist` class, the root scheduler of all task instances, or `Doer` instances. The generator returned from invoking a `Doer` is called a "deed" and is handed over to the `Doist` function. The `Doist` shown below has a list of `deeds` that are these generator functions, classic coroutines, that it runs when the `Doist` is executed.

{% image(path="/images/posts/13-hio-doist-uml.png", class="diagram",
    op="fit_width",
    alt="Doist UML Diagram") 
    %}
Doist UML Diagram
{% end %}

To run a `Doist` you invoke the `.do` function on the `Doist` as shown below in a test adapted from HIO.

```python
def test_doist_doers():
    """
    Test doist.do with .close of deeds
    """
    tock = 0.03125
    doist = doing.Doist(tock=tock)

    # creates a Doer, an example doer 
    doer0 = doing.ExDoer(tock=tock, tymth=doist.tymen())
    # creates a Doer, an example doer 
    doer2 = doing.ExDoer(tock=tock, tymth=doist.tymen())
    doers = [doer0, doer1]

    doist.do(doers=doers)  # run the Doist
    assert doer0.done == True
```

### Context Functions

The six context functions in the `Doer` are run by the `enter` and `exit` functions of the `Doist` as well as the `do` function of the `Doer`. Each of these functions serve as a lifecycle hook for a different time in the execution of the `Doer`. The `.do` function reproduced below shows where each context function is executed after calling `Doer.do`. Take special notice of the `while` loop inside of the `try/except` block. This is the loop that continues to run the body of the `Doer`, the function or generator that does the work of the `Doer`.

```python
# from github.com/ioflo/hio/src/hio/base.doing.py
class Doer(tyming.Tymee):
    ...

    def do(self, tymth, *, tock=0.0, **opts):
        """
        Generator method to run this doer. Calling this method returns generator.
        Interface matches generator function for compatibility.
        To customize create subclasses and override the lifecycle methods:
            .enter, .recur, .exit, .close, .abort

        Parameters:
            tymth is injected function wrapper closure returned by .tymen() of
                Tymist instance. Calling tymth() returns associated Tymist .tyme.
            tock is injected initial tock value
            args is dict of injected optional additional parameters
        """
        try:
            # enter context
            self.wind(tymth)  # update tymist dependencies
            self.tock = tock  #  set tock to parameter
            self.done = False  # allows enter to override completion state
            self.enter()  # (1) first context function, enter

            #recur context
            if isgeneratorfunction(self.recur):  #  .recur is generator method
                self.done = yield from self.recur()  # (2) recur context delegated, second context function
            else:  # .recur is standard method so iterate in while loop
                while (not self.done):  # recur context
                    tyme = (yield (self.tock))  # yields .tock then waits for next send
                    self.done = self.recur(tyme=tyme)   # (2) second context function, recur

        except GeneratorExit:  # close context, forced exit due to .close
            self.close()  # (3) third context function, close

        except Exception as ex:  # abort context, forced exit due to uncaught exception
            self.abort(ex=ex)   # (4) fourth context function, abort
            raise

        else:  # clean context
            self.clean()  # (5) fifth context function, clean

        finally:  # exit context, exit, unforced if normal exit of try, forced otherwise
            self.exit()  # (6) sixth context function, exit

        # return value of yield from or StopIteration.value indicates completion
        return self.done  # Only returns done state if normal return not close or abort raise
```

In the normal execution of a `Doer` the `.do()` function calls, in this order, `enter`, `recur`, `clean`, and then `exit`. The `close` context function is only executed when it is explicitly called by some higher level construct such as a `DoDoer` or the `Doist` scheduler itself.

In an error case, or abnormal execution of a `Doer`, the `abort` context function is called. This can also be called as a part of normal execution of a program to catch a shutdown signal to instruct a `DoDoer` or a `Doist` to perform a graceful shutdown.

### HIO DoDoer – where task hierarchies are defined

This post touches lightly on `DoDoer`s to say that the `DoDoer` provides hierarchical task management which means you can nest tasks for a clear hierarchy of task execution for groups of tasks. A future article will detail the definition and usage of the `DoDoer`.

### AsyncIO vs HIO – How do they compare?

Classic coroutines are very powerful constructs that provide a richer control flow construct as compared to AsyncIO's `async def` coroutine construct. This is because you can use any number of `yield` or `yield from` statements in the body of a classic coroutine, which provides you with the ability to custom-fit the execution of a generator-based coroutine to your specific use case. The `async/await` syntax does a similar thing for you, yet with a standard syntax that you cannot customize.

With HIO you can also repeatedly accept information into a classic coroutine instance through the `yield from` syntax. The fact that classic coroutines are just generator functions means you have full control over iteration of that generator, and all of its contained state including any state it has closed over, from an async context with all the power of Python iterators.

For example, you could run a classic coroutine any arbitrary number of times within a custom scheduler depending on special rules and have fine-grained access to what is sent into the coroutine with the `.send()` function.

Yet with this additional power comes the potential to have complicated and hard to understand control flow. It is understandable why there would be so much support in the Python community for a simpler, less powerful syntax, which is what `async/await` is. The linked article from Luciano Ramalho goes in depth on the features of both classic coroutines and Python's AsyncIO.

## Wrap up and Next Steps

This article focused on the "what" of the async framework side of HIO, specifically the three primary classes at the core of the async runtime in HIO, the `Doist` scheduler, `DoDoer` hierarchical task container, and the `Doer` task class. The raw power of classic coroutines significantly influenced the decision to use them in HIO as well as in KERIpy, KERIA, and SignifyPy. Yet, this is not an either-or, all-or-nothing situation. You can use HIO and AsyncIO together.

Major topics not covered in this article that are important to understand HIO include details of the `DoDoer` and the network and file I/O capabilities of the HIO package.

Future articles will delve deeper into the "why" of HIO, the rationale behind HIO, how and when to use it properly, as well as how to use HIO and AsyncIO together. To gain a deeper understanding of HIO one of your next steps would be to read some of the tests in the HIO source code repository, specifically the `test_doist_once` and `test_nested_doers` tests.

## References

1. S. Smith, "hio/README.md at main · ioflo/hio," *GitHub*, Aug. 21, 2021. [https://github.com/ioflo/hio/blob/main/README.md](https://github.com/ioflo/hio/blob/main/README.md) (accessed May 09, 2024).

2. L. Ramalho, "Classic Coroutines," *Fluent Python, the lizard book*, Apr. 2022. [https://www.fluentpython.com/extra/classic-coroutines](https://www.fluentpython.com/extra/classic-coroutines) (accessed May 11, 2024).

3. Real Python, "Async IO in python: A complete walkthrough," Real Python, [https://realpython.com/async-io-python](https://realpython.com/async-io-python) (accessed May 9, 2024).
