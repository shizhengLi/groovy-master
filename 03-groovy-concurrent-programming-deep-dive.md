# Groovy并发编程深度解析：构建高性能并发应用的终极指南

## 引言

Groovy作为JVM语言，充分利用了Java的并发编程基础设施，同时通过其动态特性提供了更简洁、更强大的并发编程模型。本文将深入探讨Groovy并发编程的各个方面，从基础概念到高级应用，帮助你构建高性能、高并发的Groovy应用程序。

## 1. Groovy并发编程基础

### 1.1 线程基础与Groovy增强

```groovy
// Groovy线程基础
class ThreadBasics {
    static void main(String[] args) {
        // 1. 基本线程创建
        def thread1 = Thread.start {
            println "线程1: ${Thread.currentThread().name}"
            (1..10).each {
                println "线程1: $it"
                Thread.sleep(100)
            }
        }

        // 2. 使用闭包创建线程
        def thread2 = new Thread({
            println "线程2: ${Thread.currentThread().name}"
            (1..10).each {
                println "线程2: $it"
                Thread.sleep(150)
            }
        }).start()

        // 3. 线程池执行器
        def executor = Executors.newFixedThreadPool(3)
        def futures = []

        (1..5).each { i ->
            def future = executor.submit {
                println "任务 $i 在线程 ${Thread.currentThread().name} 中执行"
                Thread.sleep(200)
                return "任务 $i 完成"
            }
            futures.add(future)
        }

        // 等待所有任务完成
        futures.each { future ->
            println future.get()
        }

        executor.shutdown()
    }
}
```

### 1.2 Groovy GPars并发库

```groovy
@Grab('org.codehaus.gpars:gpars:1.2.1')
import groovyx.gpars.*

class GParsBasics {
    static void main(String[] args) {
        // 1. 并行集合处理
        def numbers = 1..1000

        // 使用GParsPool进行并行处理
        GParsPool.withPool {
            def result = numbers.collectParallel { num ->
                Thread.sleep(10) // 模拟耗时操作
                num * num
            }
            println "并行处理结果: ${result.take(10)}"
        }

        // 2. 异步处理
        GParsPool.withPool {
            def asyncResult = numbers.collectAsync { num ->
                Thread.sleep(5)
                num + 100
            }
            println "异步处理结果: ${asyncResult.get().take(10)}"
        }

        // 3. 并行数组处理
        def array = [1, 2, 3, 4, 5] as int[]
        GParsPool.withPool {
            array.eachParallel { num ->
                println "处理数字: $num 在线程 ${Thread.currentThread().name}"
            }
        }
    }
}
```

## 2. 高级并发模式

### 2.1 Actor模型实现

```groovy
@Grab('org.codehaus.gpars:gpars:1.2.1')
import groovyx.gpars.actor.*

class MessageProcessor extends DynamicDispatchActor {
    def void process(String message) {
        println "处理字符串消息: $message"
    }

    def void process(Integer number) {
        println "处理数字消息: $number"
    }

    def void process(Map data) {
        println "处理Map消息: $data"
    }

    def void onException(Throwable throwable) {
        println "消息处理异常: ${throwable.message}"
    }
}

class ActorDemo {
    static void main(String[] args) {
        def processor = new MessageProcessor().start()

        // 发送不同类型的消息
        processor << "Hello Actor"
        processor << 42
        processor << [name: "张三", age: 25]

        // 等待消息处理完成
        Thread.sleep(1000)

        processor.stop()
    }
}

// 状态机Actor
class StateMachineActor extends DynamicDispatchActor {
    def currentState = "IDLE"

    def void process(String command) {
        switch (currentState) {
            case "IDLE":
                if (command == "START") {
                    currentState = "RUNNING"
                    println "状态从 IDLE 切换到 RUNNING"
                }
                break
            case "RUNNING":
                if (command == "STOP") {
                    currentState = "IDLE"
                    println "状态从 RUNNING 切换到 IDLE"
                } else if (command == "PAUSE") {
                    currentState = "PAUSED"
                    println "状态从 RUNNING 切换到 PAUSED"
                }
                break
            case "PAUSED":
                if (command == "RESUME") {
                    currentState = "RUNNING"
                    println "状态从 PAUSED 切换到 RUNNING"
                }
                break
        }
    }

    def void process(Map statusCommand) {
        println "当前状态: $currentState"
    }
}

// 事务性Actor
class TransactionalActor extends DynamicDispatchActor {
    def transactionLog = []
    def currentTransaction = null

    def void process(Map command) {
        switch (command.action) {
            case "BEGIN":
                if (currentTransaction == null) {
                    currentTransaction = [
                        id: UUID.randomUUID().toString(),
                        operations: [],
                        startTime: System.currentTimeMillis()
                    ]
                    println "事务 ${currentTransaction.id} 开始"
                }
                break

            case "OPERATION":
                if (currentTransaction) {
                    currentTransaction.operations.add([
                        type: command.type,
                        data: command.data,
                        timestamp: System.currentTimeMillis()
                    ])
                    println "添加操作到事务 ${currentTransaction.id}: ${command.type}"
                }
                break

            case "COMMIT":
                if (currentTransaction) {
                    transactionLog.add(currentTransaction)
                    println "事务 ${currentTransaction.id} 提交，包含 ${currentTransaction.operations.size()} 个操作"
                    currentTransaction = null
                }
                break

            case "ROLLBACK":
                if (currentTransaction) {
                    println "事务 ${currentTransaction.id} 回滚"
                    currentTransaction = null
                }
                break
        }
    }

    def void process(String command) {
        if (command == "STATUS") {
            println "当前活动事务: ${currentTransaction?.id ?: '无'}"
            println "已完成事务数: ${transactionLog.size()}"
        }
    }
}
```

