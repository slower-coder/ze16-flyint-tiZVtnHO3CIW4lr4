
Python的`asyncio`模块提供了基于协程（coroutines）的异步编程（asynchronous programming）模型。作为一种高效的编程范式，异步编程允许多个轻量级任务并发执行，且相比传统的多线程模型，具有更低的内存消耗。因此，`asyncio`在需要高并发处理的场景中，尤其是在Web开发、网络请求、API调用和套接字编程等领域，得到了广泛应用。本文将详细介绍如何在Python中使用`asyncio`进行异步编程。


本文在学习`asyncio`的过程中参考了以下文献：


* [asyncio — Asynchronous I/O](https://github.com)
* [Python Asyncio: The Complete Guide](https://github.com)
* [深度解密 asyncio](https://github.com)


目录* [1 异步编程介绍](https://github.com)
	+ [1\.1 什么是异步任务](https://github.com)
	+ [1\.2 Python中的异步编程](https://github.com)
		- [1\.2\.1 非阻塞I/O与异步编程](https://github.com)
		- [1\.2\.2 asyncio模块介绍](https://github.com)
	+ [1\.3 Python并发单元的选择与比较](https://github.com)
* [2 asyncio的使用](https://github.com)
	+ [2\.1 协程的使用](https://github.com)
	+ [2\.2 asyncio任务的使用](https://github.com)
		- [2\.2\.1 asyncio任务创建和运行](https://github.com)
		- [2\.2\.2 asyncio任务状态](https://github.com)
		- [2\.2\.3 asyncio任务获取](https://github.com)
		- [2\.2\.4 asyncio任务等待](https://github.com)
		- [2\.2\.5 asyncio任务保护](https://github.com)
		- [2\.2\.6 asyncio中运行阻塞任务](https://github.com)
	+ [2\.3 异步编程模型](https://github.com):[楚门加速器](https://shexiangshi.org)
		- [2\.3\.1 异步迭代器](https://github.com)
		- [2\.3\.2 异步生成器](https://github.com)
		- [2\.3\.3 异步推导式](https://github.com)
		- [2\.3\.4 异步上下文管理器](https://github.com)
	+ [2\.4 asyncio中的非阻塞流](https://github.com)
		- [2\.4\.1 非阻塞流介绍](https://github.com)
		- [2\.4\.2 使用asyncio检查HTTP状态](https://github.com)
		- [2\.4\.3 asyncio中的流使用示例](https://github.com)
* [3 参考](https://github.com)

# 1 异步编程介绍


异步编程是一种非阻塞的编程范式。在这种范式中，请求和函数调用会在未来某个时刻以某种方式在后台执行。非阻塞意味着当一个请求被发出时，程序不会停下来等待该请求的结果，而是会继续执行后续的操作。当请求的结果准备好时，程序会在适当的时机处理该结果，而不会影响程序其他部分的执行。因此，调用者可以继续执行其他任务，并在结果准备好或需要时，稍后处理已发出的调用结果。


## 1\.1 什么是异步任务


异步操作指的是在程序运行时，有些任务不会立即完成，而是安排在未来某个时刻执行。与同步操作不同，后者要求任务在当前步骤中完成。


**异步函数调用**（Asynchronous Function Call）是实现异步操作的一种方式。这种方式允许程序在等待某些任务完成时，继续执行其他工作，从而避免程序被卡住，提升效率。


通常，异步函数调用会返回一个被称为“未来”（Future）的对象（句柄）。这个对象可以看作是一个指向异步操作结果的标识符。程序可以通过它来查看任务的进度，或者等到任务完成时获取最终结果。这样，程序就可以在等待任务完成时做其他事情，而不是一直停下来等。


结合异步函数调用和Future，就得到了**异步任务**（Asynchronous Task）的概念。异步任务不仅仅是调用一个函数，它还包括了如任务取消、错误处理等更多的内容。这样，程序就可以更加灵活和高效地管理多个任务，提高并发性和整体性能。


简单来说，以下是这几个概念的总结：


* **异步函数调用**：指触发一个函数执行的请求，它会在未来某个时刻开始执行，而不会阻止程序继续做其他事情。
* **Future**：是异步函数调用的一个标识符，允许调用者检查任务的状态，并在任务完成时获取结果。
* **异步任务**：指代一个包含异步函数调用和结果（Future）的集合，用于管理和跟踪整个异步操作的过程。


![](https://gitlab.com/luohenyueji/article_picture_warehouse/-/raw/main/CSDN/%5Bpython%5D%20Python%E5%BC%82%E6%AD%A5%E7%BC%96%E7%A8%8B%E5%BA%93asyncio%E4%BD%BF%E7%94%A8%E6%8C%87%E5%8C%97/img/img1.jpg)


## 1\.2 Python中的异步编程


### 1\.2\.1 非阻塞I/O与异步编程


输入input/输出output，简称I/O，指的是从资源中读取或写入数据。以下是I/O操作的一些典型应用场景：


* 硬盘驱动器：对文件进行读取、写入、追加、重命名、删除等操作。
* 外围设备：鼠标、键盘、屏幕、打印机、串行设备、摄像头等。
* 互联网：下载和上传文件、获取网页、查询RSS等。
* 数据库：执行选择、更新、删除等SQL查询。
* 电子邮件：发送邮件、接收邮件、查询收件箱等。


相较于中央处理器（CPU）执行的计算任务，I/O操作通常具有较低的效率。在程序设计中，I/O请求通常以同步方式实现，即发起I/O请求的线程在数据传输完成之前会被挂起，等待操作完成。这种模式被称为阻塞式I/O（Blocking I/O）。在此模式下，操作系统能够识别线程的阻塞状态，并执行上下文切换，以便调度其他可执行的线程，从而优化CPU资源的利用率。尽管如此，阻塞式I/O操作会导致发起请求的线程或进程在I/O操作完成前无法继续执行。虽然这种设计不会对整个系统的运行造成影响，但它确实会在I/O操作期间暂时阻塞发起请求的线程或进程，影响其响应性和并发处理能力。


作为对阻塞I/O的替代方案，非阻塞I/O提供了更高效的选择。与阻塞I/O类似，非阻塞I/O同样需要底层操作系统的支持，但现代操作系统普遍提供了某种形式的非阻塞I/O功能。通过非阻塞I/O，应用程序可以以异步方式发起读写请求，操作系统将负责处理这些请求，并在数据准备好时通知应用程序。


异步编程（Asynchronous Programming）是一种专门用于处理非阻塞I/O操作的编程方式。与传统的阻塞I/O不同，非阻塞I/O使得系统在发出读写请求后不会等待操作完成，而是可以同时处理其他请求。操作结果或数据会在准备好时返回，或者在需要时提供给程序。因此，非阻塞I/O是实现异步编程的核心技术，通常这两者被统称为异步I/O。


### 1\.2\.2 asyncio模块介绍


在Python中，异步编程泛指非阻塞的请求处理方式，即发起请求后不暂停程序执行，而是继续处理其他任务。Python支持多种异步编程技术，其中部分与并发性紧密相关。为了支持异步编程，Python3\.4版本首次引入了asyncio（asynchronous I/O的缩写）模块，为异步编程提供了基础设施。随后，在Python 3\.5版本中，引入了async/await语法。其中：


* asyncio模块旨在支持异步编程，并提供了底层和高级API。高级API提供了执行异步任务、处理回调、执行I/O操作等工具。而底层API则为高级API提供了支撑，包括事件循环的内部机制、传输协议和策略等。
* async/await语法的引入是为了更好地支持协程，这是asyncio模块中实现并发的核心。因为协程提供了一种轻量级的并发方式，可以让单个线程在多个任务之间高效切换，从而实现并发执行，而无需使用传统的线程或进程。


协程是一种特殊的函数，它能够在执行过程中的多个点被暂停和恢复，实现协作式的多任务处理。与传统的子程序或函数相比，协程提供了更灵活的控制流，允许在多个点进行进入、退出和恢复执行。


目前，asyncio模块是Python异步编程的常用工具，它结合async/await语法和非阻塞I/O操作，为开发者提供了一个全面的异步编程框架。那么为什么要在Python程序使用异步编程：


* 提升并发性能：通过使用协程，asyncio使得程序能够以单线程的方式高效地处理大量并发任务，避免了传统多线程编程中的复杂性和资源消耗。
* 简化异步编程：asyncio提供了一套简洁的异步编程范例，使得编写和维护异步代码变得更加容易，同时也提高了代码的可读性和可维护性。
* 优化I/O操作：asyncio支持非阻塞I/O操作，这意味着程序在等待I/O操作（如文件读写、网络通信等）时，不会阻塞主线程，从而可以同时执行其他任务，显著提高了I/O操作的效率。


## 1\.3 Python并发单元的选择与比较


**线程、进程、协程**


在现代编程中，有效地处理并发是提高程序性能和响应能力的关键。Python提供了多种并发单元，包括线程、进程和协程，每种都有其特定的用途和优势。以下是对这三种并发单元的详细介绍：


1. 线程（Threads）


线程是一种并发单元，Python中由threading模块提供，并得到操作系统的支持。线程适合处理阻塞I/O任务，例如从文件、套接字和设备中进行读写操作。然而，由于全局解释器锁（GIL）的存在，Python中的线程在执行CPU密集型任务时效率不高。


2. 进程（Processes）


进程也是由操作系统支持的并发单元，Python中由multiprocessing模块提供。进程适合执行CPU密集型任务，尤其是那些不需要大量进程间通信的计算任务。与线程相比，进程可以绕过全局解释器锁，因此在处理CPU密集型任务时更为高效。


3. 协程（Coroutines）


协程是Python语言和运行时（标准解释器）提供的并发单元，Python中由asyncio模块进一步支持。相较于线程，程序中可以拥有更多的协程同时运行。协程适用于非阻塞I/O操作，如与子进程和套接字的交互。此外，虽然阻塞I/O和CPU密集型任务并非协程的直接应用场景，但可以通过在后台使用线程和进程以模拟非阻塞的方式执行这些任务。


关于线程、进程、协程的详细介绍见：[进程、线程、协程](https://github.com)。


![https://semfionetworks.com/blog/multi-threading-vs-multi-processing-programming-in-python/](https://gitlab.com/luohenyueji/article_picture_warehouse/-/raw/main/CSDN/%5Bpython%5D%20Python%E5%BC%82%E6%AD%A5%E7%BC%96%E7%A8%8B%E5%BA%93asyncio%E4%BD%BF%E7%94%A8%E6%8C%87%E5%8C%97/img/img2.jpg)


**在Python中使用协程的利弊**


在Python中，使用协程相比于线程和进程有以下几个主要好处：


* 轻量级：协程的创建和切换比线程和进程更高效，消耗的资源更少。
* 避免上下文切换开销：协程切换由程序控制，不依赖操作系统调度，减少了CPU时间消耗。
* 共享内存：协程在同一线程内运行，数据共享更简单，不需要锁和复杂的同步机制。
* 简洁的代码结构：协程使异步代码能够以同步的方式书写，代码更易理解和维护。
* 高并发处理：协程非常适合处理大量I/O密集型任务，能够高效利用单个线程实现并发。


当然在Python中使用协程也存在一些缺点：


* 编程复杂性：协程要求开发者理解异步编程模式，这增加了编程的复杂性，尤其是在编写、测试和调试异步代码时。
* 库支持有限：并非所有Python库都支持异步操作，这可能限制了协程在某些场景下的应用。
* 调试难度：异步代码的调试比同步代码更困难，因为传统的调试工具可能无法有效地跟踪协程的执行流程。
* 错误处理：异步代码中的错误处理比同步代码更为复杂，因为需要处理协程挂起和恢复时的状态。
* 执行机制：根据Python底层设计，协程在执行时是协作式的，一次只能运行一个协程。这种机制类似于全局解释器锁下的线程执行。


# 2 asyncio的使用


## 2\.1 协程的使用


了解协程的创建和运行是学习`asyncio`库的基础，因为`asyncio`正是通过协程实现异步编程的。因此，在学习`asyncio`之前，先掌握协程的基本概念非常重要。


协程是异步编程中实现并发的核心，它是一种特殊的函数，能够在执行过程中暂停并稍后恢复。与传统的函数不同，传统函数只能在一个固定的入口和出口点运行，而协程则允许在多个地方挂起、恢复或退出。这种特性使得协程在执行时可以暂停并等待其他任务完成，比如等待其他协程的执行结果、外部资源的返回（例如网络连接或数据处理），然后再继续执行。正是因为协程具备这种暂停和恢复的能力，它们能够同时执行多个任务，而且能够精确控制任务何时暂停和何时恢复。


**协程的定义**


在Python中，协程可以通过使用`async def`关键字来定义。这种定义方式允许协程接受参数，并在执行完毕后返回一个值，类似于常规函数的行为：



```


|  | # 定义一个协程 |
| --- | --- |
|  | async def custom_coroutine(): |
|  | # 协程体，可以包含异步操作 |
|  | pass |


```

使用`async def`声明的协程被称为“协程函数”，这是一种特殊的函数，其返回值是一个协程对象。协程函数在其内部使用`await`（用于等待另一个协程完成）、`async for`（用于异步迭代）和`async with`（用于异步上下文管理器）等关键字来处理异步操作。如下所示：



```


|  | # 定义一个异步协程 |
| --- | --- |
|  | async def custom_coroutine(): |
|  | # 等待另一个协程执行 |
|  | # await表达式将暂停当前协程的执行，并将控制权转交给被等待的协程，以便其能够执行 |
|  | await asyncio.sleep(1) |


```

**协程的创建**


在定义了协程之后，可以创建具体的协程实例：



```


|  | # 实例化协程 |
| --- | --- |
|  | coroutine_instance = custom_coroutine() |


```

需要注意的是，调用协程函数本身并不会导致任何用户定义的代码被执行，其作用仅限于创建并返回一个`coroutine`对象。`coroutine`对象是Python中的一种特殊对象类型，它提供了如`send()`和`close()`等方法，用于控制协程的执行流程和生命周期管理。


可以通过创建协程实例并调用`type()`函数来报告其类型来演示这一点：



```


|  | import asyncio |
| --- | --- |
|  | # 定义协程 |
|  | async def custom_coroutine(): |
|  | print("运行自定义协程") |
|  | # 等待另一个协程 |
|  | await asyncio.sleep(1) |
|  |  |
|  | # 创建协程 |
|  | coro = custom_coroutine() |
|  | # 检查协程的类型 |
|  | print(type(coro)) |


```

代码运行结果为：



```


|  | <class 'coroutine'> |
| --- | --- |
|  | sys:1: RuntimeWarning: coroutine 'custom_coroutine' was never awaited |


```

该警告是因为在Python中，当定义一个协程函数并调用它时，返回的是一个协程对象，而不是立即执行。这个协程对象需要通过`await`关键字或事件循环来执行。代码中的`print(type(coro))`正确地打印出了协程对象的类型。但是，由于协程没有被await，所以会有一个RuntimeWarning警告。


**协程的运行**


协程可以被定义和创建，但只有在事件循环中才能执行。事件循环是异步应用程序的核心，负责调度异步任务和回调，处理网络 I/O 操作。它还负责协调协程之间的协作和多任务处理。


事件循环的工作方式类似于一个不断运行的“调度器”，它会检查哪些任务已经准备好执行，并按顺序执行这些任务。如果某个任务需要等待，例如等待网络响应或文件读取，事件循环会暂时挂起这个任务，并把控制权交给其他可以继续执行的任务。这样，程序就可以在等待的同时，处理其他任务，从而避免了阻塞操作。


通常，通过调用 `asyncio.run()` 函数启动事件循环。该函数会启动事件循环并接收一个协程作为参数，等待协程执行完成并返回结果。代码示例如下：



```


|  | import asyncio |
| --- | --- |
|  |  |
|  | # 定义协程 |
|  | async def custom_coroutine(): |
|  | print("运行自定义协程") |
|  | # 等待另一个协程 |
|  | await asyncio.sleep(1) |
|  |  |
|  | # 创建协程 |
|  | coro = custom_coroutine() |
|  |  |
|  | # 运行协程 |
|  | asyncio.run(coro) |
|  |  |
|  | # 检查协程的类型 |
|  | print(type(coro)) |


```

代码运行结果为：



```


|  | 运行自定义协程 |
| --- | --- |
|  | <class 'coroutine'> |


```

## 2\.2 asyncio任务的使用


在asyncio框架中，任务（Task）是协程的封装，它将协程交给事件循环调度执行。任务通常由协程创建，并在事件循环中运行，但它独立于协程本身。创建任务时，不需要等待其完成，可以继续执行其他操作。任务对象代表一个将在事件循环中异步执行的操作，借助任务管理，可以更方便地处理异步编程中的复杂场景。


协程是异步操作的基本单元，任务则负责管理和调度这些协程。任务不仅支持同时执行多个协程，还能处理结果、取消任务和捕获异常等。如果只关注协程，而忽视任务，就无法有效调度多个异步操作，也难以管理任务的生命周期。因此，理解任务对于掌握异步编程至关重要。


任务的生命周期可以从多个阶段来描述。首先，任务是由协程（coroutine）创建的，并被安排在事件循环（event loop）中独立执行。随着时间的推移，任务会进入运行状态。在执行过程中，任务可能会因为等待其他协程或任务完成而被挂起（suspended）。任务有可能在正常情况下完成并返回结果，或者由于某些异常而失败。如果有其他协程介入，任务也可能会被取消（canceled）。一旦任务完成，它将无法再被重新执行。因此，任务的生命周期可以总结为以下几个阶段：


1. 创建（Created）：任务被创建，但尚未开始执行。
2. 调度（Scheduled）：任务被安排到事件循环中，准备开始执行。
3. 取消（Canceled）：任务在执行之前或执行过程中被取消。
4. 运行（Running）：任务开始执行，进入活跃状态。
5. 挂起（Suspended）：任务在运行过程中被挂起，等待其他操作完成。
6. 结果（Result）：任务成功完成并返回结果。
7. 异常（Exception）：任务执行过程中遇到错误或异常，导致失败。
8. 完成（Done）：任务无论是否成功，都已结束，无法再执行。


![https://superfastpython.com/asyncio-task-life-cycle/](https://gitlab.com/luohenyueji/article_picture_warehouse/-/raw/main/CSDN/%5Bpython%5D%20Python%E5%BC%82%E6%AD%A5%E7%BC%96%E7%A8%8B%E5%BA%93asyncio%E4%BD%BF%E7%94%A8%E6%8C%87%E5%8C%97/img/img3.jpg)


### 2\.2\.1 asyncio任务创建和运行


**任务的创建**


任务是通过协程实例创建的，因此只能在协程内部创建和调度。可以使用 `asyncio.create_task()` 函数来创建任务，该函数会返回一个 `asyncio.Task` 实例：



```


|  | import asyncio |
| --- | --- |
|  | # 定义协程 |
|  | async def custom_coroutine(): |
|  | # 等待另一个协程 |
|  | await asyncio.sleep(1) |
|  |  |
|  | # 创建协程 |
|  | coro = custom_coroutine() |
|  |  |
|  | # 从协程创建任务 |
|  | # name参数为设置任务的名称 |
|  | task = asyncio.create_task(coro, name='task') |
|  | # 也可以用函数设置任务名称 |
|  | task.set_name('MyTask') |


```

此外，`asyncio.ensure_future`函数也可以用来创建和安排任务，它会确保返回一个Future或Task实例：



```


|  | # 创建并安排任务 |
| --- | --- |
|  | task = asyncio.ensure_future(custom_coroutine()) |


```

当然也可以直接通过事件循环来创建任务，可以使用事件循环对象的`create_task`方法。示例如下：



```


|  | # 获取当前事件循环 |
| --- | --- |
|  | loop = asyncio.get_event_loop() |
|  | # 创建并安排任务 |
|  | task = loop.create_task(custom_coroutine()) |


```

**任务的运行**


在创建任务后，尽管可以使用`create_task`函数将协程安排为独立任务，但任务未必会立即执行。任务的执行依赖于事件循环的调度，它会在其他所有协程执行完成后才会开始。例如，在一个asyncio程序中，若某个协程创建并安排了任务，任务只有在该协程挂起后才有可能开始执行。具体来说，任务的执行通常会等到协程进入休眠、等待其他协程或任务时，才会被事件循环调度执行：



```


|  | import asyncio |
| --- | --- |
|  |  |
|  | # 定义一个简单的异步函数 |
|  | async def my_coroutine(name): |
|  | print(f"任务 {name} 开始执行") |
|  | await asyncio.sleep(1)  # 模拟任务的延时 |
|  | print(f"任务 {name} 完成") |
|  |  |
|  | # 获取事件循环 |
|  | loop = asyncio.get_event_loop() |
|  |  |
|  | print("任务创建之前") |
|  |  |
|  | # 创建任务并加入事件循环 |
|  | task = loop.create_task(my_coroutine("A")) |
|  | print("任务创建之后，任务已加入事件循环") |
|  |  |
|  | # 运行事件循环，直到任务完成 |
|  | loop.run_until_complete(task) |
|  |  |
|  | # 关闭事件循环 |
|  | loop.close() |
|  |  |
|  | print("事件循环已关闭") |


```

代码运行结果为：



```


|  | 任务创建之前 |
| --- | --- |
|  | 任务创建之后，任务已加入事件循环 |
|  | 任务 A 开始执行 |
|  | 任务 A 完成 |
|  | 事件循环已关闭 |


```

**多任务的运行**


`gather`函数可以同时启动多个协程（任务）并发执行，同时将它们存储在一个集合中进行管理。它还支持等待所有协程执行完成，并且可以提供取消操作的功能。


以下是具体的代码示例：



```


|  | # 并发执行多个协程，并返回结果 |
| --- | --- |
|  | # 如果协程没有返回值，则返回None |
|  | results = asyncio.gather(coro1(), coro2()) |


```

或者，使用展开语法：



```


|  | # 使用展开语法收集多个协程 |
| --- | --- |
|  | asyncio.gather(*[coro1(), coro2()]) |


```

需要注意的是，直接传递一个协程列表是无效的：



```


|  | # 直接传递协程列表是不允许的 |
| --- | --- |
|  | asyncio.gather([coro1(), coro2()]) |


```

### 2\.2\.2 asyncio任务状态


本节将介绍以下内容：


* 任务的完成与取消
* 获取任务结果
* 任务异常的处理
* 任务回调函数的使用


![](https://gitlab.com/luohenyueji/article_picture_warehouse/-/raw/main/CSDN/%5Bpython%5D%20Python%E5%BC%82%E6%AD%A5%E7%BC%96%E7%A8%8B%E5%BA%93asyncio%E4%BD%BF%E7%94%A8%E6%8C%87%E5%8C%97/img/img4.jpg)


**任务完成与取消**


在任务创建完成后，需要重点检查两个关键状态：一是任务是否已顺利完成；二是任务是否已被正式取消。可以通过`done()`方法来确认任务是否已完成，通过 `cancelled()`方法来检查任务是否已被取消。示例代码如下：



```


|  | import asyncio |
| --- | --- |
|  |  |
|  | # 异步任务1: 打印任务开始、等待1秒并打印任务完成 |
|  | async def task_completed(): |
|  | print("任务1正在执行") |
|  | await asyncio.sleep(1)  # 模拟异步操作，暂停1秒 |
|  | print("任务1完成") |
|  |  |
|  | # 异步任务2: 打印任务开始、等待2秒并打印任务完成 |
|  | async def task_cancelled(): |
|  | print("任务2正在执行") |
|  | await asyncio.sleep(2)  # 模拟异步操作，暂停2秒 |
|  | print("任务2完成") |
|  |  |
|  | # 主异步函数 |
|  | async def main(): |
|  | # 创建并启动两个异步任务 |
|  | task1 = asyncio.create_task(task_completed())  # 创建任务1 |
|  | task2 = asyncio.create_task(task_cancelled())  # 创建任务2 |
|  |  |
|  | # 等待task1任务完成 |
|  | await task1 |
|  |  |
|  | # 取消task2任务 |
|  | task2.cancel() |
|  |  |
|  | # 异常处理: 捕获任务2被取消的异常 |
|  | try: |
|  | await task2  # 尝试等待task2完成 |
|  | except asyncio.CancelledError: |
|  | # 如果task2被取消 |
|  | pass |
|  |  |
|  | # 检查task1是否完成 |
|  | if task1.done(): |
|  | print("任务1已完成") |
|  |  |
|  | # 检查task2是否被取消 |
|  | if task2.cancelled(): |
|  | print("任务2已取消") |
|  |  |
|  | # 运行主异步函数 |
|  | asyncio.run(main())  # 启动事件循环，执行main函数 |


```

代码运行结果为：



```


|  | 任务1正在执行 |
| --- | --- |
|  | 任务2正在执行 |
|  | 任务1完成 |
|  | 任务1已完成 |
|  | 任务2已取消 |


```

**任务结果获取**


通过调用 `result()` 方法可以获取任务执行的结果。如果任务中包含的协程函数有返回值，则 `result()` 方法将返回该值；若协程函数未显式返回任何值，则默认返回 `None`。


若任务已被取消，在尝试调用 `result()` 方法时会触发 `CancelledError` 异常。因此，建议在调用 `result()` 方法前先检查任务是否已被取消：



```


|  | # 检查任务是否未被取消 |
| --- | --- |
|  | if not task.cancelled(): |
|  | # 获取包装协程的返回值 |
|  | value = task.result() |
|  | else: |
|  | # 任务已被取消 |


```

如果任务尚未完成，在调用 `result()` 方法时会抛出 `InvalidStateError` 异常。因此，在调用 `result()` 方法之前，最好先确认任务是否已完成：



```


|  | # 检查任务是否已完成 |
| --- | --- |
|  | if not task.done(): |
|  | await task |
|  | # 获取包装协程的返回值 |
|  | value = task.result() |


```

**任务异常处理**


可以通过 `exception()` 方法获取协程未处理的异常信息。若任务执行过程中发生异常，使用该方法能够捕获并返回该异常：



```


|  | import asyncio |
| --- | --- |
|  |  |
|  | # 定义一个协程，模拟异常 |
|  | async def faulty_coroutine(): |
|  | raise ValueError("协程中发生了错误。") |
|  |  |
|  | # 主程序 |
|  | async def main(): |
|  | # 创建协程任务 |
|  | task = asyncio.create_task(faulty_coroutine()) |
|  |  |
|  | # 等待任务执行完成，捕获异常 |
|  | try: |
|  | await task |
|  | except Exception as e: |
|  | # 获取任务中的异常 |
|  | exception = task.exception() |
|  | print(f"任务异常: {exception}") |
|  |  |
|  | # 运行事件循环 |
|  | asyncio.run(main()) |


```

在这个示例中，创建了一个协程`faulty_coroutine`，该协程在执行时会引发 `ValueError`异常。通过`task.exception()`方法捕获并打印该异常信息。执行结果将显示：



```


|  | 任务异常: 协程中发生了错误。 |
| --- | --- |


```

**任务的回调函数**


通过`add_done_callback()`方法可以为任务指定一个完成时触发的回调函数。这个方法需要传入一个函数名，该函数将在任务完成时被调用。注意，任务完成可以发生在以下几种情况：包装的协程正常结束、返回结果、抛出未捕获的异常，或者任务被取消。以下是如何定义和注册一个完成回调函数的示例：



```


|  | # 定义完成回调函数 |
| --- | --- |
|  | def handle(task): |
|  | print(task) |
|  |  |
|  | # 为任务注册完成回调函数 |
|  | task.add_done_callback(handle) |


```

同样地，如果需要，可以使用`remove_done_callback()`方法来删除或取消之前注册的回调函数：



```


|  | # 取消注册的回调函数 |
| --- | --- |
|  | task.remove_done_callback(handle) |


```

但是要注意的是回调函数通常是普通的Python函数，无法进行异步操作：



```


|  | import asyncio |
| --- | --- |
|  |  |
|  | # 异步函数 |
|  | async def my_coroutine(): |
|  | print("开始任务") |
|  | await asyncio.sleep(1) |
|  | print("任务完成") |
|  | return "任务完成" |
|  |  |
|  | # 回调函数 |
|  | def my_callback(task): |
|  | print("回调函数被调用") |
|  | result = task.result()  # 获取任务的返回结果 |
|  | print(f"任务的返回结果是: {result}") |
|  |  |
|  | async def main(): |
|  | # 创建任务 |
|  | task = asyncio.create_task(my_coroutine()) |
|  |  |
|  | # 注册回调函数 |
|  | task.add_done_callback(my_callback) |
|  |  |
|  | # 等待任务完成 |
|  | await task |
|  |  |
|  | # 运行主函数 |
|  | asyncio.run(main()) |


```

代码运行结果为：



```


|  | 开始任务 |
| --- | --- |
|  | 任务完成 |
|  | 回调函数被调用 |
|  | 任务的返回结果是: 任务完成 |


```

### 2\.2\.3 asyncio任务获取


**当前任务获取**


可以使用 `asyncio.current_task()` 方法来获取当前正在执行的任务。这个方法会返回一个代表当前任务的 `Task` 对象。以下示例展示了如何在主协程中获取当前任务：



```


|  | # 从当前协程中获取当前任务 |
| --- | --- |
|  | import asyncio |
|  |  |
|  | # 定义主协程 |
|  | async def main(): |
|  | # 输出开始消息 |
|  | print('主协程已启动') |
|  | # 获取当前任务 |
|  | current_task = asyncio.current_task() |
|  | # 打印任务详情 |
|  | print(current_task) |
|  |  |
|  | # 运行主协程 |
|  | asyncio.run(main()) |


```

上述代码打印结果包含任务的名称和正在运行的协程信息：



```


|  | Task pending name='Task-1' coro= |
| --- | --- |


```

**所有任务获取**


可以使用 `asyncio.all_tasks()` 函数来检索asyncio程序中所有已安排和正在执行（尚未完成）的任务。以下示例首先创建了10个任务，每个任务都封装并执行相同的协程。随后，主协程捕获程序中所有已计划或正在运行的任务集合，并输出它们的详细信息：



```


|  | import asyncio |
| --- | --- |
|  |  |
|  | # 10个异步任务 |
|  | async def task_coroutine(value): |
|  | # 输出任务运行信息 |
|  | print(f'任务 {value} 开始运行') |
|  | # 模拟异步等待，使每个任务休眠1秒 |
|  | await asyncio.sleep(1) |
|  |  |
|  | # 输出任务运行信息 |
|  | print(f'任务 {value} 结束运行') |
|  |  |
|  | # 主协程定义 |
|  | async def main(): |
|  | # 输出主协程启动信息 |
|  | print('主协程已启动') |
|  | # 创建10个任务 |
|  | # 注意。任务的执行依赖于事件循环的调度，它会在其他所有协程执行完成后才会开始 |
|  | # 例如当前协程创建了任务，但是任务只有在当前协程挂起后才有可能开始执行 |
|  | started_tasks = [asyncio.create_task(task_coroutine(i), name=f'任务{i}') for i in range(10)] |
|  | # 使得当前的协程（即 main 协程）挂起0.1秒，从而使得子任务运行 |
|  | # 如果没有这句代码，也没有之后的gather函数，子任务会在main函数将要结束时运行，此时事件循环还存在 |
|  | await asyncio.sleep(0.1) |
|  | # 获取所有任务的集合 |
|  | all_tasks = asyncio.all_tasks() |
|  | # 输出所有任务的详细信息 |
|  | for task in all_tasks: |
|  | print(f'> {task.get_name()}, {task.get_coro()}') |
|  | print("等待任务完成") |
|  | # gather会收集传入的所有任务，并阻塞当前协程，直到所有任务都执行完毕 |
|  | # 没有gather函数，这样当前协程会直接结束，导致部分任务未能执行或未完成 |
|  | await asyncio.gather(*started_tasks) |
|  | # gather函数类似于以下代码 |
|  | # 逐个等待每个任务完成 |
|  | # for task in started_tasks: |
|  | #     await task  # 等待单个任务完成 |
|  |  |
|  | # 运行异步程序 |
|  | asyncio.run(main()) |


```

### 2\.2\.4 asyncio任务等待


**asyncio.wait函数**


`asyncio.wait()`函数用于等待多个 asyncio.Task 实例（即封装了协程的任务）完成。它允许配置等待策略，比如等待全部任务、第一个完成或第一个出错的任务。这些任务实际上是 asyncio.Task 类的实例，它们封装了协程，使得协程可以被调度并独立运行，并提供了查询状态和获取结果的接口。


`asyncio.wait()` 函数接收一组可等待对象，通常为 `Task` 实例，或者 `Task` 的列表、字典或集合。该函数会持续等待，直到任务集合中的某些条件得到满足，默认情况下，这些条件是所有任务都已完成。`asyncio.wait()` 返回一个包含两个集合的元组：第一个集合是所有已满足条件的任务对象，称为“完成集”（"done" set）；第二个集合是尚未满足条件的任务对象，称为“待处理集”（"pending" set）。


例如：



```


|  | tasks = [asyncio.create_task(task_coro(i)) for i in range(10)] |
| --- | --- |
|  | # 等待所有任务完成 |
|  | done, pending = await asyncio.wait(tasks) |


```

在上面的示例中，`asyncio.wait()`被加上了 `await`，原因在于从技术角度来看，`asyncio.wait()`是一个返回协程的协程函数。`await`用于暂停异步函数的执行，以便调用 `asyncio.wait`，从而等待所有任务完成。


**等待条件设置**


在`asyncio.wait()`函数中，`return_when`参数允许指定等待的条件，其默认值为`asyncio.ALL_COMPLETED`，意味着只有当所有任务都完成时，才会停止等待并返回结果。如果将`return_when`参数设置为`asyncio.FIRST_COMPLETED`，将等待直到列表中的第一个任务完成。一旦第一个任务完成并从等待集中移除，将继续执行当前代码，但其余的任务将继续执行，不会被取消：



```


|  | import asyncio |
| --- | --- |
|  | import random |
|  |  |
|  | # 模拟一个可能需要不同时间完成的异步任务 |
|  | async def async_task(name, duration): |
|  | print(f"任务 {name} 开始，预计耗时 {duration} 秒") |
|  | await asyncio.sleep(duration) |
|  | print(f"任务 {name} 完成") |
|  | return f"结果 {name}" |
|  |  |
|  | # 主函数，用于启动和管理异步任务 |
|  | async def main(): |
|  | # 创建两个任务，一个快一个慢 |
|  | fast_task = asyncio.create_task(async_task("任务1", random.randint(1, 3)), name='任务1') |
|  | slow_task = asyncio.create_task(async_task("任务2", random.randint(4, 6)), name='任务2') |
|  |  |
|  | # return_when=asyncio.FIRST_COMPLETED表示第一个任务以下代码完成后，会继续后续等待，而不用等待其余任务 |
|  | # done和pending都是集合类型（set），包含完成的任务集合和未完成的集合 |
|  | done, pending = await asyncio.wait([fast_task, slow_task], return_when=asyncio.FIRST_COMPLETED) |
|  |  |
|  | # 处理完成的任务 |
|  | for task in done: |
|  | # 提取结果 |
|  | result = task.result() |
|  | print(f"{task.get_name()}的执行结果：{result}") |
|  |  |
|  | print(f"已完成任务数：{len(done)}") |
|  |  |
|  | # 等待剩余的任务完成（如果需要） |
|  | if len(pending) >0: |
|  | await asyncio.wait(pending) |
|  |  |
|  | # 运行主函数 |
|  | asyncio.run(main()) |


```

此外，可以通过将 `return_when` 参数设置为 `FIRST_EXCEPTION` 来等待第一个因异常失败的任务。如果没有任务因异常失败，`done` 集合将包含所有已完成的任务，且 `wait()` 函数仅在所有任务完成后才会返回结果。


**任务超时**


可以通过`timeout`参数指定等待任务的最大时间（以秒为单位）。如果超时，则函数将返回一个包含当前满足条件的任务子集的元组，例如，如果等待所有任务完成，则返回的是已完成的任务子集。示例代码如下：



```


|  | import asyncio |
| --- | --- |
|  | import random |
|  |  |
|  | # 在新任务中执行的协程 |
|  | async def task_coro(arg): |
|  | # 模拟不同的执行时间 |
|  | value =  random.uniform(0, 2)  # 保证每个任务的执行时间为0到2秒 |
|  | await asyncio.sleep(value) |
|  | print(f'> 任务 {arg} 完成，执行时间 {value:.2f} 秒') |
|  |  |
|  | # 主协程 |
|  | async def main(): |
|  | # 创建多个任务 |
|  | tasks = [asyncio.create_task(task_coro(i)) for i in range(10)] |
|  |  |
|  | # 设置最大等待时间为1秒，超时后返回已完成的任务 |
|  | done, pending = await asyncio.wait(tasks, timeout=1) |
|  |  |
|  | # 打印结果：已完成的任务和待处理的任务 |
|  | print(f'已完成的任务数量: {len(done)}') |
|  | print(f'待处理的任务数量: {len(pending)}') |
|  |  |
|  | # 如果超时后有任务未完成，显示它们的状态 |
|  | if pending: |
|  | print('以下任务未在超时时间内完成：') |
|  | for task in pending: |
|  | print(f'- 任务 {tasks.index(task)}') |
|  |  |
|  | # 启动异步程序 |
|  | asyncio.run(main()) |


```

**单个任务等待**


`asyncio.wait_for()` 函数用于等待协程或任务的完成，并提供超时控制。与 `asyncio.wait()` 不同，`wait_for()` 仅等待一个任务，并且在超时之前会检查该任务是否已完成。如果任务未能在指定的超时时间内完成，函数会抛出 `asyncio.TimeoutError` 异常。如果没有设置超时，函数将一直等待任务完成。


`wait_for()` 会返回一个协程对象，实际执行时需要通过 `await` 显式等待结果，或者将其调度为任务。如果超时，任务将被取消。`wait_for()` 接受两个参数：


1. 第一个参数是待等待的协程或任务。
2. 第二个参数是超时时间（单位为秒），可以是整数或浮点数。如果设置为 `None`，表示没有超时限制。


示例代码如下：



```


|  | from random import random |
| --- | --- |
|  | import asyncio |
|  |  |
|  | # 定义异步函数 task_coro，作为协程任务执行 |
|  | async def task_coro(arg): |
|  | # 生成一个1到2之间的随机数 |
|  | value = 1 + random() |
|  | # 打印接收到的值 |
|  | print(f'>任务接收到 {value}') |
|  | # 异步等待，模拟耗时操作，等待时间由随机数决定 |
|  | await asyncio.sleep(value) |
|  | # 打印任务完成消息 |
|  | print('>任务完成') |
|  |  |
|  | # 定义主协程函数 main |
|  | async def main(): |
|  | # 创建协程任务并传入参数1 |
|  | task = task_coro(1) |
|  | # 尝试执行任务并设置超时为0.2秒 |
|  | try: |
|  | await asyncio.wait_for(task, timeout=0.2) |
|  | except asyncio.TimeoutError: |
|  | # 超时处理：打印任务取消消息 |
|  | print('放弃等待，任务已取消') |
|  | except Exception: |
|  | # 捕获其他可能的异常 |
|  | pass |
|  |  |
|  | # 启动异步程序，运行主协程 |
|  | asyncio.run(main()) |


```

### 2\.2\.5 asyncio任务保护


异步任务可通过调用 `cancel()` 方法取消。将任务包装在 `asyncio.shield()` 中可防止其被取消。`asyncio.shield()` 会将协程或可等待对象包装在一个特殊对象中，吸收所有取消请求。即便外部请求取消，任务仍会继续执行。此功能在异步编程中尤为重要，特别是当某些任务可取消，而其他关键任务需持续运行时。`asyncio.shield()`接受可等待对象并返回一个 `asyncio.Future` 对象，可直接等待该对象或传递给其他任务：



```


|  | # 防止任务被取消 |
| --- | --- |
|  | shielded = asyncio.shield(task) |
|  | # 等待屏蔽任务 |
|  | await shielded |
|  |  |


```

返回的 `Future` 对象可以通过调用 `cancel()`方法来取消。如果内部任务仍在执行，取消请求会被视为成功。例如：



```


|  | # 取消屏蔽任务 |
| --- | --- |
|  | was_canceled = shielded.cancel() |


```

至关重要的是，向`Future`对象发出的取消请求并不会传递给其内部任务：



```


|  | import asyncio |
| --- | --- |
|  |  |
|  | async def coro(): |
|  | print("任务开始") |
|  | await asyncio.sleep(3) |
|  | print("任务完成") |
|  |  |
|  | async def main(): |
|  | # 创建异步任务 |
|  | task = asyncio.create_task(coro()) |
|  |  |
|  | # 使用shield包装任务，以创建一个不可取消的任务 |
|  | shield = asyncio.shield(task) |
|  |  |
|  | # 尝试取消shield，但这不会影响内部的task |
|  | shield.cancel() |
|  |  |
|  | try: |
|  | # 等待任务执行完成 |
|  | await task |
|  | except asyncio.CancelledError: |
|  | print("任务被取消") |
|  |  |
|  | # 启动异步事件循环 |
|  | asyncio.run(main()) |


```

代码运行结果为：



```


|  | 任务开始 |
| --- | --- |
|  | 任务完成 |


```

如果一个正在被屏蔽的任务被取消了，那么取消请求将会传递给`shield`对象，导致`shield`对象也被取消，并且会触发`asyncio.CancelledError` 异常。以下是代码示例：



```


|  | import asyncio |
| --- | --- |
|  |  |
|  | async def coro(): |
|  | print("任务开始") |
|  | await asyncio.sleep(3) |
|  | print("任务完成") |
|  |  |
|  | async def main(): |
|  | # 创建异步任务 |
|  | task = asyncio.create_task(coro()) |
|  |  |
|  | # 使用 shield 包装任务，以创建一个不可取消的任务 |
|  | shield = asyncio.shield(task) |
|  |  |
|  | # 取消task |
|  | task.cancel() |
|  |  |
|  | try: |
|  | # 等待任务执行完成 |
|  | await task |
|  | except asyncio.CancelledError: |
|  | print("任务被取消") |
|  |  |
|  | # 启动异步事件循环 |
|  | asyncio.run(main()) |


```

代码运行结果为：



```


|  | 任务被取消 |
| --- | --- |


```

最后，以下示例展示了如何创建、调度和保护协程任务，首先，创建一个主协程 `main()`，作为应用程序的入口点，并创建一个任务协程，确保任务不会被取消。随后，使用 `asyncio.shield()` 保护任务，将其传递给 `cancel_task()` 协程，在其中模拟`shielded`任务取消请求。主协程等待该任务并捕获 `CancelledError` 异常。任务在运行一段时间后休眠，最终，任务完成并返回结果，`shielded` 任务被标记为取消，而内部任务则正常完成：



```


|  | import asyncio |
| --- | --- |
|  |  |
|  | # 定义一个简单的异步任务，模拟处理逻辑 |
|  | async def simple_task(number): |
|  | await asyncio.sleep(1) |
|  | return number |
|  |  |
|  | # 定义一个异步任务，稍后取消指定任务 |
|  | async def cancel_task(task): |
|  | await asyncio.sleep(0.2) |
|  | was_cancelled = task.cancel() |
|  | print(f'已取消: {was_cancelled}') |
|  |  |
|  | # 主协程，调度其他任务 |
|  | async def main(): |
|  | coro = simple_task(1) |
|  | task = asyncio.create_task(coro) |
|  | shielded = asyncio.shield(task) |
|  |  |
|  | # 创建取消任务的协程 |
|  | asyncio.create_task(cancel_task(shielded)) |
|  |  |
|  | try: |
|  | result = await shielded |
|  | print(f'>获得: {result}') |
|  | except asyncio.CancelledError: |
|  | print('任务已被取消') |
|  |  |
|  | await asyncio.sleep(1) |
|  |  |
|  | print(f'保护任务: {shielded}') |
|  | print(f'任务: {task}') |
|  |  |
|  | # 启动主协程 |
|  | asyncio.run(main()) |


```

### 2\.2\.6 asyncio中运行阻塞任务


asyncio专注于异步编程和非阻塞I/O操作。然而，异步应用中执行阻塞函数调用是不可避免的，原因包括：


* 执行CPU密集型任务，如复杂计算。
* 处理阻塞I/O任务，如文件读写。
* 调用未与asyncio集成的第三方库。阻塞调用会导致事件循环暂停，阻止其他协程运行。


`asyncio` 模块提供了两种方法来在 `asyncio` 程序中执行阻塞调用。第一种方法是使用 `asyncio.to_thread()` 函数，它是一个高级API，专为应用程序开发者设计。`asyncio.to_thread()`在单独的线程中执行并返回一个协程。这个协程可以被等待或调度作为独立任务执行。同时`asyncio.to_thread()` 会在后台创建一个 `ThreadPoolExecutor` 来执行阻塞操作，因此它适用于 I/O 密集型任务。


在以下代码中，`asyncio.to_thread`的作用是将一个阻塞的同步任务（即 `blocking_task()` 函数）封装成异步任务，并将其交给一个独立的线程池运行。这样做的目的是避免阻塞主事件循环，从而确保其他异步任务可以继续执行。具体而言，`blocking_task` 是一个同步函数，其中的 `time.sleep(2)` 会阻塞当前线程 2 秒。由于线程在这段时间内无法执行其他任务，如果在传统的 `asyncio` 环境中直接调用阻塞函数，事件循环会被暂停，无法继续调度其他任务。但是`asyncio.to_thread` 的使用确保了阻塞任务不会影响到主事件循环的执行：



```


|  | import asyncio |
| --- | --- |
|  | import time |
|  |  |
|  | # 定义一个阻塞 IO 绑定任务 |
|  | def blocking_task(): |
|  | # 报告任务开始 |
|  | print('任务开始') |
|  | # 模拟阻塞操作：休眠2秒 |
|  | time.sleep(2)  # 这里的 time.sleep 是阻塞当前线程的操作 |
|  | # 报告任务结束 |
|  | print('任务完成') |
|  |  |
|  | # 主协程 |
|  | async def main(): |
|  | # 报告主协程正在运行并启动阻塞任务 |
|  | print('主协程正在执行阻塞任务') |
|  |  |
|  | # 将阻塞任务封装成协程并通过asyncio.to_thread函数运行 |
|  | # asyncio.to_thread 会将阻塞任务分配给一个独立的线程池来执行 |
|  | coro = asyncio.to_thread(blocking_task) |
|  |  |
|  | # 创建一个 asyncio 任务来执行上述协程 |
|  | # asyncio.create_task 将协程封装为 Task 对象，使其能够在后台执行 |
|  | task = asyncio.create_task(coro) |
|  |  |
|  | # 主协程继续执行其他事情 |
|  | print('主协程正在做其他事情') |
|  |  |
|  | # 使用 await asyncio.sleep(0) 允许任务被调度 |
|  | # 这个操作确保协程任务有机会开始执行，因为 main() 协程的执行会在此处暂停 |
|  | await asyncio.sleep(0)  # 让出控制权，确保 task 能被执行 |
|  |  |
|  | # 等待任务执行完成 |
|  | await task |
|  |  |
|  | # 运行异步程序 |
|  | # asyncio.run(main()) 负责启动整个异步事件循环并执行 main() 协程 |
|  | asyncio.run(main()) |


```

另一种方法是使用 `loop.run_in_executor()` 函数，首先通过 `asyncio.get_running_loop()` 获取当前事件循环。`loop.run_in_executor()` 函数接收一个执行器和一个要执行的函数，如果传入 `None` 作为执行器参数，则默认使用 `ThreadPoolExecutor`。该函数返回一个可等待对象，可以选择等待它，且任务会立即开始执行，因此不需要额外等待或安排返回的可等待对象来启动阻塞调用。示例如下：



```


|  | # 获取事件循环 |
| --- | --- |
|  | loop = asyncio.get_running_loop() |
|  | # 在单独的线程中执行函数 |
|  | await loop.run_in_executor(None, task) |


```

或者，可以创建一个执行器并将其传递给`loop.run_in_executor()`函数，该函数将在执行器中执行异步调用。在这种情况下，调用者必须管理执行器，在调用者完成后将其关闭：



```


|  | # 创建进程池 |
| --- | --- |
|  | with ProcessPoolExecutor as exe: |
|  | # 获取事件循环 |
|  | loop = asyncio.get_running_loop() |
|  | # 在单独的线程中执行函数 |
|  | await loop.run_in_executor(exe, task) |
|  | # 进程池自动关闭... |


```

## 2\.3 异步编程模型


### 2\.3\.1 异步迭代器


迭代是Python中的基本操作，`asyncio`提供了对异步迭代器的支持。通过定义实现`__aiter__`和`__anext__`方法的对象，能够在`asyncio`程序中创建并使用异步迭代器（Asynchronous Iterators）。


**迭代器**


迭代器是实现了迭代协议的Python对象。具体而言，`__iter__()`方法返回迭代器自身，而`__next__()`方法使迭代器前进并返回下一个元素。当没有更多数据时，迭代器会引发`StopIteration`异常。可以通过内置函数`next()`逐步获取迭代器中的元素，或者使用`for`循环自动遍历迭代器：



```


|  | # 定义一个名为 MyIterator 的迭代器类 |
| --- | --- |
|  | class MyIterator: |
|  | # 初始化方法，接收两个参数：start 和 end，定义迭代的起始值和结束值 |
|  | def __init__(self, start, end): |
|  | self.current = start  # 设置当前迭代的位置，初始为 start |
|  | self.end = end        # 设置迭代的结束值 |
|  |  |
|  | # 定义迭代器的 __iter__ 方法，返回迭代器自身 |
|  | def __iter__(self): |
|  | return self  # 迭代器对象自身是可迭代的 |
|  |  |
|  | # 定义迭代器的 __next__ 方法，用于获取下一个值 |
|  | def __next__(self): |
|  | # 如果当前值已经达到或超过结束值，抛出 StopIteration 异常，表示迭代结束 |
|  | if self.current >= self.end: |
|  | raise StopIteration |
|  | self.current += 1  # 将当前值加 1 |
|  | return self.current - 1  # 返回当前值，减去 1 是因为在加 1 后，当前值已经递增过 |
|  |  |
|  | # 创建 MyIterator 类的一个实例，从 2开始，结束值为 5 |
|  | my_iter = MyIterator(2, 5) |
|  |  |
|  | # 逐步获取元素，调用 next(my_iter) 获取迭代器中的下一个元素 |
|  | print(next(my_iter))  # 输出 2 |
|  | print(next(my_iter))  # 输出 3 |
|  |  |
|  | # 使用 for 循环遍历MyIterator类实例，这里会自动调用 __iter__ 和 __next__ 方法 |
|  | # MyIterator(0, 3)创建一个新的迭代器实例，迭代的范围是从 0 到 3（不包含 3） |
|  | for number in MyIterator(0, 3): |
|  | print(number) |


```

**异步迭代器**


异步迭代器是实现了`__aiter__()`和`__anext__()`方法的Python对象。`__aiter__()`返回迭代器实例，`__anext__()`返回一个可等待对象，用于执行迭代步骤。异步迭代器只能在`asyncio`程序中使用，可以通过`async for`表达式遍历，自动调用`__anext__()`并等待其结果。与普通的`for`循环不同，`async for`适用于处理异步操作，如网络请求或文件读取。


要创建异步迭代器，只需定义实现这两个方法的类。`__anext__()`必须返回一个可等待对象，并使用`async def`定义。迭代结束时，`__anext__()`应抛出`StopAsyncIteration`异常。由于异步迭代器依赖`asyncio`事件循环，迭代过程中的每个对象都在协程中执行并等待：



```


|  | # 定义一个异步迭代器 |
| --- | --- |
|  | class AsyncIterator(): |
|  | # 构造函数，初始化一些状态 |
|  | def __init__(self): |
|  | self.counter = 0 |
|  |  |
|  | # 实现迭代器协议的 __aiter__方法 |
|  | def __aiter__(self): |
|  | return self |
|  |  |
|  | # 实现异步的 __anext__ 方法 |
|  | async def __anext__(self): |
|  | # 如果没有更多项目，抛出StopAsyncIteration异常 |
|  | if self.counter >= 10: |
|  | raise StopAsyncIteration |
|  | # 增加计数器 |
|  | self.counter += 1 |
|  | # 返回当前计数器值 |
|  | return self.counter |
|  | # 创建迭代器 |
|  | it = AsyncIterator() |


```

通过使用`async for`表达式在循环中遍历异步迭代器，该表达式将自动等待循环的每次迭代：



```


|  | import asyncio |
| --- | --- |
|  | it = AsyncIterator() |
|  |  |
|  | async def main(): |
|  | # 遍历异步迭代器 |
|  | async for result in AsyncIterator(): |
|  | print(result) |
|  |  |
|  | # 启动异步任务 |
|  | asyncio.run(main()) |


```

如果使用的是Python 3\.10或更高版本，可以使用`anext`内置函数遍历迭代器的一步，就像使用`next`函数的经典迭代器一样：



```


|  | import asyncio |
| --- | --- |
|  | # 获取迭代器一步的等待 |
|  | awaitable = anext(it) |
|  | # 执行迭代器的一步并得到结果 |
|  | result = await awaitable |


```

### 2\.3\.2 异步生成器


生成器是Python的基本组成部分，指的是包含至少一个`yield`表达式的函数。与常规函数不同，生成器函数可以在执行过程中暂停，并在后续恢复执行，这种特性与协程相似。实际上，Python中的协程是生成器的扩展。通过`asyncio`库，能够实现异步生成器，而异步生成器（Asynchronous Generators）则是基于协程中`yield`表达式的应用。


**生成器**


生成器是一个Python函数，它通过 `yield` 表达式逐步返回值。每当生成器遇到 `yield` 时，它会返回一个值并暂停执行。下一次调用生成器时，它会从暂停的位置继续执行，直到再次遇到 `yield`。虽然生成器可以通过内置的 `next()` 函数逐步执行，但通常更常见的做法是使用迭代器，如 `for` 循环或列表推导式，来遍历生成器并获取所有返回的值：



```


|  | # 定义一个简单的生成器函数 |
| --- | --- |
|  | def count_up_to(max): |
|  | count = 1 |
|  | while count <= max: |
|  | yield count  # 暂停并返回当前值 |
|  | count += 1 |
|  |  |
|  | # 使用 next() 函数逐步执行生成器 |
|  | counter = count_up_to(5) |
|  | print(next(counter))  # 输出 1 |
|  | print(next(counter))  # 输出 2 |
|  | print(next(counter))  # 输出 3 |
|  |  |
|  | # 使用 for 循环遍历生成器 |
|  | for num in count_up_to(5): |
|  | print(num)  # 输出 1, 2, 3, 4, 5 |
|  |  |
|  | # 也可以使用列表推导式将生成器的结果转为列表 |
|  | numbers = [num for num in count_up_to(5)] |
|  | print(numbers)  # 输出 [1, 2, 3, 4, 5] |


```

**异步生成器**


异步生成器是使用 `yield` 表达式的特殊协程，与普通生成器不同，它能在运行时暂停并等待其他协程或任务完成。异步生成器函数创建一个异步迭代器，无法通过 `next()` 遍历，而需使用 `anext()` 获取下一个值。这使得异步生成器支持 `async for` 语法，每次迭代时都会暂停，等待任务完成后继续执行。简单来说，异步生成器在每次迭代时像一个等待中的任务，`async for` 会调度并等待其结果。


异步生成器通过定义一个包含至少一个 `yield` 表达式的协程实现，且该函数需使用 `async def` 语法。由于它本质上是一个协程，每个迭代返回的是一个等待对象，这些对象在 `asyncio` 事件循环中被调度执行，可以在生成器中等待这些对象：



```


|  | # 定义一个异步生成器 |
| --- | --- |
|  | async def async_generator(): |
|  | for i in range(10): |
|  | # 暂停并睡眠一会儿 |
|  | await asyncio.sleep(1) |
|  | # 向调用者产生一个值 |
|  | yield i |
|  |  |
|  | # 创建迭代器 |
|  | gen = async_generator() |


```

可以使用 `async for` 表达式在循环中遍历异步生成器，该表达式会自动等待每次迭代的结果：



```


|  | # 异步运行的入口点 |
| --- | --- |
|  | import asyncio |
|  | async def main(): |
|  | # 异步迭代生成器并打印结果 |
|  | async for result in async_generator(): |
|  | print(result) |
|  |  |
|  | # 使用列表推导式收集所有结果 |
|  | results = [item async for item in async_generator()] |
|  | print(results) |
|  |  |
|  | asyncio.run(main()) |


```

如果使用的是 Python 3\.10 或更高版本，可以使用 anext() 内置函数遍历迭代器的一步，就像使用 next() 函数的经典迭代器一样：



```


|  | import asyncio |
| --- | --- |
|  | # 获取生成器一步的等待值 |
|  | awaitable = anext(gen) |
|  | # 执行生成器的一步并得到结果 |
|  | result = await awaitable |


```

### 2\.3\.3 异步推导式


异步推导式（Asynchronous Comprehensions）是经典推导式的异步版本，专门用于处理异步迭代和异步操作。`asyncio`支持两种类型的异步推导式，分别是`async for`推导式和`await`推导式。


**推导式**


推导式是一种通过简洁的语法构建数据集合（如列表、字典和集合）的方法。它允许在单行代码中结合 `for` 循环和条件语句，快速创建并填充数据结构。推导式通常由三部分组成：


1. 输出表达式：表示生成的元素。
2. for循环：用于遍历可迭代对象。
3. 条件表达式（可选）：用于过滤元素，仅保留符合条件的部分。


列表推导式可以通过 `for` 表达式从一个新列表中生成元素。例如：



```


|  | # 使用列表推导式创建列表 |
| --- | --- |
|  | result = [a * 2 for a in range(100)] |


```

此外，推导式还可以用于创建字典和集合。例如：



```


|  | # 使用推导式创建字典 |
| --- | --- |
|  | result = {a: i for a, i in zip(['a', 'b', 'c'], range(3))} |
|  | # 使用推导式创建集合 |
|  | result = {a for a in [1, 2, 3, 2, 3, 1, 5, 4]} |


```

**async for异步推导式**


异步推导式可通过带异步迭代的`async for`创建列表、集合或字典。该表达式按需生成协程或任务，获取其结果并存入目标容器。需要注意，`async for`仅能在协程或任务中使用。异步迭代器返回可等待对象，`async for`用于遍历并获取每个对象的结果，在内部`async for`自动等待并调度协程。异步生成器实现异步迭代器方法，亦可用于异步推导式。代码示例如下：



```


|  | import asyncio |
| --- | --- |
|  |  |
|  | # 异步函数，模拟异步操作 |
|  | async def get_data(): |
|  | await asyncio.sleep(1)  # 模拟异步等待 |
|  | return [1, 2, 3, 4, 5] |
|  |  |
|  | # 异步推导式创建列表 |
|  | async def async_list_comprehension(): |
|  | async_data = await get_data() |
|  | async_list = [x * 2 for x in async_data] |
|  | return async_list |
|  |  |
|  | # 异步推导式创建集合 |
|  | async def async_set_comprehension(): |
|  | async_data = await get_data() |
|  | async_set = {x * 2 for x in async_data} |
|  | return async_set |
|  |  |
|  | # 异步推导式创建字典 |
|  | async def async_dict_comprehension(): |
|  | async_data = await get_data() |
|  | async_dict = {x: x * 2 for x in async_data} |
|  | return async_dict |
|  |  |
|  | # 运行异步推导式 |
|  | async def main(): |
|  | list_result = await async_list_comprehension() |
|  | set_result = await async_set_comprehension() |
|  | dict_result = await async_dict_comprehension() |
|  |  |
|  | print("Async List:", list_result) |
|  | print("Async Set:", set_result) |
|  | print("Async Dict:", dict_result) |
|  |  |
|  | # 启动事件循环 |
|  | asyncio.run(main()) |


```

**await异步推导式**


`await` 表达式不仅适用于常规异步操作，还可用于列表、集合或字典推导式，称为 await 推导。无论在异步还是同步代码中，建议统一使用 await 推导或列表推导。与异步推导式类似，await 推导仅能在异步协程或任务中使用。该机制通过挂起并等待一系列可等待对象，构建数据结构（如列表）。在当前协程中，这些可等待对象按顺序执行：



```


|  | import asyncio |
| --- | --- |
|  | async def fetch_data(item): |
|  | # 模拟异步数据获取操作 |
|  | await asyncio.sleep(1)  # 模拟异步等待 |
|  | return f"Data for {item}" |
|  |  |
|  | async def main(): |
|  | # 创建一个包含可等待对象的列表 |
|  | awaitables = [fetch_data(item) for item in range(5)] |
|  |  |
|  | # 使用 await 推导式构建列表 |
|  | results = [await x for x in awaitables] |
|  |  |
|  | # 打印结果 |
|  | print(results) |
|  |  |
|  | # 运行主函数 |
|  | asyncio.run(main()) |


```

### 2\.3\.4 异步上下文管理器


上下文管理器是Python中的一种结构，它提供了类似`try-finally`的环境，具有统一的接口和简洁的语法，通常通过`with`语句来使用。上下文管理器常用于资源管理，确保资源在使用后能够被正确关闭或释放，无论使用过程是否成功，或是否因异常而导致失败。`asyncio`库支持开发异步上下文管理器。在`asyncio`程序中，异步上下文管理器（asynchronous ContextManagers）可以通过定义一个实现了`__aenter__()`和`__aexit__()`方法的协程对象来创建和使用。


**上下文管理器**


上下文管理器是 Python 中定义了 `__enter__` 和 `__exit__` 方法的对象，它们分别负责在 `with` 语句块开始和结束时执行特定的操作。这一机制使得在进入和退出特定代码块时，能够自动管理资源的生命周期，如文件、套接字或线程池的打开与关闭。通过上下文管理器，可以有效地管理资源，提升代码的安全性与可读性。


通过 `with` 语句使用上下文管理器，可在代码块执行前后自动完成资源的准备与清理工作。上下文管理器对象通常在 `with` 语句中创建，并自动触发 `__enter__` 方法。无论代码块是正常结束还是因异常退出，`__exit__` 方法都会被自动调用。


例如：



```


|  | # 使用上下文管理器 |
| --- | --- |
|  | with ContextManager() as manager: |
|  | # 在此处执行代码块 |
|  | # 自动管理资源关闭 |
|  | # 这与 try-finally 结构类似 |


```

或者，也可以手动创建对象并调用这些方法：



```


|  | # 创建对象 |
| --- | --- |
|  | manager = ContextManager() |
|  | try: |
|  | manager.__enter__() |
|  | # 执行代码块 |
|  | finally: |
|  | manager.__exit__() |


```

**异步上下文管理器**


异步上下文管理器指的是能够在其 `__aenter__` 和 `__aexit__` 方法中挂起执行的上下文管理器。这两个方法被定义为协程，并由调用者进行等待。通过 `async with` 表达式，可以实现这一功能。因此，异步上下文管理器通常用于asyncio程序中，尤其是在协程调用时。`async with` 表达式是对传统 `with` 表达式的扩展，专门用于异步上下文管理器，允许在协程中进行异步操作，其使用方式与同步的 `with` 表达式相似，但能够处理异步任务。下面是一个定义异步上下文管理器的例子：



```


|  | import asyncio |
| --- | --- |
|  | # 定义异步上下文管理器 |
|  | class AsyncContextManager: |
|  | # 进入异步上下文管理器 |
|  | async def __aenter__(self): |
|  | # 报告消息 |
|  | print('> entering the context manager') |
|  | # 模拟异步操作，暂时阻塞 |
|  | await asyncio.sleep(0.5) |
|  |  |
|  | # 退出异步上下文管理器 |
|  | async def __aexit__(self, exc_type, exc, tb): |
|  | # 报告消息 |
|  | print('> exiting the context manager') |
|  | # 模拟异步操作，暂时阻塞 |
|  | await asyncio.sleep(0.5) |


```

在使用异步上下文管理器时，通过 `async with` 表达式来调用它。这不仅会自动等待进入和退出协程，还会确保在执行过程中暂停当前协程，直到相关的异步操作完成：



```


|  | # 使用异步上下文管理器 |
| --- | --- |
|  | async with AsyncContextManager() as manager: |
|  | # 在此块中执行一些异步任务 |
|  | # ... |


```

以下示例展示了在 asyncio 程序中异步上下文管理器的常见使用模式，首先会创建 `main()` 协程，并将其作为 asyncio 程序的入口点。`main()` 协程执行时，创建了一个 `AsyncContextManager`类的实例，并在 `async with` 表达式中使用它：



```


|  | import asyncio |
| --- | --- |
|  |  |
|  | # 定义异步上下文管理器 |
|  | class AsyncContextManager: |
|  | # 进入异步上下文管理器 |
|  | async def __aenter__(self): |
|  | # 报告消息 |
|  | print('>进入上下文管理器') |
|  | # 暂停一段时间 |
|  | await asyncio.sleep(0.5) |
|  |  |
|  | # 退出异步上下文管理器 |
|  | async def __aexit__(self, exc_type, exc, tb): |
|  | # 报告消息 |
|  | print('>退出上下文管理器') |
|  | # 暂停一段时间 |
|  | await asyncio.sleep(0.5) |
|  |  |
|  | # 定义一个简单的协程 |
|  | async def custom_coroutine(): |
|  | # 创建并使用异步上下文管理器 |
|  | async with AsyncContextManager() as manager: |
|  | # 输出当前状态 |
|  | print(f'在上下文管理器内部') |
|  |  |
|  | # 启动异步程序 |
|  | asyncio.run(custom_coroutine()) |


```

代码运行结果为：



```


|  | >进入上下文管理器 |
| --- | --- |
|  | 在上下文管理器内部 |
|  | >退出上下文管理器 |


```

## 2\.4 asyncio中的非阻塞流


### 2\.4\.1 非阻塞流介绍


`asyncio`的一个重要特点是能够在进行网络操作时避免阻塞，这意味着在等待数据的过程中，程序仍然可以继续执行其他任务。这一功能通过“流”（streams）来实现，流就像一个管道，用于收发数据。借助流，数据的发送和接收变得更加简便，无需依赖复杂的回调函数或底层实现细节。


具体来说，`asyncio`中的`asyncio streams`支持通过网络连接创建“写”流和“读”流。在这些流中，可以执行数据写入和读取操作，且在等待期间程序不会被某个操作卡住。操作完成后，网络连接即可关闭。尽管在使用流功能时需要自行处理一些网络协议的细节，这种方式仍然能够支持许多常见的网络协议，例如：


* 与网站服务器通信的HTTP或HTTPS协议。
* 用于发送电子邮件的SMTP协议。
* 用于文件传输的FTP协议。


流不仅可以用于创建服务器并处理标准协议的请求，还能帮助开发者定制协议，以满足特定应用需求。接下来，将介绍如何使用异步流：


**打开连接**


可以使用 `asyncio.open_connection()` 函数打开 asyncio TCP 客户端套接字连接，建立网络连接并返回一对（reader、writer）对象。这些返回的对象是 `StreamReader` 和 `StreamWriter` 类的实例，用于与套接字交互。该函数是一个必须等待的协程，一旦套接字连接打开便返回。


例如：



```


|  | # 打开一个连接 |
| --- | --- |
|  | reader, writer = await asyncio.open_connection(...) |


```

`asyncio.open_connection()` 函数需要许多参数来配置套接字连接，其中两个必需的参数是主机和端口：


* 主机是一个字符串，指定要连接的服务器，例如域名或 IP 地址。
* 端口是套接字端口号，例如HTTP服务器为80，HTTPS 服务器为 443，SMTP为25 等。


例如，打开与 HTTP 服务器的连接：



```


|  | # 打开与 http 服务器的连接 |
| --- | --- |
|  | reader, writer = await asyncio.open_connection('www.baidu.com', 80) |


```

如果需要加密套接字连接（如 HTTPS），可以通过设置 `ssl=True` 实现 SSL 协议支持：



```


|  | # 打开与 https 服务器的连接 |
| --- | --- |
|  | reader, writer = await asyncio.open_connection('www.baidu.com', 443, ssl=True) |


```

**启动侦听服务**


要启动一个异步的TCP服务器，可以使用`asyncio.start_server()`函数。这个函数会创建一个服务器，它会在指定的地址和端口上监听来自客户端的连接请求。这个函数是一个需要等待的协程，当调用时，它会返回一个`asyncio.Server`对象，代表正在运行的服务器。


下面是如何使用这个函数的一个示例：



```


|  | # 启动一个 TCP 服务器 |
| --- | --- |
|  | server = await asyncio.start_server(...) |


```

这个函数需要三个参数：一个处理连接的函数、服务器的地址和端口号。处理连接的函数是一个用户自定义的函数，每当有客户端连接到服务器时，这个函数就会被调用。这个函数会接收一对对象作为参数，这两个对象分别用于从客户端读取数据（`StreamReader`）和向客户端发送数据（`StreamWriter`）。


**地址**是指客户端用来连接服务器的域名或IP地址，**端口号**则是服务器用来接收连接请求的网络端口，不同的服务通常会使用不同的端口，比如FTP服务常用端口21，而HTTP服务常用端口80。


下面是一个具体的使用示例：



```


|  | # 定义一个处理客户端连接的函数 |
| --- | --- |
|  | async def handler(reader, writer): |
|  | # 在这里添加处理客户端请求的逻辑 |
|  | pass |
|  |  |
|  | # 使用指定的处理器、地址和端口号启动服务器 |
|  | server = await asyncio.start_server(handler, '127.0.0.1', 80) |


```

在这个例子中，`handler`函数将负责处理每个客户端的连接，`'127.0.0.1'`是本地回环地址，意味着服务器将只在本地计算机上可用，而`80`是HTTP服务的标准端口号。


**使用StreamWriter写入数据**


数据可以通过`asyncio.StreamWriter`写入套接字，套接字是计算机之间通过网络传输数据的通信端点。`StreamWriter`提供API将字节数据写入套接字连接的I/O流，数据会尝试立即发送到目标设备，若无法立即发送，则存储在缓冲区。写入后，最好使用`drain`方法清空缓冲区：



```


|  | # 写入字节数据 |
| --- | --- |
|  | writer.write(byte_data) |
|  | # 等待数据传输 |
|  | await writer.drain() |


```

**使用StreamReader读取数据**


数据可以通过`asyncio.StreamReader`从套接字中读取。读取的数据是字节格式，因此在使用之前可能需要进行编码。所有读取操作都是必须等待的协程。可以使用`read()`方法读取任意数量的字节，该方法会一直读取，直到文件末尾（EOF）：



```


|  | # 读取字节数据 |
| --- | --- |
|  | byte_data = await reader.read() |


```

也可以通过`n`参数指定要读取的字节数：



```


|  | # 读取指定字节数的数据 |
| --- | --- |
|  | byte_data = await reader.read(n=100) |


```

使用`readline`方法可以读取单行数据，直到遇到新行字符`\n`或文件末尾（EOF），返回的是字节数据：



```


|  | # 读取一行数据 |
| --- | --- |
|  | byte_line = await reader.readline() |


```

此外，`readexactly()`方法用于读取确切数量的字节，如果读取的字节数不足，则会引发异常。而`readuntil()`方法会读取字节数据，直到遇到指定的字节字符为止。


**关闭连接**


可以通过`asyncio.StreamWriter`来关闭套接字。调用`close()`方法即可关闭套接字，该方法不会阻塞：



```


|  | #关闭套接字 |
| --- | --- |
|  | writer.close() |


```

尽管`close()`方法不会阻塞，但可以通过`wait_close()`方法等待套接字完全关闭后再继续操作：



```


|  | #关闭套接字 |
| --- | --- |
|  | writer.close() |
|  | #等待套接字关闭 |
|  | awaitwriter.wait_closed() |


```

也可以通过`is_closing()`方法检查套接字是否已经关闭或正在关闭过程中：



```


|  | #检查套接字是否已关闭或正在关闭 |
| --- | --- |
|  | ifwriter.is_closing(): |
|  | #... |


```

### 2\.4\.2 使用asyncio检查HTTP状态


本节介绍如何使用 `asyncio` 模块通过打开流并进行HTTP请求和响应的读写操作，整个过程通常包括以下四个步骤：


1. 打开连接
2. 发送请求
3. 读取响应
4. 关闭连接


专业的异步HTTP框架可以参考使用：[aiohttp](https://github.com)。


**打开链接**


使用 `asyncio.open_connection()` 函数打开连接。该函数接受主机名和端口号作为参数，并返回一个 `StreamReader` 和 `StreamWriter`，用于通过套接字进行数据的读写。这些功能通常用于在端口 80 上打开 HTTP 连接。


**发送HTTP请求**


在打开HTTSP连接后，可以向`StreamWriter`写入查询以发出 HTTP 请求。以HTTP版本1\.1 请求为例，HTTP请求的格式为纯文本，可以请求根路径“/”，其示例如下：



```


|  | GET / HTTP/1.1 |
| --- | --- |
|  | Host: www.google.com |


```

HTTP协议请求的具体介绍见：[HTTP协议请求/响应格式详解](https://github.com)。需要注意的是，每行末尾必须包含回车符和换行符（\\r\\n），且请求的末尾需有一个空行。若作为 Python 字符串表示，格式如下所示：



```


|  | 'GET / HTTP/1.1\r\n' |
| --- | --- |
|  | 'Host: www.google.com\r\n' |
|  | '\r\n' |


```

在写入 `StreamWriter` 之前，必须将该字符串编码为字节。可以通过调用 `encode()` 方法实现字符串编码，默认的“utf\-8”编码通常适用。例如：



```


|  | # 将字符串编码为字节 |
| --- | --- |
|  | byte_data = string.encode() |


```

接着，可以使用 `StreamWriter` 的 `write()` 方法将字节数据写入套接字。例如：



```


|  | # 将查询写入套接字 |
| --- | --- |
|  | # 等待套接字准备好 |
|  | await writer.drain() |


```

**读取HTTP响应**


发出HTTP请求后，可以读取响应。此操作可通过套接字的`StreamReader`实现。使用`read()`方法可一次读取一大块字节，或者使用`readline()`方法逐行读取字节。由于基于文本的HTTP协议通常每次发送一行HTML数据，因此`readline()`方法更加便捷。需要注意的是，`readline()`是一个协程，调用时需要等待其执行完成。示例如下：



```


|  | #读取一行响应 |
| --- | --- |
|  | line_bytes=awaitreader.readline() |


```

HTTP/1\.1响应由标头和正文两部分组成，二者通过空行分隔。标头部分包含关于请求是否成功以及即将发送的文件类型的信息，正文则包含文件内容，如HTML网页。HTTP标头的第一行通常表示请求页面的HTTP状态。每一行数据需要从字节解码为字符串，通常使用`decode()`方法，默认编码为"utf\_8"。示例如下：



```


|  | #将字节解码为字符串 |
| --- | --- |
|  | line_data=line_bytes.decode() |


```

**关闭HTTP连接**


可以通过调用`close()`方法关闭`StreamWriter`，从而关闭套接字连接。例如：



```


|  | #关闭连接 |
| --- | --- |
|  | writer.close() |


```

此操作不会阻塞，并且可能不会立即关闭套接字。


### 2\.4\.3 asyncio中的流使用示例


本节介绍了一个用于检查网站状态的示例，代码实现了一个异步HTTP状态码获取工具。通过结合asyncio和urlsplit模块，该工具能够并发请求多个URL，并获取这些URL的HTTP状态行（例如 HTTP/1\.1 200 OK）。代码流程如下：


1. **解析URL**：使用 `urlsplit(url)` 解析 URL，提取协议、主机名和路径等信息。
2. **建立连接**：根据协议选择相应端口（80 或 443），并建立异步网络连接。
3. **发送HTTP请求**：构建并发送简单的HTTP请求报文。
4. **读取响应**：异步读取 HTTP 响应的状态行并返回。
5. **处理异常**：若请求失败，捕获异常并输出错误信息，返回 None。
6. **并发执行任务**：通过 `asyncio.gather()` 实现并发执行所有 URL 请求，等待任务完成并返回结果。
7. **输出结果**：输出每个URL的HTTP状态行或错误信息。


![](https://gitlab.com/luohenyueji/article_picture_warehouse/-/raw/main/CSDN/%5Bpython%5D%20Python%E5%BC%82%E6%AD%A5%E7%BC%96%E7%A8%8B%E5%BA%93asyncio%E4%BD%BF%E7%94%A8%E6%8C%87%E5%8C%97/img/img5.jpg)


示例代码如下：



```


|  | import asyncio |
| --- | --- |
|  | from urllib.parse import urlsplit  # 导入 urlsplit 函数，用于解析 URL |
|  |  |
|  | # 定义一个异步函数，用于获取指定 URL 的 HTTP/S 状态 |
|  | async def get_status(url): |
|  | # 使用 urlsplit 解析 URL，将其分解为各个部分（例如：scheme, hostname, path） |
|  | url_parsed = urlsplit(url) |
|  |  |
|  | try: |
|  | # 根据 URL 的 scheme（协议）判断是 http 还是 https，选择相应的端口（80 或 443） |
|  | if url_parsed.scheme == 'https': |
|  | # 如果是 https 协议，连接到 443 端口，并启用 SSL 加密 |
|  | reader, writer = await asyncio.open_connection(url_parsed.hostname, 443, ssl=True) |
|  | else: |
|  | # 如果是 http 协议，连接到 80 端口 |
|  | reader, writer = await asyncio.open_connection(url_parsed.hostname, 80) |
|  |  |
|  | # 构建 HTTP 请求报文：请求目标是 URL 的 path 部分，使用 HTTP/1.1 协议 |
|  | query = f'GET {url_parsed.path} HTTP/1.1\r\nHost: {url_parsed.hostname}\r\n\r\n' |
|  |  |
|  | # 将请求报文写入连接，并使用 StreamWriter 将编码字节写入套接字。 |
|  | writer.write(query.encode()) |
|  | # 等待数据写入完成 |
|  | await writer.drain() |
|  |  |
|  | # 从服务器读取一行响应数据（HTTP 状态行） |
|  | response = await reader.readline() |
|  | # 解码响应并去除多余的空白字符 |
|  | status = response.decode().strip() |
|  |  |
|  | # 返回 HTTP 状态行（例如："HTTP/1.1 200 OK"） |
|  | return status |
|  | except Exception as e: |
|  | # 如果请求过程中发生任何异常，捕获并输出错误信息 |
|  | print(f"Error fetching {url}: {e}") |
|  | # 如果发生错误，返回 None |
|  | return None |
|  | finally: |
|  | # 确保连接关闭 |
|  | writer.close() |
|  | await writer.wait_closed()  # 等待连接完全关闭 |
|  | reader.feed_eof()  # 通知 reader 没有更多数据会到来 |
|  |  |
|  | # 主协程，执行多个 URL 状态获取任务 |
|  | async def main(): |
|  | # 定义一个包含多个 URL 的列表，表示我们需要检查的目标网站 |
|  | sites = [ |
|  | 'https://www.baidu.com/', |
|  | 'https://www.bilibili.com/', |
|  | 'https://www.weibo.com/', |
|  | 'https://www.douyin.com/', |
|  | 'https://www.zhihu.com/', |
|  | 'https://www.taobao.com/', |
|  | 'https://www.sohu.com/', |
|  | 'https://www.tmall.com/', |
|  | 'https://www.xinhuanet.com/', |
|  | 'https://www.163.com/' |
|  | ] |
|  |  |
|  | # 为每个 URL 创建一个获取状态的异步任务 |
|  | tasks = [get_status(url) for url in sites] |
|  |  |
|  | # 使用 asyncio.gather 并发执行所有任务，返回所有任务的结果（包括异常） |
|  | results = await asyncio.gather(*tasks, return_exceptions=True) |
|  |  |
|  | # 遍历 URL 和对应的响应状态，输出结果 |
|  | for url, status in zip(sites, results): |
|  | # 如果状态不为空，表示请求成功，输出 URL 和 HTTP 状态 |
|  | if status is not None: |
|  | print(f'{url:30}:\t{status}') |
|  | else: |
|  | # 如果状态为 None，表示请求失败，输出错误信息 |
|  | print(f'{url:30}:\tError') |
|  |  |
|  | # 运行主异步程序 |
|  | asyncio.run(main()) |


```

# 3 参考


* [asyncio — Asynchronous I/O](https://github.com)
* [Python Asyncio: The Complete Guide](https://github.com)
* [深度解密 asyncio](https://github.com)
* [进程、线程、协程](https://github.com)
* [aiohttp](https://github.com)
* [HTTP协议请求/响应格式详解](https://github.com)


