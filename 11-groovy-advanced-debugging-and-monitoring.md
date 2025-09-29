# Groovy高级调试与监控

## 目录

1. [引言](#引言)
2. [Groovy调试基础](#groovy调试基础)
3. [高级调试技术](#高级调试技术)
4. [性能监控](#性能监控)
5. [内存分析](#内存分析)
6. [线程调试](#线程调试)
7. [远程调试](#远程调试)
8. [日志与追踪](#日志与追踪)
9. [监控工具集成](#监控工具集成)
10. [调试最佳实践](#调试最佳实践)
11. [实战案例](#实战案例)
12. [总结](#总结)

## 引言

调试和监控是软件开发过程中不可或缺的环节。Groovy作为JVM上的动态语言，提供了丰富的调试和监控工具。本章节将深入探讨Groovy的高级调试技术、性能监控方法、内存分析以及各种监控工具的集成，帮助开发者构建稳定、高效的Groovy应用程序。

### 调试与监控的重要性

1. **问题定位**：快速定位和解决代码中的问题
2. **性能优化**：识别性能瓶颈并进行优化
3. **资源管理**：监控内存使用和线程状态
4. **生产环境**：在生产环境中进行问题诊断

## Groovy调试基础

### 基本调试概念

```groovy
class GroovyDebuggingBasics {

    // 1. 断点调试
    static void demonstrateBreakpoints() {
        def numbers = [1, 2, 3, 4, 5]
        def sum = 0

        // 设置断点在这里观察变量状态
        numbers.each { number ->
            sum += number  // 断点：观察sum的变化
            println "Current number: $number, Sum: $sum"
        }

        println "Final sum: $sum"
    }

    // 2. 调试输出
    static void demonstrateDebugOutput() {
        def process = { input ->
            println "[DEBUG] Processing input: $input"  // 调试信息
            def result = input * 2
            println "[DEBUG] Generated result: $result"  // 调试信息
            result
        }

        def data = [10, 20, 30]
        def results = data.collect(process)
        println "Results: $results"
    }

    // 3. 条件调试
    static void demonstrateConditionalDebugging() {
        def items = ["apple", "banana", "cherry", "date", "elderberry"]

        items.each { item ->
            if (item.length() > 5) {  // 条件调试
                println "[DEBUG] Long item found: $item (${item.length()} characters)"
            }
        }
    }

    // 4. 异常调试
    static void demonstrateExceptionDebugging() {
        def riskyOperation = { value ->
            try {
                println "[DEBUG] Attempting operation with value: $value"
                def result = 100 / value
                println "[DEBUG] Operation successful: $result"
                result
            } catch (Exception e) {
                println "[ERROR] Operation failed: ${e.message}"
                println "[ERROR] Stack trace:"
                e.stackTrace.each { println "  $it" }
                throw e
            }
        }

        riskyOperation(10)
        riskyOperation(0)  // 会抛出异常
    }
}
```

### 调试工具使用

```groovy
class DebuggingTools {

    // 1. Groovy Console调试
    static void demonstrateGroovyConsole() {
        // Groovy Console是调试Groovy代码的强大工具
        // 支持交互式执行和实时调试
        def consoleCode = """
            def data = [1, 2, 3, 4, 5]
            def result = data.collect { it * 2 }
            println "Processed data: \$result"

            // 可以在这里设置断点
            def sum = result.sum()
            println "Sum: \$sum"
        """

        println "Groovy Console code example:"
        println consoleCode

        // 在实际开发中，可以将代码复制到Groovy Console中执行
    }

    // 2. 断言调试
    static void demonstrateAssertions() {
        def calculateDiscount = { price, discountRate ->
            assert price > 0 : "Price must be positive"
            assert discountRate >= 0 && discountRate <= 1 : "Discount rate must be between 0 and 1"

            def discountedPrice = price * (1 - discountRate)
            assert discountedPrice > 0 : "Discounted price must be positive"
            assert discountedPrice <= price : "Discounted price must be less than or equal to original price"

            discountedPrice
        }

        // 正常情况
        def result1 = calculateDiscount(100, 0.1)
        println "Discounted price: $result1"

        // 会触发断言错误
        try {
            calculateDiscount(-100, 0.1)
        } catch (AssertionError e) {
            println "Assertion caught: ${e.message}"
        }
    }

    // 3. 调试包装器
    static class DebugWrapper {
        static def wrapWithDebug = { Closure operation, String operationName ->
            return { Object... args ->
                println "[DEBUG] Starting $operationName with args: $args"
                def startTime = System.currentTimeMillis()

                try {
                    def result = operation(*args)
                    def endTime = System.currentTimeMillis()
                    println "[DEBUG] $operationName completed in ${endTime - startTime}ms with result: $result"
                    result
                } catch (Exception e) {
                    def endTime = System.currentTimeMillis()
                    println "[ERROR] $operationName failed in ${endTime - startTime}ms: ${e.message}"
                    throw e
                }
            }
        }

        static void demonstrateDebugWrapper() {
            def add = { a, b -> a + b }
            def multiply = { a, b -> a * b }

            def debugAdd = wrapWithDebug(add, "add")
            def debugMultiply = wrapWithDebug(multiply, "multiply")

            println debugAdd(5, 3)
            println debugMultiply(4, 6)

            try {
                debugMultiply(4, 0)  // 会触发错误
            } catch (Exception e) {
                println "Caught exception: ${e.message}"
            }
        }
    }

    // 4. 状态检查器
    static class StateChecker {
        static def checkState = { Map state, Map expected ->
            def mismatches = []

            expected.each { key, expectedValue ->
                def actualValue = state[key]
                if (actualValue != expectedValue) {
                    mismatches.add("State mismatch for '$key': expected $expectedValue, got $actualValue")
                }
            }

            if (!mismatches.isEmpty()) {
                println "[STATE] State check failed:"
                mismatches.each { println "  $it" }
                false
            } else {
                println "[STATE] State check passed"
                true
            }
        }

        static void demonstrateStateChecker() {
            def applicationState = [
                userCount: 100,
                memoryUsage: 512,
                activeConnections: 25,
                lastUpdate: new Date()
            ]

            def expectedState = [
                userCount: 100,
                memoryUsage: 512,
                activeConnections: 25
            ]

            checkState(applicationState, expectedState)

            // 修改状态以触发检查失败
            applicationState.userCount = 150
            checkState(applicationState, expectedState)
        }
    }
}
```

## 高级调试技术

### 动态代码注入

```groovy
class DynamicCodeInjection {

    // 1. 运行时代码修改
    static class RuntimeCodeModifier {
        static def injectDebugCode = { Class targetClass, String methodName, Closure debugLogic ->
            def originalMethod = targetClass.metaClass.getMetaMethod(methodName)

            targetClass.metaClass."${methodName}Debug" = { Object... args ->
                println "[DEBUG] Calling $methodName with args: $args"
                def result = debugLogic(args)
                println "[DEBUG] $methodName result: $result"
                result
            }
        }

        static def replaceMethod = { Class targetClass, String methodName, Closure newImplementation ->
            targetClass.metaClass."$methodName" = { Object... args ->
                println "[DEBUG] Intercepted call to $methodName"
                newImplementation(*args)
            }
        }

        static def addBeforeAdvice = { Class targetClass, String methodName, Closure beforeAdvice ->
            def originalMethod = targetClass.metaClass.getMetaMethod(methodName)

            targetClass.metaClass."$methodName" = { Object... args ->
                beforeAdvice(args)
                originalMethod.invoke(delegate, args)
            }
        }

        static def addAfterAdvice = { Class targetClass, String methodName, Closure afterAdvice ->
            def originalMethod = targetClass.metaClass.getMetaMethod(methodName)

            targetClass.metaClass."$methodName" = { Object... args ->
                def result = originalMethod.invoke(delegate, args)
                afterAdvice(result)
                result
            }
        }
    }

    // 2. 调试代理
    static class DebugProxy {
        static def createDebugProxy = { Object target ->
            def proxy = Proxy.newProxyInstance(
                target.class.classLoader,
                target.class.interfaces,
                { Object proxy, Method method, Object[] args ->
                    println "[DEBUG] Proxy: Calling ${method.name} with args: $args"
                    def startTime = System.currentTimeMillis()

                    try {
                        def result = method.invoke(target, args)
                        def endTime = System.currentTimeMillis()
                        println "[DEBUG] Proxy: ${method.name} completed in ${endTime - startTime}ms"
                        result
                    } catch (Exception e) {
                        def endTime = System.currentTimeMillis()
                        println "[ERROR] Proxy: ${method.name} failed in ${endTime - startTime}ms: ${e.message}"
                        throw e
                    }
                }
            )
            proxy
        }
    }

    // 3. 方法拦截器
    static class MethodInterceptor {
        static def interceptMethod = { Class targetClass, String methodName, Closure interceptor ->
            def originalMethod = targetClass.metaClass.getMetaMethod(methodName)

            targetClass.metaClass."$methodName" = { Object... args ->
                def context = [
                    methodName: methodName,
                    args: args,
                    target: delegate,
                    startTime: System.currentTimeMillis()
                ]

                try {
                    interceptor(context)
                    def result = originalMethod.invoke(delegate, args)
                    context.result = result
                    context.endTime = System.currentTimeMillis()
                    interceptor(context)
                    result
                } catch (Exception e) {
                    context.exception = e
                    context.endTime = System.currentTimeMillis()
                    interceptor(context)
                    throw e
                }
            }
        }
    }

    static void demonstrateDynamicInjection() {
        // 示例类
        class Calculator {
            def add(a, b) { a + b }
            def multiply(a, b) { a * b }
            def divide(a, b) { a / b }
        }

        def calculator = new Calculator()

        // 注入调试代码
        RuntimeCodeModifier.injectDebugCode(Calculator, "add") { args ->
            println "[INJECT] Add operation: ${args[0]} + ${args[1]}"
            args[0] + args[1]
        }

        println "Add with debug: ${calculator.addDebug(5, 3)}"

        // 方法拦截
        RuntimeCodeModifier.interceptMethod(Calculator, "multiply") { context ->
            println "[INTERCEPT] Before multiply: ${context.args}"
        }

        println "Multiply with interceptor: ${calculator.multiply(4, 6)}"

        // 创建调试代理
        def proxy = DebugProxy.createDebugProxy(calculator)
        println "Proxy divide: ${proxy.divide(10, 2)}"
    }
}
```

### 调试元编程

```groovy
class DebuggingMetaprogramming {

    // 1. 调试MOP
    static class DebugMOP {
        static def enableMethodTracing = { Class targetClass ->
            targetClass.metaClass.invokeMethod = { String name, Object[] args ->
                println "[MOP] Intercepted method call: $name with args: $args"
                def metaMethod = delegate.metaClass.getMetaMethod(name, *args)
                if (metaMethod) {
                    def result = metaMethod.invoke(delegate, args)
                    println "[MOP] Method $name returned: $result"
                    result
                } else {
                    println "[MOP] Method $name not found"
                    throw new MissingMethodException(name, delegate.class, args)
                }
            }
        }

        static def enablePropertyTracing = { Class targetClass ->
            targetClass.metaClass.getProperty = { String name ->
                println "[MOP] Getting property: $name"
                def result = delegate.metaClass.getMetaProperty(name)?.getProperty(delegate)
                println "[MOP] Property $name value: $result"
                result
            }

            targetClass.metaClass.setProperty = { String name, Object value ->
                println "[MOP] Setting property: $name = $value"
                delegate.metaClass.getMetaProperty(name)?.setProperty(delegate, value)
            }
        }
    }

    // 2. 调试Expando
    static class DebugExpando {
        static def createDebugExpando = { Object target ->
            def debugExpando = new Expando(target)

            // 添加调试方法
            debugExpando.debug = { message ->
                println "[DEBUG] ${new Date().format('yyyy-MM-dd HH:mm:ss')} - $message"
            }

            debugExpando.debugMethod = { String methodName, Closure method ->
                debugExpando."${methodName}WithDebug" = { Object... args ->
                    debugExpando.debug("Calling method $methodName with args: $args")
                    def result = method(*args)
                    debugExpando.debug("Method $methodName returned: $result")
                    result
                }
            }

            debugExpando.debugProperty = { String propertyName, Object value ->
                debugExpando.debug("Setting property $propertyName = $value")
                debugExpando."$propertyName" = value
            }

            debugExpando
        }
    }

    // 3. 调试Category
    static class DebugCategory {
        static def debug(Closure self, String message) {
            println "[DEBUG] ${new Date().format('yyyy-MM-dd HH:mm:ss')} - $message"
            self()
        }

        static def time(Closure self, String operationName) {
            def startTime = System.currentTimeMillis()
            println "[TIMER] Starting $operationName"
            def result = self()
            def endTime = System.currentTimeMillis()
            println "[TIMER] $operationName completed in ${endTime - startTime}ms"
            result
        }

        static def trace(Closure self, String traceName) {
            println "[TRACE] Entering $traceName"
            def result = self()
            println "[TRACE] Exiting $traceName"
            result
        }
    }

    static void demonstrateDebuggingMetaprogramming() {
        // 示例类
        class Service {
            def processData(data) {
                println "Processing data: $data"
                data.toUpperCase()
            }

            def getConfig(key) {
                def configs = [timeout: 30, retries: 3]
                configs[key]
            }
        }

        def service = new Service()

        // 启用MOP调试
        DebugMOP.enableMethodTracing(Service)

        println "Calling processData with MOP debugging:"
        println service.processData("hello")

        // 使用调试Expando
        def debugService = DebugExpando.createDebugExpando(service)
        debugService.debug("This is a debug message")

        debugService.debugMethod("enhancedProcess") { data ->
            data.reverse()
        }

        println debugService.enhancedProcess("hello")

        // 使用调试Category
        use(DebugCategory) {
            def result = "hello world".debug("Processing string") {
                it.toUpperCase()
            }.time("string processing") {
                it.reverse()
            }.trace("final transformation")
            println "Final result: $result"
        }
    }
}
```

## 性能监控

### 执行时间监控

```groovy
class PerformanceMonitoring {

    // 1. 方法执行时间监控
    static class ExecutionTimeMonitor {
        private static final Map<String, List<Long>> executionTimes = [:]

        static def monitorExecution = { Closure operation, String operationName ->
            def startTime = System.nanoTime()
            try {
                def result = operation()
                def endTime = System.nanoTime()
                def duration = endTime - startTime
                recordExecutionTime(operationName, duration)
                result
            } catch (Exception e) {
                def endTime = System.nanoTime()
                def duration = endTime - startTime
                recordExecutionTime(operationName, duration)
                throw e
            }
        }

        static def recordExecutionTime = { String operationName, long durationNanos ->
            if (!executionTimes.containsKey(operationName)) {
                executionTimes[operationName] = []
            }
            executionTimes[operationName] << (durationNanos / 1_000_000)  // 转换为毫秒
        }

        static def getExecutionStats = { String operationName ->
            def times = executionTimes[operationName] ?: []
            if (times.isEmpty()) {
                return [count: 0, average: 0, min: 0, max: 0]
            }

            [
                count: times.size(),
                average: times.sum() / times.size(),
                min: times.min(),
                max: times.max()
            ]
        }

        static def printStats = { String operationName ->
            def stats = getExecutionStats(operationName)
            println """
                [PERF] Statistics for '$operationName':
                  Count: ${stats.count}
                  Average: ${stats.average.round(2)}ms
                  Min: ${stats.min}ms
                  Max: ${stats.max}ms
            """.stripIndent()
        }
    }

    // 2. 方法拦截监控
    static class MethodInterceptor {
        static def interceptForMonitoring = { Class targetClass, String methodName ->
            def originalMethod = targetClass.metaClass.getMetaMethod(methodName)

            targetClass.metaClass."$methodName" = { Object... args ->
                def startTime = System.nanoTime()
                try {
                    def result = originalMethod.invoke(delegate, args)
                    def endTime = System.nanoTime()
                    def duration = (endTime - startTime) / 1_000_000
                    println "[PERF] $methodName executed in ${duration.round(2)}ms"
                    result
                } catch (Exception e) {
                    def endTime = System.nanoTime()
                    def duration = (endTime - startTime) / 1_000_000
                    println "[PERF] $methodName failed in ${duration.round(2)}ms"
                    throw e
                }
            }
        }
    }

    // 3. 性能阈值监控
    static class PerformanceThreshold {
        private static final Map<String, Long> thresholds = [:]
        private static final Map<String, List<Long>> violations = [:]

        static def setThreshold = { String operationName, long thresholdMs ->
            thresholds[operationName] = thresholdMs
        }

        static def monitorWithThreshold = { Closure operation, String operationName ->
            def startTime = System.nanoTime()
            try {
                def result = operation()
                def endTime = System.nanoTime()
                def duration = (endTime - startTime) / 1_000_000

                if (duration > (thresholds[operationName] ?: Long.MAX_VALUE)) {
                    recordViolation(operationName, duration)
                    println "[PERF] WARNING: $operationName exceeded threshold (${duration.round(2)}ms)"
                }

                result
            } catch (Exception e) {
                def endTime = System.nanoTime()
                def duration = (endTime - startTime) / 1_000_000
                recordViolation(operationName, duration)
                throw e
            }
        }

        static def recordViolation = { String operationName, long duration ->
            if (!violations.containsKey(operationName)) {
                violations[operationName] = []
            }
            violations[operationName] << duration
        }

        static def getViolations = { String operationName ->
            violations[operationName] ?: []
        }
    }

    static void demonstratePerformanceMonitoring() {
        // 示例操作
        def fastOperation = {
            Thread.sleep(10)
            "Fast result"
        }

        def slowOperation = {
            Thread.sleep(100)
            "Slow result"
        }

        def variableOperation = {
            def sleepTime = Math.random() * 200
            Thread.sleep(sleepTime as long)
            "Variable result in ${sleepTime}ms"
        }

        // 执行时间监控
        println "=== Execution Time Monitoring ==="
        5.times {
            ExecutionTimeMonitor.monitorExecution(fastOperation, "fastOperation")
        }

        3.times {
            ExecutionTimeMonitor.monitorExecution(slowOperation, "slowOperation")
        }

        ExecutionTimeMonitor.printStats("fastOperation")
        ExecutionTimeMonitor.printStats("slowOperation")

        // 阈值监控
        println "\n=== Threshold Monitoring ==="
        PerformanceThreshold.setThreshold("variableOperation", 50)

        10.times {
            PerformanceThreshold.monitorWithThreshold(variableOperation, "variableOperation")
        }

        def violations = PerformanceThreshold.getViolations("variableOperation")
        println "Threshold violations: ${violations.size()}"
        violations.each { println "  ${it.round(2)}ms" }
    }
}
```

### 内存使用监控

```groovy
class MemoryMonitoring {

    // 1. 内存使用监控
    static class MemoryMonitor {
        static def getMemoryUsage = {
            def runtime = Runtime.runtime
            def totalMemory = runtime.totalMemory()
            def freeMemory = runtime.freeMemory()
            def usedMemory = totalMemory - freeMemory
            def maxMemory = runtime.maxMemory()

            [
                total: totalMemory / (1024 * 1024),
                used: usedMemory / (1024 * 1024),
                free: freeMemory / (1024 * 1024),
                max: maxMemory / (1024 * 1024),
                usagePercent: (usedMemory * 100 / maxMemory)
            ]
        }

        static def printMemoryUsage = { String context = "" ->
            def usage = getMemoryUsage()
            println """
                [MEMORY] $context
                  Total: ${usage.total.round(2)}MB
                  Used: ${usage.used.round(2)}MB
                  Free: ${usage.free.round(2)}MB
                  Max: ${usage.max.round(2)}MB
                  Usage: ${usage.usagePercent.round(2)}%
            """.stripIndent()
        }

        static def monitorMemoryUsage = { Closure operation, String operationName ->
            def beforeMemory = getMemoryUsage()
            println "[MEMORY] Before $operationName: ${beforeMemory.used.round(2)}MB"

            def result = operation()

            def afterMemory = getMemoryUsage()
            println "[MEMORY] After $operationName: ${afterMemory.used.round(2)}MB"
            println "[MEMORY] Memory delta: ${(afterMemory.used - beforeMemory.used).round(2)}MB"

            result
        }
    }

    // 2. 对象创建监控
    static class ObjectCreationMonitor {
        private static final Map<String, Integer> creationCounts = [:]

        static def monitorObjectCreation = { Closure operation, String objectName ->
            def result = operation()
            incrementCreationCount(objectName)
            result
        }

        static def incrementCreationCount = { String objectName ->
            creationCounts[objectName] = (creationCounts[objectName] ?: 0) + 1
        }

        static def getCreationCount = { String objectName ->
            creationCounts[objectName] ?: 0
        }

        static def printCreationStats = {
            println "[OBJECT] Creation statistics:"
            creationCounts.each { objectName, count ->
                println "  $objectName: $count"
            }
        }

        static def resetCounts = {
            creationCounts.clear()
        }
    }

    // 3. 内存泄漏检测
    static class MemoryLeakDetector {
        private static final Map<String, WeakReference> trackedObjects = [:]
        private static final Map<String, Long> creationTimes = [:]

        static def trackObject = { String objectId, Object obj ->
            trackedObjects[objectId] = new WeakReference(obj)
            creationTimes[objectId] = System.currentTimeMillis()
        }

        static def checkForLeaks = {
            def leakedObjects = []

            trackedObjects.each { objectId, weakRef ->
                def obj = weakRef.get()
                if (obj != null) {
                    def age = System.currentTimeMillis() - creationTimes[objectId]
                    if (age > 60_000) {  // 超过1分钟
                        leakedObjects << [id: objectId, age: age, object: obj]
                    }
                }
            }

            leakedObjects
        }

        static def printLeakReport = {
            def leaks = checkForLeaks()
            if (leaks.isEmpty()) {
                println "[LEAK] No potential memory leaks detected"
            } else {
                println "[LEAK] Potential memory leaks detected:"
                leaks.each { leak ->
                    println "  Object ${leak.id}: age ${leak.age / 1000}s, type: ${leak.object.class.simpleName}"
                }
            }
        }

        static def cleanupTracking = {
            def iterator = trackedObjects.entrySet().iterator()
            while (iterator.hasNext()) {
                def entry = iterator.next()
                if (entry.value.get() == null) {
                    iterator.remove()
                    creationTimes.remove(entry.key)
                }
            }
        }
    }

    static void demonstrateMemoryMonitoring() {
        // 内存使用监控
        println "=== Memory Usage Monitoring ==="
        MemoryMonitor.printMemoryUsage("Initial")

        def createLargeData = {
            def data = []
            10000.times { data.add("Item $it" * 100) }
            data
        }

        MemoryMonitor.monitorMemoryUsage(createLargeData, "createLargeData")
        MemoryMonitor.printMemoryUsage("After creating large data")

        // 对象创建监控
        println "\n=== Object Creation Monitoring ==="
        def createUser = { id ->
            ObjectCreationMonitor.monitorObjectCreation("user") {
                [id: id, name: "User $id", created: new Date()]
            }
        }

        10.times { createUser(it) }

        ObjectCreationMonitor.printCreationStats()

        // 内存泄漏检测
        println "\n=== Memory Leak Detection ==="
        def objects = []
        5.times { i ->
            def obj = [id: i, data: "Data" * 1000]
            MemoryLeakDetector.trackObject("obj$i", obj)
            objects.add(obj)
        }

        MemoryLeakDetector.printLeakReport()

        // 清理一些对象
        objects = objects[0..2]
        System.gc()
        Thread.sleep(1000)

        MemoryLeakDetector.printLeakReport()
    }
}
```

## 线程调试

### 线程状态监控

```groovy
class ThreadDebugging {

    // 1. 线程状态监控
    static class ThreadMonitor {
        static def getAllThreadInfo = {
            Thread.allStackTraces.collectMany { thread, stackTrace ->
                [[
                    name: thread.name,
                    id: thread.id,
                    state: thread.state,
                    priority: thread.priority,
                    isDaemon: thread.daemon,
                    isAlive: thread.alive,
                    isInterrupted: thread.interrupted,
                    stackTrace: stackTrace
                ]]
            }
        }

        static def printThreadInfo = {
            def threadInfo = getAllThreadInfo()
            println "[THREAD] Active threads: ${threadInfo.size()}"
            threadInfo.each { info ->
                println """
                    [THREAD] ${info.name} (ID: ${info.id})
                      State: ${info.state}
                      Priority: ${info.priority}
                      Daemon: ${info.isDaemon}
                      Alive: ${info.isAlive}
                      Interrupted: ${info.isInterrupted}
                      Stack trace: ${info.stackTrace.take(3).join(' <- ')}
                """.stripIndent()
            }
        }

        static def findThreadByName = { String name ->
            Thread.allStackTraces.keySet().find { it.name == name }
        }

        static def getThreadDump = {
            def dump = new StringBuilder()
            dump.append("===== Thread Dump =====\n")

            Thread.allStackTraces.each { thread, stackTrace ->
                dump.append("\"${thread.name}\" #${thread.id} ${thread.state}\n")
                dump.append("   java.lang.Thread.State: ${thread.state}\n")

                stackTrace.each { element ->
                    dump.append("\t at $element\n")
                }
                dump.append("\n")
            }

            dump.toString()
        }
    }

    // 2. 线程死锁检测
    static class DeadlockDetector {
        static def detectDeadlocks = {
            def threads = Thread.allStackTraces.keySet()
            def deadlocks = []

            threads.each { thread ->
                if (thread.state == Thread.State.BLOCKED) {
                    def blockedOn = findLockOwner(thread)
                    if (blockedOn && isWaitingFor(blockedOn, thread)) {
                        deadlocks << [thread1: thread, thread2: blockedOn]
                    }
                }
            }

            deadlocks
        }

        static def findLockOwner = { Thread blockedThread ->
            def stackTrace = Thread.allStackTraces[blockedThread]
            if (stackTrace) {
                def lockElement = stackTrace.find {
                    it.className.contains("Lock") || it.className.contains("synchronized")
                }
                if (lockElement) {
                    return threads.find { it != blockedThread && isHoldingLock(it, lockElement) }
                }
            }
            null
        }

        static def isHoldingLock = { Thread thread, StackTraceElement lockElement ->
            def stackTrace = Thread.allStackTraces[thread]
            stackTrace && stackTrace.any { it == lockElement }
        }

        static def isWaitingFor = { Thread owner, Thread waiter ->
            def ownerStackTrace = Thread.allStackTraces[owner]
            def waiterStackTrace = Thread.allStackTraces[waiter]

            ownerStackTrace && waiterStackTrace &&
            ownerStackTrace.any { ownerElement ->
                waiterStackTrace.any { waiterElement ->
                    ownerElement.fileName == waiterElement.fileName &&
                    ownerElement.lineNumber == waiterElement.lineNumber
                }
            }
        }

        static def printDeadlockReport = {
            def deadlocks = detectDeadlocks()
            if (deadlocks.isEmpty()) {
                println "[DEADLOCK] No deadlocks detected"
            } else {
                println "[DEADLOCK] Deadlocks detected:"
                deadlocks.each { deadlock ->
                    println "  Deadlock between ${deadlock.thread1.name} and ${deadlock.thread2.name}"
                }
            }
        }
    }

    // 3. 线程活动监控
    static class ThreadActivityMonitor {
        private static final Map<String, Long> threadStartTimes = [:]
        private static final Map<String, Long> threadActivityCounts = [:]

        static def monitorThreadActivity = { Thread thread, String threadName ->
            threadStartTimes[threadName] = System.currentTimeMillis()
            threadActivityCounts[threadName] = 0

            thread.metaClass.invokeMethod = { String name, Object[] args ->
                threadActivityCounts[threadName] = (threadActivityCounts[threadName] ?: 0) + 1
                delegate.metaClass.getMetaMethod(name, *args).invoke(delegate, args)
            }
        }

        static def getThreadActivityStats = { String threadName ->
            def startTime = threadStartTimes[threadName]
            def activityCount = threadActivityCounts[threadName] ?: 0
            def duration = startTime ? System.currentTimeMillis() - startTime : 0

            [
                name: threadName,
                startTime: startTime,
                duration: duration,
                activityCount: activityCount,
                activityRate: duration > 0 ? (activityCount * 1000 / duration).round(2) : 0
            ]
        }

        static def printActivityStats = { String threadName ->
            def stats = getThreadActivityStats(threadName)
            println """
                [ACTIVITY] Thread: ${stats.name}
                  Duration: ${stats.duration}ms
                  Activities: ${stats.activityCount}
                  Activity rate: ${stats.activityRate}/sec
            """.stripIndent()
        }
    }

    static void demonstrateThreadDebugging() {
        // 线程状态监控
        println "=== Thread State Monitoring ==="
        ThreadMonitor.printThreadInfo()

        // 创建一些线程
        def threads = []
        3.times { i ->
            def thread = Thread.start {
                5.times { j ->
                    println "[THREAD-${Thread.currentThread().name}] Processing item $j"
                    Thread.sleep(100)
                }
            }
            thread.name = "Worker-$i"
            threads.add(thread)
        }

        Thread.sleep(500)

        // 监控线程活动
        println "\n=== Thread Activity Monitoring ==="
        threads.each { thread ->
            ThreadActivityMonitor.monitorThreadActivity(thread, thread.name)
        }

        threads.each { it.join() }

        threads.each { thread ->
            ThreadActivityMonitor.printActivityStats(thread.name)
        }

        // 死锁检测
        println "\n=== Deadlock Detection ==="
        DeadlockDetector.printDeadlockReport()

        // 获取线程转储
        println "\n=== Thread Dump ==="
        def threadDump = ThreadMonitor.getThreadDump()
        println threadDump.length() > 1000 ? threadDump[0..1000] + "..." : threadDump
    }
}
```

### 并发问题调试

```groovy
class ConcurrencyDebugging {

    // 1. 竞态条件检测
    static class RaceConditionDetector {
        private static final Map<String, List<Long>> accessTimes = [:]

        static def detectRaceCondition = { String sharedResource, Closure operation ->
            def startTime = System.nanoTime()
            def result = operation()
            def endTime = System.nanoTime()

            recordAccess(sharedResource, startTime, endTime)
            checkForRaceConditions(sharedResource)

            result
        }

        static def recordAccess = { String resource, long startTime, long endTime ->
            if (!accessTimes.containsKey(resource)) {
                accessTimes[resource] = []
            }
            accessTimes[resource] << [start: startTime, end: endTime]
        }

        static def checkForRaceConditions = { String resource ->
            def accesses = accessTimes[resource] ?: []
            if (accesses.size() < 2) return

            // 检查重叠的访问时间
            for (int i = 0; i < accesses.size() - 1; i++) {
                for (int j = i + 1; j < accesses.size(); j++) {
                    def access1 = accesses[i]
                    def access2 = accesses[j]

                    if (isOverlapping(access1, access2)) {
                        println "[RACE] Potential race condition detected for resource '$resource'"
                        println "  Access 1: ${access1.start} - ${access1.end}"
                        println "  Access 2: ${access2.start} - ${access2.end}"
                    }
                }
            }
        }

        static def isOverlapping = { access1, access2 ->
            !(access1.end < access2.start || access2.end < access1.start)
        }

        static def reset = {
            accessTimes.clear()
        }
    }

    // 2. 同步监控
    static class SynchronizationMonitor {
        private static final Map<String, Integer> lockCounts = [:]
        private static final Map<String, Long> lockTimes = [:]

        static def monitorSynchronization = { String lockName, Closure operation ->
            def startTime = System.currentTimeMillis()

            try {
                incrementLockCount(lockName)
                def result = operation()
                recordLockTime(lockName, System.currentTimeMillis() - startTime)
                result
            } catch (Exception e) {
                recordLockTime(lockName, System.currentTimeMillis() - startTime)
                throw e
            } finally {
                decrementLockCount(lockName)
            }
        }

        static def incrementLockCount = { String lockName ->
            lockCounts[lockName] = (lockCounts[lockName] ?: 0) + 1
        }

        static def decrementLockCount = { String lockName ->
            if (lockCounts.containsKey(lockName)) {
                lockCounts[lockName] = Math.max(0, lockCounts[lockName] - 1)
            }
        }

        static def recordLockTime = { String lockName, long duration ->
            if (!lockTimes.containsKey(lockName)) {
                lockTimes[lockName] = 0
            }
            lockTimes[lockName] += duration
        }

        static def getLockStats = { String lockName ->
            [
                name: lockName,
                count: lockCounts[lockName] ?: 0,
                totalTime: lockTimes[lockName] ?: 0,
                averageTime: (lockCounts[lockName] ?: 0) > 0 ?
                    (lockTimes[lockName] ?: 0) / (lockCounts[lockName] ?: 1) : 0
            ]
        }

        static def printLockStats = { String lockName ->
            def stats = getLockStats(lockName)
            println """
                [SYNC] Lock statistics for '$lockName':
                  Count: ${stats.count}
                  Total time: ${stats.totalTime}ms
                  Average time: ${stats.averageTime.round(2)}ms
            """.stripIndent()
        }
    }

    // 3. 线程池监控
    static class ThreadPoolMonitor {
        private static final Map<String, ThreadPoolExecutor> threadPools = [:]
        private static final Map<String, List<Long>> taskExecutionTimes = [:]

        static def createMonitoredThreadPool = { String poolName, int coreSize, int maxSize ->
            def executor = new ThreadPoolExecutor(
                coreSize, maxSize, 60L, TimeUnit.SECONDS,
                new LinkedBlockingQueue<Runnable>()
            )
            threadPools[poolName] = executor
            executor
        }

        static def monitorTaskExecution = { String poolName, Closure task ->
            def executor = threadPools[poolName]
            if (!executor) {
                throw new IllegalArgumentException("Thread pool '$poolName' not found")
            }

            def startTime = System.currentTimeMillis()
            def future = executor.submit(task as Callable)

            future.whenComplete { result, exception ->
                def endTime = System.currentTimeMillis()
                recordTaskExecution(poolName, endTime - startTime)
            }

            future.get()
        }

        static def recordTaskExecution = { String poolName, long duration ->
            if (!taskExecutionTimes.containsKey(poolName)) {
                taskExecutionTimes[poolName] = []
            }
            taskExecutionTimes[poolName] << duration
        }

        static def getPoolStats = { String poolName ->
            def executor = threadPools[poolName]
            if (!executor) return null

            def executionTimes = taskExecutionTimes[poolName] ?: []

            [
                name: poolName,
                activeThreads: executor.activeCount,
                corePoolSize: executor.corePoolSize,
                maxPoolSize: executor.maximumPoolSize,
                queueSize: executor.queue.size(),
                completedTasks: executor.completedTaskCount,
                taskCount: executionTimes.size(),
                averageTaskTime: executionTimes.size() > 0 ?
                    executionTimes.sum() / executionTimes.size() : 0
            ]
        }

        static def printPoolStats = { String poolName ->
            def stats = getPoolStats(poolName)
            if (!stats) return

            println """
                [POOL] Thread pool statistics for '$poolName':
                  Active threads: ${stats.activeThreads}
                  Core size: ${stats.corePoolSize}
                  Max size: ${stats.maxPoolSize}
                  Queue size: ${stats.queueSize}
                  Completed tasks: ${stats.completedTasks}
                  Average task time: ${stats.averageTaskTime.round(2)}ms
            """.stripIndent()
        }
    }

    static void demonstrateConcurrencyDebugging() {
        // 竞态条件检测
        println "=== Race Condition Detection ==="
        def sharedCounter = 0
        def threads = []

        10.times { i ->
            def thread = Thread.start {
                100.times { j ->
                    RaceConditionDetector.detectRaceCondition("counter") {
                        sharedCounter++
                    }
                }
            }
            threads.add(thread)
        }

        threads.each { it.join() }
        println "Final counter value: $sharedCounter"

        // 同步监控
        println "\n=== Synchronization Monitoring ==="
        def sharedResource = new Object()
        def syncCounter = 0

        5.times { i ->
            Thread.start {
                10.times { j ->
                    SynchronizationMonitor.monitorSynchronization("resourceLock") {
                        syncCounter++
                        Thread.sleep(10)
                    }
                }
            }
        }

        Thread.sleep(1000)
        SynchronizationMonitor.printLockStats("resourceLock")

        // 线程池监控
        println "\n=== Thread Pool Monitoring ==="
        def pool = ThreadPoolMonitor.createMonitoredThreadPool("workerPool", 2, 4)

        20.times { i ->
            ThreadPoolMonitor.monitorTaskExecution("workerPool") {
                Thread.sleep(Math.random() * 100)
                "Task $i completed"
            }
        }

        Thread.sleep(1000)
        ThreadPoolMonitor.printPoolStats("workerPool")
    }
}
```

## 远程调试

### 远程调试设置

```groovy
class RemoteDebugging {

    // 1. 远程调试配置
    static class RemoteDebugConfig {
        static def createDebugArguments = { int port = 5005, boolean suspend = false ->
            def suspendFlag = suspend ? "y" : "n"
            "-agentlib:jdwp=transport=dt_socket,server=y,suspend=$suspendFlag,address=$port"
        }

        static def createJvmArguments = { Map config ->
            def debugArgs = createDebugArguments(config.port, config.suspend)
            def memoryArgs = "-Xmx${config.maxMemory}m -Xms${config.minMemory}m"
            def gcArgs = config.gcArgs ?: ""

            "$debugArgs $memoryArgs $gcArgs"
        }

        static def validateConfig = { Map config ->
            def required = ['port', 'maxMemory', 'minMemory']
            def missing = required.findAll { !config.containsKey(it) }
            if (!missing.isEmpty()) {
                throw new IllegalArgumentException("Missing required config: $missing")
            }

            if (config.port < 1024 || config.port > 65535) {
                throw new IllegalArgumentException("Port must be between 1024 and 65535")
            }

            true
        }
    }

    // 2. 远程调试服务器
    static class RemoteDebugServer {
        private ServerSocket serverSocket
        private int port
        private boolean running = false
        private List<ClientConnection> clients = []

        RemoteDebugServer(int port) {
            this.port = port
        }

        def start = {
            serverSocket = new ServerSocket(port)
            running = true
            println "[REMOTE] Debug server started on port $port"

            Thread.start {
                while (running) {
                    try {
                        def clientSocket = serverSocket.accept()
                        def connection = new ClientConnection(clientSocket)
                        clients.add(connection)
                        handleClient(connection)
                    } catch (Exception e) {
                        if (running) {
                            println "[REMOTE] Error accepting client: ${e.message}"
                        }
                    }
                }
            }
        }

        def stop = {
            running = false
            clients.each { it.close() }
            serverSocket?.close()
            println "[REMOTE] Debug server stopped"
        }

        def handleClient = { ClientConnection connection ->
            Thread.start {
                try {
                    while (connection.isConnected()) {
                        def command = connection.readCommand()
                        if (command) {
                            def response = processCommand(command)
                            connection.sendResponse(response)
                        }
                    }
                } catch (Exception e) {
                    println "[REMOTE] Error handling client: ${e.message}"
                } finally {
                    clients.remove(connection)
                    connection.close()
                }
            }
        }

        def processCommand = { String command ->
            def parts = command.split(' ', 2)
            def cmd = parts[0]
            def args = parts.length > 1 ? parts[1] : ''

            switch (cmd.toLowerCase()) {
                case 'info':
                    getSystemInfo()
                    break
                case 'threads':
                    getThreadInfo()
                    break
                case 'memory':
                    getMemoryInfo()
                    break
                case 'gc':
                    System.gc()
                    "GC triggered"
                    break
                case 'classes':
                    getClassInfo()
                    break
                default:
                    "Unknown command: $cmd"
            }
        }

        def getSystemInfo = {
            def runtime = Runtime.runtime
            def props = System.properties

            """
                System Information:
                  OS: ${props['os.name']} ${props['os.version']}
                  Java: ${props['java.version']}
                  JVM: ${props['java.vm.name']}
                  Processors: ${runtime.availableProcessors()}
                  Max Memory: ${runtime.maxMemory() / (1024*1024)}MB
            """.stripIndent()
        }

        def getThreadInfo = {
            def threadInfo = Thread.allStackTraces.collectMany { thread, stackTrace ->
                ["${thread.name}: ${thread.state} (${stackTrace.size()} frames)"]
            }
            "Active Threads:\n" + threadInfo.join('\n')
        }

        def getMemoryInfo = {
            def runtime = Runtime.runtime
            def total = runtime.totalMemory() / (1024*1024)
            def free = runtime.freeMemory() / (1024*1024)
            def used = total - free
            def max = runtime.maxMemory() / (1024*1024)

            """
                Memory Usage:
                  Total: ${total.round(2)}MB
                  Used: ${used.round(2)}MB
                  Free: ${free.round(2)}MB
                  Max: ${max.round(2)}MB
                  Usage: ${(used/max*100).round(2)}%
            """.stripIndent()
        }

        def getClassInfo = {
            def classLoader = Thread.currentThread().contextClassLoader
            def classes = []
            try {
                def field = classLoader.class.getDeclaredField('classes')
                field.accessible = true
                classes = field.get(classLoader)
            } catch (Exception e) {
                // 忽略异常，使用备用方法
            }

            "Loaded Classes: ${classes.size()}"
        }
    }

    // 3. 远程调试客户端
    static class RemoteDebugClient {
        private Socket socket
        private BufferedReader reader
        private PrintWriter writer

        RemoteDebugClient(String host, int port) {
            this.socket = new Socket(host, port)
            this.reader = new BufferedReader(new InputStreamReader(socket.inputStream))
            this.writer = new PrintWriter(socket.outputStream, true)
        }

        def sendCommand = { String command ->
            writer.println(command)
            writer.flush()
        }

        def readResponse = {
            def response = new StringBuilder()
            def line
            while ((line = reader.readLine()) != null) {
                if (line.isEmpty()) break
                response.append(line).append('\n')
            }
            response.toString().trim()
        }

        def executeCommand = { String command ->
            sendCommand(command)
            readResponse()
        }

        def close = {
            reader?.close()
            writer?.close()
            socket?.close()
        }

        def isConnected = {
            socket && !socket.closed
        }
    }

    // 4. 客户端连接
    static class ClientConnection {
        private Socket socket
        private BufferedReader reader
        private PrintWriter writer

        ClientConnection(Socket socket) {
            this.socket = socket
            this.reader = new BufferedReader(new InputStreamReader(socket.inputStream))
            this.writer = new PrintWriter(socket.outputStream, true)
        }

        def readCommand = {
            try {
                reader.readLine()
            } catch (Exception e) {
                null
            }
        }

        def sendResponse = { String response ->
            try {
                writer.println(response)
                writer.println()  // 空行表示结束
                writer.flush()
            } catch (Exception e) {
                println "[REMOTE] Error sending response: ${e.message}"
            }
        }

        def close = {
            reader?.close()
            writer?.close()
            socket?.close()
        }

        def isConnected = {
            socket && !socket.closed
        }
    }

    static void demonstrateRemoteDebugging() {
        // 创建远程调试配置
        def config = [
            port: 5005,
            maxMemory: 512,
            minMemory: 256,
            suspend: false
        ]

        println "=== Remote Debugging Configuration ==="
        println "Debug arguments: ${RemoteDebugConfig.createDebugArguments(config.port, config.suspend)}"
        println "JVM arguments: ${RemoteDebugConfig.createJvmArguments(config)}"

        // 模拟远程调试服务器（实际使用时需要在单独的JVM中运行）
        println "\n=== Starting Remote Debug Server ==="
        def server = new RemoteDebugServer(config.port)

        // 在实际应用中，服务器应该在后台运行
        // 这里只是演示配置

        println "Remote debugging server configured for port ${config.port}"
        println "To enable remote debugging, add these JVM arguments:"
        println "  ${RemoteDebugConfig.createDebugArguments(config.port, config.suspend)}"

        // 清理
        server.stop()
    }
}
```

### 远程监控工具

```groovy
class RemoteMonitoringTools {

    // 1. JMX监控
    static class JMXMonitor {
        static def getMBeanServer = {
            ManagementFactory.platformMBeanServer
        }

        static def registerMBean = { Object mbean, String name ->
            def mbeanServer = getMBeanServer()
            def objectName = new ObjectName(name)
            mbeanServer.registerMBean(mbean, objectName)
            objectName
        }

        static def getMBeanAttribute = { String objectName, String attribute ->
            def mbeanServer = getMBeanServer()
            def name = new ObjectName(objectName)
            mbeanServer.getAttribute(name, attribute)
        }

        static def invokeMBeanOperation = { String objectName, String operation, Object[] params = [], String[] signature = [] ->
            def mbeanServer = getMBeanServer()
            def name = new ObjectName(objectName)
            mbeanServer.invoke(name, operation, params, signature)
        }

        static def getMemoryUsage = {
            def memory = getMBeanAttribute("java.lang:type=Memory", "HeapMemoryUsage") as CompositeData
            [
                used: memory.get("used"),
                max: memory.get("max"),
                committed: memory.get("committed"),
                init: memory.get("init")
            ]
        }

        static def getThreadInfo = {
            def threadCount = getMBeanAttribute("java.lang:type=Threading", "ThreadCount")
            def peakThreadCount = getMBeanAttribute("java.lang:type=Threading", "PeakThreadCount")
            def daemonThreadCount = getMBeanAttribute("java.lang:type=Threading", "DaemonThreadCount")

            [
                threadCount: threadCount,
                peakThreadCount: peakThreadCount,
                daemonThreadCount: daemonThreadCount
            ]
        }

        static def getOperatingSystemInfo = {
            def os = getMBeanAttribute("java.lang:type=OperatingSystem", "AvailableProcessors")
            def systemLoad = getMBeanAttribute("java.lang:type=OperatingSystem", "SystemLoadAverage")

            [
                availableProcessors: os,
                systemLoadAverage: systemLoad
            ]
        }
    }

    // 2. 自定义监控MBean
    static interface ApplicationMonitorMBean {
        int getActiveSessions()
        double getAverageResponseTime()
        int getTotalRequests()
        void resetStatistics()
        Map<String, Object> getApplicationMetrics()
    }

    static class ApplicationMonitor implements ApplicationMonitorMBean {
        private int activeSessions = 0
        private int totalRequests = 0
        private long totalResponseTime = 0
        private final Map<String, Integer> endpointCounts = [:]

        synchronized void incrementSession() {
            activeSessions++
        }

        synchronized void decrementSession() {
            activeSessions = Math.max(0, activeSessions - 1)
        }

        synchronized void recordRequest(String endpoint, long responseTime) {
            totalRequests++
            totalResponseTime += responseTime
            endpointCounts[endpoint] = (endpointCounts[endpoint] ?: 0) + 1
        }

        @Override
        int getActiveSessions() {
            activeSessions
        }

        @Override
        double getAverageResponseTime() {
            totalRequests > 0 ? (double) totalResponseTime / totalRequests : 0.0
        }

        @Override
        int getTotalRequests() {
            totalRequests
        }

        @Override
        void resetStatistics() {
            totalRequests = 0
            totalResponseTime = 0
            endpointCounts.clear()
        }

        @Override
        Map<String, Object> getApplicationMetrics() {
            [
                activeSessions: activeSessions,
                totalRequests: totalRequests,
                averageResponseTime: getAverageResponseTime(),
                endpointCounts: new HashMap(endpointCounts)
            ]
        }
    }

    // 3. 远程健康检查
    static class HealthChecker {
        private static final Map<String, Closure> healthChecks = [:]

        static def registerHealthCheck = { String name, Closure check ->
            healthChecks[name] = check
        }

        static def performHealthCheck = { String name ->
            def check = healthChecks[name]
            if (!check) {
                throw new IllegalArgumentException("Health check '$name' not found")
            }

            try {
                def result = check()
                [name: name, status: "healthy", result: result, timestamp: new Date()]
            } catch (Exception e) {
                [name: name, status: "unhealthy", error: e.message, timestamp: new Date()]
            }
        }

        static def performAllHealthChecks = {
            healthChecks.collectMany { name, check ->
                [performHealthCheck(name)]
            }
        }

        static def getHealthSummary = {
            def results = performAllHealthChecks()
            def healthyCount = results.count { it.status == "healthy" }
            def totalCount = results.size()

            [
                status: healthyCount == totalCount ? "healthy" : "degraded",
                healthy: healthyCount,
                total: totalCount,
                checks: results
            ]
        }
    }

    // 4. 性能指标收集
    static class MetricsCollector {
        private static final Map<String, List<Long>> metrics = [:]
        private static final Map<String, Long> counters = [:]

        static def recordMetric = { String metricName, long value ->
            if (!metrics.containsKey(metricName)) {
                metrics[metricName] = []
            }
            metrics[metricName] << value
        }

        static def incrementCounter = { String counterName, long increment = 1 ->
            counters[counterName] = (counters[counterName] ?: 0) + increment
        }

        static def getMetricStats = { String metricName ->
            def values = metrics[metricName] ?: []
            if (values.isEmpty()) {
                return [count: 0, average: 0, min: 0, max: 0]
            }

            [
                count: values.size(),
                average: values.sum() / values.size(),
                min: values.min(),
                max: values.max(),
                percentile95: calculatePercentile(values, 95)
            ]
        }

        static def getCounter = { String counterName ->
            counters[counterName] ?: 0
        }

        static def calculatePercentile = { List<Long> values, double percentile ->
            if (values.isEmpty()) return 0

            def sorted = values.sort()
            def index = (percentile / 100) * (sorted.size() - 1)
            def lowerIndex = (int) index
            def upperIndex = Math.min(lowerIndex + 1, sorted.size() - 1)
            def weight = index - lowerIndex

            sorted[lowerIndex] * (1 - weight) + sorted[upperIndex] * weight
        }

        static def getAllMetrics = {
            def result = [:]
            metrics.each { name, values ->
                result[name] = getMetricStats(name)
            }
            result
        }

        static def getAllCounters = {
            new HashMap(counters)
        }

        static def reset = {
            metrics.clear()
            counters.clear()
        }
    }

    static void demonstrateRemoteMonitoring() {
        // JMX监控
        println "=== JMX Monitoring ==="
        def memoryUsage = JMXMonitor.getMemoryUsage()
        println "Memory Usage: ${memoryUsage.used / (1024*1024)}MB / ${memoryUsage.max / (1024*1024)}MB"

        def threadInfo = JMXMonitor.getThreadInfo()
        println "Threads: ${threadInfo.threadCount} (Peak: ${threadInfo.peakThreadCount})"

        def osInfo = JMXMonitor.getOperatingSystemInfo()
        println "CPU: ${osInfo.systemLoadAverage} load average on ${osInfo.availableProcessors} cores"

        // 自定义监控MBean
        println "\n=== Custom Monitoring MBean ==="
        def monitor = new ApplicationMonitor()
        def objectName = JMXMonitor.registerMBean(monitor, "com.example:type=ApplicationMonitor")
        println "MBean registered: $objectName"

        monitor.incrementSession()
        monitor.recordRequest("/api/users", 150)
        monitor.recordRequest("/api/products", 200)

        def metrics = monitor.applicationMetrics
        println "Application Metrics: $metrics"

        // 健康检查
        println "\n=== Health Checks ==="
        HealthChecker.registerHealthCheck("database") {
            // 模拟数据库连接检查
            Math.random() > 0.1 ? "connected" : "disconnected"
        }

        HealthChecker.registerHealthCheck("memory") {
            def memory = JMXMonitor.getMemoryUsage()
            def usagePercent = (memory.used * 100 / memory.max)
            usagePercent < 90 ? "ok" : "high"
        }

        def healthSummary = HealthChecker.getHealthSummary()
        println "Health Status: ${healthSummary.status}"
        println "Healthy checks: ${healthSummary.healthy}/${healthSummary.total}"

        // 性能指标收集
        println "\n=== Metrics Collection ==="
        50.times { i ->
            MetricsCollector.recordMetric("response.time", (long)(Math.random() * 1000))
            MetricsCollector.incrementCounter("requests.processed")
        }

        def responseStats = MetricsCollector.getMetricStats("response.time")
        println "Response Time Stats:"
        println "  Count: ${responseStats.count}"
        println "  Average: ${responseStats.average.round(2)}ms"
        println "  95th percentile: ${responseStats.percentile95.round(2)}ms"

        def requestCount = MetricsCollector.getCounter("requests.processed")
        println "Requests processed: $requestCount"
    }
}
```

## 日志与追踪

### 结构化日志

```groovy
class StructuredLogging {

    // 1. 结构化日志记录器
    static class StructuredLogger {
        private final String loggerName
        private final LogLevel level
        private final List<Appender> appenders = []

        StructuredLogger(String name, LogLevel level = LogLevel.INFO) {
            this.loggerName = name
            this.level = level
        }

        def addAppender = { Appender appender ->
            appenders.add(appender)
            this
        }

        def log = { LogLevel messageLevel, String message, Map data = [:], Throwable throwable = null ->
            if (messageLevel.ordinal() <= level.ordinal()) {
                def logEntry = new LogEntry(
                    timestamp: new Date(),
                    level: messageLevel,
                    logger: loggerName,
                    message: message,
                    data: data,
                    thread: Thread.currentThread().name,
                    throwable: throwable
                )

                appenders.each { appender.append(logEntry) }
            }
        }

        def debug = { String message, Map data = [:] -> log(LogLevel.DEBUG, message, data) }
        def info = { String message, Map data = [:] -> log(LogLevel.INFO, message, data) }
        def warn = { String message, Map data = [:] -> log(LogLevel.WARN, message, data) }
        def error = { String message, Map data = [:], Throwable throwable = null ->
            log(LogLevel.ERROR, message, data, throwable)
        }
    }

    // 2. 日志级别枚举
    static enum LogLevel {
        DEBUG(0), INFO(1), WARN(2), ERROR(3)

        final int level

        LogLevel(int level) {
            this.level = level
        }

        int ordinal() { level }
    }

    // 3. 日志条目
    static class LogEntry {
        Date timestamp
        LogLevel level
        String logger
        String message
        Map data
        String thread
        Throwable throwable

        String toJson() {
            def json = [
                timestamp: timestamp.format("yyyy-MM-dd'T'HH:mm:ss.SSS'Z'"),
                level: level.name(),
                logger: logger,
                message: message,
                thread: thread,
                data: data
            ]

            if (throwable) {
                json.exception = [
                    class: throwable.class.name,
                    message: throwable.message,
                    stackTrace: throwable.stackTrace.collect { it.toString() }
                ]
            }

            new groovy.json.JsonBuilder(json).toString()
        }

        String toText() {
            def text = "${timestamp.format('yyyy-MM-dd HH:mm:ss.SSS')} [${thread}] ${level.name()} $logger - $message"
            if (!data.isEmpty()) {
                text += " ${data}"
            }
            if (throwable) {
                text += "\n${throwable.class.name}: ${throwable.message}"
                throwable.stackTrace.each { text += "\n    at $it" }
            }
            text
        }
    }

    // 4. 日志追加器
    static interface Appender {
        void append(LogEntry entry)
    }

    static class ConsoleAppender implements Appender {
        private final boolean jsonFormat

        ConsoleAppender(boolean jsonFormat = false) {
            this.jsonFormat = jsonFormat
        }

        void append(LogEntry entry) {
            println jsonFormat ? entry.toJson() : entry.toText()
        }
    }

    static class FileAppender implements Appender {
        private final File file
        private final boolean jsonFormat

        FileAppender(String filename, boolean jsonFormat = false) {
            this.file = new File(filename)
            this.jsonFormat = jsonFormat
            this.file.parentFile.mkdirs()
        }

        void append(LogEntry entry) {
            file.append(jsonFormat ? entry.toJson() + "\n" : entry.toText() + "\n")
        }
    }

    static class RollingFileAppender implements Appender {
        private final String baseFilename
        private final long maxSize
        private final int maxFiles
        private File currentFile
        private long currentSize

        RollingFileAppender(String baseFilename, long maxSize = 10 * 1024 * 1024, int maxFiles = 5) {
            this.baseFilename = baseFilename
            this.maxSize = maxSize
            this.maxFiles = maxFiles
            this.currentFile = new File(baseFilename)
            this.currentSize = currentFile.exists() ? currentFile.length() : 0
        }

        void append(LogEntry entry) {
            def message = entry.toJson() + "\n"
            def messageSize = message.bytes.length

            if (currentSize + messageSize > maxSize) {
                rollFile()
            }

            currentFile.append(message)
            currentSize += messageSize
        }

        private void rollFile() {
            // 删除最老的文件
            def oldestFile = new File("${baseFilename}.${maxFiles}")
            if (oldestFile.exists()) {
                oldestFile.delete()
            }

            // 重命名文件
            for (int i = maxFiles - 1; i >= 1; i--) {
                def oldFile = new File("${baseFilename}.${i}")
                def newFile = new File("${baseFilename}.${i + 1}")
                if (oldFile.exists()) {
                    oldFile.renameTo(newFile)
                }
            }

            // 重命名当前文件
            def backupFile = new File("${baseFilename}.1")
            currentFile.renameTo(backupFile)

            // 创建新文件
            currentFile = new File(baseFilename)
            currentSize = 0
        }
    }

    // 5. 日志工厂
    static class LoggerFactory {
        private static final Map<String, StructuredLogger> loggers = [:]

        static synchronized StructuredLogger getLogger(String name) {
            if (!loggers.containsKey(name)) {
                def logger = new StructuredLogger(name)

                // 默认添加控制台追加器
                logger.addAppender(new ConsoleAppender())

                loggers[name] = logger
            }
            loggers[name]
        }

        static synchronized StructuredLogger getLogger(String name, LogLevel level) {
            if (!loggers.containsKey(name)) {
                def logger = new StructuredLogger(name, level)
                logger.addAppender(new ConsoleAppender())
                loggers[name] = logger
            }
            loggers[name]
        }

        static void configure = { Map config ->
            config.each { loggerName, loggerConfig ->
                def logger = getLogger(loggerName)

                // 清除现有追加器
                logger.appenders.clear()

                // 添加配置的追加器
                loggerConfig.appenders?.each { appenderConfig ->
                    def appender = createAppender(appenderConfig)
                    logger.addAppender(appender)
                }
            }
        }

        private static Appender createAppender(Map config) {
            def type = config.type
            switch (type) {
                case 'console':
                    return new ConsoleAppender(config.jsonFormat ?: false)
                case 'file':
                    return new FileAppender(config.filename, config.jsonFormat ?: false)
                case 'rolling':
                    return new RollingFileAppender(
                        config.filename,
                        config.maxSize ?: 10 * 1024 * 1024,
                        config.maxFiles ?: 5
                    )
                default:
                    throw new IllegalArgumentException("Unknown appender type: $type")
            }
        }
    }

    static void demonstrateStructuredLogging() {
        // 创建日志记录器
        def logger = LoggerFactory.getLogger("MyApp", LogLevel.DEBUG)

        // 添加文件追加器
        logger.addAppender(new FileAppender("application.log"))
        logger.addAppender(new RollingFileAppender("application-rolling.log"))

        // 记录不同级别的日志
        logger.debug("Application starting", [version: "1.0.0", environment: "development"])
        logger.info("User logged in", [userId: "123", username: "john.doe"])
        logger.warn("Cache miss ratio high", [cacheName: "userCache", missRatio: 0.25])

        try {
            throw new RuntimeException("Simulated error")
        } catch (Exception e) {
            logger.error("Processing failed", [requestId: "abc123"], e)
        }

        // 结构化数据日志
        def complexData = [
            transaction: [
                id: "txn_789",
                amount: 150.00,
                currency: "USD"
            ],
            user: [
                id: "123",
                tier: "premium"
            ],
            metadata: [
                source: "mobile",
                version: "2.1.0"
            ]
        ]

        logger.info("Transaction processed", complexData)

        println "Structured logging demonstration completed"
        println "Check application.log and application-rolling.log for output"
    }
}
```

### 分布式追踪

```groovy
class DistributedTracing {

    // 1. 追踪上下文
    static class TraceContext {
        private final String traceId
        private final String spanId
        private final String parentSpanId
        private final Map<String, String> baggage

        TraceContext(String traceId, String spanId, String parentSpanId = null, Map<String, String> baggage = [:]) {
            this.traceId = traceId
            this.spanId = spanId
            this.parentSpanId = parentSpanId
            this.baggage = new HashMap(baggage)
        }

        static TraceContext generateRoot() {
            def traceId = UUID.randomUUID().toString().replace('-', '')
            def spanId = UUID.randomUUID().toString().replace('-', '').substring(0, 16)
            new TraceContext(traceId, spanId)
        }

        TraceContext createChild() {
            def childSpanId = UUID.randomUUID().toString().replace('-', '').substring(0, 16)
            new TraceContext(traceId, childSpanId, spanId, baggage)
        }

        Map<String, String> toHeaders() {
            [
                "X-Trace-Id": traceId,
                "X-Span-Id": spanId,
                "X-Parent-Span-Id": parentSpanId
            ] + baggage.collectEntries { key, value -> ["X-Baggage-$key": value] }.flatten()
        }

        static TraceContext fromHeaders(Map<String, String> headers) {
            def traceId = headers["X-Trace-Id"]
            def spanId = headers["X-Span-Id"]
            def parentSpanId = headers["X-Parent-Span-Id"]

            def baggage = headers.findAll { key, value -> key.startsWith("X-Baggage-") }
                .collectEntries { key, value -> [key.replace("X-Baggage-", ""), value] }

            new TraceContext(traceId, spanId, parentSpanId, baggage)
        }

        String toString() {
            "TraceContext(traceId=$traceId, spanId=$spanId, parentSpanId=$parentSpanId)"
        }
    }

    // 2. 追踪Span
    static class Span {
        final TraceContext context
        final String operationName
        final long startTime
        long endTime
        final Map<String, Object> tags = [:]
        final List<SpanLog> logs = []
        final List<Span> children = []

        Span(TraceContext context, String operationName) {
            this.context = context
            this.operationName = operationName
            this.startTime = System.currentTimeMillis()
        }

        def setTag = { String key, Object value ->
            tags[key] = value
            this
        }

        def log = { String message, Map data = [:] ->
            logs.add(new SpanLog(System.currentTimeMillis(), message, data))
            this
        }

        def finish = {
            endTime = System.currentTimeMillis()
        }

        def duration = {
            endTime > 0 ? endTime - startTime : -1
        }

        def addChild = { Span child ->
            children.add(child)
            this
        }

        Map<String, Object> toMap() {
            [
                traceId: context.traceId,
                spanId: context.spanId,
                parentSpanId: context.parentSpanId,
                operationName: operationName,
                startTime: startTime,
                duration: duration(),
                tags: tags,
                logs: logs.collect { it.toMap() },
                children: children.collect { it.toMap() }
            ]
        }
    }

    // 3. Span日志
    static class SpanLog {
        final long timestamp
        final String message
        final Map<String, Object> data

        SpanLog(long timestamp, String message, Map<String, Object> data) {
            this.timestamp = timestamp
            this.message = message
            this.data = new HashMap(data)
        }

        Map<String, Object> toMap() {
            [
                timestamp: timestamp,
                message: message,
                data: data
            ]
        }
    }

    // 4. 追踪器
    static class Tracer {
        private static final ThreadLocal<TraceContext> currentContext = new ThreadLocal<>()
        private static final List<SpanReporter> reporters = []

        static def addReporter = { SpanReporter reporter ->
            reporters.add(reporter)
        }

        static def startSpan = { String operationName ->
            def context = currentContext.get() ?: TraceContext.generateRoot()
            def span = new Span(context, operationName)
            span
        }

        static def startChildSpan = { String operationName, TraceContext parentContext ->
            def childContext = parentContext.createChild()
            def span = new Span(childContext, operationName)
            span
        }

        static def withSpan = { String operationName, Closure operation ->
            def span = startSpan(operationName)
            try {
                currentContext.set(span.context)
                def result = operation()
                span.finish()
                result
            } catch (Exception e) {
                span.setTag("error", true)
                span.log("Exception", [exception: e.message])
                span.finish()
                throw e
            } finally {
                currentContext.set(null)
                reportSpan(span)
            }
        }

        static def withChildSpan = { String operationName, Closure operation ->
            def parentContext = currentContext.get()
            if (!parentContext) {
                return withSpan(operationName, operation)
            }

            def span = startChildSpan(operationName, parentContext)
            try {
                currentContext.set(span.context)
                def result = operation()
                span.finish()
                result
            } catch (Exception e) {
                span.setTag("error", true)
                span.log("Exception", [exception: e.message])
                span.finish()
                throw e
            } finally {
                currentContext.set(parentContext)
                reportSpan(span)
            }
        }

        static def reportSpan = { Span span ->
            reporters.each { reporter ->
                try {
                    reporter.report(span)
                } catch (Exception e) {
                    println "Error reporting span: ${e.message}"
                }
            }
        }

        static def getCurrentContext = {
            currentContext.get()
        }
    }

    // 5. Span报告器
    static interface SpanReporter {
        void report(Span span)
    }

    static class ConsoleSpanReporter implements SpanReporter {
        void report(Span span) {
            def duration = span.duration()
            def status = span.tags.error ? "ERROR" : "OK"
            println "[TRACE] ${span.operationName} - ${duration}ms - $status"

            if (!span.tags.isEmpty()) {
                println "  Tags: $span.tags"
            }

            if (!span.logs.isEmpty()) {
                println "  Logs:"
                span.logs.each { log ->
                    println "    ${new Date(log.timestamp)}: ${log.message}"
                }
            }
        }
    }

    static class JsonFileSpanReporter implements SpanReporter {
        private final File file

        JsonFileSpanReporter(String filename) {
            this.file = new File(filename)
            this.file.parentFile.mkdirs()
        }

        void report(Span span) {
            def json = new groovy.json.JsonBuilder(span.toMap()).toString()
            file.append(json + "\n")
        }
    }

    // 6. HTTP追踪集成
    static class HttpTracing {
        static def injectHeaders = { Map<String, String> headers, TraceContext context ->
            headers.putAll(context.toHeaders())
        }

        static def extractContext = { Map<String, String> headers ->
            TraceContext.fromHeaders(headers)
        }

        static def traceHttpRequest = { String url, Closure request, TraceContext context = null ->
            Tracer.withChildSpan("HTTP $url") {
                def span = Tracer.startSpan("HTTP $url")
                span.setTag("http.url", url)
                span.setTag("http.method", "GET")

                try {
                    def result = request()
                    span.setTag("http.status_code", 200)
                    span.finish()
                    result
                } catch (Exception e) {
                    span.setTag("http.status_code", 500)
                    span.setTag("error", true)
                    span.log("HTTP request failed", [exception: e.message])
                    span.finish()
                    throw e
                }
            }
        }
    }

    static void demonstrateDistributedTracing() {
        // 添加报告器
        Tracer.addReporter(new ConsoleSpanReporter())
        Tracer.addReporter(new JsonFileSpanReporter("traces.json"))

        // 基本追踪
        println "=== Basic Tracing ==="
        Tracer.withSpan("mainOperation") {
            println "Doing main work"
            Thread.sleep(100)

            Tracer.withChildSpan("subOperation") {
                println "Doing sub work"
                Thread.sleep(50)
            }

            Tracer.withChildSpan("anotherSubOperation") {
                println "Doing another sub work"
                Thread.sleep(75)
            }
        }

        // 带标签和日志的追踪
        println "\n=== Tracing with Tags and Logs ==="
        Tracer.withSpan("processData") { span ->
            span.setTag("data.size", 1000)
            span.setTag("data.type", "JSON")

            span.log("Processing started", [recordCount: 1000])

            Thread.sleep(200)

            span.log("Processing completed", [success: true, processedCount: 1000])
        }

        // HTTP追踪
        println "\n=== HTTP Tracing ==="
        def mockHttpRequest = {
            println "Making HTTP request"
            Thread.sleep(150)
            "HTTP Response"
        }

        def result = HttpTracing.traceHttpRequest("https://api.example.com/data", mockHttpRequest)
        println "HTTP result: $result"

        // 错误追踪
        println "\n=== Error Tracing ==="
        try {
            Tracer.withSpan("riskyOperation") {
                println "Doing risky work"
                Thread.sleep(100)
                throw new RuntimeException("Something went wrong")
            }
        } catch (Exception e) {
            println "Caught expected exception: ${e.message}"
        }

        println "Distributed tracing demonstration completed"
        println "Check traces.json for detailed trace data"
    }
}
```

## 总结

Groovy高级调试与监控是构建稳定、高效应用程序的关键。通过本章节的学习，我们深入了解了：

### 核心调试技术总结

1. **基础调试**：断点调试、调试输出、条件调试、异常调试
2. **高级调试**：动态代码注入、调试元编程、方法拦截
3. **性能监控**：执行时间监控、内存使用监控、对象创建监控
4. **线程调试**：线程状态监控、死锁检测、并发问题调试
5. **远程调试**：远程调试设置、JMX监控、健康检查
6. **日志追踪**：结构化日志、分布式追踪、监控工具集成

### 关键技术要点

1. **调试工具**：Groovy Console、断言、调试包装器、状态检查器
2. **监控策略**：性能阈值监控、内存泄漏检测、线程池监控
3. **远程工具**：JMX MBean、远程健康检查、性能指标收集
4. **日志系统**：结构化日志、多种追加器、日志工厂
5. **追踪系统**：追踪上下文、Span管理、分布式追踪

### 实际应用价值

1. **问题定位**：快速定位和解决生产环境中的问题
2. **性能优化**：识别性能瓶颈并进行针对性优化
3. **系统监控**：实时监控系统状态和性能指标
4. **调试效率**：提高调试效率，减少调试时间

通过掌握这些高级调试和监控技术，开发者可以构建更加稳定、高效的Groovy应用程序，并能够在复杂的分布式环境中快速定位和解决问题。这些技术不仅适用于开发环境，更重要的是在生产环境中发挥关键作用，确保应用程序的稳定运行和性能优化。