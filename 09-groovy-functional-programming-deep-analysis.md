# Groovy函数式编程深度解析

## 目录

1. [引言](#引言)
2. [函数式编程基础](#函数式编程基础)
3. [Groovy闭包详解](#groovy闭包详解)
4. [函数式数据结构](#函数式数据结构)
5. [高阶函数与组合](#高阶函数与组合)
6. [函数式设计模式](#函数式设计模式)
7. [函数式并发编程](#函数式并发编程)
8. [函数式性能优化](#函数式性能优化)
9. [实战案例](#实战案例)
10. [最佳实践](#最佳实践)
11. [总结](#总结)

## 引言

函数式编程是一种编程范式，强调将计算视为数学函数的求值，避免状态变化和可变数据。Groovy作为JVM上的动态语言，提供了丰富的函数式编程特性，包括闭包、高阶函数、函数组合等。本章节将深入探讨Groovy中的函数式编程概念、技术和最佳实践。

### 函数式编程的优势

1. **代码简洁性**：使用高阶函数和闭包减少样板代码
2. **可测试性**：纯函数更容易进行单元测试
3. **并发安全**：不可变数据结构减少并发问题
4. **组合性**：函数组合提供强大的代码复用能力
5. **声明式编程**：关注"做什么"而不是"怎么做"

## 函数式编程基础

### 纯函数概念

```groovy
class FunctionalProgrammingBasics {

    // 1. 纯函数示例
    static int add(int a, int b) {
        a + b  // 纯函数：相同的输入总是产生相同的输出
    }

    static List<Integer> doubleList(List<Integer> numbers) {
        numbers.collect { it * 2 }  // 纯函数：不修改输入参数
    }

    // 2. 非纯函数示例（对比）
    static int nonPureAdd(int a, int b) {
        def result = a + b
        println "Result: $result"  // 副作用：打印到控制台
        result
    }

    static List<Integer> nonPureFilter(List<Integer> numbers) {
        numbers.removeAll { it <= 0 }  // 副作用：修改输入参数
        numbers
    }

    // 3. 函数的幂等性
    static class IdempotentFunctions {
        static List<Integer> sortList(List<Integer> numbers) {
            new ArrayList(numbers).sort()  // 幂等函数：多次调用结果相同
        }

        static Map<String, Integer> countWords(String text) {
            def words = text.split('\\s+')
            def counts = [:]
            words.each { word ->
                counts[word] = (counts[word] ?: 0) + 1
            }
            counts  // 幂等函数：相同输入产生相同输出
        }
    }
}
```

### 不可变性原则

```groovy
class Immutability {

    // 1. 不可变集合
    static void demonstrateImmutableCollections() {
        def immutableList = [1, 2, 3].asImmutable()
        def immutableSet = [1, 2, 3].asImmutable()
        def immutableMap = [a: 1, b: 2].asImmutable()

        // 尝试修改会抛出异常
        try {
            immutableList.add(4)
        } catch (UnsupportedOperationException e) {
            println "Cannot modify immutable list: ${e.message}"
        }

        // 创建新集合而不是修改原集合
        def newList = immutableList + [4, 5]
        def newMap = immutableMap + [c: 3]
    }

    // 2. 不可变对象
    @groovy.transform.Immutable
    static class ImmutablePerson {
        String name
        int age
        List<String> hobbies
    }

    static void demonstrateImmutableObjects() {
        def person = new ImmutablePerson(name: "John", age: 30, hobbies: ["Reading", "Music"])

        // 尝试修改会创建新对象
        def updatedPerson = person.withAge(31)
        println "Original: $person"
        println "Updated: $updatedPerson"
    }

    // 3. 不可变构建器
    static class ImmutableBuilder {
        static class PersonBuilder {
            private String name
            private int age = 0
            private List<String> hobbies = []

            PersonBuilder withName(String name) {
                this.name = name
                this
            }

            PersonBuilder withAge(int age) {
                this.age = age
                this
            }

            PersonBuilder withHobby(String hobby) {
                this.hobbies = new ArrayList(this.hobbies)
                this.hobbies.add(hobby)
                this
            }

            ImmutablePerson build() {
                new ImmutablePerson(name: name, age: age, hobbies: hobbies.asImmutable())
            }
        }

        static PersonBuilder person() {
            new PersonBuilder()
        }
    }
}
```

### 函数作为一等公民

```groovy
class FirstClassFunctions {

    // 1. 函数作为参数
    static List<Integer> processNumbers(List<Integer> numbers, Closure<Integer> processor) {
        numbers.collect(processor)
    }

    static void demonstrateFunctionAsParameter() {
        def numbers = [1, 2, 3, 4, 5]

        def doubled = processNumbers(numbers) { it * 2 }
        def squared = processNumbers(numbers) { it ** 2 }
        def absolute = processNumbers(numbers) { Math.abs(it) }

        println "Original: $numbers"
        println "Doubled: $doubled"
        println "Squared: $squared"
        println "Absolute: $absolute"
    }

    // 2. 函数作为返回值
    static Closure<Integer> createMultiplier(int factor) {
        return { int value -> value * factor }
    }

    static Closure<String> createFormatter(String prefix, String suffix) {
        return { String input -> "$prefix$input$suffix" }
    }

    static void demonstrateFunctionAsReturn() {
        def doubler = createMultiplier(2)
        def tripler = createMultiplier(3)

        def boldFormatter = createFormatter("**", "**")
        def quoteFormatter = createFormatter('"', '"')

        println "5 * 2 = ${doubler(5)}"
        println "5 * 3 = ${tripler(5)}"
        println "Bold: ${boldFormatter('Hello')}"
        println "Quote: ${quoteFormatter('World')}"
    }

    // 3. 函数组合
    static Closure compose(Closure f, Closure g) {
        return { x -> f(g(x)) }
    }

    static void demonstrateFunctionComposition() {
        def addOne = { x -> x + 1 }
        def multiplyByTwo = { x -> x * 2 }
        def toString = { x -> x.toString() }

        // 组合函数
        def addOneThenMultiply = compose(multiplyByTwo, addOne)
        def addOneThenToString = compose(toString, addOne)

        println "Add one then multiply: ${addOneThenMultiply(5)}"  // (5 + 1) * 2 = 12
        println "Add one then to string: ${addOneThenToString(5)}"  // "6"
    }
}
```

## Groovy闭包详解

### 闭包基础

```groovy
class ClosureBasics {

    // 1. 闭包定义和调用
    static void demonstrateBasicClosure() {
        // 简单闭包
        def simpleClosure = { println "Hello, World!" }
        simpleClosure()

        // 带参数的闭包
        def greeting = { name -> println "Hello, $name!" }
        greeting("Alice")
        greeting.call("Bob")  // 显式调用

        // 带默认参数的闭包
        def power = { base, exponent = 2 -> base ** exponent }
        println "3^2 = ${power(3)}"
        println "3^3 = ${power(3, 3)}"
    }

    // 2. 隐式参数
    static void demonstrateImplicitParameters() {
        // it作为隐式参数
        def doubler = { it * 2 }
        println "5 doubled: ${doubler(5)}"

        // 集合操作中的隐式参数
        def numbers = [1, 2, 3, 4, 5]
        def doubledNumbers = numbers.collect { it * 2 }
        println "Original: $numbers"
        println "Doubled: $doubledNumbers"

        // 多个参数的闭包
        def sum = { a, b -> a + b }
        println "2 + 3 = ${sum(2, 3)}"
    }

    // 3. 闭包委托
    static class ClosureDelegate {
        String name
        int age

        void process(Closure closure) {
            closure.delegate = this
            closure.call()
        }
    }

    static void demonstrateClosureDelegate() {
        def delegate = new ClosureDelegate(name: "John", age: 30)

        delegate.process {
            println "Name: $name"  // 访问委托对象的属性
            println "Age: $age"
            println "Age in 10 years: ${age + 10}"
        }
    }
}
```

### 闭包高级特性

```groovy
class AdvancedClosureFeatures {

    // 1. 闭包柯里化
    static void demonstrateCurrying() {
        def add = { a, b -> a + b }

        // 柯里化
        def addFive = add.curry(5)
        def addTen = add.curry(10)

        println "5 + 3 = ${addFive(3)}"
        println "10 + 7 = ${addTen(7)}"

        // 多参数柯里化
        def multiply = { a, b, c -> a * b * c }
        def multiplyByTwo = multiply.curry(2)
        def multiplyByTwoAndThree = multiply.curry(2, 3)

        println "2 * 3 * 4 = ${multiplyByTwo(3, 4)}"
        println "2 * 3 * 5 = ${multiplyByTwoAndThree(5)}"
    }

    // 2. 闭包记忆化
    static class MemoizedClosures {
        static def expensiveOperation = { n ->
            println "Calculating factorial of $n"
            (1..n).inject(1) { acc, i -> acc * i }
        }.memoize()

        static def fibonacci = { int n ->
            if (n <= 1) return n
            fibonacci(n - 1) + fibonacci(n - 2)
        }.memoize()

        static void demonstrateMemoization() {
            println "First call to factorial(5): ${expensiveOperation(5)}"
            println "Second call to factorial(5): ${expensiveOperation(5)}"  // 不会重新计算

            println "Fibonacci(10): ${fibonacci(10)}"
            println "Fibonacci(10) again: ${fibonacci(10)}"  // 使用缓存结果
        }
    }

    // 3. 闭包组合
    static class ClosureComposition {
        static def compose = { Closure f, Closure g ->
            return { x -> f(g(x)) }
        }

        static def pipe = { Closure f, Closure g ->
            return { x -> g(f(x)) }
        }

        static void demonstrateComposition() {
            def addOne = { x -> x + 1 }
            def multiplyByTwo = { x -> x * 2 }
            def toString = { x -> x.toString() }

            // 函数组合
            def addOneThenMultiply = compose(multiplyByTwo, addOne)
            def addOneThenToString = compose(toString, addOne)

            // 函数管道
            def multiplyThenAddOne = pipe(multiplyByTwo, addOne)

            println "Compose - Add one then multiply: ${addOneThenMultiply(5)}"
            println "Compose - Add one then string: ${addOneThenToString(5)}"
            println "Pipe - Multiply then add one: ${multiplyThenAddOne(5)}"
        }
    }
}
```

### 闭包作用域和上下文

```groovy
class ClosureScopeAndContext {

    // 1. 闭包捕获外部变量
    static void demonstrateVariableCapture() {
        def counter = 0

        def increment = { counter++ }
        def getValue = { counter }

        println "Initial value: ${getValue()}"
        increment()
        println "After increment: ${getValue()}"
        increment()
        increment()
        println "After multiple increments: ${getValue()}"
    }

    // 2. 闭包与对象作用域
    static class ClosureWithObjectScope {
        private String message = "Hello"

        def createGreeter() {
            return { name -> "$message, $name!" }
        }

        def updateMessage(String newMessage) {
            this.message = newMessage
        }
    }

    static void demonstrateObjectScope() {
        def obj = new ClosureWithObjectScope()
        def greeter = obj.createGreeter()

        println greeter("Alice")
        obj.updateMessage("Hi")
        println greeter("Bob")  // 使用更新后的消息
    }

    // 3. 闭包委托策略
    static class ClosureDelegationStrategies {
        static class Owner {
            String name = "Owner"
            def method() { "Owner method" }
        }

        static class Delegate {
            String name = "Delegate"
            def method() { "Delegate method" }
        }

        static class Resolver {
            String name = "Resolver"
            def method() { "Resolver method" }
        }

        static void demonstrateDelegationStrategies() {
            def owner = new Owner()
            def delegate = new Delegate()
            def resolver = new Resolver()

            def closure = {
                println "Name: $name"
                println "Method result: ${method()}"
            }

            // 默认策略：OWNER_FIRST
            closure.delegate = delegate
            closure.resolveStrategy = Closure.OWNER_FIRST
            println "OWNER_FIRST strategy:"
            closure()

            // DELEGATE_FIRST策略
            closure.resolveStrategy = Closure.DELEGATE_FIRST
            println "\nDELEGATE_FIRST strategy:"
            closure()

            // DELEGATE_ONLY策略
            closure.resolveStrategy = Closure.DELEGATE_ONLY
            println "\nDELEGATE_ONLY strategy:"
            closure()
        }
    }
}
```

## 函数式数据结构

### 不可变集合

```groovy
class ImmutableDataStructures {

    // 1. 不可变列表
    static class ImmutableList<T> {
        private final List<T> elements

        ImmutableList(List<T> elements) {
            this.elements = elements.asImmutable()
        }

        ImmutableList<T> add(T element) {
            def newList = new ArrayList(elements)
            newList.add(element)
            new ImmutableList(newList)
        }

        ImmutableList<T> remove(T element) {
            def newList = new ArrayList(elements)
            newList.remove(element)
            new ImmutableList(newList)
        }

        ImmutableList<T> filter(Closure<Boolean> predicate) {
            new ImmutableList(elements.findAll(predicate))
        }

        ImmutableList<T> map(Closure<T> transformer) {
            new ImmutableList(elements.collect(transformer))
        }

        List<T> toList() {
            elements
        }

        String toString() {
            elements.toString()
        }
    }

    // 2. 不可变映射
    static class ImmutableMap<K, V> {
        private final Map<K, V> data

        ImmutableMap(Map<K, V> data) {
            this.data = data.asImmutable()
        }

        ImmutableMap<K, V> put(K key, V value) {
            def newMap = new HashMap(data)
            newMap.put(key, value)
            new ImmutableMap(newMap)
        }

        ImmutableMap<K, V> remove(K key) {
            def newMap = new HashMap(data)
            newMap.remove(key)
            new ImmutableMap(newMap)
        }

        ImmutableMap<K, V> filterKeys(Closure<Boolean> predicate) {
            def newMap = data.findAll { key, value -> predicate(key) }
            new ImmutableMap(newMap)
        }

        ImmutableMap<K, V> mapValues(Closure<V> transformer) {
            def newMap = data.collectEntries { key, value -> [key, transformer(value)] }
            new ImmutableMap(newMap)
        }

        V get(K key) {
            data.get(key)
        }

        Map<K, V> toMap() {
            data
        }

        String toString() {
            data.toString()
        }
    }

    // 3. 函数式集合操作
    static class FunctionalCollectionOperations {
        static <T> List<T> filter(List<T> list, Closure<Boolean> predicate) {
            list.findAll(predicate)
        }

        static <T, R> List<R> map(List<T> list, Closure<R> transformer) {
            list.collect(transformer)
        }

        static <T> T reduce(List<T> list, T initialValue, Closure<T> reducer) {
            list.inject(initialValue, reducer)
        }

        static <T> List<T> distinct(List<T> list) {
            list.unique()
        }

        static <T> List<List<T>> groupBy(List<T> list, Closure<?> classifier) {
            list.groupBy(classifier).values().toList()
        }

        static void demonstrateOperations() {
            def numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

            // 过滤偶数
            def evenNumbers = filter(numbers) { it % 2 == 0 }
            println "Even numbers: $evenNumbers"

            // 计算平方
            def squares = map(numbers) { it ** 2 }
            println "Squares: $squares"

            // 计算总和
            def sum = reduce(numbers, 0) { acc, num -> acc + num }
            println "Sum: $sum"

            // 去重
            def withDuplicates = [1, 2, 2, 3, 3, 3, 4, 5]
            def uniqueNumbers = distinct(withDuplicates)
            println "Unique numbers: $uniqueNumbers"

            // 分组
            def words = ["apple", "banana", "cherry", "date", "elderberry"]
            def byLength = groupBy(words) { it.length() }
            println "Words by length: $byLength"
        }
    }
}
```

### 函数式流处理

```groovy
class FunctionalStreamProcessing {

    // 1. 流式数据源
    static class DataSource {
        static List<Integer> generateNumbers(int count) {
            (1..count).toList()
        }

        static List<String> generateWords() {
            ["functional", "programming", "groovy", "closure", "immutable", "pure"]
        }

        static List<Map<String, Object>> generateUsers() {
            [
                [name: "Alice", age: 25, city: "New York"],
                [name: "Bob", age: 30, city: "London"],
                [name: "Charlie", age: 35, city: "Paris"],
                [name: "Diana", age: 28, city: "Tokyo"]
            ]
        }
    }

    // 2. 流处理操作
    static class StreamOperations {
        static <T> List<T> filter(List<T> source, Closure<Boolean> predicate) {
            source.findAll(predicate)
        }

        static <T, R> List<R> map(List<T> source, Closure<R> transformer) {
            source.collect(transformer)
        }

        static <T> List<T> take(List<T> source, int count) {
            source.take(count)
        }

        static <T> List<T> skip(List<T> source, int count) {
            source.drop(count)
        }

        static <T> List<T> distinct(List<T> source) {
            source.unique()
        }

        static <T> void forEach(List<T> source, Closure<T> consumer) {
            source.each(consumer)
        }

        static <T> Optional<T> findFirst(List<T> source, Closure<Boolean> predicate) {
            def result = source.find(predicate)
            result != null ? Optional.of(result) : Optional.empty()
        }

        static <T> List<T> sorted(List<T> source, Closure<Integer> comparator) {
            source.sort(comparator)
        }
    }

    // 3. 流处理管道
    static class StreamPipeline {
        static <T, R> List<R> createPipeline(List<T> source, List<Closure> operations) {
            def result = source
            operations.each { operation ->
                result = operation.call(result)
            }
            result
        }

        static void demonstratePipeline() {
            def numbers = DataSource.generateNumbers(20)
            def users = DataSource.generateUsers()

            // 数字处理管道
            def numberPipeline = [
                { list -> StreamOperations.filter(list) { it % 2 == 0 } },
                { list -> StreamOperations.map(list) { it ** 2 } },
                { list -> StreamOperations.take(list, 5) },
                { list -> StreamOperations.sorted(list) { a, b -> a <=> b } }
            ]

            def result = createPipeline(numbers, numberPipeline)
            println "Number pipeline result: $result"

            // 用户处理管道
            def userPipeline = [
                { list -> StreamOperations.filter(list) { it.age > 25 } },
                { list -> StreamOperations.map(list) { it.name.toUpperCase() } },
                { list -> StreamOperations.sorted(list) { a, b -> a.length() <=> b.length() } }
            ]

            def userResult = createPipeline(users, userPipeline)
            println "User pipeline result: $userResult"
        }
    }
}
```

### 函数式树结构

```groovy
class FunctionalTreeStructures {

    // 1. 不可变二叉树
    static class ImmutableBinaryTree<T> {
        final T value
        final ImmutableBinaryTree<T> left
        final ImmutableBinaryTree<T> right

        ImmutableBinaryTree(T value, ImmutableBinaryTree<T> left = null, ImmutableBinaryTree<T> right = null) {
            this.value = value
            this.left = left
            this.right = right
        }

        ImmutableBinaryTree<T> insert(T newValue) {
            if (newValue < value) {
                def newLeft = left ? left.insert(newValue) : new ImmutableBinaryTree(newValue)
                new ImmutableBinaryTree(value, newLeft, right)
            } else if (newValue > value) {
                def newRight = right ? right.insert(newValue) : new ImmutableBinaryTree(newValue)
                new ImmutableBinaryTree(value, left, newRight)
            } else {
                this  // 值已存在，返回原树
            }
        }

        boolean contains(T target) {
            if (target == value) {
                return true
            } else if (target < value && left) {
                return left.contains(target)
            } else if (target > value && right) {
                return right.contains(target)
            }
            return false
        }

        List<T> inOrderTraversal() {
            def result = []
            if (left) result.addAll(left.inOrderTraversal())
            result.add(value)
            if (right) result.addAll(right.inOrderTraversal())
            result
        }

        int size() {
            int count = 1
            if (left) count += left.size()
            if (right) count += right.size()
            count
        }

        int depth() {
            int leftDepth = left ? left.depth() : 0
            int rightDepth = right ? right.depth() : 0
            Math.max(leftDepth, rightDepth) + 1
        }
    }

    // 2. 函数式树操作
    static class TreeOperations {
        static <T> ImmutableBinaryTree<T> buildTree(List<T> values) {
            def root = new ImmutableBinaryTree(values[0])
            values[1..-1].each { value ->
                root = root.insert(value)
            }
            root
        }

        static <T> List<T> breadthFirstTraversal(ImmutableBinaryTree<T> tree) {
            if (!tree) return []

            def result = []
            def queue = [tree]

            while (queue) {
                def node = queue.remove(0)
                result.add(node.value)

                if (node.left) queue.add(node.left)
                if (node.right) queue.add(node.right)
            }

            result
        }

        static <T> ImmutableBinaryTree<T> mapTree(ImmutableBinaryTree<T> tree, Closure<T> mapper) {
            if (!tree) return null

            def newValue = mapper(tree.value)
            def newLeft = mapTree(tree.left, mapper)
            def newRight = mapTree(tree.right, mapper)

            new ImmutableBinaryTree(newValue, newLeft, newRight)
        }

        static <T> ImmutableBinaryTree<T> filterTree(ImmutableBinaryTree<T> tree, Closure<Boolean> predicate) {
            if (!tree) return null

            def newLeft = filterTree(tree.left, predicate)
            def newRight = filterTree(tree.right, predicate)

            if (predicate(tree.value)) {
                new ImmutableBinaryTree(tree.value, newLeft, newRight)
            } else if (newLeft && newRight) {
                // 合并子树
                newLeft.insertAll(newRight)
            } else {
                newLeft ?: newRight
            }
        }
    }

    // 3. 树的实际应用
    static class TreeApplications {
        static void demonstrateTreeOperations() {
            def values = [5, 3, 7, 1, 4, 6, 8]
            def tree = TreeOperations.buildTree(values)

            println "Original values: $values"
            println "Tree in-order traversal: ${tree.inOrderTraversal()}"
            println "Tree size: ${tree.size()}"
            println "Tree depth: ${tree.depth()}"

            println "Breadth-first traversal: ${TreeOperations.breadthFirstTraversal(tree)}"
            println "Contains 4: ${tree.contains(4)}"
            println "Contains 9: ${tree.contains(9)}"

            // 映射操作
            def doubledTree = TreeOperations.mapTree(tree) { it * 2 }
            println "Doubled tree traversal: ${doubledTree.inOrderTraversal()}"

            // 过滤操作
            def filteredTree = TreeOperations.filterTree(tree) { it > 4 }
            println "Filtered tree traversal: ${filteredTree?.inOrderTraversal()}"
        }
    }
}
```

## 高阶函数与组合

### 函数组合模式

```groovy
class FunctionComposition {

    // 1. 基础函数组合
    static class BasicComposition {
        static def compose = { Closure f, Closure g ->
            return { x -> f(g(x)) }
        }

        static def pipe = { Closure f, Closure g ->
            return { x -> g(f(x)) }
        }

        static void demonstrateBasicComposition() {
            def addOne = { x -> x + 1 }
            def multiplyByTwo = { x -> x * 2 }
            def toString = { x -> x.toString() }

            // 函数组合
            def addOneThenMultiply = compose(multiplyByTwo, addOne)
            def multiplyThenAddOne = pipe(multiplyByTwo, addOne)

            println "5 -> addOne -> multiplyByTwo: ${addOneThenMultiply(5)}"  // 12
            println "5 -> multiplyByTwo -> addOne: ${multiplyThenAddOne(5)}"  // 11

            // 多重组合
            def complex = compose(toString, multiplyByTwo, addOne)
            println "5 -> complex: ${complex(5)}"  // "12"
        }
    }

    // 2. 高阶函数
    static class HigherOrderFunctions {
        static def applyTwice = { Closure f, x -> f(f(x)) }

        static def repeat = { Closure f, int n ->
            return { x ->
                def result = x
                n.times { result = f(result) }
                result
            }
        }

        static def composeAll = { List<Closure> functions ->
            if (functions.isEmpty()) {
                return { x -> x }
            }
            functions.reverse().inject { acc, f -> compose(f, acc) }
        }

        static void demonstrateHigherOrderFunctions() {
            def increment = { x -> x + 1 }
            def double = { x -> x * 2 }

            println "Apply increment twice: ${applyTwice(increment, 5)}"  // 7
            println "Apply double twice: ${applyTwice(double, 5)}"  // 20

            def incrementThreeTimes = repeat(increment, 3)
            println "Increment 5 three times: ${incrementThreeTimes(5)}"  // 8

            def pipeline = composeAll([increment, double, increment])
            println "Pipeline: ${pipeline(5)}"  // (5 + 1) * 2 + 1 = 13
        }
    }

    // 3. 函数式工具
    static class FunctionalUtils {
        static def identity = { x -> x }

        static def constant = { value -> { x -> value } }

        static def alwaysTrue = { x -> true }
        static def alwaysFalse = { x -> false }

        static def complement = { Closure predicate ->
            return { x -> !predicate(x) }
        }

        static def both = { Closure p1, Closure p2 ->
            return { x -> p1(x) && p2(x) }
        }

        static def either = { Closure p1, Closure p2 ->
            return { x -> p1(x) || p2(x) }
        }

        static void demonstrateFunctionalUtils() {
            def isEven = { x -> x % 2 == 0 }
            def isPositive = { x -> x > 0 }

            def isOdd = complement(isEven)
            def isNonPositive = complement(isPositive)

            def isEvenAndPositive = both(isEven, isPositive)
            def isEvenOrPositive = either(isEven, isPositive)

            println "Is 4 even and positive: ${isEvenAndPositive(4)}"  // true
            println "Is 3 even or positive: ${isEvenOrPositive(3)}"  // true
            println "Is -2 even: ${isEven(-2)}, odd: ${isOdd(-2)}"  // true, false
        }
    }
}
```

### 柯里化与部分应用

```groovy
class CurryingAndPartialApplication {

    // 1. 柯里化函数
    static class CurriedFunctions {
        static def curry = { Closure f, Object... args ->
            return { Object... remainingArgs ->
                f(*args, *remainingArgs)
            }
        }

        static def curryN = { Closure f, int n ->
            if (n == 1) {
                return { x -> f(x) }
            } else {
                return { x ->
                    def remaining = curryN(f, n - 1)
                    return { y -> remaining(x, y) }
                }
            }
        }

        static void demonstrateCurrying() {
            def add = { a, b, c -> a + b + c }

            // 柯里化
            def add5 = curry(add, 5)
            def add5And3 = curry(add, 5, 3)

            println "5 + 2 + 1 = ${add5(2, 1)}"  // 8
            println "5 + 3 + 2 = ${add5And3(2)}"  // 10

            // 多参数柯里化
            def multiply = { a, b, c, d -> a * b * c * d }
            def multiply2 = curryN(multiply, 4)
            def multiply2And3 = multiply2(2)
            def multiply2And3And4 = multiply2And3(3)

            println "2 * 3 * 4 * 5 = ${multiply2And3And4(5)}"  // 120
        }
    }

    // 2. 部分应用
    static class PartialApplication {
        static def partial = { Closure f, Object... args ->
            return { Object... remainingArgs ->
                f(*args, *remainingArgs)
            }
        }

        static def partialRight = { Closure f, Object... args ->
            return { Object... remainingArgs ->
                f(*remainingArgs, *args)
            }
        }

        static void demonstratePartialApplication() {
            def greet = { greeting, name, punctuation -> "$greeting, $name$punctuation" }

            // 部分应用（从左到右）
            def sayHello = partial(greet, "Hello")
            def sayHelloToJohn = partial(greet, "Hello", "John")

            println "Hello to Alice: ${sayHello("Alice", "!")}"  // "Hello, Alice!"
            println "Hello to John: ${sayHelloToJohn(".")}"  // "Hello, John."

            // 部分应用（从右到左）
            def withExclamation = partialRight(greet, "!")
            def greetJohnWithExclamation = partialRight(greet, "John", "!")

            println "Hi to Alice!: ${withExclamation("Hi", "Alice")}"  // "Hi, Alice!"
            println "Hello to John!: ${greetJohnWithExclamation("Hello")}"  // "Hello, John!"
        }
    }

    // 3. 实际应用场景
    static class PracticalApplications {
        static def createValidator = { Map rules ->
            return { Map data ->
                def errors = []
                rules.each { field, validator ->
                    def value = data[field]
                    if (!validator(value)) {
                        errors.add("Invalid $field: $value")
                    }
                }
                errors
            }
        }

        static def createProcessor = { List<Closure> operations ->
            return { input ->
                operations.inject(input) { acc, operation ->
                    operation(acc)
                }
            }
        }

        static void demonstrateApplications() {
            // 验证器
            def userValidator = createValidator([
                name: { it instanceof String && it.length() > 0 },
                age: { it instanceof Integer && it > 0 && it < 150 },
                email: { it =~ /[^@]+@[^@]+\.[^@]+/ }
            ])

            def user1 = [name: "John", age: 30, email: "john@example.com"]
            def user2 = [name: "", age: 200, email: "invalid-email"]

            println "User1 errors: ${userValidator(user1)}"  // []
            println "User2 errors: ${userValidator(user2)}"  // ["Invalid name: ", "Invalid age: 200", "Invalid email: invalid-email"]

            // 数据处理器
            def dataProcessor = createProcessor([
                { data -> data.collect { it.toUpperCase() } },
                { data -> data.findAll { it.length() > 3 } },
                { data -> data.sort() }
            ])

            def words = ["apple", "banana", "cat", "dog", "elephant"]
            println "Processed words: ${dataProcessor(words)}"  // ["APPLE", "BANANA", "ELEPHANT"]
        }
    }
}
```

### 函数式管道

```groovy
class FunctionalPipelines {

    // 1. 管道构建器
    static class PipelineBuilder {
        private List<Closure> operations = []

        def add(Closure operation) {
            operations.add(operation)
            this
        }

        def filter(Closure predicate) {
            operations.add { list -> list.findAll(predicate) }
            this
        }

        def map(Closure transformer) {
            operations.add { list -> list.collect(transformer) }
            this
        }

        def take(int count) {
            operations.add { list -> list.take(count) }
            this
        }

        def skip(int count) {
            operations.add { list -> list.drop(count) }
            this
        }

        def sort(Closure comparator = { a, b -> a <=> b }) {
            operations.add { list -> list.sort(comparator) }
            this
        }

        def distinct() {
            operations.add { list -> list.unique() }
            this
        }

        def build() {
            return { input ->
                operations.inject(input) { result, operation ->
                    operation(result)
                }
            }
        }
    }

    // 2. 管道操作符
    static class PipelineOperators {
        static def pipe = { Object input, Closure... operations ->
            operations.inject(input) { result, operation ->
                operation(result)
            }
        }

        static def compose = { Closure... operations ->
            return { input ->
                operations.reverse().inject(input) { result, operation ->
                    operation(result)
                }
            }
        }

        static void demonstrateOperators() {
            def numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

            // 使用管道操作符
            def result = pipe(numbers,
                { it.findAll { it % 2 == 0 } },
                { it.collect { it ** 2 } },
                { it.take(3) },
                { it.sort() }
            )

            println "Pipe result: $result"

            // 使用组合操作符
            def processor = compose(
                { it.collect { it ** 2 } },
                { it.findAll { it > 10 } },
                { it.take(3) }
            )

            println "Compose result: ${processor([3, 4, 5, 6])}"
        }
    }

    // 3. 高级管道模式
    static class AdvancedPipelines {
        static def parallelPipeline = { List<Closure> operations ->
            return { input ->
                def pool = Executors.newFixedThreadPool(operations.size())
                try {
                    def futures = operations.collect { operation ->
                        pool.submit { operation(input) }
                    }
                    futures.collect { it.get() }
                } finally {
                    pool.shutdown()
                }
            }
        }

        static def conditionalPipeline = { Map<Closure, Closure> branches ->
            return { input ->
                def result = input
                branches.each { condition, operation ->
                    if (condition(result)) {
                        result = operation(result)
                    }
                }
                result
            }
        }

        static def memoizedPipeline = { List<Closure> operations ->
            def memoizedOperations = operations.collect { it.memoize() }
            return { input ->
                memoizedOperations.inject(input) { result, operation ->
                    operation(result)
                }
            }
        }

        static void demonstrateAdvancedPipelines() {
            def numbers = [1, 2, 3, 4, 5]

            // 并行管道
            def parallelProcessor = parallelPipeline([
                { it.collect { it * 2 } },
                { it.collect { it ** 2 } },
                { it.collect { it + 10 } }
            ])

            println "Parallel processing: ${parallelProcessor(numbers)}"

            // 条件管道
            def conditionalProcessor = conditionalPipeline([
                ({ it.size() > 3 }): { it.take(3) },
                ({ it.any { it > 10 } }): { it.findAll { it <= 10 } }
            ])

            println "Conditional processing: ${conditionalProcessor([15, 2, 8, 12, 3])}"

            // 记忆化管道
            def expensiveOperation = { list ->
                println "Processing expensive operation"
                list.collect { it ** 2 }
            }

            def memoizedProcessor = memoizedPipeline([expensiveOperation])
            println "First call: ${memoizedProcessor([1, 2, 3])}"
            println "Second call: ${memoizedProcessor([1, 2, 3])}"  // 不会重新计算
        }
    }
}
```

## 函数式设计模式

### 函数式策略模式

```groovy
class FunctionalStrategyPattern {

    // 1. 策略函数
    static class StrategyFunctions {
        static def add = { a, b -> a + b }
        static def subtract = { a, b -> a - b }
        static def multiply = { a, b -> a * b }
        static def divide = { a, b -> a / b }
        static def power = { a, b -> a ** b }
        static def max = { a, b -> Math.max(a, b) }
        static def min = { a, b -> Math.min(a, b) }
    }

    // 2. 策略上下文
    static class StrategyContext {
        private Closure strategy

        StrategyContext(Closure strategy) {
            this.strategy = strategy
        }

        def execute(Object... args) {
            strategy(*args)
        }

        def setStrategy(Closure strategy) {
            this.strategy = strategy
        }
    }

    // 3. 策略工厂
    static class StrategyFactory {
        static def createStrategy(String operation) {
            switch (operation.toLowerCase()) {
                case "add":
                    return StrategyFunctions.add
                case "subtract":
                    return StrategyFunctions.subtract
                case "multiply":
                    return StrategyFunctions.multiply
                case "divide":
                    return StrategyFunctions.divide
                case "power":
                    return StrategyFunctions.power
                case "max":
                    return StrategyFunctions.max
                case "min":
                    return StrategyFunctions.min
                default:
                    throw new IllegalArgumentException("Unknown operation: $operation")
            }
        }

        static def createComposedStrategy(List<String> operations) {
            def strategies = operations.collect { createStrategy(it) }
            return { a, b ->
                strategies.inject(a) { result, strategy ->
                    strategy(result, b)
                }
            }
        }
    }

    static void demonstrateStrategyPattern() {
        def context = new StrategyContext(StrategyFunctions.add)
        println "5 + 3 = ${context.execute(5, 3)}"

        context.setStrategy(StrategyFunctions.multiply)
        println "5 * 3 = ${context.execute(5, 3)}"

        def strategy = StrategyFactory.createStrategy("power")
        println "5 ^ 3 = ${strategy(5, 3)}"

        def composed = StrategyFactory.createComposedStrategy(["add", "multiply"])
        println "(5 + 3) * 2 = ${composed(5, 3)}"  // 16
    }
}
```

### 函数式责任链模式

```groovy
class FunctionalChainOfResponsibility {

    // 1. 处理器函数
    static class HandlerFunctions {
        static def authenticate = { request ->
            if (request.token) {
                println "Authentication successful"
                request
            } else {
                throw new SecurityException("Authentication failed")
            }
        }

        static def validate = { request ->
            if (request.data && request.data.size() > 0) {
                println "Validation successful"
                request
            } else {
                throw new IllegalArgumentException("Invalid data")
            }
        }

        static def authorize = { request ->
            if (request.role == "admin") {
                println "Authorization successful"
                request
            } else {
                throw new SecurityException("Authorization failed")
            }
        }

        static def process = { request ->
            println "Processing request: ${request.data}"
            [result: "Processed ${request.data.size()} items"]
        }
    }

    // 2. 链构建器
    static class ChainBuilder {
        private List<Closure> handlers = []

        def addHandler(Closure handler) {
            handlers.add(handler)
            this
        }

        def build() {
            return { request ->
                handlers.inject(request) { result, handler ->
                    handler(result)
                }
            }
        }
    }

    // 3. 条件链
    static class ConditionalChain {
        static def createChain(List<Map> handlerSpecs) {
            return { request ->
                handlerSpecs.inject(request) { result, spec ->
                    def condition = spec.condition
                    def handler = spec.handler
                    if (condition(result)) {
                        handler(result)
                    } else {
                        result
                    }
                }
            }
        }
    }

    static void demonstrateChainOfResponsibility() {
        // 基本链
        def chain = new ChainBuilder()
            .addHandler(HandlerFunctions.authenticate)
            .addHandler(HandlerFunctions.validate)
            .addHandler(HandlerFunctions.authorize)
            .addHandler(HandlerFunctions.process)
            .build()

        def request = [
            token: "valid-token",
            data: ["item1", "item2", "item3"],
            role: "admin"
        ]

        try {
            def result = chain(request)
            println "Chain result: $result"
        } catch (Exception e) {
            println "Chain failed: ${e.message}"
        }

        // 条件链
        def conditionalChain = ConditionalChain.createChain([
            [condition: { it.containsKey('token') }, handler: HandlerFunctions.authenticate],
            [condition: { it.containsKey('data') }, handler: HandlerFunctions.validate],
            [condition: { it.containsKey('role') }, handler: HandlerFunctions.authorize]
        ])

        try {
            def conditionalResult = conditionalChain(request)
            println "Conditional chain result: $conditionalResult"
        } catch (Exception e) {
            println "Conditional chain failed: ${e.message}"
        }
    }
}
```

### 函数式观察者模式

```groovy
class FunctionalObserverPattern {

    // 1. 事件系统
    static class EventSystem {
        private Map<String, List<Closure>> observers = [:]

        def subscribe(String eventType, Closure observer) {
            if (!observers.containsKey(eventType)) {
                observers[eventType] = []
            }
            observers[eventType].add(observer)
            this
        }

        def unsubscribe(String eventType, Closure observer) {
            if (observers.containsKey(eventType)) {
                observers[eventType].remove(observer)
            }
            this
        }

        def publish(String eventType, Object event) {
            if (observers.containsKey(eventType)) {
                observers[eventType].each { observer ->
                    try {
                        observer(event)
                    } catch (Exception e) {
                        println "Observer failed: ${e.message}"
                    }
                }
            }
            this
        }

        def clear() {
            observers.clear()
            this
        }
    }

    // 2. 事件处理器
    static class EventHandlers {
        static def logHandler = { event ->
            println "[LOG] Event received: ${event.type} - ${event.data}"
        }

        static def emailHandler = { event ->
            println "[EMAIL] Sending email notification for: ${event.type}"
        }

        static def analyticsHandler = { event ->
            println "[ANALYTICS] Tracking event: ${event.type}"
        }

        static def errorHandler = { event ->
            println "[ERROR] Error occurred: ${event.error}"
        }
    }

    // 3. 事件过滤器
    static class EventFilters {
        static def filterByType = { String type ->
            return { event ->
                event.type == type
            }
        }

        static def filterByData = { Closure predicate ->
            return { event ->
                predicate(event.data)
            }
        }

        static def createFilteredObserver = { Closure observer, Closure filter ->
            return { event ->
                if (filter(event)) {
                    observer(event)
                }
            }
        }
    }

    static void demonstrateObserverPattern() {
        def eventSystem = new EventSystem()

        // 订阅事件
        eventSystem.subscribe("user.created", EventHandlers.logHandler)
        eventSystem.subscribe("user.created", EventHandlers.emailHandler)
        eventSystem.subscribe("user.created", EventHandlers.analyticsHandler)

        eventSystem.subscribe("error.occurred", EventHandlers.logHandler)
        eventSystem.subscribe("error.occurred", EventHandlers.errorHandler)

        // 发布事件
        eventSystem.publish("user.created", [
            type: "user.created",
            data: [userId: "123", name: "John Doe"],
            timestamp: new Date()
        ])

        eventSystem.publish("error.occurred", [
            type: "error.occurred",
            error: "Database connection failed",
            timestamp: new Date()
        ])

        // 使用过滤器
        def highPriorityHandler = EventFilters.createFilteredObserver(
            EventHandlers.emailHandler,
            EventFilters.filterByData { it.priority == "high" }
        )

        eventSystem.subscribe("alert", highPriorityHandler)

        eventSystem.publish("alert", [
            type: "alert",
            data: [message: "System overload", priority: "high"],
            timestamp: new Date()
        ])
    }
}
```

## 函数式并发编程

### 不可变并发数据结构

```groovy
class ImmutableConcurrentDataStructures {

    // 1. 并发安全不可变列表
    static class ConcurrentImmutableList<T> {
        private final List<T> elements
        private final ReadWriteLock lock = new ReentrantReadWriteLock()

        ConcurrentImmutableList(List<T> elements) {
            this.elements = elements.asImmutable()
        }

        ConcurrentImmutableList<T> add(T element) {
            lock.readLock().lock()
            try {
                def newList = new ArrayList(elements)
                newList.add(element)
                new ConcurrentImmutableList(newList)
            } finally {
                lock.readLock().unlock()
            }
        }

        ConcurrentImmutableList<T> remove(T element) {
            lock.readLock().lock()
            try {
                def newList = new ArrayList(elements)
                newList.remove(element)
                new ConcurrentImmutableList(newList)
            } finally {
                lock.readLock().unlock()
            }
        }

        List<T> toList() {
            lock.readLock().lock()
            try {
                new ArrayList(elements)
            } finally {
                lock.readLock().unlock()
            }
        }

        int size() {
            lock.readLock().lock()
            try {
                elements.size()
            } finally {
                lock.readLock().unlock()
            }
        }
    }

    // 2. 函数式共享状态
    static class FunctionalSharedState {
        private final AtomicReference<Map<String, Object>> stateRef

        FunctionalSharedState(Map<String, Object> initialState) {
            this.stateRef = new AtomicReference(initialState.asImmutable())
        }

        def update(Closure<Map> updater) {
            def oldState = stateRef.get()
            def newState = updater(oldState).asImmutable()

            while (!stateRef.compareAndSet(oldState, newState)) {
                oldState = stateRef.get()
                newState = updater(oldState).asImmutable()
            }

            newState
        }

        def get(String key) {
            stateRef.get().get(key)
        }

        def getSnapshot() {
            new HashMap(stateRef.get())
        }
    }

    // 3. 函数式队列
    static class FunctionalQueue<T> {
        private final List<T> front
        private final List<T> rear

        FunctionalQueue(List<T> front = [], List<T> rear = []) {
            this.front = front.asImmutable()
            this.rear = rear.asImmutable()
        }

        FunctionalQueue<T> enqueue(T item) {
            new FunctionalQueue(front, [item] + rear)
        }

        FunctionalQueue<T> dequeue() {
            if (front.isEmpty()) {
                def newFront = rear.reverse()
                new FunctionalQueue(newFront[1..-1], [])
            } else {
                new FunctionalQueue(front[1..-1], rear)
            }
        }

        T peek() {
            if (front.isEmpty()) {
                rear.isEmpty() ? null : rear.last()
            } else {
                front.first()
            }
        }

        boolean isEmpty() {
            front.isEmpty() && rear.isEmpty()
        }

        int size() {
            front.size() + rear.size()
        }
    }

    static void demonstrateConcurrentStructures() {
        // 并发不可变列表
        def concurrentList = new ConcurrentImmutableList([1, 2, 3])
        def newList = concurrentList.add(4)
        println "Original size: ${concurrentList.size()}, New size: ${newList.size()}"

        // 函数式共享状态
        def sharedState = new FunctionalSharedState([count: 0, users: []])

        def updater = { state ->
            def newCount = state.count + 1
            def newUsers = state.users + ["user$newCount"]
            [count: newCount, users: newUsers]
        }

        def updatedState = sharedState.update(updater)
        println "Updated state: $updatedState"

        // 函数式队列
        def queue = new FunctionalQueue()
        def queue1 = queue.enqueue("A")
        def queue2 = queue1.enqueue("B")
        def queue3 = queue2.enqueue("C")

        println "Queue size: ${queue3.size()}"
        println "Peek: ${queue3.peek()}"

        def queue4 = queue3.dequeue()
        println "After dequeue - size: ${queue4.size()}, peek: ${queue4.peek()}"
    }
}
```

### 函数式并行处理

```groovy
class FunctionalParallelProcessing {

    // 1. 并行映射
    static class ParallelMap {
        static <T, R> List<R> parallelMap(List<T> list, Closure<R> mapper, int threads = 4) {
            if (list.size() < 1000) {
                return list.collect(mapper)
            }

            def pool = Executors.newFixedThreadPool(threads)
            try {
                def batchSize = Math.max(list.size() / threads, 1000) as int
                def batches = list.collate(batchSize)

                def futures = batches.collect { batch ->
                    pool.submit { batch.collect(mapper) }
                }

                futures.collect { it.get() }.flatten()
            } finally {
                pool.shutdown()
            }
        }
    }

    // 2. 并行过滤
    static class ParallelFilter {
        static <T> List<T> parallelFilter(List<T> list, Closure<Boolean> predicate, int threads = 4) {
            if (list.size() < 1000) {
                return list.findAll(predicate)
            }

            def pool = Executors.newFixedThreadPool(threads)
            try {
                def batchSize = Math.max(list.size() / threads, 1000) as int
                def batches = list.collate(batchSize)

                def futures = batches.collect { batch ->
                    pool.submit { batch.findAll(predicate) }
                }

                futures.collect { it.get() }.flatten()
            } finally {
                pool.shutdown()
            }
        }
    }

    // 3. 并行归约
    static class ParallelReduce {
        static <T> T parallelReduce(List<T> list, T identity, Closure<T> reducer, int threads = 4) {
            if (list.size() < 1000) {
                return list.inject(identity, reducer)
            }

            def pool = Executors.newFixedThreadPool(threads)
            try {
                def batchSize = Math.max(list.size() / threads, 1000) as int
                def batches = list.collate(batchSize)

                def futures = batches.collect { batch ->
                    pool.submit { batch.inject(identity, reducer) }
                }

                def partialResults = futures.collect { it.get() }
                partialResults.inject(identity, reducer)
            } finally {
                pool.shutdown()
            }
        }
    }

    static void demonstrateParallelProcessing() {
        def numbers = (1..10000).toList()

        // 并行映射
        def startTime = System.currentTimeMillis()
        def squares = ParallelMap.parallelMap(numbers) { it ** 2 }
        def endTime = System.currentTimeMillis()
        println "Parallel map took ${endTime - startTime}ms"

        // 并行过滤
        startTime = System.currentTimeMillis()
        def evenNumbers = ParallelFilter.parallelFilter(numbers) { it % 2 == 0 }
        endTime = System.currentTimeMillis()
        println "Parallel filter took ${endTime - startTime}ms"

        // 并行归约
        startTime = System.currentTimeMillis()
        def sum = ParallelReduce.parallelReduce(numbers, 0) { acc, num -> acc + num }
        endTime = System.currentTimeMillis()
        println "Parallel reduce took ${endTime - startTime}ms, sum: $sum"

        // 使用GPars
        GParsPool.withPool(4) {
            def gparsSquares = numbers.collectParallel { it ** 2 }
            def gparsEven = numbers.findAllParallel { it % 2 == 0 }
            def gparsSum = numbers.parallel.inject(0) { acc, num -> acc + num }

            println "GPars parallel operations completed"
        }
    }
}
```

### 函数式Actor模型

```groovy
class FunctionalActorModel {

    // 1. 函数式Actor
    static class FunctionalActor {
        private final BlockingQueue<Closure> mailbox = new LinkedBlockingQueue<>()
        private final AtomicBoolean running = new AtomicBoolean(true)
        private final Thread thread

        FunctionalActor(String name = "Actor") {
            this.thread = Thread.start(name) {
                while (running.get()) {
                    def message = mailbox.poll(100, TimeUnit.MILLISECONDS)
                    if (message) {
                        try {
                            message()
                        } catch (Exception e) {
                            println "Actor error: ${e.message}"
                        }
                    }
                }
            }
        }

        def send(Closure message) {
            mailbox.offer(message)
            this
        }

        def stop() {
            running.set(false)
            thread.join()
            this
        }
    }

    // 2. 状态管理Actor
    static class StatefulActor {
        private final FunctionalActor actor
        private final AtomicReference<Map> stateRef

        StatefulActor(Map initialState) {
            this.stateRef = new AtomicReference(initialState)
            this.actor = new FunctionalActor()

            // 初始化处理器
            actor.send { msg ->
                def currentState = stateRef.get()
                def newState = processMessage(currentState, msg)
                stateRef.set(newState)
            }
        }

        def send(Map message) {
            actor.send { msg ->
                def currentState = stateRef.get()
                def newState = processMessage(currentState, msg)
                stateRef.set(newState)
            }
            this
        }

        def getState() {
            stateRef.get()
        }

        def stop() {
            actor.stop()
            this
        }

        private Map processMessage(Map state, Map message) {
            switch (message.type) {
                case "add":
                    def count = state.count ?: 0
                    def items = state.items ?: []
                    return [
                        count: count + 1,
                        items: items + [message.item]
                    ]
                case "remove":
                    def items = state.items ?: []
                    return [
                        count: Math.max((state.count ?: 0) - 1, 0),
                        items: items - [message.item]
                    ]
                case "get":
                    return state
                default:
                    return state
            }
        }
    }

    // 3. 函数式Actor系统
    static class ActorSystem {
        private final Map<String, FunctionalActor> actors = [:]
        private final Map<String, StatefulActor> statefulActors = [:]

        def createActor(String name) {
            def actor = new FunctionalActor(name)
            actors[name] = actor
            actor
        }

        def createStatefulActor(String name, Map initialState) {
            def actor = new StatefulActor(initialState)
            statefulActors[name] = actor
            actor
        }

        def sendToActor(String name, Closure message) {
            def actor = actors[name]
            if (actor) {
                actor.send(message)
            } else {
                throw new IllegalArgumentException("Actor $name not found")
            }
            this
        }

        def sendToStatefulActor(String name, Map message) {
            def actor = statefulActors[name]
            if (actor) {
                actor.send(message)
            } else {
                throw new IllegalArgumentException("Stateful actor $name not found")
            }
            this
        }

        def stopAll() {
            actors.values().each { it.stop() }
            statefulActors.values().each { it.stop() }
            actors.clear()
            statefulActors.clear()
            this
        }
    }

    static void demonstrateActorModel() {
        def system = new ActorSystem()

        // 创建普通Actor
        def processor = system.createActor("processor")
        processor.send { println "Processing message 1" }
        processor.send { println "Processing message 2" }

        // 创建状态管理Actor
        def counter = system.createStatefulActor("counter", [count: 0, items: []])

        counter.send(type: "add", item: "item1")
        counter.send(type: "add", item: "item2")
        counter.send(type: "add", item: "item3")
        counter.send(type: "remove", item: "item2")

        Thread.sleep(1000)  // 等待处理完成

        println "Counter state: ${counter.getState()}"

        system.stopAll()
    }
}
```

## 函数式性能优化

### 惰性求值

```groovy
class LazyEvaluation {

    // 1. 惰性序列
    static class LazySequence<T> {
        private final Closure<T> generator
        private final Closure<Boolean> hasNext
        private T currentValue
        private boolean evaluated = false

        LazySequence(Closure<T> generator, Closure<Boolean> hasNext) {
            this.generator = generator
            this.hasNext = hasNext
        }

        T getCurrent() {
            if (!evaluated) {
                currentValue = generator()
                evaluated = true
            }
            currentValue
        }

        boolean hasNext() {
            hasNext()
        }

        LazySequence<T> next() {
            new LazySequence(generator, hasNext)
        }

        List<T> take(int count) {
            def result = []
            def sequence = this
            count.times {
                if (sequence.hasNext()) {
                    result.add(sequence.getCurrent())
                    sequence = sequence.next()
                }
            }
            result
        }
    }

    // 2. 惰性集合
    static class LazyCollection<T> {
        private final List<T> source
        private final List<Closure> operations = []

        LazyCollection(List<T> source) {
            this.source = source
        }

        LazyCollection<T> filter(Closure<Boolean> predicate) {
            def newOps = new ArrayList(operations)
            newOps.add { list -> list.findAll(predicate) }
            new LazyCollection(source) {
                this.operations = newOps
            }
        }

        LazyCollection<T> map(Closure<T> transformer) {
            def newOps = new ArrayList(operations)
            newOps.add { list -> list.collect(transformer) }
            new LazyCollection(source) {
                this.operations = newOps
            }
        }

        List<T> evaluate() {
            operations.inject(source) { result, operation ->
                operation(result)
            }
        }
    }

    // 3. 惰性计算
    static class LazyComputation {
        private final Closure computation
        private Object value
        private boolean computed = false

        LazyComputation(Closure computation) {
            this.computation = computation
        }

        Object getValue() {
            if (!computed) {
                value = computation()
                computed = true
            }
            value
        }

        boolean isComputed() {
            computed
        }

        void reset() {
            computed = false
            value = null
        }
    }

    static void demonstrateLazyEvaluation() {
        // 惰性序列
        def fibonacci = new LazySequence(
            { def (a, b) = [0, 1]; (0..100).collect { def result = a; (a, b) = [b, a + b]; result } },
            { true }
        )

        def fibNumbers = fibonacci.take(10)
        println "First 10 Fibonacci numbers: $fibNumbers"

        // 惰性集合
        def numbers = (1..10000).toList()
        def lazyCollection = new LazyCollection(numbers)
            .filter { it % 2 == 0 }
            .map { it ** 2 }
            .filter { it < 1000 }

        def result = lazyCollection.evaluate()
        println "Lazy collection result (first 10): ${result.take(10)}"

        // 惰性计算
        def expensive = new LazyComputation {
            println "Computing expensive value..."
            Thread.sleep(1000)
            "Expensive Result"
        }

        println "Before first getValue()"
        println "Result: ${expensive.getValue()}"
        println "After first getValue()"
        println "Second getValue(): ${expensive.getValue()}"
    }
}
```

### 记忆化优化

```groovy
class MemoizationOptimization {

    // 1. 基础记忆化
    static class BasicMemoization {
        static def memoize = { Closure closure ->
            def cache = [:]
            return { Object... args ->
                def key = args.toList()
                if (!cache.containsKey(key)) {
                    cache[key] = closure(*args)
                }
                cache[key]
            }
        }

        static def fibonacci = { int n ->
            if (n <= 1) return n
            fibonacci(n - 1) + fibonacci(n - 2)
        }.memoize()

        static def factorial = { int n ->
            if (n <= 1) return 1
            n * factorial(n - 1)
        }.memoize()

        static void demonstrateBasicMemoization() {
            println "Fibonacci(10): ${fibonacci(10)}"
            println "Fibonacci(10) again: ${fibonacci(10)}"  // 使用缓存

            println "Factorial(5): ${factorial(5)}"
            println "Factorial(5) again: ${factorial(5)}"  // 使用缓存
        }
    }

    // 2. LRU缓存记忆化
    static class LruMemoization {
        static def memoizeAtMost = { Closure closure, int maxSize ->
            def cache = new LinkedHashMap<Object, Object>(maxSize, 0.75f, true) {
                protected boolean removeEldestEntry(Map.Entry eldest) {
                    size() > maxSize
                }
            }
            return { Object... args ->
                def key = args.toList()
                if (!cache.containsKey(key)) {
                    cache[key] = closure(*args)
                }
                cache[key]
            }
        }

        static def expensiveOperation = { int n ->
            println "Computing expensive operation for $n"
            Thread.sleep(100)
            n ** 2
        }

        static def memoizedExpensive = memoizeAtMost(expensiveOperation, 5)

        static void demonstrateLruMemoization() {
            (1..10).each { i ->
                println "Result for $i: ${memoizedExpensive(i)}"
            }

            println "Accessing cached values:"
            (1..3).each { i ->
                println "Result for $i: ${memoizedExpensive(i)}"
            }
        }
    }

    // 3. 定时记忆化
    static class TimedMemoization {
        static def memoizeFor = { Closure closure, long duration ->
            def cache = [:]
            def timestamps = [:]
            return { Object... args ->
                def key = args.toList()
                def now = System.currentTimeMillis()

                if (!cache.containsKey(key) || now - timestamps[key] > duration) {
                    cache[key] = closure(*args)
                    timestamps[key] = now
                }
                cache[key]
            }
        }

        static def timeBasedOperation = { String key ->
            println "Computing time-based operation for $key"
            new Date().toString()
        }

        static def memoizedTimed = memoizeFor(timeBasedOperation, 5000)  // 5秒缓存

        static void demonstrateTimedMemoization() {
            println "First call: ${memoizedTimed('test')}"
            println "Second call (should be cached): ${memoizedTimed('test')}"

            Thread.sleep(6000)
            println "Third call (should recompute): ${memoizedTimed('test')}"
        }
    }
}
```

### 流式处理优化

```groovy
class StreamProcessingOptimization {

    // 1. 流式处理管道
    static class OptimizedPipeline {
        static def createPipeline(List<Closure> operations) {
            return { input ->
                operations.inject(input) { result, operation ->
                    operation(result)
                }
            }
        }

        static def shortCircuitPipeline = { List<Closure> operations ->
            return { input ->
                operations.inject(input) { result, operation ->
                    def intermediate = operation(result)
                    if (intermediate == null || intermediate.isEmpty()) {
                        return intermediate  // 提前终止
                    }
                    intermediate
                }
            }
        }

        static def parallelPipeline = { List<Closure> operations, int threads = 4 ->
            return { input ->
                def pool = Executors.newFixedThreadPool(threads)
                try {
                    def batches = input.collate(Math.max(input.size() / threads, 1000) as int)
                    def futures = batches.collect { batch ->
                        pool.submit {
                            operations.inject(batch) { result, operation ->
                                operation(result)
                            }
                        }
                    }
                    futures.collect { it.get() }.flatten()
                } finally {
                    pool.shutdown()
                }
            }
        }
    }

    // 2. 批处理优化
    static class BatchProcessing {
        static def batchProcess = { List data, Closure operation, int batchSize = 1000 ->
            def results = []
            def batches = data.collate(batchSize)

            batches.each { batch ->
                def batchResult = operation(batch)
                if (batchResult instanceof Collection) {
                    results.addAll(batchResult)
                } else {
                    results.add(batchResult)
                }

                // 批次间暂停
                Thread.sleep(10)
            }

            results
        }

        static def streamBatchProcess = { InputStream input, Closure operation, int batchSize = 1000 ->
            def results = []
            def buffer = new byte[batchSize]
            def bytesRead

            while ((bytesRead = input.read(buffer)) != -1) {
                def batch = buffer[0..bytesRead-1]
                def batchResult = operation(batch)

                if (batchResult instanceof Collection) {
                    results.addAll(batchResult)
                } else {
                    results.add(batchResult)
                }
            }

            results
        }
    }

    // 3. 内存优化
    static class MemoryOptimization {
        static def memoryEfficientProcess = { List data, Closure operation ->
            def result = []
            data.each { item ->
                def processed = operation(item)
                result.add(processed)

                // 定期清理内存
                if (result.size() % 1000 == 0) {
                    System.gc()
                }
            }
            result
        }

        static def streamingProcess = { Iterator iterator, Closure operation ->
            def result = []
            while (iterator.hasNext()) {
                def item = iterator.next()
                def processed = operation(item)
                result.add(processed)

                // 控制内存使用
                if (result.size() >= 10000) {
                    yield result
                    result = []
                }
            }
            result
        }
    }

    static void demonstrateStreamOptimization() {
        def largeData = (1..100000).toList()

        // 流式管道
        def pipeline = OptimizedPipeline.createPipeline([
            { list -> list.findAll { it % 2 == 0 } },
            { list -> list.collect { it ** 2 } },
            { list -> list.take(100) }
        ])

        def result = pipeline(largeData)
        println "Pipeline result size: ${result.size()}"

        // 并行处理
        def parallelPipeline = OptimizedPipeline.parallelPipeline([
            { list -> list.findAll { it % 2 == 0 } },
            { list -> list.collect { it ** 2 } }
        ])

        def parallelResult = parallelPipeline(largeData)
        println "Parallel pipeline result size: ${parallelResult.size()}"

        // 批处理
        def batchResult = BatchProcessing.batchProcess(largeData) { batch ->
            batch.findAll { it % 3 == 0 }
        }
        println "Batch processing result size: ${batchResult.size()}"
    }
}
```

## 实战案例

### 函数式数据处理管道

```groovy
class FunctionalDataProcessingPipeline {

    // 1. 数据源
    static class DataSources {
        static List<Map> generateSalesData(int count) {
            (1..count).collect { i ->
                [
                    id: i,
                    product: "Product-${i % 10}",
                    quantity: (int)(Math.random() * 100) + 1,
                    price: (Math.random() * 1000).round(2),
                    date: new Date() - (int)(Math.random() * 365),
                    region: ["North", "South", "East", "West"][i % 4],
                    salesperson: ["Alice", "Bob", "Charlie", "Diana"][i % 4]
                ]
            }
        }

        static List<Map> generateUserData(int count) {
            (1..count).collect { i ->
                [
                    id: i,
                    name: "User$i",
                    email: "user$i@example.com",
                    age: (int)(Math.random() * 50) + 18,
                    city: ["New York", "London", "Paris", "Tokyo"][i % 4],
                    registrationDate: new Date() - (int)(Math.random() * 365),
                    isActive: Math.random() > 0.2
                ]
            }
        }
    }

    // 2. 处理器
    static class Processors {
        static def filterByRegion = { region ->
            return { data -> data.findAll { it.region == region } }
        }

        static def filterByDateRange = { startDate, endDate ->
            return { data -> data.findAll { it.date >= startDate && it.date <= endDate } }
        }

        static def calculateTotal = { data ->
            data.collect { it.quantity * it.price }.sum()
        }

        static def groupByField = { field ->
            return { data -> data.groupBy { it[field] } }
        }

        static def aggregateByGroup = { field, aggregation ->
            return { data ->
                data.groupBy { it[field] }.collectEntries { key, group ->
                    [(key): aggregation(group)]
                }
            }
        }

        static def enrichData = { enrichment ->
            return { data -> data.collect { item -> item + enrichment(item) } }
        }
    }

    // 3. 管道构建器
    static class PipelineBuilder {
        private List<Closure> operations = []

        def add(Closure operation) {
            operations.add(operation)
            this
        }

        def filter(Closure predicate) {
            operations.add { data -> data.findAll(predicate) }
            this
        }

        def map(Closure transformer) {
            operations.add { data -> data.collect(transformer) }
            this
        }

        def groupBy(Closure classifier) {
            operations.add { data -> data.groupBy(classifier) }
            this
        }

        def aggregate(Closure aggregator) {
            operations.add { data -> aggregator(data) }
            this
        }

        def build() {
            return { input ->
                operations.inject(input) { result, operation ->
                    operation(result)
                }
            }
        }
    }

    static void demonstrateDataPipeline() {
        // 生成测试数据
        def salesData = DataSources.generateSalesData(1000)

        // 构建处理管道
        def salesPipeline = new PipelineBuilder()
            .filter { it.quantity > 50 }
            .filter { it.price > 500 }
            .map { item ->
                item + [total: item.quantity * item.price]
            }
            .groupBy { it.region }
            .map { region, items ->
                [
                    region: region,
                    totalSales: items.collect { it.total }.sum(),
                    averagePrice: items.collect { it.price }.sum() / items.size(),
                    itemCount: items.size()
                ]
            }
            .filter { it.totalSales > 10000 }
            .sort { a, b -> b.totalSales <=> a.totalSales }
            .build()

        def result = salesPipeline(salesData)
        println "Sales Analysis Results:"
        result.each { println "  ${it.region}: \$${it.totalSales.round(2)} (avg: \$${it.averagePrice.round(2)}, count: ${it.itemCount})" }
    }
}
```

### 函数式配置管理

```groovy
class FunctionalConfigurationManagement {

    // 1. 配置源
    static class ConfigurationSources {
        static def fromMap = { Map config ->
            { key -> config[key] }
        }

        static def fromProperties = { Properties props ->
            { key -> props[key] }
        }

        static def fromJson = { String json ->
            def config = new groovy.json.JsonSlurper().parseText(json)
            { key -> config[key] }
        }

        static def fromEnvironment = {
            { key -> System.getenv(key) }
        }

        static def fromSystemProperties = {
            { key -> System.getProperty(key) }
        }
    }

    // 2. 配置组合器
    static class ConfigurationCombiners {
        static def chain = { List<Closure> sources ->
            return { key ->
                sources.findResult { source ->
                    def value = source(key)
                    if (value != null) value
                }
            }
        }

        static def merge = { List<Closure> sources ->
            return { key ->
                def values = sources.collect { it(key) }.findAll { it != null }
                values.size() > 0 ? values.last() : null
            }
        }

        static def cascade = { List<Closure> sources ->
            return { key ->
                def result = [:]
                sources.each { source ->
                    def value = source(key)
                    if (value != null && value instanceof Map) {
                        result.putAll(value)
                    }
                }
                result
            }
        }
    }

    // 3. 配置验证器
    static class ConfigurationValidators {
        static def required = { key ->
            return { config ->
                def value = config(key)
                if (value == null) {
                    throw new IllegalArgumentException("Required configuration '$key' is missing")
                }
                value
            }
        }

        static def ofType = { Class type ->
            return { key ->
                return { config ->
                    def value = config(key)
                    if (value != null && !type.isInstance(value)) {
                        throw new IllegalArgumentException("Configuration '$key' must be of type $type")
                    }
                    value
                }
            }
        }

        static def matching = { Closure validator ->
            return { key ->
                return { config ->
                    def value = config(key)
                    if (value != null && !validator(value)) {
                        throw new IllegalArgumentException("Configuration '$key' failed validation")
                    }
                    value
                }
            }
        }
    }

    // 4. 配置构建器
    static class ConfigurationBuilder {
        private List<Closure> sources = []
        private Map<String, Closure> validators = [:]

        def addSource(Closure source) {
            sources.add(source)
            this
        }

        def addMapSource(Map config) {
            sources.add(ConfigurationSources.fromMap(config))
            this
        }

        def addJsonSource(String json) {
            sources.add(ConfigurationSources.fromJson(json))
            this
        }

        def addEnvironmentSource() {
            sources.add(ConfigurationSources.fromEnvironment())
            this
        }

        def addSystemPropertiesSource() {
            sources.add(ConfigurationSources.fromSystemProperties())
            this
        }

        def addValidator(String key, Closure validator) {
            validators[key] = validator
            this
        }

        def build() {
            def configSource = ConfigurationCombiners.chain(sources)

            return { key ->
                def value = configSource(key)
                if (validators.containsKey(key)) {
                    value = validators[key](value)
                }
                value
            }
        }
    }

    static void demonstrateConfigurationManagement() {
        // 构建配置
        def config = new ConfigurationBuilder()
            .addMapSource([database: [url: "jdbc:mysql://localhost:3306/mydb", username: "user"]])
            .addEnvironmentSource()
            .addSystemPropertiesSource()
            .addValidator("database.url", ConfigurationValidators.required())
            .addValidator("database.username", ConfigurationValidators.required())
            .addValidator("database.maxConnections", ConfigurationValidators.ofType(Integer))
            .build()

        // 使用配置
        println "Database URL: ${config('database.url')}"
        println "Database username: ${config('database.username')}"
        println "Database max connections: ${config('database.maxConnections')}"
    }
}
```

### 函数式事件处理

```groovy
class FunctionalEventHandling {

    // 1. 事件类型
    static class EventTypes {
        static Map createEvent(String type, Map data = [:]) {
            [
                type: type,
                data: data,
                timestamp: new Date(),
                id: UUID.randomUUID().toString()
            ]
        }

        static Map createErrorEvent(String error, Map data = [:]) {
            createEvent("error", data + [error: error])
        }

        static Map createUserEvent(String action, Map userData) {
            createEvent("user.$action", userData)
        }

        static Map createSystemEvent(String action, Map systemData) {
            createEvent("system.$action", systemData)
        }
    }

    // 2. 事件处理器
    static class EventHandlers {
        static def logHandler = { event ->
            println "[${event.timestamp.format('yyyy-MM-dd HH:mm:ss')}] ${event.type}: ${event.data}"
        }

        static def emailHandler = { event ->
            // 模拟发送邮件
            println "Sending email notification for ${event.type}"
        }

        static def metricsHandler = { event ->
            // 模拟记录指标
            println "Recording metrics for ${event.type}"
        }

        static def errorHandler = { event ->
            println "ERROR: ${event.data.error}"
        }

        static def userEventHandler = { event ->
            println "User event: ${event.type} - User ${event.data.userId}"
        }
    }

    // 3. 事件路由器
    static class EventRouter {
        private Map<String, List<Closure>> routes = [:]

        def route(String eventType, Closure handler) {
            if (!routes.containsKey(eventType)) {
                routes[eventType] = []
            }
            routes[eventType].add(handler)
            this
        }

        def routePattern(String pattern, Closure handler) {
            def regex = pattern.replaceAll('\\*', '.*')
            return { event ->
                if (event.type ==~ regex) {
                    handler(event)
                }
            }
        }

        def dispatch(Map event) {
            def handlers = routes[event.type] ?: []
            handlers.each { handler ->
                try {
                    handler(event)
                } catch (Exception e) {
                    println "Handler failed for event ${event.type}: ${e.message}"
                }
            }
            this
        }

        def dispatchAll(List<Map> events) {
            events.each { event ->
                dispatch(event)
            }
            this
        }
    }

    // 4. 事件处理器链
    static class EventChain {
        private List<Closure> handlers = []

        def addHandler(Closure handler) {
            handlers.add(handler)
            this
        }

        def process(Map event) {
            handlers.inject(event) { result, handler ->
                handler(result)
            }
        }

        def processAll(List<Map> events) {
            events.collect { event ->
                process(event)
            }
        }
    }

    static void demonstrateEventHandling() {
        // 创建事件路由器
        def router = new EventRouter()
            .route("user.created", EventHandlers.userEventHandler)
            .route("user.updated", EventHandlers.userEventHandler)
            .route("error", EventHandlers.errorHandler)
            .routePattern("system.*", EventHandlers.logHandler)

        // 创建事件处理器链
        def chain = new EventChain()
            .addHandler(EventHandlers.logHandler)
            .addHandler(EventHandlers.metricsHandler)

        // 生成和处理事件
        def events = [
            EventTypes.createUserEvent("created", [userId: "123", name: "John Doe"]),
            EventTypes.createSystemEvent("startup", [component: "database"]),
            EventTypes.createErrorEvent("Connection failed", [component: "cache"])
        ]

        println "=== Router Processing ==="
        router.dispatchAll(events)

        println "\n=== Chain Processing ==="
        def chainResults = chain.processAll(events)
        chainResults.each { result ->
            println "Processed event: ${result.type}"
        }
    }
}
```

## 最佳实践

### 函数式设计原则

```groovy
class FunctionalDesignPrinciples {

    // 1. 纯函数优先
    static class PureFirstPrinciple {
        // 纯函数 - 无副作用
        static def calculateTotal = { List<Map> items ->
            items.collect { it.quantity * it.price }.sum()
        }

        // 纯函数 - 无副作用
        static def filterValidUsers = { List<Map> users ->
            users.findAll { it.email && it.age >= 18 }
        }

        // 非纯函数 - 有副作用
        static def logUserAction = { Map user, String action ->
            println "User ${user.id} performed $action"
            user.lastAction = action
        }

        // 纯函数包装器
        static def withLogging = { Closure operation, String operationName ->
            return { Object... args ->
                println "Starting $operationName with args: $args"
                def result = operation(*args)
                println "Completed $operationName with result: $result"
                result
            }
        }

        static void demonstratePureFunctions() {
            def items = [
                [quantity: 2, price: 10.0],
                [quantity: 3, price: 15.0]
            ]

            def total = calculateTotal(items)
            println "Total: $total"

            def loggedCalculateTotal = withLogging(calculateTotal, "calculateTotal")
            def loggedTotal = loggedCalculateTotal(items)
            println "Logged total: $loggedTotal"
        }
    }

    // 2. 不可变性原则
    static class ImmutabilityPrinciple {
        // 不可变数据构建
        static def buildImmutableUser = { Map data ->
            [
                id: data.id,
                name: data.name,
                email: data.email,
                createdAt: new Date(),
                version: 1
            ].asImmutable()
        }

        // 不可变更新
        static def updateUser = { Map user, Map updates ->
            def newUser = new HashMap(user)
            updates.each { key, value ->
                newUser[key] = value
            }
            newUser.version = user.version + 1
            newUser.updatedAt = new Date()
            newUser.asImmutable()
        }

        // 不可变集合操作
        static def addItem = { List list, item ->
            (list + [item]).asImmutable()
        }

        static def removeItem = { List list, item ->
            (list - [item]).asImmutable()
        }

        static void demonstrateImmutability() {
            def user = buildImmutableUser([id: 1, name: "John", email: "john@example.com"])
            println "Original user: $user"

            def updatedUser = updateUser(user, [name: "John Doe"])
            println "Updated user: $updatedUser"
            println "Original user unchanged: $user"

            def numbers = [1, 2, 3].asImmutable()
            def newNumbers = addItem(numbers, 4)
            println "Original: $numbers"
            println "New: $newNumbers"
        }
    }

    // 3. 组合优于继承
    static class CompositionOverInheritance {
        // 函数组合
        static def compose = { Closure f, Closure g ->
            return { x -> f(g(x)) }
        }

        // 特性组合
        static def withValidation = { Closure validator, Closure operation ->
            return { Object... args ->
                if (validator(*args)) {
                    operation(*args)
                } else {
                    throw new IllegalArgumentException("Validation failed")
                }
            }
        }

        static def withLogging = { Closure logger, Closure operation ->
            return { Object... args ->
                logger("Starting operation", args)
                def result = operation(*args)
                logger("Operation completed", [result: result])
                result
            }
        }

        static def withRetry = { Closure operation, int maxRetries = 3 ->
            return { Object... args ->
                def attempt = 0
                while (attempt < maxRetries) {
                    try {
                        return operation(*args)
                    } catch (Exception e) {
                        attempt++
                        if (attempt == maxRetries) {
                            throw e
                        }
                        Thread.sleep(1000)
                    }
                }
            }
        }

        static void demonstrateComposition() {
            def add = { a, b -> a + b }
            def multiply = { a, b -> a * b }

            def addThenMultiply = compose(multiply, add)
            println "(2 + 3) * 4 = ${addThenMultiply(2, 3, 4)}"

            def validatedAdd = withValidation({ a, b -> a >= 0 && b >= 0 }, add)
            def loggedAdd = withLogging({ msg, args -> println "$msg: $args" }, validatedAdd)
            def retriedAdd = withRetry(loggedAdd)

            println "Composed add: ${retriedAdd(5, 3)}"
        }
    }
}
```

### 性能优化策略

```groovy
class PerformanceOptimizationStrategies {

    // 1. 惰性求值策略
    static class LazyEvaluationStrategy {
        static def lazyList = { Closure generator ->
            def result = []
            return { index ->
                while (result.size() <= index) {
                    result.add(generator(result.size()))
                }
                result[index]
            }
        }

        static def lazyCache = { Closure loader ->
            def cachedValue = null
            def loaded = false
            return {
                if (!loaded) {
                    cachedValue = loader()
                    loaded = true
                }
                cachedValue
            }
        }

        static def memoizeWithTTL = { Closure operation, long ttl ->
            def cache = [:]
            def timestamps = [:]
            return { Object... args ->
                def key = args.toList()
                def now = System.currentTimeMillis()

                if (!cache.containsKey(key) || now - timestamps[key] > ttl) {
                    cache[key] = operation(*args)
                    timestamps[key] = now
                }

                cache[key]
            }
        }

        static void demonstrateLazyEvaluation() {
            def fibonacci = lazyList { n ->
                if (n <= 1) n
                else fibonacci(n - 1) + fibonacci(n - 2)
            }

            println "Fibonacci(10): ${fibonacci(10)}"
            println "Fibonacci(15): ${fibonacci(15)}"

            def expensive = lazyCache {
                println "Loading expensive data..."
                Thread.sleep(1000)
                "Expensive Result"
            }

            println "First access: ${expensive()}"
            println "Second access: ${expensive()}"
        }
    }

    // 2. 并发处理策略
    static class ConcurrentProcessingStrategy {
        static def parallelProcess = { List data, Closure operation, int threads = 4 ->
            if (data.size() < 1000) {
                return data.collect(operation)
            }

            def pool = Executors.newFixedThreadPool(threads)
            try {
                def batchSize = Math.max(data.size() / threads, 1000) as int
                def batches = data.collate(batchSize)

                def futures = batches.collect { batch ->
                    pool.submit { batch.collect(operation) }
                }

                futures.collect { it.get() }.flatten()
            } finally {
                pool.shutdown()
            }
        }

        static def asyncProcess = { Closure operation ->
            def future = Executors.newSingleThreadExecutor().submit(operation as Callable)
            return { -> future.get() }
        }

        static def batchProcess = { List data, Closure operation, int batchSize = 1000 ->
            def results = []
            def batches = data.collate(batchSize)

            batches.each { batch ->
                def batchResult = operation(batch)
                results.addAll(batchResult instanceof Collection ? batchResult : [batchResult])
            }

            results
        }

        static void demonstrateConcurrentProcessing() {
            def largeData = (1..10000).toList()

            def start = System.currentTimeMillis()
            def squares = parallelProcess(largeData) { it ** 2 }
            def end = System.currentTimeMillis()
            println "Parallel processing took ${end - start}ms"

            def asyncOperation = asyncProcess {
                println "Async operation started"
                Thread.sleep(1000)
                "Async Result"
            }

            println "Async operation submitted"
            println "Async result: ${asyncOperation()}"
        }
    }

    // 3. 内存优化策略
    static class MemoryOptimizationStrategy {
        static def streamProcess = { Iterator iterator, Closure operation ->
            def result = []
            def counter = 0

            while (iterator.hasNext()) {
                def item = iterator.next()
                def processed = operation(item)
                result.add(processed)

                counter++
                if (counter % 1000 == 0) {
                    yield result
                    result = []
                }
            }

            result
        }

        static def chunkedProcess = { List data, Closure operation, int chunkSize = 1000 ->
            def results = []
            def chunks = data.collate(chunkSize)

            chunks.each { chunk ->
                def chunkResult = operation(chunk)
                results.addAll(chunkResult instanceof Collection ? chunkResult : [chunkResult])

                // 释放内存
                chunk.clear()
                System.gc()
            }

            results
        }

        static def weakCache = { Closure loader ->
            def cache = new WeakHashMap()
            return { key ->
                def value = cache.get(key)
                if (value == null) {
                    value = loader(key)
                    cache.put(key, value)
                }
                value
            }
        }

        static void demonstrateMemoryOptimization() {
            def largeList = (1..100000).toList()
            def iterator = largeList.iterator()

            def results = streamProcess(iterator) { item ->
                item ** 2
            }

            println "Stream processing results: ${results.size()}"

            def chunkedResults = chunkedProcess(largeList) { chunk ->
                chunk.collect { it ** 2 }
            }

            println "Chunked processing results: ${chunkedResults.size()}"
        }
    }
}
```

### 错误处理策略

```groovy
class ErrorHandlingStrategies {

    // 1. 函数式错误处理
    static class FunctionalErrorHandling {
        static def Either = { Closure operation ->
            try {
                def result = operation()
                [success: true, value: result]
            } catch (Exception e) {
                [success: false, error: e.message]
            }
        }

        static def Option = { Closure operation ->
            try {
                def result = operation()
                result != null ? [present: true, value: result] : [present: false]
            } catch (Exception e) {
                [present: false, error: e.message]
            }
        }

        static def Try = { Closure operation ->
            try {
                def result = operation()
                [success: true, value: result]
            } catch (Exception e) {
                [success: false, exception: e]
            }
        }

        static def mapEither = { Map either, Closure transformer ->
            if (either.success) {
                def newResult = transformer(either.value)
                [success: true, value: newResult]
            } else {
                either
            }
        }

        static def flatMapEither = { Map either, Closure transformer ->
            if (either.success) {
                transformer(either.value)
            } else {
                either
            }
        }

        static void demonstrateFunctionalErrorHandling() {
            def safeDivide = { a, b ->
                Either { a / b }
            }

            def result1 = safeDivide(10, 2)
            println "10 / 2 = ${result1.success ? result1.value : result1.error}"

            def result2 = safeDivide(10, 0)
            println "10 / 0 = ${result2.success ? result2.value : result2.error}"

            def chained = flatMapEither(result1) { value ->
                safeDivide(value, 5)
            }
            println "Chained result: ${chained.success ? chained.value : chained.error}"
        }
    }

    // 2. 恢复策略
    static class RecoveryStrategies {
        static def withFallback = { Closure operation, Closure fallback ->
            try {
                def result = operation()
                result != null ? result : fallback()
            } catch (Exception e) {
                fallback()
            }
        }

        static def withRetry = { Closure operation, int maxRetries = 3, long delay = 1000 ->
            def attempt = 0
            while (attempt < maxRetries) {
                try {
                    return operation()
                } catch (Exception e) {
                    attempt++
                    if (attempt == maxRetries) {
                        throw e
                    }
                    Thread.sleep(delay)
                }
            }
        }

        static def withCircuitBreaker = { Closure operation, int failureThreshold = 5, long timeout = 60000 ->
            def state = [:].withDefault { key ->
                switch (key) {
                    case 'state': return 'CLOSED'
                    case 'failures': return 0
                    case 'lastFailure': return 0
                    default: return null
                }
            }

            return {
                if (state.state == 'OPEN') {
                    if (System.currentTimeMillis() - state.lastFailure > timeout) {
                        state.state = 'HALF_OPEN'
                    } else {
                        throw new RuntimeException("Circuit breaker is OPEN")
                    }
                }

                try {
                    def result = operation()
                    if (state.state == 'HALF_OPEN') {
                        state.state = 'CLOSED'
                        state.failures = 0
                    }
                    result
                } catch (Exception e) {
                    state.failures++
                    state.lastFailure = System.currentTimeMillis()

                    if (state.failures >= failureThreshold) {
                        state.state = 'OPEN'
                    }

                    throw e
                }
            }
        }

        static void demonstrateRecoveryStrategies() {
            def unreliableOperation = {
                if (Math.random() > 0.7) {
                    throw new RuntimeException("Random failure")
                }
                "Success"
            }

            def result1 = withFallback(unreliableOperation) { "Fallback result" }
            println "With fallback: $result1"

            def result2 = withRetry(unreliableOperation, 5)
            println "With retry: $result2"

            def circuitBreaker = withCircuitBreaker(unreliableOperation, 3, 10000)
            try {
                def result3 = circuitBreaker()
                println "Circuit breaker result: $result3"
            } catch (Exception e) {
                println "Circuit breaker error: ${e.message}"
            }
        }
    }

    // 3. 日志和监控
    static class LoggingAndMonitoring {
        static def withLogging = { Closure operation, String operationName ->
            return { Object... args ->
                def start = System.currentTimeMillis()
                println "Starting $operationName with args: $args"

                try {
                    def result = operation(*args)
                    def duration = System.currentTimeMillis() - start
                    println "Completed $operationName in ${duration}ms with result: $result"
                    result
                } catch (Exception e) {
                    def duration = System.currentTimeMillis() - start
                    println "Failed $operationName in ${duration}ms with error: ${e.message}"
                    throw e
                }
            }
        }

        static def withMetrics = { Closure operation, String metricName ->
            return { Object... args ->
                def start = System.currentTimeMillis()
                try {
                    def result = operation(*args)
                    def duration = System.currentTimeMillis() - start
                    // 记录指标
                    println "Metric $metricName: ${duration}ms"
                    result
                } catch (Exception e) {
                    def duration = System.currentTimeMillis() - start
                    println "Metric $metricName: FAILED in ${duration}ms"
                    throw e
                }
            }
        }

        static def withTracing = { Closure operation, String traceId ->
            return { Object... args ->
                println "[$traceId] Starting operation"
                try {
                    def result = operation(*args)
                    println "[$traceId] Operation completed successfully"
                    result
                } catch (Exception e) {
                    println "[$traceId] Operation failed: ${e.message}"
                    throw e
                }
            }
        }

        static void demonstrateLoggingAndMonitoring() {
            def add = { a, b -> a + b }

            def loggedAdd = withLogging(add, "add")
            def metricsAdd = withMetrics(add, "addOperation")
            def tracedAdd = withTracing(add, "trace-123")

            println "Logged: ${loggedAdd(5, 3)}"
            println "Metrics: ${metricsAdd(10, 20)}"
            println "Traced: ${tracedAdd(7, 8)}"
        }
    }
}
```

## 总结

Groovy函数式编程提供了一种强大而优雅的编程范式，结合了函数式编程的理论优势与Groovy的实用特性。通过本章节的学习，我们深入了解了：

### 核心概念总结

1. **纯函数与不可变性**：构建可预测、可测试的代码基础
2. **闭包与高阶函数**：实现灵活的函数组合和抽象
3. **函数式数据结构**：处理数据的不可变操作
4. **函数式设计模式**：应用函数式思想解决实际问题
5. **并发编程**：利用不可变性和函数式特性构建并发安全的应用

### 关键技术要点

1. **闭包的强大功能**：柯里化、记忆化、组合、委托
2. **函数组合模式**：构建可复用、可组合的函数库
3. **惰性求值**：优化性能，减少不必要的计算
4. **函数式并发**：避免共享状态，构建安全的并发应用
5. **错误处理策略**：使用Either、Option等函数式错误处理模式

### 实际应用建议

1. **渐进式采用**：在现有代码中逐步引入函数式特性
2. **性能考虑**：合理使用记忆化、惰性求值等优化技术
3. **代码可读性**：保持函数简洁，命名清晰
4. **测试策略**：利用纯函数的可测试性进行单元测试
5. **团队协作**：建立函数式编程的团队标准和最佳实践

通过掌握Groovy函数式编程，开发者可以编写出更简洁、更安全、更易维护的代码，同时充分利用JVM的性能优势。函数式编程不仅是一种编程技术，更是一种思维方式的转变，值得每个Groovy开发者深入学习和实践。