### 2.2 CSP（Communicating Sequential Processes）模型

```groovy
@Grab('org.codehaus.gpars:gpars:1.2.1')
import groovyx.gpars.csp.*

class CSPDemo {
    static void main(String[] args) {
        // 创建通道
        def channel = Channel.createChannel()

        // 生产者
        def producer = {
            5.times { i ->
                println "生产者发送: 消息 $i"
                channel.write("消息 $i")
                Thread.sleep(200)
            }
            channel.write("END") // 结束标记
        }

        // 消费者
        def consumer = {
            def message
            while ((message = channel.read()) != "END") {
                println "消费者接收: $message"
                Thread.sleep(300)
            }
            println "消费者结束"
        }

        // 启动生产者和消费者
        GParsPool.withPool {
            def producerTask = task(producer)
            def consumerTask = task(consumer)

            producerTask.join()
            consumerTask.join()
        }

        channel.close()
    }
}

// 多通道通信示例
class MultiChannelCSP {
    static void main(String[] args) {
        def requestChannel = Channel.createChannel()
        def responseChannel = Channel.createChannel()

        // 服务器Actor
        def server = {
            while (true) {
                def request = requestChannel.read()
                if (request == "SHUTDOWN") break

                println "服务器处理请求: $request"
                Thread.sleep(100) // 模拟处理时间

                def response = "处理结果: $request"
                responseChannel.write(response)
            }
            println "服务器关闭"
        }

        // 客户端Actor
        def client = { clientId ->
            3.times { i ->
                def request = "Client-$clientId-Request-$i"
                println "客户端 $clientId 发送请求: $request"
                requestChannel.write(request)

                def response = responseChannel.read()
                println "客户端 $clientId 接收响应: $response"
            }
        }

        GParsPool.withPool {
            def serverTask = task(server)

            // 启动多个客户端
            def clientTasks = (1..3).collect { clientId ->
                task { client(clientId) }
            }

            // 等待所有客户端完成
            clientTasks*.join()

            // 关闭服务器
            requestChannel.write("SHUTDOWN")
            serverTask.join()
        }

        requestChannel.close()
        responseChannel.close()
    }
}
```

## 3. 数据并行处理

### 3.1 并行集合操作

```groovy
class ParallelCollections {
    static void main(String[] args) {
        def largeList = (1..1000000).collect { it }

        // 1. 并行映射
        GParsPool.withPool {
            def start = System.currentTimeMillis()
            def result = largeList.parallel.map { it * 2 }
            def end = System.currentTimeMillis()
            println "并行映射耗时: ${end - start}ms"
            println "结果数量: ${result.size()}"
        }

        // 2. 并行过滤
        GParsPool.withPool {
            def start = System.currentTimeMillis()
            def filtered = largeList.parallel.filter { it % 2 == 0 }
            def end = System.currentTimeMillis()
            println "并行过滤耗时: ${end - start}ms"
            println "偶数数量: ${filtered.size()}"
        }

        // 3. 并行归约
        GParsPool.withPool {
            def start = System.currentTimeMillis()
            def sum = largeList.parallel.reduce { a, b -> a + b }
            def end = System.currentTimeMillis()
            println "并行归约耗时: ${end - start}ms"
            println "总和: $sum"
        }

        // 4. 并行分组
        GParsPool.withPool {
            def start = System.currentTimeMillis()
            def grouped = largeList.parallel.groupBy { it % 10 }
            def end = System.currentTimeMillis()
            println "并行分组耗时: ${end - start}ms"
            println "分组数量: ${grouped.size()}"
        }

        // 5. 并行排序
        GParsPool.withPool {
            def start = System.currentTimeMillis()
            def sorted = largeList.parallel.sort { -it } // 降序
            def end = System.currentTimeMillis()
            println "并行排序耗时: ${end - start}ms"
            println "前10个元素: ${sorted.take(10)}"
        }
    }
}
```

### 3.2 并行数据流处理

```groovy
class ParallelDataFlow {
    static void main(String[] args) {
        // 创建数据流变量
        def source = DataFlow.task {
            println "数据源开始生成数据"
            (1..100).collect {
                Thread.sleep(10)
                it
            }
        }

        def stage1 = DataFlow.task {
            def data = source.val
            println "阶段1: 开始处理数据"
            data.collect { it * 2 }
        }

        def stage2 = DataFlow.task {
            def data = stage1.val
            println "阶段2: 开始处理数据"
            data.findAll { it % 4 == 0 }
        }

        def stage3 = DataFlow.task {
            def data = stage2.val
            println "阶段3: 开始处理数据"
            data.sum()
        }

        def result = stage3.val
        println "最终结果: $result"
    }
}

// 复杂数据流网络
class ComplexDataFlowNetwork {
    static void main(String[] args) {
        // 数据源
        def dataSource = DataFlow.task {
            println "开始生成数据流"
            (1..50).collect {
                Thread.sleep(50)
                [id: it, value: it * 10, timestamp: System.currentTimeMillis()]
            }
        }

        // 过滤有效数据
        def validData = DataFlow.task {
            def data = dataSource.val
            println "过滤阶段: 保留有效数据"
            data.findAll { it.value > 100 }
        }

        // 数据转换
        def transformedData = DataFlow.task {
            def data = validData.val
            println "转换阶段: 数据标准化"
            data.collect {
                [id: it.id, normalizedValue: it.value / 100.0, timestamp: it.timestamp]
            }
        }

        // 数据聚合
        def aggregatedData = DataFlow.task {
            def data = transformedData.val
            println "聚合阶段: 计算统计信息"
            [
                count: data.size(),
                average: data.sum { it.normalizedValue } / data.size(),
                max: data.max { it.normalizedValue }.normalizedValue,
                min: data.min { it.normalizedValue }.normalizedValue
            ]
        }

        // 结果输出
        def resultOutput = DataFlow.task {
            def stats = aggregatedData.val
            println "最终统计结果:"
            println "数据数量: ${stats.count}"
            println "平均值: ${String.format("%.2f", stats.average)}"
            println "最大值: ${String.format("%.2f", stats.max)}"
            println "最小值: ${String.format("%.2f", stats.min)}"
        }

        // 等待所有任务完成
        resultOutput.val
    }
}
```

