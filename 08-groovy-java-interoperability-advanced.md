# Groovy与Java互操作高级技巧

## 目录

1. [引言](#引言)
2. [Groovy与Java互操作基础](#groovy与java互操作基础)
3. [类型转换与映射](#类型转换与映射)
4. [集合操作互操作](#集合操作互操作)
5. [异常处理机制](#异常处理机制)
6. [注解处理](#注解处理)
7. [反射与动态调用](#反射与动态调用)
8. [性能优化技巧](#性能优化技巧)
9. [高级互操作模式](#高级互操作模式)
10. [实战案例](#实战案例)
11. [最佳实践与注意事项](#最佳实践与注意事项)
12. [总结](#总结)

## 引言

Groovy作为JVM上的动态语言，与Java有着天然的互操作性。这种互操作性不仅体现在语法层面，更深入到类型系统、内存管理、类加载机制等底层实现。本章节将深入探讨Groovy与Java互操作的高级技巧，帮助开发者充分利用两种语言的优势。

### 互操作性的重要性

在实际开发中，Groovy与Java的互操作主要体现在以下几个方面：

1. **渐进式迁移**：从Java迁移到Groovy的渐进式过程
2. **生态系统整合**：利用Java丰富的第三方库
3. **性能优化**：关键路径使用Java，灵活开发使用Groovy
4. **团队协作**：不同背景的开发者共同工作

## Groovy与Java互操作基础

### 基本互操作原理

Groovy与Java的互操作性建立在以下基础之上：

1. **共享JVM**：两种语言运行在相同的JVM上
2. **字节码兼容**：都编译为JVM字节码
3. **类加载机制**：使用相同的类加载器
4. **类型系统**：基于Java类型系统进行扩展

### 基础互操作示例

```groovy
// 1. 调用Java类
import java.util.ArrayList
import java.util.HashMap

def javaList = new ArrayList<String>()
javaList.add("Groovy")
javaList.add("Java")

def javaMap = new HashMap<String, Integer>()
javaMap.put("count", 100)

// 2. Groovy与Java类型自动转换
def groovyList = ["Groovy", "Java"] as ArrayList<String>
def groovyMap = [count: 100] as HashMap<String, Integer>

// 3. Java接口实现
interface JavaInterface {
    String process(String input)
}

class GroovyImpl implements JavaInterface {
    String process(String input) {
        "Processed: $input"
    }
}

def impl = new GroovyImpl()
assert impl.process("Hello") == "Processed: Hello"
```

### 互操作核心机制

```groovy
class InteropCore {

    // 1. 类型检查与转换
    static void demonstrateTypeConversion() {
        def groovyString = "Hello"
        String javaString = groovyString as String

        def groovyNumber = 42
        int javaInt = groovyNumber as int

        // 数组转换
        def groovyArray = [1, 2, 3]
        int[] javaArray = groovyArray as int[]
    }

    // 2. 集合互操作
    static void demonstrateCollectionInterop() {
        // Java集合 -> Groovy集合
        def javaList = new ArrayList<String>()
        javaList.add("Item1")
        javaList.add("Item2")

        // 自动获得Groovy集合方法
        javaList.each { println it }
        javaList*.toUpperCase()

        // Groovy集合 -> Java集合
        def groovyList = ["Item1", "Item2"]
        List<String> javaListFromGroovy = groovyList
    }

    // 3. 异常处理
    static void demonstrateExceptionHandling() {
        try {
            // Java异常可以在Groovy中处理
            throw new JavaException("Java Exception")
        } catch (JavaException e) {
            println "Caught Java exception: ${e.message}"
        }

        try {
            // Groovy异常可以在Java中处理
            throw new GroovyException("Groovy Exception")
        } catch (Exception e) {
            println "Caught exception: ${e.message}"
        }
    }
}
```

## 类型转换与映射

### 基本类型转换

```groovy
class TypeConversion {

    // 1. 自动类型转换
    static void demonstrateAutoConversion() {
        // String <-> 基本类型
        String str = "123"
        int num = str as int
        String backToStr = num as String

        // 数字类型转换
        def d = 3.14
        int i = d as int  // 截断
        long l = i as long

        // 布尔转换
        def boolStr = "true"
        boolean bool = boolStr as boolean

        // 字符转换
        def charStr = "A"
        char c = charStr as char
    }

    // 2. 数组类型转换
    static void demonstrateArrayConversion() {
        // Groovy列表 -> Java数组
        def list = [1, 2, 3, 4, 5]
        int[] intArray = list as int[]
        String[] strArray = ["a", "b", "c"] as String[]

        // 多维数组
        def matrix = [[1, 2], [3, 4]]
        int[][] intMatrix = matrix as int[][]

        // Java数组 -> Groovy列表
        def javaArray = new int[]{1, 2, 3}
        def groovyList = javaArray.toList()
    }

    // 3. 集合类型转换
    static void demonstrateCollectionConversion() {
        // List转换
        def groovyList = ["a", "b", "c"]
        ArrayList<String> arrayList = groovyList as ArrayList<String>
        LinkedList<String> linkedList = groovyList as LinkedList<String>

        // Set转换
        def groovySet = ["a", "b", "c"] as Set
        HashSet<String> hashSet = groovySet as HashSet<String>
        TreeSet<String> treeSet = groovySet as TreeSet<String>

        // Map转换
        def groovyMap = [key1: "value1", key2: "value2"]
        HashMap<String, String> hashMap = groovyMap as HashMap<String, String>
        TreeMap<String, String> treeMap = groovyMap as TreeMap<String, String>
    }
}
```

### 自定义类型转换

```groovy
class CustomTypeConversion {

    // 1. 实现asType方法
    static class Person {
        String name
        int age

        Object asType(Class clazz) {
            if (clazz == Map) {
                return [name: name, age: age]
            } else if (clazz == String) {
                return "Person($name, $age)"
            }
            super.asType(clazz)
        }
    }

    // 2. 自定义转换器
    static class DateConverter {
        static Object convertToDate(String str) {
            if (str.matches("\\d{4}-\\d{2}-\\d{2}")) {
                return Date.parse("yyyy-MM-dd", str)
            }
            throw new IllegalArgumentException("Invalid date format")
        }

        static Object convertFromDateString(String str, Class targetType) {
            switch (targetType) {
                case Date:
                    return convertToDate(str)
                case Calendar:
                    def cal = Calendar.getInstance()
                    cal.time = convertToDate(str)
                    return cal
                case Long:
                    return convertToDate(str).time
                default:
                    throw new IllegalArgumentException("Unsupported target type")
            }
        }
    }

    // 3. 类型转换注册
    static void registerCustomConverters() {
        // 注册全局类型转换
        def registry = GroovySystem.metaClassRegistry

        // 为String注册转换为Date的方法
        String.metaClass.asType << { Class clazz ->
            if (clazz == Date) {
                return DateConverter.convertToDate(delegate)
            }
            delegate.asType(clazz)
        }

        // 为自定义类注册转换方法
        Person.metaClass.asType << { Class clazz ->
            if (clazz == JsonElement) {
                def json = new JsonObject()
                json.addProperty("name", delegate.name)
                json.addProperty("age", delegate.age)
                return json
            }
            delegate.asType(clazz)
        }
    }
}
```

### 类型映射策略

```groovy
class TypeMapping {

    // 1. 泛型类型映射
    static void demonstrateGenericMapping() {
        // Java泛型 -> Groovy
        def javaList = new ArrayList<String>()
        javaList.add("Hello")

        // 保持类型信息
        def groovyList = javaList as List<String>
        groovyList.add("World")

        // 泛型方法调用
        def processor = new GenericProcessor<String>()
        processor.process("Test")

        // 复杂泛型
        Map<String, List<Integer>> complexMap = [
            "list1": [1, 2, 3],
            "list2": [4, 5, 6]
        ]
    }

    // 2. 枚举类型映射
    static void demonstrateEnumMapping() {
        enum JavaEnum {
            VALUE1, VALUE2, VALUE3
        }

        enum GroovyEnum {
            VALUE1, VALUE2, VALUE3
        }

        // Java枚举 -> Groovy
        def javaEnum = JavaEnum.VALUE1
        def groovyEnum = GroovyEnum.valueOf(javaEnum.name())

        // 枚举集合
        def javaEnumArray = [JavaEnum.VALUE1, JavaEnum.VALUE2] as JavaEnum[]
        def groovyEnumList = javaEnumArray.collect { GroovyEnum.valueOf(it.name()) }
    }

    // 3. 自定义类型映射
    static class TypeMapper {
        static Map<Class, Class> typeMappings = [:]

        static void registerMapping(Class source, Class target) {
            typeMappings[source] = target
        }

        static Object map(Object source) {
            def targetType = typeMappings[source.getClass()]
            if (targetType) {
                return mapToTarget(source, targetType)
            }
            source
        }

        static Object mapToTarget(Object source, Class targetType) {
            // 实现具体的映射逻辑
            if (targetType == Map) {
                return source.properties.findAll { !['class', 'metaClass'].contains(it.key) }
            }
            // 其他映射逻辑
            source
        }
    }
}
```

## 集合操作互操作

### Java集合增强

```groovy
class JavaCollectionEnhancement {

    // 1. 为Java集合添加Groovy方法
    static void enhanceJavaCollections() {
        // 为ArrayList添加Groovy风格的方法
        ArrayList.metaClass.eachWithIndex { Closure closure ->
            for (int i = 0; i < delegate.size(); i++) {
                closure.call(delegate.get(i), i)
            }
        }

        ArrayList.metaClass.collectMany { Closure closure ->
            def result = []
            delegate.each { item ->
                def collected = closure.call(item)
                if (collected instanceof Collection) {
                    result.addAll(collected)
                } else {
                    result.add(collected)
                }
            }
            result
        }

        ArrayList.metaClass.groupBy { Closure closure ->
            def groups = [:].withDefault { [] }
            delegate.each { item ->
                def key = closure.call(item)
                groups[key] << item
            }
            groups
        }
    }

    // 2. Java集合使用Groovy闭包
    static void demonstrateClosureWithJavaCollections() {
        def javaList = new ArrayList<String>()
        javaList.add("Apple")
        javaList.add("Banana")
        javaList.add("Cherry")

        // 使用Groovy闭包处理Java集合
        javaList.each { println it.toUpperCase() }

        def upperList = javaList.collect { it.toUpperCase() }
        def filteredList = javaList.findAll { it.length() > 5 }

        // 使用inject进行聚合
        def concatenated = javaList.inject("") { acc, item -> acc + item }
    }

    // 3. Java集合的流式操作
    static void demonstrateStreamOperations() {
        def javaList = Arrays.asList(1, 2, 3, 4, 5)

        // Java Stream + Groovy
        def result = javaList.stream()
            .filter { it % 2 == 0 }
            .map { it * 2 }
            .collect(Collectors.toList())

        // 结合使用
        def combined = javaList.stream()
            .filter { it > 2 }
            .collect()
            .collectMany { [it, it * 2] }
    }
}
```

### 集合性能优化

```groovy
class CollectionPerformanceOptimization {

    // 1. 集合选择策略
    static class CollectionSelector {
        static List selectOptimalList(int expectedSize) {
            if (expectedSize < 10) {
                return new ArrayList(expectedSize)
            } else if (expectedSize < 1000) {
                return new ArrayList(expectedSize)
            } else {
                return new ArrayList(expectedSize)
            }
        }

        static Set selectOptimalSet(int expectedSize) {
            if (expectedSize < 100) {
                return new HashSet(expectedSize)
            } else {
                return new HashSet(expectedSize)
            }
        }

        static Map selectOptimalMap(int expectedSize) {
            if (expectedSize < 100) {
                return new HashMap(expectedSize)
            } else {
                return new HashMap(expectedSize)
            }
        }
    }

    // 2. 批量操作优化
    static class BatchOperationOptimizer {
        static void batchAdd(List target, Collection source) {
            if (target instanceof ArrayList) {
                target.addAll(source)
            } else {
                source.each { target.add(it) }
            }
        }

        static void batchProcess(Collection data, Closure processor) {
            if (data.size() > 1000) {
                // 分批处理
                def batchSize = 1000
                def batches = data.collate(batchSize)
                batches.each { batch ->
                    batch.each(processor)
                }
            } else {
                data.each(processor)
            }
        }
    }

    // 3. 内存使用优化
    static class MemoryOptimizer {
        static def optimizeCollectionUsage(Collection collection) {
            if (collection instanceof ArrayList) {
                collection.trimToSize()
            }
            collection
        }

        static def createSizedCollection(int size) {
            def list = new ArrayList(size)
            for (int i = 0; i < size; i++) {
                list.add(null)
            }
            list
        }
    }
}
```

### 集合转换工具

```groovy
class CollectionConversionUtils {

    // 1. 深度转换
    static Map deepConvertToJavaMap(Map groovyMap) {
        def result = new HashMap()
        groovyMap.each { key, value ->
            result.put(key, deepConvertToJava(value))
        }
        result
    }

    static List deepConvertToJavaList(List groovyList) {
        def result = new ArrayList()
        groovyList.each { item ->
            result.add(deepConvertToJava(item))
        }
        result
    }

    static Object deepConvertToJava(Object obj) {
        if (obj instanceof Map) {
            return deepConvertToJavaMap(obj)
        } else if (obj instanceof List) {
            return deepConvertToJavaList(obj)
        } else if (obj instanceof Set) {
            return new HashSet(deepConvertToJavaList(obj as List))
        } else {
            return obj
        }
    }

    // 2. 类型安全的集合转换
    static <T> List<T> toTypedList(Collection collection, Class<T> type) {
        collection.collect { it as T }
    }

    static <K, V> Map<K, V> toTypedMap(Map map, Class<K> keyType, Class<V> valueType) {
        def result = new HashMap<K, V>()
        map.each { key, value ->
            result.put(key as K, value as V)
        }
        result
    }

    // 3. 集合工厂方法
    static class CollectionFactory {
        static <T> ArrayList<T> newArrayList(T... items) {
            def list = new ArrayList<T>()
            items.each { list.add(it) }
            list
        }

        static <T> HashSet<T> newHashSet(T... items) {
            def set = new HashSet<T>()
            items.each { set.add(it) }
            set
        }

        static <K, V> HashMap<K, V> newHashMap(Map<K, V> map) {
            def result = new HashMap<K, V>()
            result.putAll(map)
            result
        }
    }
}
```

## 异常处理机制

### 异常类型兼容性

```groovy
class ExceptionCompatibility {

    // 1. 异常继承体系
    static void demonstrateExceptionHierarchy() {
        try {
            // Java异常
            throw new IOException("Java IO Exception")
        } catch (IOException e) {
            println "Caught IOException: ${e.message}"
        }

        try {
            // Groovy异常
            throw new MissingPropertyException("Groovy Exception")
        } catch (Exception e) {
            println "Caught Exception: ${e.message}"
        }

        try {
            // 混合异常处理
            def operation = { ->
                if (Math.random() > 0.5) {
                    throw new IOException("IO Error")
                } else {
                    throw new MissingMethodException("Method Error", String, [])
                }
            }
            operation()
        } catch (IOException | MissingMethodException e) {
            println "Caught exception: ${e.message}"
        }
    }

    // 2. 自定义异常互操作
    static class CustomGroovyException extends Exception {
        CustomGroovyException(String message) {
            super(message)
        }

        CustomGroovyException(String message, Throwable cause) {
            super(message, cause)
        }
    }

    static class JavaExceptionHandler {
        void handleException(Exception e) {
            if (e instanceof CustomGroovyException) {
                println "Handling Groovy exception: ${e.message}"
            } else {
                println "Handling Java exception: ${e.message}"
            }
        }
    }

    // 3. 异常链处理
    static void demonstrateExceptionChaining() {
        try {
            try {
                throw new IOException("Original IO Error")
            } catch (IOException e) {
                throw new CustomGroovyException("Wrapped exception", e)
            }
        } catch (CustomGroovyException e) {
            println "Chained exception: ${e.message}"
            println "Cause: ${e.cause?.message}"
        }
    }
}
```

### 异常处理策略

```groovy
class ExceptionHandlingStrategy {

    // 1. 多重异常捕获
    static void demonstrateMultiCatch() {
        try {
            def operation = { ->
                def random = Math.random()
                if (random < 0.3) {
                    throw new IOException("IO Error")
                } else if (random < 0.6) {
                    throw new IllegalArgumentException("Argument Error")
                } else {
                    throw new RuntimeException("Runtime Error")
                }
            }
            operation()
        } catch (IOException e) {
            println "IO Exception handled: ${e.message}"
        } catch (IllegalArgumentException e) {
            println "Illegal Argument Exception handled: ${e.message}"
        } catch (RuntimeException e) {
            println "Runtime Exception handled: ${e.message}"
        }
    }

    // 2. 异常恢复策略
    static class ExceptionRecovery {
        static def withRecovery(Closure operation, Closure recovery) {
            try {
                operation.call()
            } catch (Exception e) {
                println "Exception occurred: ${e.message}"
                recovery.call(e)
            }
        }

        static def withRetry(Closure operation, int maxRetries = 3) {
            int attempts = 0
            while (attempts < maxRetries) {
                try {
                    return operation.call()
                } catch (Exception e) {
                    attempts++
                    if (attempts == maxRetries) {
                        throw e
                    }
                    println "Attempt $attempts failed, retrying..."
                    Thread.sleep(1000)
                }
            }
        }
    }

    // 3. 异常转换
    static class ExceptionTranslator {
        static Exception translateToGroovyException(Exception e) {
            if (e instanceof IOException) {
                return new CustomGroovyException("IO operation failed", e)
            } else if (e instanceof IllegalArgumentException) {
                return new CustomGroovyException("Invalid argument", e)
            } else {
                return new CustomGroovyException("Unexpected error", e)
            }
        }

        static Exception translateToJavaException(Exception e) {
            if (e instanceof MissingMethodException) {
                return new IllegalArgumentException("Method not found", e)
            } else if (e instanceof MissingPropertyException) {
                return new IllegalArgumentException("Property not found", e)
            } else {
                return new RuntimeException("Groovy error", e)
            }
        }
    }
}
```

### 异常监控与日志

```groovy
class ExceptionMonitoring {

    // 1. 异常监控器
    static class ExceptionMonitor {
        static Map<String, Integer> exceptionCounts = [:]
        static Map<String, List<Exception>> exceptionHistory = [:].withDefault { [] }

        static void monitor(Closure operation) {
            try {
                operation.call()
            } catch (Exception e) {
                def exceptionType = e.getClass().simpleName
                exceptionCounts[exceptionType] = (exceptionCounts[exceptionType] ?: 0) + 1
                exceptionHistory[exceptionType] << e
                throw e
            }
        }

        static void printStatistics() {
            println "Exception Statistics:"
            exceptionCounts.each { type, count ->
                println "  $type: $count occurrences"
            }
        }
    }

    // 2. 异常日志记录
    static class ExceptionLogger {
        static void logException(Exception e, String context = "") {
            def timestamp = new Date().format("yyyy-MM-dd HH:mm:ss")
            def logEntry = "[$timestamp] $context - ${e.class.simpleName}: ${e.message}"

            // 记录堆栈跟踪
            def stackTrace = e.stackTrace.take(5).join("\n")
            def fullLog = logEntry + "\n" + stackTrace

            println fullLog

            // 写入日志文件
            new File("exceptions.log").append(fullLog + "\n\n")
        }

        static void logWithDetails(Exception e, Map additionalInfo = [:]) {
            def details = [
                timestamp: new Date().format("yyyy-MM-dd HH:mm:ss.SSS"),
                exception: e.class.name,
                message: e.message,
                cause: e.cause?.message,
                additional: additionalInfo
            ]

            def json = new groovy.json.JsonBuilder(details).toString()
            new File("exceptions.json").append(json + "\n")
        }
    }

    // 3. 异常通知
    static class ExceptionNotifier {
        static void notifyAdmin(Exception e, String context) {
            def message = """
                Exception Alert!
                Context: $context
                Exception: ${e.class.simpleName}
                Message: ${e.message}
                Time: ${new Date()}
            """.stripIndent()

            // 发送邮件通知
            sendEmail("admin@example.com", "Exception Alert", message)

            // 发送Slack通知
            sendSlackNotification(message)
        }

        static void sendEmail(String to, String subject, String body) {
            // 实现邮件发送逻辑
            println "Email sent to $to: $subject"
        }

        static void sendSlackNotification(String message) {
            // 实现Slack通知逻辑
            println "Slack notification: $message"
        }
    }
}
```

## 注解处理

### 注解基础互操作

```groovy
class AnnotationInterop {

    // 1. Java注解在Groovy中使用
    @Deprecated
    @SuppressWarnings("unused")
    static void demonstrateJavaAnnotations() {
        println "This method uses Java annotations"
    }

    // 2. Groovy注解
    @Grab('org.apache.commons:commons-lang3:3.12.0')
    @CompileStatic
    static void demonstrateGroovyAnnotations() {
        def lang = org.apache.commons.lang3.StringUtils
        println lang.upperCase("hello from commons-lang3")
    }

    // 3. 自定义注解
    @Retention(RetentionPolicy.RUNTIME)
    @Target(ElementType.METHOD)
    @interface MyAnnotation {
        String value() default ""
        int priority() default 0
    }

    @MyAnnotation(value = "Important Method", priority = 1)
    static void importantMethod() {
        println "This is an important method"
    }
}
```

### 注解处理器

```groovy
class AnnotationProcessor {

    // 1. 运行时注解处理
    static void processRuntimeAnnotations() {
        def clazz = AnnotationInterop.class

        // 处理方法注解
        clazz.methods.each { method ->
            def myAnnotation = method.getAnnotation(MyAnnotation)
            if (myAnnotation) {
                println "Method ${method.name} has annotation: ${myAnnotation.value()}"
                println "Priority: ${myAnnotation.priority()}"
            }

            def deprecated = method.getAnnotation(Deprecated)
            if (deprecated) {
                println "Method ${method.name} is deprecated"
            }
        }
    }

    // 2. 编译时注解处理
    @Retention(RetentionPolicy.SOURCE)
    @Target(ElementType.TYPE)
    @interface GenerateBuilder {
        String builderName() default ""
    }

    @GenerateBuilder(builderName = "PersonBuilder")
    static class Person {
        String name
        int age
    }

    // 3. 注解驱动的代码生成
    static class CodeGenerator {
        static void generateBuilder(Class<?> clazz) {
            def annotation = clazz.getAnnotation(GenerateBuilder)
            if (!annotation) return

            def builderName = annotation.builderName() ?: "${clazz.simpleName}Builder"
            def fields = clazz.declaredFields.findAll { !it.synthetic }

            def builderCode = """
                class ${builderName} {
                    ${fields.collect { field ->
                        "${field.type.simpleName} ${field.name}"
                    }.join('\n')}

                    ${fields.collect { field ->
                        """
                        ${builderName} with${field.name.capitalize()}(${field.type.simpleName} ${field.name}) {
                            this.${field.name} = ${field.name}
                            this
                        }
                        """.stripIndent()
                    }.join('\n')}

                    ${clazz.simpleName} build() {
                        new ${clazz.simpleName}(
                            ${fields.collect { "${it.name}: ${it.name}" }.join(', ')}
                        )
                    }
                }
            """.stripIndent()

            // 动态编译和加载
            def shell = new GroovyShell()
            def builderClass = shell.evaluate(builderCode)

            println "Generated builder class: $builderName"
        }
    }
}
```

### 高级注解应用

```groovy
class AdvancedAnnotationApplication {

    // 1. 组合注解
    @Retention(RetentionPolicy.RUNTIME)
    @Target(ElementType.METHOD)
    @interface ServiceEndpoint {
        String path()
        String method() default "GET"
    }

    @Retention(RetentionPolicy.RUNTIME)
    @Target(ElementType.METHOD)
    @interface AuthRequired {
        String[] roles() default []
    }

    @Retention(RetentionPolicy.RUNTIME)
    @Target(ElementType.METHOD)
    @interface RateLimited {
        int requests() default 100
        int perSeconds() default 60
    }

    @ServiceEndpoint(path = "/api/users", method = "POST")
    @AuthRequired(roles = ["ADMIN"])
    @RateLimited(requests = 10, perSeconds = 60)
    static void createUser() {
        println "Creating user..."
    }

    // 2. 注解驱动的AOP
    static class AnnotationAOP {
        static void around(Closure target) {
            def method = target.method
            def startTime = System.currentTimeMillis()

            try {
                // 前置处理
                handleBeforeAdvice(method)

                // 执行目标方法
                def result = target.call()

                // 后置处理
                handleAfterAdvice(method, result)

                return result
            } catch (Exception e) {
                // 异常处理
                handleExceptionAdvice(method, e)
                throw e
            } finally {
                // 最终处理
                def endTime = System.currentTimeMillis()
                handleFinallyAdvice(method, endTime - startTime)
            }
        }

        static void handleBeforeAdvice(Method method) {
            def logAnnotation = method.getAnnotation(LogAround)
            if (logAnnotation) {
                println "Entering ${method.name}"
            }
        }

        static void handleAfterAdvice(Method method, Object result) {
            def logAnnotation = method.getAnnotation(LogAround)
            if (logAnnotation) {
                println "Exiting ${method.name} with result: $result"
            }
        }

        static void handleExceptionAdvice(Method method, Exception e) {
            def logAnnotation = method.getAnnotation(LogAround)
            if (logAnnotation) {
                println "Exception in ${method.name}: ${e.message}"
            }
        }

        static void handleFinallyAdvice(Method method, long duration) {
            def logAnnotation = method.getAnnotation(LogAround)
            if (logAnnotation) {
                println "Method ${method.name} executed in ${duration}ms"
            }
        }
    }

    // 3. 注解与配置结合
    static class AnnotationConfiguration {
        static Map<String, Object> loadConfiguration(Class<?> configClass) {
            def config = [:]

            configClass.fields.each { field ->
                def valueAnnotation = field.getAnnotation(ConfigurationValue)
                if (valueAnnotation) {
                    def key = valueAnnotation.key() ?: field.name
                    def defaultValue = valueAnnotation.defaultValue()
                    def value = System.getProperty(key, defaultValue)

                    // 类型转换
                    if (field.type == Integer) {
                        value = value as Integer
                    } else if (field.type == Boolean) {
                        value = value as Boolean
                    }

                    config[key] = value
                }
            }

            config
        }
    }
}
```

## 反射与动态调用

### 反射基础操作

```groovy
class ReflectionInterop {

    // 1. 基本反射操作
    static void demonstrateBasicReflection() {
        def clazz = String.class

        // 获取类信息
        println "Class name: ${clazz.name}"
        println "Package: ${clazz.package}"
        println "Superclass: ${clazz.superclass}"

        // 获取字段
        def fields = clazz.declaredFields
        println "Fields: ${fields.collect { it.name }}"

        // 获取方法
        def methods = clazz.declaredMethods
        println "Methods: ${methods.collect { it.name }}"

        // 获取构造器
        def constructors = clazz.constructors
        println "Constructors: ${constructors.collect { it.parameterTypes.collect { it.simpleName } }}"
    }

    // 2. 动态创建对象
    static void demonstrateDynamicObjectCreation() {
        def clazz = String.class

        // 使用无参构造器
        def instance1 = clazz.newInstance()

        // 使用带参数的构造器
        def constructor = clazz.getConstructor(String.class)
        def instance2 = constructor.newInstance("Hello World")

        // 使用Groovy的反射
        def instance3 = "Test String"

        println "Instance1: $instance1"
        println "Instance2: $instance2"
        println "Instance3: $instance3"
    }

    // 3. 动态方法调用
    static void demonstrateDynamicMethodInvocation() {
        def str = "Hello World"

        // 反射调用方法
        def method = str.getClass().getMethod("toUpperCase")
        def result1 = method.invoke(str)

        // Groovy方式
        def result2 = str.toUpperCase()

        // 动态调用任意方法
        def methodName = "substring"
        def args = [0, 5] as Object[]
        def method2 = str.getClass().getMethod(methodName, int.class, int.class)
        def result3 = method2.invoke(str, args)

        println "Result1: $result1"
        println "Result2: $result2"
        println "Result3: $result3"
    }
}
```

### 高级反射技术

```groovy
class AdvancedReflection {

    // 1. 私有成员访问
    static void demonstratePrivateAccess() {
        def clazz = String.class

        // 访问私有字段
        def field = clazz.getDeclaredField("value")
        field.accessible = true
        def str = "Hello"
        def value = field.get(str)
        println "Private field value: $value"

        // 访问私有方法
        def method = clazz.getDeclaredMethod("indexOf", String.class, int.class)
        method.accessible = true
        def result = method.invoke(str, "l", 2)
        println "Private method result: $result"

        // 访问私有构造器
        def constructor = clazz.getDeclaredConstructor(byte[].class)
        constructor.accessible = true
        def bytes = [72, 101, 108, 108, 111] as byte[]
        def newStr = constructor.newInstance(bytes)
        println "Private constructor result: $newStr"
    }

    // 2. 反射性能优化
    static class ReflectionCache {
        static Map<String, Method> methodCache = [:]
        static Map<String, Field> fieldCache = [:]
        static Map<String, Constructor> constructorCache = [:]

        static Method getCachedMethod(Class clazz, String name, Class... paramTypes) {
            def key = "${clazz.name}.$name(${paramTypes.collect { it.name }.join(',')})"
            if (!methodCache.containsKey(key)) {
                methodCache[key] = clazz.getMethod(name, paramTypes)
            }
            methodCache[key]
        }

        static Field getCachedField(Class clazz, String name) {
            def key = "${clazz.name}.$name"
            if (!fieldCache.containsKey(key)) {
                def field = clazz.getDeclaredField(name)
                field.accessible = true
                fieldCache[key] = field
            }
            fieldCache[key]
        }

        static Object invokeCached(Object obj, String methodName, Object... args) {
            def paramTypes = args.collect { it.class }
            def method = getCachedMethod(obj.class, methodName, *paramTypes)
            method.invoke(obj, args)
        }
    }

    // 3. 动态代理
    static void demonstrateDynamicProxy() {
        def interface1 = { String name -> "Hello, $name" } as Runnable
        def interface2 = { int a, int b -> a + b } as Callable

        // 创建动态代理
        def handler = { Object proxy, Method method, Object[] args ->
            println "Proxy method called: ${method.name}"
            if (method.name == "run") {
                println "Runnable run method"
            } else if (method.name == "call") {
                println "Callable call method with args: ${args}"
                return args[0] as int + args[1] as int
            }
            null
        }

        def proxy = Proxy.newProxyInstance(
            ReflectionInterop.classLoader,
            [Runnable, Callable] as Class[],
            handler
        )

        // 使用代理
        if (proxy instanceof Runnable) {
            proxy.run()
        }

        if (proxy instanceof Callable) {
            def result = proxy.call(5, 3)
            println "Proxy result: $result"
        }
    }
}
```

### 元编程与反射结合

```groovy
class MetaprogrammingReflection {

    // 1. 动态添加方法
    static void demonstrateDynamicMethodAddition() {
        def clazz = String.class

        // 使用Groovy MetaClass动态添加方法
        String.metaClass.reverseWords = { ->
            delegate.split(' ').reverse().join(' ')
        }

        def str = "Hello World"
        println "Reversed words: ${str.reverseWords()}"

        // 动态添加静态方法
        String.metaClass.static.createRandom = { length ->
            def chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789'
            def random = new Random()
            (1..length).collect { chars[random.nextInt(chars.length())] }.join('')
        }

        println "Random string: ${String.createRandom(10)}"
    }

    // 2. 动态属性访问
    static void demonstrateDynamicPropertyAccess() {
        def obj = new Expando()

        // 动态添加属性
        obj.name = "Dynamic Object"
        obj.age = 25

        // 动态添加方法
        obj.greet = { "Hello, my name is $name and I'm $age years old" }

        println obj.name
        println obj.age
        println obj.greet()

        // 使用Groovy属性访问
        obj.metaClass.getAddress = { -> "123 Dynamic Street" }
        println obj.address
    }

    // 3. 方法拦截
    static void demonstrateMethodInterception() {
        def interceptor = { Object[] args ->
            println "Method called with args: $args"
            def result = delegate.call(*args)
            println "Method result: $result"
            result
        }

        def enhancedString = "Hello World"
        enhancedString.metaClass.invokeMethod = { String name, Object[] args ->
            if (name == "toUpperCase") {
                println "Intercepting toUpperCase"
                def result = delegate.toUpperCase()
                println "Result: $result"
                result
            } else {
                delegate.metaClass.getMetaMethod(name, *args).invoke(delegate, args)
            }
        }

        println enhancedString.toUpperCase()
        println enhancedString.length()
    }
}
```

## 性能优化技巧

### 互操作性能优化

```groovy
class InteropPerformanceOptimization {

    // 1. 静态编译优化
    @CompileStatic
    static class OptimizedInterop {
        static void processJavaCollections(List<String> javaList) {
            // 静态编译避免动态调度开销
            def result = new ArrayList<String>()
            javaList.each { item ->
                result.add(item.toUpperCase())
            }
            result
        }

        @CompileStatic
        static int calculateSum(List<Integer> numbers) {
            int sum = 0
            for (int num : numbers) {
                sum += num
            }
            sum
        }

        @CompileStatic
        static Map<String, Integer> processMap(Map<String, Integer> input) {
            def result = new HashMap<String, Integer>()
            input.each { key, value ->
                result.put(key.toUpperCase(), value * 2)
            }
            result
        }
    }

    // 2. 类型推断优化
    @TypeChecked
    static class TypeCheckedInterop {
        def processList(List<String> list) {
            // 类型检查避免运行时类型检查
            list.collect { it.length() }
        }

        def filterNumbers(List<Integer> numbers) {
            numbers.findAll { it > 0 }
        }

        def transformMap(Map<String, Integer> map) {
            map.collectEntries { key, value -> [key.toLowerCase(), value * 2] }
        }
    }

    // 3. 缓存优化
    static class CacheOptimizedInterop {
        static Map<Class, List<Method>> methodCache = [:]

        static List<Method> getMethodsCached(Class clazz) {
            if (!methodCache.containsKey(clazz)) {
                methodCache[clazz] = clazz.declaredMethods.toList()
            }
            methodCache[clazz]
        }

        static Method getMethodCached(Class clazz, String name, Class... paramTypes) {
            def methods = getMethodsCached(clazz)
            methods.find { it.name == name && it.parameterTypes == paramTypes }
        }

        static def invokeCached(Object obj, String methodName, Object... args) {
            def method = getMethodCached(obj.class, methodName, *args.collect { it.class })
            method?.invoke(obj, args)
        }
    }
}
```

### 内存管理优化

```groovy
class MemoryOptimization {

    // 1. 对象池优化
    static class ObjectPool<T> {
        private Queue<T> pool = new LinkedList<>()
        private Closure<T> factory

        ObjectPool(Closure<T> factory) {
            this.factory = factory
        }

        T borrowObject() {
            def obj = pool.poll()
            if (obj == null) {
                obj = factory.call()
            }
            obj
        }

        void returnObject(T obj) {
            pool.offer(obj)
        }

        void clear() {
            pool.clear()
        }

        int size() {
            pool.size()
        }
    }

    // 2. 集合内存优化
    static class CollectionMemoryOptimizer {
        static <T> List<T> createOptimizedList(int expectedSize) {
            def list = new ArrayList<T>(expectedSize)
            list
        }

        static <T> Set<T> createOptimizedSet(int expectedSize) {
            def set = new HashSet<T>(expectedSize)
            set
        }

        static <K, V> Map<K, V> createOptimizedMap(int expectedSize) {
            def map = new HashMap<K, V>(expectedSize)
            map
        }

        static void trimCollections(List... collections) {
            collections.each { collection ->
                if (collection instanceof ArrayList) {
                    collection.trimToSize()
                }
            }
        }
    }

    // 3. 弱引用优化
    static class WeakReferenceCache<K, V> {
        private Map<K, WeakReference<V>> cache = [:]

        void put(K key, V value) {
            cache[key] = new WeakReference<V>(value)
        }

        V get(K key) {
            def ref = cache[key]
            ref?.get()
        }

        void cleanup() {
            def iterator = cache.entrySet().iterator()
            while (iterator.hasNext()) {
                def entry = iterator.next()
                if (entry.value.get() == null) {
                    iterator.remove()
                }
            }
        }

        int size() {
            cache.size()
        }
    }
}
```

### 并发优化

```groovy
class ConcurrentOptimization {

    // 1. 线程安全集合
    static class ThreadSafeCollections {
        static <T> List<T> synchronizedList(List<T> list) {
            Collections.synchronizedList(list)
        }

        static <K, V> Map<K, V> synchronizedMap(Map<K, V> map) {
            Collections.synchronizedMap(map)
        }

        static <T> Set<T> synchronizedSet(Set<T> set) {
            Collections.synchronizedSet(set)
        }

        static <T> List<T> concurrentList() {
            new CopyOnWriteArrayList<T>()
        }

        static <K, V> Map<K, V> concurrentMap() {
            new ConcurrentHashMap<K, V>()
        }

        static <T> Set<T> concurrentSet() {
            Collections.newSetFromMap(new ConcurrentHashMap<T, Boolean>())
        }
    }

    // 2. 并行处理优化
    static class ParallelProcessor {
        static <T, R> List<R> parallelProcess(List<T> data, Closure<R> processor) {
            if (data.size() < 1000) {
                return data.collect(processor)
            }

            def cores = Runtime.runtime.availableProcessors()
            def batchSize = Math.max(data.size() / cores, 1000) as int
            def batches = data.collate(batchSize)

            def futures = batches.collect { batch ->
                GParsPool.withPool {
                    batch.collectParallel(processor)
                }
            }

            futures.flatten()
        }

        static <T> void parallelForEach(List<T> data, Closure processor) {
            if (data.size() < 1000) {
                data.each(processor)
                return
            }

            GParsPool.withPool {
                data.eachParallel(processor)
            }
        }
    }

    // 3. 锁优化
    static class LockOptimization {
        private final ReentrantReadWriteLock lock = new ReentrantReadWriteLock()

        def readOperation(Closure operation) {
            lock.readLock().lock()
            try {
                operation.call()
            } finally {
                lock.readLock().unlock()
            }
        }

        def writeOperation(Closure operation) {
            lock.writeLock().lock()
            try {
                operation.call()
            } finally {
                lock.writeLock().unlock()
            }
        }

        def tryWriteOperation(Closure operation, long timeout = 1000) {
            if (lock.writeLock().tryLock(timeout, TimeUnit.MILLISECONDS)) {
                try {
                    return operation.call()
                } finally {
                    lock.writeLock().unlock()
                }
            }
            throw new RuntimeException("Failed to acquire write lock")
        }
    }
}
```

## 高级互操作模式

### 桥接模式

```groovy
class BridgePattern {

    // 1. Java接口桥接
    interface JavaService {
        String processData(String input)
        void saveData(String data)
    }

    class JavaServiceImpl implements JavaService {
        String processData(String input) {
            "Processed: $input"
        }

        void saveData(String data) {
            println "Saving data: $data"
        }
    }

    // 2. Groovy适配器
    class GroovyServiceAdapter {
        private JavaService javaService

        GroovyServiceAdapter(JavaService javaService) {
            this.javaService = javaService
        }

        def processWithClosure(String input, Closure processor) {
            def processed = javaService.processData(input)
            processor(processed)
        }

        def saveWithValidation(String data, Closure validator) {
            if (validator(data)) {
                javaService.saveData(data)
            } else {
                throw new IllegalArgumentException("Invalid data: $data")
            }
        }
    }

    // 3. 动态桥接
    class DynamicBridge {
        static def createBridge(Object target) {
            def bridge = new Expando()

            // 动态添加方法
            target.class.methods.each { method ->
                if (method.parameterCount == 0) {
                    bridge."${method.name}" = { -> method.invoke(target) }
                } else if (method.parameterCount == 1) {
                    bridge."${method.name}" = { arg -> method.invoke(target, arg) }
                } else {
                    bridge."${method.name}" = { Object... args -> method.invoke(target, args) }
                }
            }

            // 添加Groovy特有方法
            bridge.with = { Closure closure ->
                closure.delegate = bridge
                closure.call()
            }

            bridge
        }
    }
}
```

### 装饰器模式

```groovy
class DecoratorPattern {

    // 1. Java装饰器
    interface JavaComponent {
        String operation()
    }

    class JavaComponentImpl implements JavaComponent {
        String operation() {
            "Basic Operation"
        }
    }

    class JavaDecorator implements JavaComponent {
        private JavaComponent component

        JavaDecorator(JavaComponent component) {
            this.component = component
        }

        String operation() {
            component.operation()
        }
    }

    // 2. Groovy装饰器
    class GroovyDecorator {
        private JavaComponent component

        GroovyDecorator(JavaComponent component) {
            this.component = component
        }

        String operation() {
            def base = component.operation()
            "Enhanced: $base"
        }

        String operationWithClosure(Closure enhancer) {
            def base = component.operation()
            enhancer(base)
        }

        def withLogging(Closure operation) {
            println "Before operation"
            def result = operation()
            println "After operation: $result"
            result
        }
    }

    // 3. 动态装饰器
    class DynamicDecorator {
        static def decorate(Object target, Map enhancements) {
            def decorated = new Expando()

            // 复制原始方法
            target.class.methods.each { method ->
                if (method.parameterCount == 0) {
                    decorated."${method.name}" = { -> method.invoke(target) }
                } else if (method.parameterCount == 1) {
                    decorated."${method.name}" = { arg -> method.invoke(target, arg) }
                } else {
                    decorated."${method.name}" = { Object... args -> method.invoke(target, args) }
                }
            }

            // 添加增强方法
            enhancements.each { methodName, closure ->
                decorated."${methodName}" = closure
            }

            decorated
        }
    }
}
```

### 策略模式

```groovy
class StrategyPattern {

    // 1. Java策略接口
    interface JavaStrategy {
        int execute(int a, int b)
    }

    class AddStrategy implements JavaStrategy {
        int execute(int a, int b) {
            a + b
        }
    }

    class MultiplyStrategy implements JavaStrategy {
        int execute(int a, int b) {
            a * b
        }
    }

    // 2. Groovy策略实现
    class GroovyStrategies {
        static def add = { a, b -> a + b }
        static def multiply = { a, b -> a * b }
        static def power = { a, b -> Math.pow(a, b) as int }
        static def max = { a, b -> Math.max(a, b) }
        static def min = { a, b -> Math.min(a, b) }
    }

    // 3. 策略上下文
    class StrategyContext {
        private JavaStrategy javaStrategy
        private Closure groovyStrategy

        StrategyContext(JavaStrategy strategy) {
            this.javaStrategy = strategy
        }

        StrategyContext(Closure strategy) {
            this.groovyStrategy = strategy
        }

        int execute(int a, int b) {
            if (javaStrategy) {
                return javaStrategy.execute(a, b)
            } else if (groovyStrategy) {
                return groovyStrategy(a, b)
            }
            throw new IllegalStateException("No strategy set")
        }

        void setStrategy(JavaStrategy strategy) {
            this.javaStrategy = strategy
            this.groovyStrategy = null
        }

        void setStrategy(Closure strategy) {
            this.groovyStrategy = strategy
            this.javaStrategy = null
        }
    }

    // 4. 动态策略选择
    class DynamicStrategySelector {
        static def selectStrategy(String operation) {
            switch (operation.toLowerCase()) {
                case "add":
                    return GroovyStrategies.add
                case "multiply":
                    return GroovyStrategies.multiply
                case "power":
                    return GroovyStrategies.power
                case "max":
                    return GroovyStrategies.max
                case "min":
                    return GroovyStrategies.min
                default:
                    throw new IllegalArgumentException("Unknown operation: $operation")
            }
        }

        static def executeWithStrategy(int a, int b, String operation) {
            def strategy = selectStrategy(operation)
            strategy(a, b)
        }
    }
}
```

## 实战案例

### Java库集成案例

```groovy
class JavaLibraryIntegration {

    // 1. Apache Commons集成
    @Grab('org.apache.commons:commons-lang3:3.12.0')
    @Grab('org.apache.commons:commons-collections4:4.4')
    @Grab('org.apache.commons:commons-io:1.3.2')

    static void demonstrateCommonsIntegration() {
        // Commons Lang
        def lang = org.apache.commons.lang3.StringUtils
        println "Abbreviation: ${lang.abbreviate("Hello World", 5)}"
        println "Is numeric: ${lang.isNumeric("12345")}"

        // Commons Collections
        def coll = org.apache.commons.collections4.CollectionUtils
        def list1 = [1, 2, 3]
        def list2 = [3, 4, 5]
        println "Intersection: ${coll.intersection(list1, list2)}"

        // Commons IO
        def io = org.apache.commons.io.FileUtils
        def tempFile = File.createTempFile("test", ".txt")
        io.writeStringToFile(tempFile, "Hello from Commons IO")
        println "Temp file content: ${io.readFileToString(tempFile)}"
    }

    // 2. Guava集成
    @Grab('com.google.guava:guava:31.0.1-jre')

    static void demonstrateGuavaIntegration() {
        // Guava Collections
        def immutableList = com.google.common.collect.ImmutableList.of(1, 2, 3)
        def immutableMap = com.google.common.collect.ImmutableMap.of("a", 1, "b", 2)

        println "Immutable list: $immutableList"
        println "Immutable map: $immutableMap"

        // Guava Cache
        def cache = com.google.common.cache.CacheBuilder.newBuilder()
            .maximumSize(1000)
            .expireAfterWrite(10, TimeUnit.MINUTES)
            .build()

        cache.put("key1", "value1")
        cache.put("key2", "value2")

        println "Cache value: ${cache.getIfPresent("key1")}"

        // Guava Preconditions
        def preconditions = com.google.common.base.Preconditions
        def value = preconditions.checkNotNull(null, "Value cannot be null")
    }

    // 3. Spring框架集成
    @Grab('org.springframework:spring-context:5.3.21')
    @Grab('org.springframework:spring-beans:5.3.21')

    static void demonstrateSpringIntegration() {
        // Spring Application Context
        def context = new org.springframework.context.annotation.AnnotationConfigApplicationContext()

        // Groovy Bean
        @groovy.transform.ToString
        class GroovyBean {
            String name
            int value

            String process() {
                "Processing $name with value $value"
            }
        }

        // 注册Groovy Bean
        context.beanFactory.registerSingleton("groovyBean", new GroovyBean(name: "Test", value: 42))

        // 获取Bean
        def bean = context.getBean("groovyBean")
        println "Bean: $bean"
        println "Bean process: ${bean.process()}"

        context.close()
    }
}
```

### 数据库集成案例

```groovy
class DatabaseIntegration {

    // 1. JDBC集成
    @Grab('com.h2database:h2:2.1.214')

    static void demonstrateJdbcIntegration() {
        def sql = groovy.sql.Sql.newInstance(
            "jdbc:h2:mem:testdb", "sa", "", "org.h2.Driver"
        )

        try {
            // 创建表
            sql.execute("""
                CREATE TABLE users (
                    id INT PRIMARY KEY AUTO_INCREMENT,
                    name VARCHAR(100),
                    email VARCHAR(100),
                    created_at TIMESTAMP
                )
            """)

            // 插入数据
            def insertSql = "INSERT INTO users (name, email, created_at) VALUES (?, ?, ?)"
            sql.executeInsert(insertSql, ["John Doe", "john@example.com", new Date()])
            sql.executeInsert(insertSql, ["Jane Smith", "jane@example.com", new Date()])

            // 查询数据
            def users = sql.rows("SELECT * FROM users")
            println "Users: $users"

            // 使用Groovy闭包处理结果
            sql.eachRow("SELECT * FROM users") { row ->
                println "User: ${row.name}, Email: ${row.email}"
            }

            // 参数化查询
            def user = sql.firstRow("SELECT * FROM users WHERE name = ?", ["John Doe"])
            println "Found user: $user"

        } finally {
            sql.close()
        }
    }

    // 2. JPA集成
    @Grab('org.hibernate:hibernate-core:5.6.9.Final')
    @Grab('com.h2database:h2:2.1.214')

    @Entity
    @Table(name = "products")
    static class Product {
        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        Long id

        @Column(name = "name")
        String name

        @Column(name = "price")
        BigDecimal price

        @Column(name = "description")
        String description

        @Column(name = "created_at")
        Date createdAt

        // Groovy属性
        def getFormattedPrice() {
            "\$${price}"
        }

        def getShortDescription() {
            description ? (description.length() > 50 ? description[0..47] + "..." : description) : ""
        }
    }

    static void demonstrateJPAIntegration() {
        def emf = Persistence.createEntityManagerFactory("test-pu")
        def em = emf.createEntityManager()

        try {
            // 开始事务
            em.transaction.begin()

            // 创建产品
            def product = new Product(
                name: "Laptop",
                price: 999.99,
                description: "High-performance laptop with SSD storage",
                createdAt: new Date()
            )

            em.persist(product)

            // 提交事务
            em.transaction.commit()

            // 查询产品
            def foundProduct = em.find(Product.class, product.id)
            println "Found product: $foundProduct"
            println "Formatted price: ${foundProduct.formattedPrice}"
            println "Short description: ${foundProduct.shortDescription}"

            // JPQL查询
            def query = em.createQuery("SELECT p FROM Product p WHERE p.price > :price", Product.class)
            query.setParameter("price", 500)
            def expensiveProducts = query.resultList

            println "Expensive products: $expensiveProducts"

        } finally {
            em.close()
            emf.close()
        }
    }

    // 3. MyBatis集成
    @Grab('org.mybatis:mybatis:3.5.10')
    @Grab('com.h2database:h2:2.1.214')

    static interface UserMapper {
        @Select("SELECT * FROM users WHERE id = #{id}")
        User findById(@Param("id") Long id)

        @Select("SELECT * FROM users WHERE name = #{name}")
        User findByName(@Param("name") String name)

        @Insert("INSERT INTO users (name, email, created_at) VALUES (#{name}, #{email}, #{createdAt})")
        @Options(useGeneratedKeys = true, keyProperty = "id")
        int insert(User user)

        @Update("UPDATE users SET name = #{name}, email = #{email} WHERE id = #{id}")
        int update(User user)

        @Delete("DELETE FROM users WHERE id = #{id}")
        int delete(Long id)
    }

    static void demonstrateMyBatisIntegration() {
        def dataSource = groovy.sql.Sql.newInstance(
            "jdbc:h2:mem:testdb", "sa", "", "org.h2.Driver"
        ).dataSource

        def sessionFactory = new org.apache.ibatis.session.SqlSessionFactoryBuilder().build(
            org.apache.ibatis.io.Resources.getResourceAsStream("mybatis-config.xml")
        )

        def session = sessionFactory.openSession()

        try {
            def mapper = session.getMapper(UserMapper)

            // 插入用户
            def user = new User(name: "Alice", email: "alice@example.com", createdAt: new Date())
            mapper.insert(user)

            // 查询用户
            def foundUser = mapper.findById(user.id)
            println "Found user: $foundUser"

            // 更新用户
            user.name = "Alice Updated"
            mapper.update(user)

            // 删除用户
            mapper.delete(user.id)

        } finally {
            session.close()
        }
    }
}
```

### 微服务集成案例

```groovy
class MicroserviceIntegration {

    // 1. REST客户端集成
    @Grab('org.apache.httpcomponents:httpclient:4.5.13')
    @Grab('com.fasterxml.jackson.core:jackson-databind:2.13.3')

    static void demonstrateRestClientIntegration() {
        def httpClient = org.apache.http.impl.client.HttpClients.createDefault()
        def objectMapper = new com.fasterxml.jackson.databind.ObjectMapper()

        try {
            // GET请求
            def getRequest = new org.apache.http.client.methods.HttpGet("https://jsonplaceholder.typicode.com/posts/1")
            def response = httpClient.execute(getRequest)

            if (response.statusLine.statusCode == 200) {
                def content = org.apache.http.util.EntityUtils.toString(response.entity)
                def post = objectMapper.readValue(content, Map.class)
                println "Post: $post"
            }

            // POST请求
            def postData = [
                title: "Groovy Post",
                body: "This is a post from Groovy",
                userId: 1
            ]

            def postRequest = new org.apache.http.client.methods.HttpPost("https://jsonplaceholder.typicode.com/posts")
            postRequest.setHeader("Content-Type", "application/json")
            postRequest.entity = new org.apache.http.entity.StringEntity(
                objectMapper.writeValueAsString(postData)
            )

            def postResponse = httpClient.execute(postRequest)
            if (postResponse.statusLine.statusCode == 201) {
                def responseContent = org.apache.http.util.EntityUtils.toString(postResponse.entity)
                def createdPost = objectMapper.readValue(responseContent, Map.class)
                println "Created post: $createdPost"
            }

        } finally {
            httpClient.close()
        }
    }

    // 2. Spring Boot集成
    @Grab('org.springframework.boot:spring-boot-starter-web:2.7.5')
    @Grab('org.springframework.boot:spring-boot-starter-actuator:2.7.5')

    @RestController
    @RequestMapping("/api")
    static class GroovyController {

        @GetMapping("/hello")
        def hello() {
            [message: "Hello from Groovy Controller!", timestamp: new Date()]
        }

        @PostMapping("/process")
        def process(@RequestBody Map data) {
            def result = [
                original: data,
                processed: data.collectEntries { key, value -> [key, value?.toString()?.toUpperCase()] },
                processedAt: new Date()
            ]
            result
        }

        @GetMapping("/users/{id}")
        def getUser(@PathVariable Long id) {
            // 模拟用户数据
            def users = [
                1: [id: 1, name: "John Doe", email: "john@example.com"],
                2: [id: 2, name: "Jane Smith", email: "jane@example.com"],
                3: [id: 3, name: "Bob Johnson", email: "bob@example.com"]
            ]

            def user = users[id]
            if (user) {
                user
            } else {
                throw new org.springframework.web.server.ResponseStatusException(
                    org.springframework.http.HttpStatus.NOT_FOUND, "User not found"
                )
            }
        }
    }

    @SpringBootApplication
    static class GroovyApplication {
        static void main(String[] args) {
            org.springframework.boot.SpringApplication.run(GroovyApplication, args)
        }
    }

    // 3. 消息队列集成
    @Grab('org.apache.activemq:activemq-broker:5.17.0')
    @Grab('org.springframework.boot:spring-boot-starter-activemq:2.7.5')

    @Service
    static class MessageProducer {
        @Autowired
        private JmsTemplate jmsTemplate

        void sendMessage(String destination, Object message) {
            jmsTemplate.convertAndSend(destination, message)
        }

        void sendUserMessage(User user) {
            def message = [
                type: "USER_CREATED",
                data: user,
                timestamp: new Date()
            ]
            sendMessage("user.queue", message)
        }
    }

    @Service
    static class MessageConsumer {
        @JmsListener(destination = "user.queue")
        void receiveUserMessage(Map message) {
            println "Received user message: $message"

            if (message.type == "USER_CREATED") {
                def user = message.data
                println "Processing user creation: ${user.name}"

                // 发送确认消息
                def confirmation = [
                    type: "USER_PROCESSED",
                    userId: user.id,
                    processedAt: new Date()
                ]
                jmsTemplate.convertAndSend("user.confirmation.queue", confirmation)
            }
        }
    }
}
```

## 最佳实践与注意事项

### 类型安全实践

```groovy
class TypeSafetyBestPractices {

    // 1. 使用@CompileStatic和@TypeChecked
    @CompileStatic
    static class TypeSafeService {
        List<String> processStrings(List<String> input) {
            input.collect { it.toUpperCase() }
        }

        Map<String, Integer> analyzeText(String text) {
            def words = text.split('\\s+')
            def wordCounts = [:]

            words.each { word ->
                wordCounts[word] = (wordCounts[word] ?: 0) + 1
            }

            wordCounts
        }
    }

    // 2. 类型安全集合操作
    @TypeChecked
    static class TypeSafeCollections {
        static <T> List<T> filterByType(Collection<?> collection, Class<T> type) {
            collection.findAll { it != null && type.isInstance(it) }.collect { type.cast(it) }
        }

        static <K, V> Map<K, V> ensureMapTypes(Map<?, ?> map, Class<K> keyType, Class<V> valueType) {
            def result = [:]
            map.each { key, value ->
                if (keyType.isInstance(key) && valueType.isInstance(value)) {
                    result[keyType.cast(key)] = valueType.cast(value)
                }
            }
            result
        }
    }

    // 3. 类型转换工具
    static class TypeConversionUtils {
        static <T> T safeCast(Object obj, Class<T> type) {
            if (obj == null) return null
            if (type.isInstance(obj)) return type.cast(obj)

            try {
                return obj as T
            } catch (Exception e) {
                throw new ClassCastException("Cannot cast ${obj.class} to $type")
            }
        }

        static <T> List<T> safeCastList(Collection<?> collection, Class<T> type) {
            collection.collect { safeCast(it, type) }
        }

        static <K, V> Map<K, V> safeCastMap(Map<?, ?> map, Class<K> keyType, Class<V> valueType) {
            map.collectEntries { key, value ->
                [(safeCast(key, keyType)): safeCast(value, valueType)]
            }
        }
    }
}
```

### 性能优化实践

```groovy
class PerformanceBestPractices {

    // 1. 缓存策略
    static class CacheManager {
        private static final Map<String, Object> cache = [:]
        private static final Map<String, Long> cacheTimestamps = [:]
        private static final long CACHE_TTL = 5 * 60 * 1000 // 5分钟

        static <T> T cached(String key, Closure<T> loader) {
            def now = System.currentTimeMillis()

            if (cache.containsKey(key)) {
                def timestamp = cacheTimestamps[key]
                if (now - timestamp < CACHE_TTL) {
                    return cache[key] as T
                }
            }

            def result = loader.call()
            cache[key] = result
            cacheTimestamps[key] = now

            return result
        }

        static void clearCache() {
            cache.clear()
            cacheTimestamps.clear()
        }

        static void clearExpiredCache() {
            def now = System.currentTimeMillis()
            def expiredKeys = cacheTimestamps.findAll { now - it.value > CACHE_TTL }.keySet()

            expiredKeys.each { key ->
                cache.remove(key)
                cacheTimestamps.remove(key)
            }
        }
    }

    // 2. 连接池管理
    static class ConnectionPoolManager {
        private static final Map<String, DataSource> connectionPools = [:]

        static DataSource getConnectionPool(String url, String username, String password) {
            if (!connectionPools.containsKey(url)) {
                def pool = new org.apache.commons.dbcp2.BasicDataSource()
                pool.driverClassName = "org.h2.Driver"
                pool.url = url
                pool.username = username
                pool.password = password
                pool.initialSize = 5
                pool.maxTotal = 20
                pool.maxIdle = 10
                pool.minIdle = 5

                connectionPools[url] = pool
            }

            connectionPools[url]
        }

        static void closeAllPools() {
            connectionPools.values().each { pool ->
                try {
                    pool.close()
                } catch (Exception e) {
                    println "Error closing connection pool: ${e.message}"
                }
            }
            connectionPools.clear()
        }
    }

    // 3. 批量处理优化
    static class BatchProcessor {
        static <T, R> List<R> processBatch(List<T> items, Closure<R> processor, int batchSize = 100) {
            def results = []
            def batches = items.collate(batchSize)

            batches.each { batch ->
                def batchResults = batch.collect(processor)
                results.addAll(batchResults)

                // 批次间暂停，避免资源耗尽
                Thread.sleep(100)
            }

            results
        }

        static <T> void processInParallel(List<T> items, Closure processor, int threadCount = 4) {
            def pool = Executors.newFixedThreadPool(threadCount)

            try {
                def futures = items.collect { item ->
                    pool.submit { processor(item) }
                }

                futures.each { future ->
                    try {
                        future.get()
                    } catch (Exception e) {
                        println "Error processing item: ${e.message}"
                    }
                }
            } finally {
                pool.shutdown()
            }
        }
    }
}
```

### 错误处理实践

```groovy
class ErrorHandlingBestPractices {

    // 1. 统一异常处理
    static class ExceptionHandler {
        static <T> T withExceptionHandling(Closure<T> operation, Closure<T> errorHandler) {
            try {
                operation.call()
            } catch (Exception e) {
                errorHandler.call(e)
            }
        }

        static <T> T withRetry(Closure<T> operation, int maxRetries = 3, long delayMs = 1000) {
            int attempt = 0
            while (attempt < maxRetries) {
                try {
                    return operation.call()
                } catch (Exception e) {
                    attempt++
                    if (attempt == maxRetries) {
                        throw e
                    }
                    Thread.sleep(delayMs)
                }
            }
        }

        static <T> T withTimeout(Closure<T> operation, long timeoutMs) {
            def future = Executors.newSingleThreadExecutor().submit(operation as Callable)
            try {
                return future.get(timeoutMs, TimeUnit.MILLISECONDS)
            } catch (TimeoutException e) {
                future.cancel(true)
                throw new RuntimeException("Operation timed out after ${timeoutMs}ms")
            } finally {
                future.cancel(true)
            }
        }
    }

    // 2. 日志记录
    static class LoggingUtils {
        static enum LogLevel {
            DEBUG, INFO, WARN, ERROR
        }

        static void log(LogLevel level, String message, Throwable throwable = null) {
            def timestamp = new Date().format("yyyy-MM-dd HH:mm:ss.SSS")
            def threadName = Thread.currentThread().name

            def logEntry = "[$timestamp] [$level] [$threadName] $message"

            if (throwable) {
                logEntry += "\n" + throwable.stackTrace.join("\n")
            }

            println logEntry

            // 写入日志文件
            new File("application.log").append(logEntry + "\n")
        }

        static void debug(String message) {
            log(LogLevel.DEBUG, message)
        }

        static void info(String message) {
            log(LogLevel.INFO, message)
        }

        static void warn(String message) {
            log(LogLevel.WARN, message)
        }

        static void error(String message, Throwable throwable = null) {
            log(LogLevel.ERROR, message, throwable)
        }
    }

    // 3. 资源管理
    static class ResourceManager {
        static <T> T withResource(Closure<T> resourceCreator, Closure<T> operation) {
            def resource = resourceCreator.call()
            try {
                operation.call(resource)
            } finally {
                if (resource instanceof AutoCloseable) {
                    try {
                        resource.close()
                    } catch (Exception e) {
                        LoggingUtils.error("Error closing resource", e)
                    }
                }
            }
        }

        static void withConnection(String url, String username, String password, Closure operation) {
            withResource({ DriverManager.getConnection(url, username, password) }, operation)
        }

        static void withFile(String filename, String mode, Closure operation) {
            withResource({ new File(filename) }, operation)
        }
    }
}
```

## 总结

Groovy与Java的互操作性是Groovy语言最强大的特性之一。通过深入理解和掌握本章介绍的各种互操作技巧，开发者可以：

1. **无缝集成**：在现有Java项目中引入Groovy，或反之亦然
2. **性能优化**：利用Groovy的简洁性和Java的性能优势
3. **生态扩展**：充分利用Java丰富的第三方库和工具
4. **渐进式迁移**：从Java逐步迁移到Groovy，降低风险

### 关键要点回顾

1. **类型转换**：掌握Groovy与Java之间的类型转换机制
2. **集合操作**：利用Groovy的集合方法增强Java集合
3. **异常处理**：统一处理两种语言的异常类型
4. **注解处理**：在Groovy中使用Java注解，反之亦然
5. **反射机制**：利用反射实现动态调用和元编程
6. **性能优化**：通过静态编译和缓存优化互操作性能
7. **设计模式**：在互操作场景中应用各种设计模式
8. **实战应用**：在实际项目中应用互操作技术

### 最佳实践建议

1. **类型安全**：在关键路径使用`@CompileStatic`和`@TypeChecked`
2. **性能监控**：监控互操作调用的性能影响
3. **错误处理**：建立统一的异常处理机制
4. **资源管理**：正确管理跨语言的资源生命周期
5. **测试覆盖**：确保互操作代码有充分的测试覆盖

通过合理应用这些技巧和最佳实践，开发者可以构建出既高效又灵活的Groovy-Java混合应用程序，充分发挥两种语言的优势。