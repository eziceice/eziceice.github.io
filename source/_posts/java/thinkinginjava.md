---
title: Thinking in Java
date: 2019-05-19 12:07:06
tags: java
categories:
- programming languages
---
# Thinking in Java

**_Note: Review this book and try to find something that didn't understand correctly._**
_Number of times now: **3rd**_


## 1. 对象导论

- Late Binding：当在Java中调用方法时，被调用的方法只有在运行时才会确定它的Object的具体类型，这也是Java多态能够实现的核心。

## 2. Everything is Object

- Stack：位于通用RAM中，但是可以通过Stack pointer可有从处理器那里直接获得支持。Pointer如果向下移动，则分配新的内存，如果向上移动，则释放内存。reference通常都存在与stack中；如果想要在在stack中创建对象，必须要知道对象的生命周期。**基本类型都在stack中存放。**
- Heap：也位于通用RAM中，用来存放Java所有的对象，不同于stack，compiler不需要知道存储在heap中的数据存活多长时间。因此，在heap中分配内存有着很大的灵活性，但是因此也有trade off，用heap进行存储分配和clean比用stack需要更多的时间。
- Static pool：常量值通常直接存放在程序代码内部，这样做是安全的，因为它们永远不会被改变。
- 基本类型初始值：（**只适用于class variable， 对于局部变量可能为任意值，因此对于局部变量尽量赋予初值**）
  -  boolean：false
  - char：null
  - byte：0
  - short：0
  - int：0
  - long：0L
  - float：0.0f
  - double：0.0d
- 当声明一个事务是static的时候，就意味着这个field或者方法不会与包含它的那个类的任何对象实例关联在一起。

## 2. Operator

- 对于基本的数据类型的赋值，由于其存储了实际的值，而并非指向一个对象的引用，所以在为其赋值时，是直接将一个地方的内容复制到了另一个地方。但是对于对象则是复制的引用。**将对象和基本类型在参数中传递时也是这样的效果。**

## 5. Constructor and Clean

- 如果定义的类中已经有了constructor，那么compiler就不会自动创建一个default constructor。
- this关键字表示获取当前对象的引用。static方法就是没有this的方法。
- Java的GC机制：从static pool和stack开始，遍历所有的reference，就能找到所有活的对象，然后清理所有不需要的对象。**两种GC处理技术会来回切换。**
    - **Stop and Copy**：暂停程序的运行，将所有活的对象从当前的heap复制到另一个新的heap，然后清理。该方法效率低但是清理后的对象都是紧挨着的，因此分配新空间比较简单直接。该方法在程序稳定以后效率更低，因为垃圾此时产生的较少，在进行大量的复制则是浪费操作。如果没有大量的垃圾产生将会进入到另一种模式。
    - **Mark and Sweep**：当找到一个活的对象以后，给对象设置标记，当对于所有的活的对象标记完成以后，开始进行清理，没标记的对象的空间将会被释放。但是此时heap里的空间是不连续的。如果需要得到连续空间则需要对空间进行整理。
- Initialise Order:
  ![InitizliseOrder][1]

## 6. Access Modifier

- 对于类的访问权限只有两种，一种是default访问权限，另一种是public。内部类可以是private或者protected的。

## 7. 复用

- 使用final的通常原因是把方法锁定，以防止任何继承类修改它的含义。想要确保在继承中使方法的行为保持不变，并且不会被覆盖，则使用final。类中所有的private方法都是隐式的指定final的。
- 当将某个类整体定义为final时，就表明你不打算继承该类，而且也不允许别人这样做。final类所中所有的方法都是隐式的指定为final的，因此无法被覆盖。
- constructor也是static方法。
- 优先使用组合或者代理，最后考虑使用继承。

## 7. Polymorphism

- Java中除了static方法和final方法之外，其他所有方法都是动态绑定。
- static方法和field都是没有polymorphism的。

## 8. Interface