## 4. 高级并发工具

### 4.1 自定义线程池配置

```groovy
class AdvancedThreadPool {
    static def createOptimizedThreadPool() {
        // 1. 自定义线程工厂
        def threadFactory = { Runnable runnable ->
            def thread = new Thread(runnable)
            thread.name = "CustomPool-Thread-${System.currentTimeMillis()}"
            thread.priority = Thread.NORM_PRIORITY
            thread.daemon = true
            return thread
        }

        // 2. 自定义拒绝策略
        def rejectionHandler = { Runnable runnable, ThreadPoolExecutor executor ->
            println "任务被拒绝: ${runnable.class.simpleName}"
            // 尝试重新提交
            Thread.sleep(100)
            executor.execute(runnable)
        }

        // 3. 创建优化的线程池
        def executor = new ThreadPoolExecutor(
            4,                              // 核心线程数
            16,                             // 最大线程数
            60L,                            // 空闲线程存活时间
            TimeUnit.SECONDS,               // 时间单位
            new LinkedBlockingQueue<>(1000), // 任务队列
            threadFactory as ThreadFactory, // 线程工厂
            rejectionHandler as RejectedExecutionHandler // 拒绝策略
        )

        // 4. 监控线程池状态
        def monitor = Thread.start {
            while (!executor.isTerminated()) {
                println """
                    线程池状态:
                    - 活跃线程数: ${executor.activeCount}
                    - 已完成任务数: ${executor.completedTaskCount}
                    - 队列大小: ${executor.queue.size()}
                    - 总任务数: ${executor.taskCount}
                """
                Thread.sleep(5000)
            }
        }

        return [executor: executor, monitor: monitor]
    }

    static void main(String[] args) {
        def poolConfig = createOptimizedThreadPool()
        def executor = poolConfig.executor

        // 提交任务
        def futures = []
        (1..100).each { i ->
            def future = executor.submit {
                Thread.sleep(100)
                println "任务 $i 完成"
                return "Result-$i"
            }
            futures.add(future)
        }

        // 获取结果
        futures.each { future ->
            try {
                println future.get(1, TimeUnit.SECONDS)
            } catch (TimeoutException e) {
                println "任务超时"
            }
        }

        // 关闭线程池
        executor.shutdown()
        executor.awaitTermination(1, TimeUnit.MINUTES)
        poolConfig.monitor.interrupt()
    }
}
```

### 4.2 并发集合优化

```groovy
class ConcurrentCollections {
    static void main(String[] args) {
        // 1. 并发HashMap
        def concurrentMap = new ConcurrentHashMap<String, Integer>()

        // 并发写入
        def threads = []
        10.times { threadId ->
            def thread = Thread.start {
                100.times { i ->
                    def key = "Thread-${threadId}-Key-${i}"
                    concurrentMap.put(key, threadId * 1000 + i)
                }
            }
            threads.add(thread)
        }

        threads*.join()
        println "ConcurrentHashMap 大小: ${concurrentMap.size()}"

        // 2. 并发CopyOnWriteArrayList
        def copyOnWriteList = new CopyOnWriteArrayList<String>()

        threads.clear()
        5.times { threadId ->
            def thread = Thread.start {
                20.times { i ->
                    copyOnWriteList.add("Thread-${threadId}-Item-${i}")
                    Thread.sleep(10)
                }
            }
            threads.add(thread)
        }

        threads*.join()
        println "CopyOnWriteArrayList 大小: ${copyOnWriteList.size()}"

        // 3. 并发BlockingQueue
        def blockingQueue = new LinkedBlockingQueue<String>(10)

        // 生产者
        def producer = Thread.start {
            20.times { i ->
                def item = "Item-${i}"
                println "生产者: 尝试放入 $item"
                blockingQueue.put(item) // 阻塞直到有空间
                println "生产者: 成功放入 $item"
                Thread.sleep(50)
            }
        }

        // 消费者
        def consumer = Thread.start {
            20.times {
                def item = blockingQueue.take() // 阻塞直到有元素
                println "消费者: 获取 $item"
                Thread.sleep(100)
            }
        }

        producer.join()
        consumer.join()

        // 4. 并发SkipList
        def skipList = new ConcurrentSkipListMap<String, Integer>()

        threads.clear()
        5.times { threadId ->
            def thread = Thread.start {
                100.times { i ->
                    def key = "Key-${threadId}-${i}"
                    skipList.put(key, i)
                }
            }
            threads.add(thread)
        }

        threads*.join()
        println "ConcurrentSkipListMap 大小: ${skipList.size()}"
        println "第一个键: ${skipList.firstKey()}"
        println "最后一个键: ${skipList.lastKey()}"
    }
}
```

## 5. 异步编程模式

### 5.1 Promise与Future模式

```groovy
class AsyncProgramming {
    static void main(String[] args) {
        // 1. 基本Future使用
        def executor = Executors.newFixedThreadPool(4)

        def future1 = executor.submit {
            Thread.sleep(1000)
            return "异步任务1完成"
        }

        def future2 = executor.submit {
            Thread.sleep(1500)
            return "异步任务2完成"
        }

        // 等待所有Future完成
        def results = [future1, future2].collect { it.get() }
        println "所有任务完成: $results"

        // 2. Future回调
        def callbackFuture = executor.submit {
            Thread.sleep(800)
            return "回调任务结果"
        }

        callbackFuture.whenComplete { result, exception ->
            if (exception) {
                println "回调任务异常: ${exception.message}"
            } else {
                println "回调任务成功: $result"
            }
        }

        // 3. 自定义Promise
        def customPromise = createCustomPromise()

        customPromise.onSuccess { result ->
            println "Promise成功: $result"
        }

        customPromise.onFailure { error ->
            println "Promise失败: ${error.message}"
        }

        // 模拟异步操作
        Thread.start {
            Thread.sleep(500)
            customPromise.complete("自定义Promise完成")
        }

        // 4. Future组合
        def futureA = executor.submit { "A" }
        def futureB = executor.submit { "B" }
        def futureC = executor.submit { "C" }

        def combinedFuture = combineFutures([futureA, futureB, futureC])
        combinedFuture.whenComplete { result, exception ->
            if (exception) {
                println "组合Future异常: ${exception.message}"
            } else {
                println "组合Future结果: $result"
            }
        }

        Thread.sleep(2000)
        executor.shutdown()
    }

    // 自定义Promise实现
    static def createCustomPromise() {
        def state = "PENDING"
        def result = null
        def error = null
        def successCallbacks = []
        def failureCallbacks = []

        def promise = [
            complete: { value ->
                if (state == "PENDING") {
                    state = "FULFILLED"
                    result = value
                    successCallbacks.each { callback -> callback(value) }
                }
            },
            fail: { throwable ->
                if (state == "PENDING") {
                    state = "REJECTED"
                    error = throwable
                    failureCallbacks.each { callback -> callback(throwable) }
                }
            },
            onSuccess: { callback ->
                if (state == "FULFILLED") {
                    callback(result)
                } else if (state == "PENDING") {
                    successCallbacks.add(callback)
                }
            },
            onFailure: { callback ->
                if (state == "REJECTED") {
                    callback(error)
                } else if (state == "PENDING") {
                    failureCallbacks.add(callback)
                }
            },
            getState: { state }
        ]

        return promise
    }

    // 组合多个Future
    static def combineFutures(List<Future> futures) {
        def executor = Executors.newSingleThreadExecutor()
        def combinedFuture = executor.submit {
            def results = []
            futures.each { future ->
                results.add(future.get())
            }
            return results
        }
        executor.shutdown()
        return combinedFuture
    }
}
```

### 5.2 响应式编程

```groovy
@Grab('io.reactivex.rxjava2:rxjava:2.2.21')
import io.reactivex.*

class ReactiveProgramming {
    static void main(String[] args) {
        // 1. 创建Observable
        def observable = Observable.create { emitter ->
            5.times { i ->
                if (!emitter.isDisposed()) {
                    println "发射: $i"
                    emitter.onNext(i)
                    Thread.sleep(200)
                }
            }
            if (!emitter.isDisposed()) {
                emitter.onComplete()
            }
        }

        // 2. 订阅Observable
        def disposable = observable
            .subscribeOn(Schedulers.io())           // 在IO线程执行
            .observeOn(Schedulers.single())          // 在单线程观察
            .map { item -> item * 2 }               // 转换数据
            .filter { item -> item % 3 == 0 }       // 过滤数据
            .subscribe(
                { item -> println "接收: $item" },  // onNext
                { error -> println "错误: ${error.message}" },  // onError
                { println "完成" }                   // onComplete
            )

        // 3. 组合操作
        def observable1 = Observable.just(1, 2, 3)
        def observable2 = Observable.just(4, 5, 6)

        Observable.zip(observable1, observable2) { a, b -> a + b }
            .subscribe { result -> println "Zip结果: $result" }

        // 4. 背压处理
        def backpressureObservable = Observable.range(1, 1000)
            .subscribeOn(Schedulers.io())
            .observeOn(Schedulers.single(), false, 10) // 缓冲区大小为10

        backpressureObservable.subscribe(
            { item ->
                println "处理: $item"
                Thread.sleep(100) // 模拟慢速处理
            },
            { error -> println "背压错误: ${error.message}" }
        )

        Thread.sleep(5000)
        disposable.dispose()
    }
}

// 自定义响应式操作符
class CustomReactiveOperators {
    static void main(String[] args) {
        // 1. 自定义操作符
        Observable.range(1, 10)
            .lift { observer ->
                new ObservableSource<String>() {
                    @Override
                    void subscribe(Observer<? super String> observer) {
                        observer.onSubscribe(Disposables.empty())
                        observer.onNext("开始处理")
                    }
                }
            }
            .subscribe { println "自定义操作符: $it" }

        // 2. 重试机制
        def retryObservable = Observable.create { emitter ->
            def attempt = 0
            while (attempt < 3 && !emitter.isDisposed()) {
                attempt++
                try {
                    println "尝试 $attempt"
                    if (attempt < 3) {
                        throw new RuntimeException("模拟失败")
                    }
                    emitter.onNext("成功")
                    emitter.onComplete()
                } catch (Exception e) {
                    if (attempt == 3) {
                        emitter.onError(e)
                    }
                }
            }
        }

        retryObservable
            .retryWhen { errors ->
                errors.zipWith(Observable.range(1, 3)) { error, attempt ->
                    if (attempt < 3) {
                        println "重试 $attempt"
                        Observable.timer(1000, TimeUnit.MILLISECONDS)
                    } else {
                        Observable.error(error)
                    }
                }.flatMap { it }
            }
            .subscribe(
                { result -> println "最终结果: $result" },
                { error -> println "最终错误: ${error.message}" }
            )

        Thread.sleep(5000)
    }
}
```

## 6. 并发性能优化

### 6.1 锁优化技术