- Interface只可以使用public或者包访问权限，interface的field隐式的为static和final的，interface中的方法是隐式的为public的。Java8中可以为interface中添加default和static方法。
- 对于Interface类之间的继承，可以是多重继承。
- Factory模式就是使用的interface来实现的。

## 8. Inner Class

- Inner Class对象会包含有外部对象的引用，并且可以使用外部对象的一切成员，不需要任何条件。通过使用外部对象的名字.this,可以获取外部对象的引用。
- 如果一个匿名内部类想要使用一个在其外部类定义的对象作为参数，则该参数必须为final。
- 嵌套类（static内部类在外部类的内部）可以使用在interface中作为公有代码使用(Java8之后不需要)。
- 每个内部类都能独立地继承自一个接口的实现，所以无论外围类是否已经继承了某个接口的实现，对于内部类都没有影响。内部类有效的解决了多重继承的问题，也就是说，内部类（不是一个内部类）允许继承多个非接口类型。

## 9. Collection

- LinkedList可以用来实现stack/queue等数据结构。`peek`方法返回第一个元素，但不会移除。
- HashMap结构(Array+LinkedList)
  ![HashMap][2]
- LinkedHashMap(HashMap + Doubly LinkedList - 用来维护顺序)
  ![LinkedHashMap][3]
- TreeMap(Red-Black Tree)
- Collection的线程安全性：如果需要线程安全的集合，使用`ConcurrentHashMap/CopyonWriteArrayList/CopyonWriteArraySet`或者使用`Collections.synchronizedList/Map/Set`方法。
  ![集合是否线程安全][4]
- Queue是线程安全的，通过LinkedList向上转型变可得到Queue，peek()和element()方法都在不移除的情况下返回队头，当队列为空时，peek返回null，element返回NoSuchElementException。poll()和remove()方法将移除并返回队头，当队列为空时，poll返回null，remove返回NoSuchElementException。
- 任何是实现Iterable接口的类都可以用于foreach循环。foreach可以用于数组，但是数组并不是一个Iterable。
- Java 中的Collection关系：
  ![Collection][5]

## 10. String

- String是immutable对象。

## 11. Type Information

- RTTI：Run Time Type Information。在运行时，识别一个对象的类型。
- Class对象仅在需要的时候才被加载，static初始化是在类加载的时候进行的。
- 使用.class语法来获得对类的引用不会引发初始化，但为了产生class引用，使用Class.forName()立刻就进行了初始化(static区域初始化)。
- 使用instanceof或者isInstance()方法与直接比较Class对象有一个很重要的差别，instanceof保持了类型的概念，它指的是该类或者该类的派生类，而比较getClass和.class则不会考虑继承或者实现。

## 12. Generics

- 泛型不能显示的用于运行时类型的操作之中，例如转型，instanceof操作或者new表达式，因为所有关于参数的类型信息都被擦除掉了。
- 在泛型中创建数组推荐使用`Array.newInstance()`。
- 在使用泛型的边界中可以使用extends关键字例如`<T extends SomeClass & SomeInterface>`。
- 使用`List<? extends Fruit> fList = new ArrayList<Apple>()`时，任何类型的object都不能被添加到这个容器中，因为该容器的上界为Fruit类型无法确保插入非Fruit类型会对AppleList造成影响，编译器会对插入的任何类型进行限制。
- 使用`List<? super Apple> aList = new ArrayList<Apple>()`时，此时编译器允许插入任何的Apple类型或者其子类型，因为此时Apple是下界，因此可以保证插入的值总是Apple或者Apple的子类型。
- 一个类不能实现同一个泛型接口的两种变体，由于擦除的原因，这两个变体会成为相同的接口。

## 13. Arrays

- 使用`System.arraycopy()`来复制数组比for循环复制要快很多。

## 14. More in Collections

- TreeSet是SortedSet在Java中的唯一实现。
    - subSet(fromElement, toElement)生成Set的子集。
    - headSet(toElement), 返回所有比toElement小的元素的子集。
    - tailSet(fromElement), 返回大于等于fromElement的元素的子集。