```groovy
class LockOptimization {
    private final Object lock = new Object()
    private final ReadWriteLock readWriteLock = new ReentrantReadWriteLock()
    private final StampedLock stampedLock = new StampedLock()

    def sharedData = [:]
    def counter = 0

    // 1. 同步块优化
    def synchronizedMethod() {
        synchronized (lock) {
            counter++
            Thread.sleep(50)
            return counter
        }
    }

    // 2. 读写锁优化
    def readWithLock() {
        readWriteLock.readLock().lock()
        try {
            return sharedData.size()
        } finally {
            readWriteLock.readLock().unlock()
        }
    }

    def writeWithLock(key, value) {
        readWriteLock.writeLock().lock()
        try {
            sharedData[key] = value
            Thread.sleep(100)
        } finally {
            readWriteLock.writeLock().unlock()
        }
    }

    // 3. StampedLock优化
    def optimisticRead() {
        def stamp = stampedLock.tryOptimisticRead()
        def result = sharedData.size()

        if (!stampedLock.validate(stamp)) {
            // 乐观读失败，使用悲观读
            stamp = stampedLock.readLock()
            try {
                result = sharedData.size()
            } finally {
                stampedLock.unlockRead(stamp)
            }
        }

        return result
    }

    def conditionalWrite(key, value) {
        def stamp = stampedLock.writeLock()
        try {
            sharedData[key] = value
            Thread.sleep(50)
            return true
        } finally {
            stampedLock.unlockWrite(stamp)
        }
    }

    // 4. 锁粗化与细化
    def coarseGrainedOperation() {
        synchronized (lock) {
            // 粗粒度锁 - 多个操作在同一个锁内
            def data1 = fetchDataFromDatabase("table1")
            def data2 = fetchDataFromDatabase("table2")
            processData(data1, data2)
            saveToCache("result", data1 + data2)
        }
    }

    def fineGrainedOperation() {
        // 细粒度锁 - 每个操作单独加锁
        def data1, data2
        synchronized (lock) {
            data1 = fetchDataFromDatabase("table1")
        }

        synchronized (lock) {
            data2 = fetchDataFromDatabase("table2")
        }

        def result
        synchronized (lock) {
            result = processData(data1, data2)
        }

        synchronized (lock) {
            saveToCache("result", result)
        }
    }

    private def fetchDataFromDatabase(String table) {
        Thread.sleep(100)
        return "Data from $table"
    }

    private def processData(data1, data2) {
        Thread.sleep(50)
        return "$data1 + $data2"
    }

    private def saveToCache(String key, String value) {
        Thread.sleep(30)
        sharedData[key] = value
    }
}
```

### 6.2 无锁算法

```groovy
class LockFreeAlgorithms {
    // 1. 无锁计数器
    private final AtomicInteger atomicCounter = new AtomicInteger(0)

    def incrementCounter() {
        return atomicCounter.incrementAndGet()
    }

    def getCounter() {
        return atomicCounter.get()
    }

    // 2. 无锁栈
    private static class Node {
        final Object value
        volatile Node next

        Node(Object value) {
            this.value = value
        }
    }

    private final AtomicReference<Node> stackTop = new AtomicReference<>()

    def push(Object value) {
        def newNode = new Node(value)
        def oldTop
        do {
            oldTop = stackTop.get()
            newNode.next = oldTop
        } while (!stackTop.compareAndSet(oldTop, newNode))
    }

    def pop() {
        def oldTop
        def newTop
        do {
            oldTop = stackTop.get()
            if (oldTop == null) return null
            newTop = oldTop.next
        } while (!stackTop.compareAndSet(oldTop, newTop))

        return oldTop.value
    }

    // 3. 无锁队列
    private static class QueueNode {
        final Object value
        volatile QueueNode next

        QueueNode(Object value) {
            this.value = value
        }
    }

    private final AtomicReference<QueueNode> queueHead = new AtomicReference<>()
    private final AtomicReference<QueueNode> queueTail = new AtomicReference<>()

    LockFreeAlgorithms() {
        def dummy = new QueueNode(null)
        queueHead.set(dummy)
        queueTail.set(dummy)
    }

    def enqueue(Object value) {
        def newNode = new QueueNode(value)
        def tail
        do {
            tail = queueTail.get()
            tail.next = newNode
        } while (!queueTail.compareAndSet(tail, newNode))
    }

    def dequeue() {
        def head
        def nextHead
        do {
            head = queueHead.get()
            nextHead = head.next
            if (nextHead == null) return null
        } while (!queueHead.compareAndSet(head, nextHead))

        return nextHead.value
    }

    // 4. CAS操作示例
    def compareAndSwapExample(Object expectedValue, Object newValue) {
        def atomicRef = new AtomicReference(expectedValue)

        def success = atomicRef.compareAndSet(expectedValue, newValue)
        if (success) {
            println "CAS操作成功: $expectedValue -> $newValue"
        } else {
            println "CAS操作失败: 当前值为 ${atomicRef.get()}"
        }

        return success
    }
}
```

## 7. 并发测试与调试

### 7.1 并发测试框架

```groovy
import spock.lang.Specification
import spock.lang.Unroll
import java.util.concurrent.*
import java.util.concurrent.atomic.*

class ConcurrencyTesting extends Specification {

    def "测试并发计数器"() {
        given:
        def counter = new AtomicInteger(0)
        def threadCount = 10
        def incrementsPerThread = 1000
        def executor = Executors.newFixedThreadPool(threadCount)
        def futures = []

        when:
        threadCount.times {
            def future = executor.submit {
                incrementsPerThread.times {
                    counter.incrementAndGet()
                    Thread.sleep(1) // 模拟工作
                }
            }
            futures.add(future)
        }

        futures.each { it.get() }
        def finalCount = counter.get()

        then:
        finalCount == threadCount * incrementsPerThread

        cleanup:
        executor.shutdown()
    }

    def "测试线程安全集合"() {
        given:
        def concurrentList = new CopyOnWriteArrayList<String>()
        def threadCount = 5
        def itemsPerThread = 100
        def executor = Executors.newFixedThreadPool(threadCount)

        when:
        threadCount.times { threadId ->
            executor.submit {
                itemsPerThread.times { i ->
                    concurrentList.add("Thread-${threadId}-Item-${i}")
                    Thread.sleep(10)
                }
            }
        }

        executor.shutdown()
        executor.awaitTermination(1, TimeUnit.MINUTES)

        then:
        concurrentList.size() == threadCount * itemsPerThread
        concurrentList.every { it != null }
    }

    def "测试死锁检测"() {
        given:
        def lock1 = new Object()
        def lock2 = new Object()
        def deadlockDetected = new AtomicBoolean(false)

        when:
        def thread1 = Thread.start {
            synchronized (lock1) {
                println "线程1获取lock1"
                Thread.sleep(100)
                synchronized (lock2) {
                    println "线程1获取lock2"
                }
            }
        }

        def thread2 = Thread.start {
            synchronized (lock2) {
                println "线程2获取lock2"
                Thread.sleep(100)
                synchronized (lock1) {
                    println "线程2获取lock1"
                }
            }
        }

        // 检测死锁
        def monitor = Thread.start {
            Thread.sleep(500)
            def deadlockedThreads = ManagementFactory.threadMXBean.findDeadlockedThreads()
            if (deadlockedThreads) {
                deadlockDetected.set(true)
                println "检测到死锁"
            }
        }

        thread1.join(2000)
        thread2.join(2000)
        monitor.join(1000)

        then:
        deadlockDetected.get() || (thread1.alive && thread2.alive)
    }

    @Unroll
    def "测试并发性能 - 线程数: #threadCount"() {
        given:
        def executor = Executors.newFixedThreadPool(threadCount)
        def startTime = System.currentTimeMillis()
        def taskCount = 10000

        when:
        def futures = []
        taskCount.times {
            def future = executor.submit {
                Thread.sleep(1) // 模拟工作
                return it
            }
            futures.add(future)
        }

        futures.each { it.get() }
        def endTime = System.currentTimeMillis()
        def duration = endTime - startTime

        then:
        duration < 5000 // 5秒内完成

        cleanup:
        executor.shutdown()

        where:
        threadCount << [1, 2, 4, 8, 16]
    }
}

// 压力测试工具
class StressTesting {
    static def runLoadTest(Closure task, int threadCount, int durationSeconds) {
        def executor = Executors.newFixedThreadPool(threadCount)
        def completionTimes = new ConcurrentLinkedQueue<Long>()
        def startTime = System.currentTimeMillis()
        def endTime = startTime + durationSeconds * 1000
        def running = new AtomicBoolean(true)

        def threads = []
        threadCount.times {
            def thread = Thread.start {
                while (running.get() && System.currentTimeMillis() < endTime) {
                    def taskStart = System.currentTimeMillis()
                    task()
                    def taskEnd = System.currentTimeMillis()
                    completionTimes.add(taskEnd - taskStart)
                }
            }
            threads.add(thread)
        }

        Thread.sleep(durationSeconds * 1000)
        running.set(false)

        threads*.join()
        executor.shutdown()

        return analyzeResults(completionTimes)
    }

    static def analyzeResults(Queue<Long> completionTimes) {
        def times = completionTimes as List<Long>
        if (times.isEmpty()) return [:]

        def sorted = times.sort()
        def count = times.size()
        def total = times.sum()
        def avg = total / count
        def min = sorted.first()
        def max = sorted.last()
        def median = sorted[count / 2] as Long
        def p95 = sorted[(count * 0.95) as int]
        def p99 = sorted[(count * 0.99) as int]

        return [
            count: count,
            total: total,
            average: avg,
            min: min,
            max: max,
            median: median,
            p95: p95,
            p99: p99,
            tps: count / (total / 1000.0)
        ]
    }

    static void main(String[] args) {
        def results = runLoadTest({
            // 模拟数据库查询
            Thread.sleep(10 + Math.random() * 20)
        }, 10, 30)

        println "压力测试结果:"
        println "总请求数: ${results.count}"
        println "平均响应时间: ${String.format("%.2f", results.average)}ms"
        println "最小响应时间: ${results.min}ms"
        println "最大响应时间: ${results.max}ms"
        println "中位数响应时间: ${results.median}ms"
        println "95分位响应时间: ${results.p95}ms"
        println "99分位响应时间: ${results.p99}ms"
        println "TPS: ${String.format("%.2f", results.tps)}"
    }
}
```

### 7.2 并发调试技术