- 对于插入或者删除操作，LinkedHashSet比HashSet的代价更高，因为维护LinkedHashSet中的链表需要额外的开销。同理于LinkedHashMap和HashMap。迭代速度Linked结构将会更快，由于有链表。

## 15. Thread

- 实现了Runnable接口的类必须附着在一个Thread上才能开启多线程任务，单独实现Runnable接口并不具备多线程能力。
- 尽量使用Executors启动线程任务。**execute()执行runnable, submit()执行callable。**
    - CachedThreadPool：每个任务都创建一个线程，回收旧线程时停止创建新线程。
    - FixedThreadPool：有限的线程集来创建线程。
    - SingleThreadPool：只有一个线程的线程池，如果有多个任务其他任务将会被挂起然后按顺序执行。
- 使用Callable接口的线程将可以产生返回值，实现的是call()方法，会产生一个`Future<T>`的泛型对象，在使用该对象之前可以使用isDone()方法查看线程是否完成。直接调用get()会阻塞在该方法直到线程执行完成返回。
- 通过yield()方法可以提醒线程调度器该线程已经执行完成。但是并不会一定保证线程执行器就会去执行其他的线程。
- Daemon线程（后台线程），是指程序运行的时候在后台提供一种通用服务的线程，并且这种线程并不属于程序中不可缺少的部分。当所有的非后台线程结束时，程序也就终止了，并且会杀死进程中所有的后台线程。main()就是一个后台线程。如果一个线程是后台线程，那么该后台线程创建的任何线程都将自动被设置为后台线程。
- 一个线程可以在另一个线程上调用join()方法，其效果就是等待一段时间直到新的线程结束在继续执行旧的线程。使用interrupt()方法可以将线程打断。
- 线程中的exception如果不在run()方法中处理的话，就无法被捕获。通过使用在每个Thread上添加一个Thread.UncaughtExceptionHandler异常处理器，就可以在线程中捕获到未被捕获的异常。
- 使用synchronized关键字，可以保护对方法的同步(上锁)，意味着同一时间只有一个线程可以对该方法进行操作。如果该方法目前在使用中，任何想要使用该方法的线程都会被阻塞。synchroinzed也可以使用在方法内部，比如锁一个对象然后进行一系列操作。尽量在block中使用synchronized而不是synchronized方法。
- 使用ReentrantLock对象，可以对方法进行显示的lock和unlock。同时ReentrantLock对象有tryLock的方法，可以尝试着去获取锁。
- volatile关键字可以保证一个域是具备原子性的。尽量使用synchronized而不是volatile，因为Java中只有赋值和返回操作是原子性的，递增操作不是原子性的。
- wait()可以使线程挂起，直到收到了notify()或者notifyAll()或者wait()时间结束才可以继续执行。wait()期间会释放对锁的资源。
- 使用LinkedBlockingQueue(无大小限制)可以非常容易的实现消费者和生产者的模型。还可以使用ArrayBlockingQueue，它是一个具有固定尺寸的阻塞队列。
- 使用PipedWriter类或者PipedReader类可以对多线程下的输入输出提供支持。它们实际上就是一个BlockingQueue。
- Executor结构：
  ![Executor][6]
- Thread状态：
  ![Thread][7]

[1]:https://raw.githubusercontent.com/eziceice/blog/master/java/JavaInitializeOrder.png
[2]:https://raw.githubusercontent.com/eziceice/blog/master/java/HashMap%E7%BB%93%E6%9E%84.png
[3]:https://raw.githubusercontent.com/eziceice/blog/master/java/LinkedHashMap%E7%BB%93%E6%9E%84.png
[4]:https://raw.githubusercontent.com/eziceice/blog/master/java/CollectionThreadFeature.png
[5]:https://raw.githubusercontent.com/eziceice/blog/master/java/Collection.png
[6]:https://raw.githubusercontent.com/eziceice/blog/master/java/Executor.png
[7]:https://raw.githubusercontent.com/eziceice/blog/master/java/ThreadStatus.png