```groovy
class ConcurrencyDebugging {
    // 1. 线程状态监控
    static def monitorThreadStates() {
        def monitor = Thread.start {
            while (true) {
                println "\n=== 线程状态监控 ==="
                Thread.getAllStackTraces().each { thread, stackTrace ->
                    println "${thread.name}: ${thread.state}"
                    if (thread.state == Thread.State.BLOCKED) {
                        println "  阻塞堆栈:"
                        stackTrace.each { println "    $it" }
                    }
                }
                Thread.sleep(5000)
            }
        }
        return monitor
    }

    // 2. 死锁检测
    static def detectDeadlocks() {
        def threadMXBean = ManagementFactory.threadMXBean
        def deadlockedThreads = threadMXBean.findDeadlockedThreads()

        if (deadlockedThreads) {
            println "检测到死锁线程:"
            deadlockedThreads.each { threadId ->
                def threadInfo = threadMXBean.getThreadInfo(threadId, 100)
                println "  线程名称: ${threadInfo.threadName}"
                println "  线程状态: ${threadInfo.threadState}"
                println "  阻塞计数: ${threadInfo.blockedCount}"
                println "  等待计数: ${threadInfo.waitedCount}"
                println "  锁定信息:"
                threadInfo.lockInfo?.with { lockInfo ->
                    println "    锁定对象: $lockInfo"
                    println "    锁定模式: ${threadInfo.lockName}"
                }
                println "  堆栈跟踪:"
                threadInfo.stackTrace.each { println "    $it" }
                println ""
            }
            return true
        } else {
            println "未检测到死锁"
            return false
        }
    }

    // 3. 线程转储分析
    static def analyzeThreadDump() {
        def threadDump = new StringBuilder()

        threadDump.append("\n=== 线程转储 ===\n")
        Thread.getAllStackTraces().each { thread, stackTrace ->
            threadDump.append("\"${thread.name}\"")
            threadDump.append(" #${thread.priority}")
            threadDump.append(" daemon=${thread.daemon}")
            threadDump.append(" prio=${thread.priority}")
            threadDump.append(" tid=${thread.id}")
            threadDump.append(" nid=${thread.nativeThreadId}")
            threadDump.append(" ${thread.state}\n")

            if (thread.lock) {
                threadDump.append("    at ${thread.lock}\n")
            }

            stackTrace.each { frame ->
                threadDump.append("    at $frame\n")
            }
            threadDump.append("\n")
        }

        println threadDump.toString()
        return threadDump.toString()
    }

    // 4. 性能分析器
    static class PerformanceProfiler {
        def methodStats = new ConcurrentHashMap<String, List<Long>>()
        def enabled = new AtomicBoolean(false)

        def startProfiling() {
            enabled.set(true)
        }

        def stopProfiling() {
            enabled.set(false)
        }

        def profileMethod(String methodName, Closure closure) {
            if (!enabled.get()) {
                return closure.call()
            }

            def startTime = System.nanoTime()
            try {
                return closure.call()
            } finally {
                def duration = System.nanoTime() - startTime
                methodStats.computeIfAbsent(methodName) { new ArrayList<>() }.add(duration)
            }
        }

        def printStats() {
            println "\n=== 性能分析结果 ==="
            methodStats.each { methodName, durations ->
                def count = durations.size()
                def total = durations.sum()
                def avg = total / count
                def min = durations.min()
                def max = durations.max()
                def sorted = durations.sort()
                def median = sorted[count / 2] as Long
                def p95 = sorted[(count * 0.95) as int]

                println "方法: $methodName"
                println "  调用次数: $count"
                println "  总时间: ${total / 1_000_000}ms"
                println "  平均时间: ${avg / 1_000_000}ms"
                println "  最小时间: ${min / 1_000_000}ms"
                println "  最大时间: ${max / 1_000_000}ms"
                println "  中位数: ${median / 1_000_000}ms"
                println "  95分位: ${p95 / 1_000_000}ms"
                println ""
            }
        }
    }

    static void main(String[] args) {
        // 1. 启动线程监控
        def monitor = monitorThreadStates()

        // 2. 创建一些测试线程
        def threads = []
        3.times { i ->
            def thread = Thread.start {
                10.times { j ->
                    println "线程 $i 执行任务 $j"
                    Thread.sleep(1000 + Math.random() * 1000)
                }
            }
            threads.add(thread)
        }

        // 3. 检测死锁
        detectDeadlocks()

        // 4. 性能分析
        def profiler = new PerformanceProfiler()
        profiler.startProfiling()

        threads*.join()

        profiler.stopProfiling()
        profiler.printStats()

        // 5. 分析线程转储
        analyzeThreadDump()

        monitor.interrupt()
    }
}
```

## 8. 实战案例：高性能并发Web服务

```groovy
@Grab('org.apache.httpcomponents:httpclient:4.5.13')
@Grab('org.codehaus.gpars:gpars:1.2.1')
import groovyx.gpars.*
import org.apache.http.impl.client.*
import org.apache.http.client.methods.*

class HighPerformanceWebServer {
    def serverPort = 8080
    def executorService
    def requestProcessor
    def connectionPool
    def metrics = [:].withDefault { 0L }

    def start() {
        // 1. 配置线程池
        executorService = Executors.newFixedThreadPool(
            Runtime.getRuntime().availableProcessors() * 2,
            { Runnable r ->
                def thread = new Thread(r)
                thread.name = "WebServer-Worker-${System.currentTimeMillis()}"
                thread.priority = Thread.NORM_PRIORITY
                return thread
            }
        )

        // 2. 配置连接池
        connectionPool = new PoolingHttpClientConnectionManager()
        connectionPool.setMaxTotal(200)
        connectionPool.setDefaultMaxPerRoute(20)

        // 3. 启动HTTP服务器
        def server = new ServerSocket(serverPort)
        println "Web服务器启动在端口 $serverPort"

        // 4. 启动监控线程
        startMonitoring()

        // 5. 接受连接
        while (true) {
            def client = server.accept()
            executorService.submit {
                handleClient(client)
            }
        }
    }

    def handleClient(Socket client) {
        def startTime = System.currentTimeMillis()
        metrics.requests++

        try {
            def reader = new BufferedReader(new InputStreamReader(client.inputStream))
            def writer = new PrintWriter(client.outputStream)

            // 读取HTTP请求
            def requestLine = reader.readLine()
            if (requestLine) {
                def (method, path, version) = requestLine.split(' ')

                // 处理请求头
                def headers = [:]
                def line
                while ((line = reader.readLine()) && !line.isEmpty()) {
                    def (key, value) = line.split(': ', 2)
                    headers[key] = value
                }

                // 异步处理请求
                def response = processRequestAsync(method, path, headers)

                // 发送响应
                writer.println "HTTP/1.1 ${response.status}"
                writer.println "Content-Type: ${response.contentType}"
                writer.println "Content-Length: ${response.content.length()}"
                writer.println "Connection: close"
                writer.println ""
                writer.println response.content
                writer.flush()
            }

            metrics.successfulRequests++
        } catch (Exception e) {
            metrics.failedRequests++
            println "请求处理失败: ${e.message}"
        } finally {
            client.close()

            def duration = System.currentTimeMillis() - startTime
            metrics.totalProcessingTime += duration
            metrics.averageProcessingTime = metrics.totalProcessingTime / metrics.requests
        }
    }

    def processRequestAsync(String method, String path, Map headers) {
        // 使用GPars进行异步处理
        def result = GParsPool.withPool {
            GParsPoolAsync.async {
                processRequest(method, path, headers)
            }
        }

        return result.get(5000) // 5秒超时
    }

    def processRequest(String method, String path, Map headers) {
        metrics.processedRequests++

        try {
            switch (path) {
                case '/api/users':
                    return processUsersRequest(method, headers)
                case '/api/orders':
                    return processOrdersRequest(method, headers)
                case '/api/metrics':
                    return processMetricsRequest(method, headers)
                case '/health':
                    return [status: '200 OK', contentType: 'application/json', content: '{"status": "healthy"}']
                default:
                    return [status: '404 Not Found', contentType: 'text/plain', content: 'Not Found']
            }
        } catch (Exception e) {
            metrics.failedRequests++
            return [status: '500 Internal Server Error', contentType: 'text/plain', content: 'Internal Server Error']
        }
    }

    def processUsersRequest(String method, Map headers) {
        if (method == 'GET') {
            def users = [
                [id: 1, name: '张三', email: 'zhangsan@example.com'],
                [id: 2, name: '李四', email: 'lisi@example.com'],
                [id: 3, name: '王五', email: 'wangwu@example.com']
            ]
            return [status: '200 OK', contentType: 'application/json', content: groovy.json.JsonOutput.toJson(users)]
        } else {
            return [status: '405 Method Not Allowed', contentType: 'text/plain', content: 'Method Not Allowed']
        }
    }

    def processOrdersRequest(String method, Map headers) {
        if (method == 'GET') {
            def orders = [
                [id: 1, userId: 1, product: '商品A', amount: 100.0],
                [id: 2, userId: 2, product: '商品B', amount: 200.0],
                [id: 3, userId: 3, product: '商品C', amount: 300.0]
            ]
            return [status: '200 OK', contentType: 'application/json', content: groovy.json.JsonOutput.toJson(orders)]
        } else {
            return [status: '405 Method Not Allowed', contentType: 'text/plain', content: 'Method Not Allowed']
        }
    }

    def processMetricsRequest(String method, Map headers) {
        if (method == 'GET') {
            def metricsData = [
                requests: metrics.requests,
                successfulRequests: metrics.successfulRequests,
                failedRequests: metrics.failedRequests,
                processedRequests: metrics.processedRequests,
                averageProcessingTime: metrics.averageProcessingTime,
                uptime: System.currentTimeMillis() - metrics.startTime,
                threadPool: [
                    activeThreads: ((ThreadPoolExecutor) executorService).activeCount,
                    totalThreads: ((ThreadPoolExecutor) executorService).poolSize,
                    queueSize: ((ThreadPoolExecutor) executorService).queue.size()
                ],
                connectionPool: [
                    totalConnections: connectionPool.totalStats.max,
                    availableConnections: connectionPool.totalStats.available,
                    pendingConnections: connectionPool.totalStats.pending
                ]
            ]
            return [status: '200 OK', contentType: 'application/json', content: groovy.json.JsonOutput.toJson(metricsData)]
        } else {
            return [status: '405 Method Not Allowed', contentType: 'text/plain', content: 'Method Not Allowed']
        }
    }

    def startMonitoring() {
        metrics.startTime = System.currentTimeMillis()

        Thread.start {
            while (true) {
                println "\n=== Web服务器监控 ==="
                println "总请求数: ${metrics.requests}"
                println "成功请求数: ${metrics.successfulRequests}"
                println "失败请求数: ${metrics.failedRequests}"
                println "处理请求数: ${metrics.processedRequests}"
                println "平均处理时间: ${String.format("%.2f", metrics.averageProcessingTime)}ms"
                println "运行时间: ${System.currentTimeMillis() - metrics.startTime}ms"

                def executor = executorService as ThreadPoolExecutor
                println "线程池状态: ${executor.activeCount}/${executor.poolSize} 活跃, 队列大小: ${executor.queue.size()}"

                println "连接池状态: ${connectionPool.totalStats.available}/${connectionPool.totalStats.max} 可用"

                Thread.sleep(10000)
            }
        }
    }

    static void main(String[] args) {
        def server = new HighPerformanceWebServer()
        server.start()
    }
}
```

## 9. 总结

Groovy并发编程结合了Java的强大并发基础设施和Groovy的简洁语法，为开发者提供了构建高性能并发应用的强大工具。通过本文的深入解析，我们掌握了：

1. **基础并发概念**：线程、线程池、同步机制
2. **高级并发模式**：Actor模型、CSP、数据并行
3. **并发工具**：GPars库、并发集合、异步编程
4. **性能优化**：锁优化、无锁算法、性能分析
5. **测试与调试**：并发测试、死锁检测、性能监控
6. **实战应用**：高性能Web服务、数据处理系统

### 最佳实践建议：

1. **选择合适的并发模型**：根据应用场景选择最适合的并发模式
2. **避免过度同步**：合理使用锁，减少锁竞争
3. **使用并发集合**：优先选择线程安全的集合类
4. **监控性能**：建立完善的性能监控体系
5. **充分测试**：进行并发测试和压力测试
6. **处理异常**：正确处理并发环境中的异常情况

掌握Groovy并发编程技术将帮助你构建出高性能、高可用性的并发应用程序，充分发挥多核处理器的计算能力。