# Groovy性能优化高级技巧：构建高性能Groovy应用的终极指南

## 引言

Groovy作为JVM上的动态语言，虽然提供了强大的动态特性和简洁的语法，但也需要特别关注性能优化。本文将深入探讨Groovy性能优化的各个方面，从编译优化到运行时调优，帮助你构建高性能的Groovy应用程序。

## 1. Groovy性能基础

### 1.1 Groovy与Java性能对比

```groovy
class PerformanceComparison {
    static void main(String[] args) {
        def iterations = 1000000

        // 测试Groovy闭包性能
        def groovyClosureTime = measureTime {
            def list = (1..iterations).collect { it * 2 }
        }

        // 测试Java Stream性能
        def javaStreamTime = measureTime {
            def list = (1..iterations).stream()
                .map { it * 2 }
                .collect()
        }

        // 测试传统for循环性能
        def forLoopTime = measureTime {
            def list = []
            for (int i = 1; i <= iterations; i++) {
                list.add(i * 2)
            }
        }

        println "Groovy闭包: ${groovyClosureTime}ms"
        println "Java Stream: ${javaStreamTime}ms"
        println "传统for循环: ${forLoopTime}ms"
    }

    static def measureTime(Closure closure) {
        def start = System.currentTimeMillis()
        closure()
        def end = System.currentTimeMillis()
        return end - start
    }
}
```

### 1.2 编译优化策略

```groovy
@CompileStatic
class CompileStaticExample {
    // 使用@CompileStatic注解进行静态编译
    def staticMethod() {
        def list = [1, 2, 3, 4, 5]
        return list.sum()
    }

    @TypeChecked
    def typeCheckedMethod(String input) {
        // 类型检查方法
        return input.toUpperCase()
    }
}

// 使用@CompileStatic优化性能
@CompileStatic
class OptimizedMath {
    static int fibonacci(int n) {
        if (n <= 1) return n
        return fibonacci(n - 1) + fibonacci(n - 2)
    }

    static boolean isPrime(int n) {
        if (n <= 1) return false
        if (n <= 3) return true
        if (n % 2 == 0 || n % 3 == 0) return false

        for (int i = 5; i * i <= n; i += 6) {
            if (n % i == 0 || n % (i + 2) == 0) return false
        }
        return true
    }
}
```

## 2. 内存优化技术

### 2.1 对象池优化

```groovy
class ObjectPool<T> {
    private final Queue<T> pool = new ConcurrentLinkedQueue<>()
    private final Closure<T> factory
    private final Closure<Void> reset
    private final int maxSize

    ObjectPool(Closure<T> factory, Closure<Void> reset = null, int maxSize = 100) {
        this.factory = factory
        this.reset = reset
        this.maxSize = maxSize
    }

    T borrowObject() {
        def obj = pool.poll()
        if (obj == null) {
            obj = factory()
        }
        return obj
    }

    void returnObject(T obj) {
        if (reset) {
            reset(obj)
        }
        if (pool.size() < maxSize) {
            pool.offer(obj)
        }
    }

    int size() {
        return pool.size()
    }
}

// 使用对象池优化字符串构建
class StringBuilderPool {
    private static final ObjectPool<StringBuilder> pool = new ObjectPool<>(
        { -> new StringBuilder() },
        { sb -> sb.setLength(0) },
        50
    )

    static String buildString(Closure<StringBuilder> builder) {
        def sb = pool.borrowObject()
        try {
            builder(sb)
            return sb.toString()
        } finally {
            pool.returnObject(sb)
        }
    }
}

// 使用示例
def result = StringBuilderPool.buildString { sb ->
    1000.times { i ->
        sb.append("Item $i\n")
    }
}
```

### 2.2 缓存优化

```groovy
class CacheManager<K, V> {
    private final Map<K, V> cache = new ConcurrentHashMap<>()
    private final Map<K, Long> timestamps = new ConcurrentHashMap<>()
    private final long expireTime
    private final int maxSize
    private final Closure<V> loader

    CacheManager(long expireTime, int maxSize, Closure<V> loader) {
        this.expireTime = expireTime
        this.maxSize = maxSize
        this.loader = loader
    }

    V get(K key) {
        def now = System.currentTimeMillis()
        def timestamp = timestamps.get(key)

        // 检查是否过期
        if (timestamp && now - timestamp > expireTime) {
            cache.remove(key)
            timestamps.remove(key)
        }

        // 从缓存获取或加载
        def value = cache.computeIfAbsent(key) { k ->
            def loadedValue = loader(k)
            timestamps.put(k, now)

            // 检查缓存大小
            if (cache.size() > maxSize) {
                evictExpiredEntries()
            }

            return loadedValue
        }

        return value
    }

    private void evictExpiredEntries() {
        def now = System.currentTimeMillis()
        def expiredKeys = timestamps.findAll { k, v -> now - v > expireTime }.keySet()

        expiredKeys.each { key ->
            cache.remove(key)
            timestamps.remove(key)
        }
    }

    void clear() {
        cache.clear()
        timestamps.clear()
    }

    int size() {
        return cache.size()
    }
}

// LRU缓存实现
class LRUCache<K, V> {
    private final Map<K, V> cache
    private final int capacity

    LRUCache(int capacity) {
        this.capacity = capacity
        this.cache = new LinkedHashMap<K, V>(capacity, 0.75f, true) {
            @Override
            protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
                return size() > capacity
            }
        }
    }

    V get(K key) {
        return cache.get(key)
    }

    void put(K key, V value) {
        cache.put(key, value)
    }

    int size() {
        return cache.size()
    }

    void clear() {
        cache.clear()
    }
}
```

## 3. 集合性能优化

### 3.1 集合选择优化

```groovy
@CompileStatic
class CollectionOptimization {
    // 1. 使用合适大小的集合
    static List<Integer> optimizedListCreation(int expectedSize) {
        // 预分配大小，避免扩容
        return new ArrayList<>(expectedSize)
    }

    // 2. 优化的Map操作
    static Map<String, Integer> optimizedMapOperations() {
        def map = new HashMap<String, Integer>(1000)

        // 批量操作
        (1..1000).each { i ->
            map.put("key_$i", i)
        }

        return map
    }

    // 3. 集合遍历优化
    static int optimizedIteration(List<Integer> list) {
        def sum = 0

        // 使用索引遍历（最快）
        for (int i = 0; i < list.size(); i++) {
            sum += list[i]
        }

        return sum
    }

    // 4. 并行集合处理
    static List<Integer> parallelProcessing(List<Integer> data) {
        GParsPool.withPool {
            return data.parallel.collect { it * 2 }
        }
    }

    // 5. 集合过滤优化
    static List<Integer> optimizedFiltering(List<Integer> data) {
        def result = new ArrayList<>(data.size() / 4) // 预估结果大小

        for (int i = 0; i < data.size(); i++) {
            if (data[i] % 2 == 0) {
                result.add(data[i])
            }
        }

        return result
    }
}
```

### 3.2 流式处理优化

```groovy
@CompileStatic
class StreamOptimization {
    // 1. 避免中间操作
    static List<String> optimizedStreamProcessing(List<String> strings) {
        return strings.stream()
            .filter { s -> s.length() > 5 }
            .map { s -> s.toUpperCase() }
            .collect(Collectors.toList())
    }

    // 2. 使用并行流
    static List<Integer> optimizedParallelStream(List<Integer> numbers) {
        return numbers.parallelStream()
            .filter { n -> n % 2 == 0 }
            .map { n -> n * n }
            .collect(Collectors.toList())
    }

    // 3. 自定义收集器
    static Map<Integer, List<String>> customCollector(List<String> strings) {
        return strings.stream()
            .collect(Collectors.groupingBy(
                { s -> s.length() },
                Collectors.mapping(
                    { s -> s.toUpperCase() },
                    Collectors.toList()
                )
            ))
    }

    // 4. 流操作的短路优化
    static Optional<String> findFirstMatch(List<String> strings, Predicate<String> predicate) {
        return strings.stream()
            .filter(predicate)
            .findFirst()
    }
}
```

## 4. 字符串处理优化

### 4.1 字符串构建优化

```groovy
@CompileStatic
class StringOptimization {
    // 1. 使用StringBuilder代替字符串连接
    static String buildStringWithBuilder(List<String> parts) {
        def builder = new StringBuilder(parts.sum { it.length() })
        parts.each { builder.append(it) }
        return builder.toString()
    }

    // 2. 预分配StringBuilder大小
    static String buildStringWithPreAllocation(List<String> parts) {
        def totalLength = parts.sum { it.length() }
        def builder = new StringBuilder(totalLength)
        parts.each { builder.append(it) }
        return builder.toString()
    }

    // 3. 字符串格式化优化
    static String optimizedFormatting(String template, Map<String, Object> values) {
        def result = template
        values.each { key, value ->
            result = result.replace("\${$key}", value.toString())
        }
        return result
    }

    // 4. 字符串池优化
    private static final Map<String, String> stringPool = new ConcurrentHashMap<>()

    static String internString(String str) {
        return stringPool.computeIfAbsent(str, { it })
    }

    // 5. 正则表达式优化
    private static final Pattern EMAIL_PATTERN = Pattern.compile(
        "^[A-Za-z0-9+_.-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,}\$"
    )

    static boolean isValidEmail(String email) {
        return EMAIL_PATTERN.matcher(email).matches()
    }
}
```

### 4.2 字符串处理模式

```groovy
@CompileStatic
class StringProcessingPatterns {
    // 1. 模板引擎优化
    static String renderTemplate(String template, Map<String, Object> context) {
        def engine = new groovy.text.SimpleTemplateEngine()
        return engine.createTemplate(template).make(context).toString()
    }

    // 2. 字符串分割优化
    static List<String> optimizedSplit(String str, String delimiter) {
        if (!str) return []
        return str.split(delimiter) as List<String>
    }

    // 3. 字符串连接优化
    static String optimizedJoin(List<String> strings, String delimiter) {
        if (!strings) return ""
        if (strings.size() == 1) return strings[0]

        def length = strings.sum { it.length() } + delimiter.length() * (strings.size() - 1)
        def builder = new StringBuilder(length)

        builder.append(strings[0])
        for (int i = 1; i < strings.size(); i++) {
            builder.append(delimiter)
            builder.append(strings[i])
        }

        return builder.toString()
    }

    // 4. 字符串查找优化
    static int optimizedIndexOf(String text, String pattern) {
        return text.indexOf(pattern)
    }

    // 5. 字符串替换优化
    static String optimizedReplace(String text, String target, String replacement) {
        return text.replace(target, replacement)
    }
}
```

## 5. 方法调用优化

### 5.1 方法缓存

```groovy
@CompileStatic
class MethodCache {
    private static final Map<String, Method> methodCache = new ConcurrentHashMap<>()

    static Method getCachedMethod(Class<?> clazz, String methodName, Class<?>... parameterTypes) {
        def key = "${clazz.name}.$methodName(${parameterTypes*.name.join(', ')})"

        return methodCache.computeIfAbsent(key) { k ->
            try {
                return clazz.getMethod(methodName, parameterTypes)
            } catch (NoSuchMethodException e) {
                throw new RuntimeException("Method not found: $k", e)
            }
        }
    }

    static Object invokeCachedMethod(Object target, String methodName, Object... args) {
        def parameterTypes = args.collect { it.class } as Class<?>[]
        def method = getCachedMethod(target.class, methodName, parameterTypes)
        return method.invoke(target, args)
    }
}

// 动态方法调用优化
class DynamicMethodInvoker {
    private final Map<String, Closure> methodCache = new ConcurrentHashMap<>()

    def invokeMethod(String name, Object... args) {
        def cacheKey = "$name(${args*.class*.name.join(', ')})"

        def closure = methodCache.computeIfAbsent(cacheKey) { key ->
            return { Object[] arguments ->
                return metaClass.invokeMethod(this, name, arguments)
            }
        }

        return closure.call(args)
    }
}
```

### 5.2 反射优化

```groovy
@CompileStatic
class ReflectionOptimization {
    private static final Map<Class<?>, Map<String, Field>> fieldCache = new ConcurrentHashMap<>()
    private static final Map<Class<?>, Map<String, Method>> methodCache = new ConcurrentHashMap<>()

    // 缓存字段访问
    static Field getCachedField(Class<?> clazz, String fieldName) {
        def classFields = fieldCache.computeIfAbsent(clazz) {
            def fields = new HashMap<String, Field>()
            clazz.declaredFields.each { field ->
                field.setAccessible(true)
                fields[field.name] = field
            }
            return fields
        }

        return classFields[fieldName]
    }

    // 优化的字段访问
    static Object getFieldValue(Object obj, String fieldName) {
        try {
            def field = getCachedField(obj.class, fieldName)
            return field.get(obj)
        } catch (Exception e) {
            throw new RuntimeException("Failed to access field: $fieldName", e)
        }
    }

    static void setFieldValue(Object obj, String fieldName, Object value) {
        try {
            def field = getCachedField(obj.class, fieldName)
            field.set(obj, value)
        } catch (Exception e) {
            throw new RuntimeException("Failed to set field: $fieldName", e)
        }
    }

    // 缓存方法调用
    static Method getCachedMethod(Class<?> clazz, String methodName, Class<?>... parameterTypes) {
        def classMethods = methodCache.computeIfAbsent(clazz) {
            def methods = new HashMap<String, Method>()
            clazz.methods.each { method ->
                def key = "${method.name}(${method.parameterTypes*.name.join(', ')})"
                methods[key] = method
            }
            return methods
        }

        def key = "$methodName(${parameterTypes*.name.join(', ')})"
        return classMethods[key]
    }
}
```

## 6. I/O性能优化

### 6.1 文件I/O优化

```groovy
@CompileStatic
class FileIOOptimization {
    // 1. 缓冲区优化
    static String readFileOptimized(String filePath) {
        def file = new File(filePath)
        def buffer = new byte[file.length() as int]

        try (def input = new FileInputStream(file)) {
            input.read(buffer)
        }

        return new String(buffer)
    }

    // 2. 批量读取
    static List<String> readLinesOptimized(String filePath) {
        def lines = new ArrayList<String>(1000)
        try (def reader = new BufferedReader(new FileReader(filePath))) {
            def line
            while ((line = reader.readLine()) != null) {
                lines.add(line)
            }
        }
        return lines
    }

    // 3. 内存映射文件
    static String readWithMemoryMapping(String filePath) {
        def file = new File(filePath)
        def channel = new RandomAccessFile(file, "r").channel

        try {
            def buffer = channel.map(
                FileChannel.MapMode.READ_ONLY,
                0,
                channel.size()
            )
            return Charset.defaultCharset().decode(buffer).toString()
        } finally {
            channel.close()
        }
    }

    // 4. 批量写入
    static void writeLinesOptimized(String filePath, List<String> lines) {
        try (def writer = new BufferedWriter(new FileWriter(filePath))) {
            lines.each { line ->
                writer.write(line)
                writer.newLine()
            }
        }
    }

    // 5. 并行文件处理
    static void processFilesInParallel(String directory, Closure processor) {
        def dir = new File(directory)
        def files = dir.listFiles(new FileFilter() {
            boolean accept(File pathname) {
                return pathname.isFile()
            }
        })

        GParsPool.withPool {
            files.parallel.each { file ->
                processor(file)
            }
        }
    }
}
```

### 6.2 网络I/O优化

```groovy
@CompileStatic
class NetworkIOOptimization {
    // 1. 连接池优化
    private static final HttpClient httpClient = HttpClient.newBuilder()
        .version(HttpClient.Version.HTTP_2)
        .connectTimeout(Duration.ofSeconds(10))
        .build()

    static HttpResponse<String> sendHttpRequest(String url) {
        def request = HttpRequest.newBuilder()
            .uri(URI.create(url))
            .timeout(Duration.ofSeconds(10))
            .build()

        return httpClient.send(request, HttpResponse.BodyHandlers.ofString())
    }

    // 2. 异步HTTP请求
    static CompletableFuture<HttpResponse<String>> sendAsyncRequest(String url) {
        def request = HttpRequest.newBuilder()
            .uri(URI.create(url))
            .timeout(Duration.ofSeconds(10))
            .build()

        return httpClient.sendAsync(request, HttpResponse.BodyHandlers.ofString())
    }

    // 3. 批量请求
    static List<HttpResponse<String>> sendBatchRequests(List<String> urls) {
        def futures = urls.collect { url ->
            sendAsyncRequest(url)
        }

        return futures.collect { it.get() }
    }

    // 4. WebSocket连接优化
    static WebSocketClient createWebSocketClient(String url) {
        return new WebSocketClient(URI.create(url)) {
            @Override
            void onOpen(ServerHandshake handshakedata) {
                println "WebSocket连接已打开"
            }

            @Override
            void onMessage(String message) {
                println "收到消息: $message"
            }

            @Override
            void onClose(int code, String reason, boolean remote) {
                println "WebSocket连接已关闭"
            }

            @Override
            void onError(Exception ex) {
                println "WebSocket错误: ${ex.message}"
            }
        }
    }
}
```

## 7. 数据库性能优化

### 7.1 连接池优化

```groovy
@CompileStatic
class DatabaseOptimization {
    // 1. 优化的连接池配置
    static DataSource createOptimizedDataSource(String url, String username, String password) {
        def dataSource = new HikariDataSource()
        dataSource.setJdbcUrl(url)
        dataSource.setUsername(username)
        dataSource.setPassword(password)
        dataSource.setMaximumPoolSize(20)
        dataSource.setMinimumIdle(5)
        dataSource.setConnectionTimeout(30000)
        dataSource.setIdleTimeout(600000)
        dataSource.setMaxLifetime(1800000)
        dataSource.setLeakDetectionThreshold(60000)
        return dataSource
    }

    // 2. 批量操作优化
    static void batchInsert(DataSource dataSource, List<Map<String, Object>> data) {
        def sql = "INSERT INTO users (name, email, age) VALUES (?, ?, ?)"

        try (def connection = dataSource.getConnection();
             def statement = connection.prepareStatement(sql)) {

            data.each { row ->
                statement.setString(1, row.name as String)
                statement.setString(2, row.email as String)
                statement.setInt(3, row.age as Integer)
                statement.addBatch()
            }

            statement.executeBatch()
        }
    }

    // 3. 预编译语句缓存
    private static final Map<String, PreparedStatement> statementCache = new ConcurrentHashMap<>()

    static PreparedStatement getCachedStatement(Connection connection, String sql) {
        return statementCache.computeIfAbsent(sql) { key ->
            connection.prepareStatement(key)
        }
    }

    // 4. 事务优化
    static void executeInTransaction(DataSource dataSource, Closure operation) {
        def connection = dataSource.connection
        def originalAutoCommit = connection.autoCommit

        try {
            connection.autoCommit = false
            operation(connection)
            connection.commit()
        } catch (Exception e) {
            connection.rollback()
            throw e
        } finally {
            connection.autoCommit = originalAutoCommit
            connection.close()
        }
    }
}
```

### 7.2 查询优化

```groovy
@CompileStatic
class QueryOptimization {
    // 1. 分页查询优化
    static List<Map<String, Object>> paginatedQuery(
        DataSource dataSource,
        String sql,
        int page,
        int pageSize
    ) {
        def offset = (page - 1) * pageSize
        def pagedSql = "$sql LIMIT ? OFFSET ?"

        try (def connection = dataSource.getConnection();
             def statement = connection.prepareStatement(pagedSql)) {

            statement.setInt(1, pageSize)
            statement.setInt(2, offset)

            def resultSet = statement.executeQuery()
            def results = []

            while (resultSet.next()) {
                def metaData = resultSet.metaData
                def row = [:]

                for (int i = 1; i <= metaData.columnCount; i++) {
                    row[metaData.getColumnName(i)] = resultSet.getObject(i)
                }

                results.add(row)
            }

            return results
        }
    }

    // 2. 查询结果缓存
    private static final Map<String, Tuple2<Long, List<Map<String, Object>>>> queryCache = new ConcurrentHashMap<>()
    private static final long CACHE_EXPIRY = 5 * 60 * 1000 // 5分钟

    static List<Map<String, Object>> cachedQuery(
        DataSource dataSource,
        String sql,
        Object[] params = []
    ) {
        def now = System.currentTimeMillis()
        def cacheKey = sql + params.hashCode()

        def cached = queryCache.get(cacheKey)
        if (cached && now - cached.v1 < CACHE_EXPIRY) {
            return cached.v2
        }

        def results = executeQuery(dataSource, sql, params)
        queryCache.put(cacheKey, new Tuple2(now, results))

        return results
    }

    private static List<Map<String, Object>> executeQuery(
        DataSource dataSource,
        String sql,
        Object[] params
    ) {
        try (def connection = dataSource.getConnection();
             def statement = connection.prepareStatement(sql)) {

            params.eachWithIndex { param, index ->
                statement.setObject(index + 1, param)
            }

            def resultSet = statement.executeQuery()
            def results = []

            while (resultSet.next()) {
                def metaData = resultSet.metaData
                def row = [:]

                for (int i = 1; i <= metaData.columnCount; i++) {
                    row[metaData.getColumnName(i)] = resultSet.getObject(i)
                }

                results.add(row)
            }

            return results
        }
    }
}
```

## 8. 算法优化

### 8.1 算法选择优化

```groovy
@CompileStatic
class AlgorithmOptimization {
    // 1. 快速排序优化
    static List<Integer> quickSort(List<Integer> list) {
        if (list.size() <= 1) return list

        def pivot = list[list.size() / 2 as int]
        def left = []
        def right = []
        def equal = []

        list.each { num ->
            if (num < pivot) left.add(num)
            else if (num > pivot) right.add(num)
            else equal.add(num)
        }

        return quickSort(left) + equal + quickSort(right)
    }

    // 2. 二分查找优化
    static int binarySearch(List<Integer> sortedList, int target) {
        def left = 0
        def right = sortedList.size() - 1

        while (left <= right) {
            def mid = left + (right - left) / 2 as int

            if (sortedList[mid] == target) {
                return mid
            } else if (sortedList[mid] < target) {
                left = mid + 1
            } else {
                right = mid - 1
            }
        }

        return -1
    }

    // 3. 动态规划优化
    static int longestCommonSubsequence(String text1, String text2) {
        def m = text1.length()
        def n = text2.length()
        def dp = new int[m + 1][n + 1]

        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {
                if (text1[i - 1] == text2[j - 1]) {
                    dp[i][j] = dp[i - 1][j - 1] + 1
                } else {
                    dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1])
                }
            }
        }

        return dp[m][n]
    }

    // 4. 图算法优化
    static List<Integer> dijkstra(int[][] graph, int start, int end) {
        def n = graph.length
        def distances = new int[n]
        def visited = new boolean[n]
        def previous = new int[n]

        Arrays.fill(distances, Integer.MAX_VALUE)
        Arrays.fill(previous, -1)
        distances[start] = 0

        for (int i = 0; i < n - 1; i++) {
            def minDistance = Integer.MAX_VALUE
            def minIndex = -1

            for (int j = 0; j < n; j++) {
                if (!visited[j] && distances[j] < minDistance) {
                    minDistance = distances[j]
                    minIndex = j
                }
            }

            if (minIndex == -1) break
            visited[minIndex] = true

            for (int j = 0; j < n; j++) {
                if (!visited[j] && graph[minIndex][j] > 0) {
                    def newDistance = distances[minIndex] + graph[minIndex][j]
                    if (newDistance < distances[j]) {
                        distances[j] = newDistance
                        previous[j] = minIndex
                    }
                }
            }
        }

        // 构建路径
        def path = []
        def current = end
        while (current != -1) {
            path.add(current)
            current = previous[current]
        }

        return path.reverse()
    }
}
```

### 8.2 数据结构优化

```groovy
@CompileStatic
class DataStructureOptimization {
    // 1. 优化的树结构
    static class TreeNode<T> {
        T value
        TreeNode<T> left
        TreeNode<T> right

        TreeNode(T value) {
            this.value = value
        }

        void insert(T value) {
            if (value < this.value) {
                if (left == null) {
                    left = new TreeNode(value)
                } else {
                    left.insert(value)
                }
            } else if (value > this.value) {
                if (right == null) {
                    right = new TreeNode(value)
                } else {
                    right.insert(value)
                }
            }
        }

        boolean contains(T value) {
            if (value == this.value) {
                return true
            } else if (value < this.value) {
                return left != null && left.contains(value)
            } else {
                return right != null && right.contains(value)
            }
        }
    }

    // 2. 优化的哈希表
    static class OptimizedHashMap<K, V> {
        private static final int DEFAULT_CAPACITY = 16
        private static final float LOAD_FACTOR = 0.75f

        private Entry<K, V>[] table
        private int size

        OptimizedHashMap() {
            this(DEFAULT_CAPACITY)
        }

        OptimizedHashMap(int capacity) {
            table = new Entry[capacity]
        }

        void put(K key, V value) {
            if (size >= table.length * LOAD_FACTOR) {
                resize()
            }

            def index = getIndex(key)
            def entry = table[index]

            if (entry == null) {
                table[index] = new Entry(key, value)
                size++
            } else {
                def current = entry
                while (current.next != null) {
                    if (current.key.equals(key)) {
                        current.value = value
                        return
                    }
                    current = current.next
                }
                current.next = new Entry(key, value)
                size++
            }
        }

        V get(K key) {
            def index = getIndex(key)
            def entry = table[index]

            while (entry != null) {
                if (entry.key.equals(key)) {
                    return entry.value
                }
                entry = entry.next
            }

            return null
        }

        private int getIndex(K key) {
            return (key.hashCode() & 0x7FFFFFFF) % table.length
        }

        private void resize() {
            def newTable = new Entry[table.length * 2]
            def oldTable = table
            table = newTable
            size = 0

            oldTable.each { entry ->
                while (entry != null) {
                    put(entry.key, entry.value)
                    entry = entry.next
                }
            }
        }

        private static class Entry<K, V> {
            K key
            V value
            Entry<K, V> next

            Entry(K key, V value) {
                this.key = key
                this.value = value
            }
        }
    }
}
```

## 9. 性能监控与分析

### 9.1 性能监控工具

```groovy
@CompileStatic
class PerformanceMonitor {
    private static final Map<String, List<Long>> methodStats = new ConcurrentHashMap<>()
    private static final Map<String, Long> executionTimes = new ConcurrentHashMap<>()

    static <T> T monitorMethod(String methodName, Closure<T> closure) {
        def startTime = System.nanoTime()
        try {
            return closure.call()
        } finally {
            def endTime = System.nanoTime()
            def duration = endTime - startTime

            methodStats.computeIfAbsent(methodName) { new ArrayList<>() }.add(duration)
            executionTimes[methodName] = (executionTimes[methodName] ?: 0) + duration
        }
    }

    static void printStats() {
        println "\n=== 性能统计 ==="
        methodStats.each { methodName, durations ->
            def count = durations.size()
            def total = durations.sum()
            def avg = total / count
            def min = durations.min()
            def max = durations.max()

            println "方法: $methodName"
            println "  调用次数: $count"
            println "  总时间: ${total / 1_000_000}ms"
            println "  平均时间: ${avg / 1_000_000}ms"
            println "  最小时间: ${min / 1_000_000}ms"
            println "  最大时间: ${max / 1_000_000}ms"
            println ""
        }
    }

    static void clearStats() {
        methodStats.clear()
        executionTimes.clear()
    }
}

// 性能分析器
class PerformanceProfiler {
    private final Map<String, List<Long>> profiles = new ConcurrentHashMap<>()
    private final AtomicBoolean enabled = new AtomicBoolean(false)

    void startProfiling() {
        enabled.set(true)
    }

    void stopProfiling() {
        enabled.set(false)
    }

    def profile(String name, Closure closure) {
        if (!enabled.get()) {
            return closure.call()
        }

        def startTime = System.nanoTime()
        try {
            return closure.call()
        } finally {
            def duration = System.nanoTime() - startTime
            profiles.computeIfAbsent(name) { new ArrayList<>() }.add(duration)
        }
    }

    void printProfile() {
        println "\n=== 性能分析结果 ==="
        profiles.each { name, durations ->
            def count = durations.size()
            def total = durations.sum()
            def avg = total / count
            def min = durations.min()
            def max = durations.max()
            def sorted = durations.sort()
            def median = sorted[count / 2 as int]
            def p95 = sorted[(count * 0.95) as int]
            def p99 = sorted[(count * 0.99) as int]

            println "操作: $name"
            println "  执行次数: $count"
            println "  总时间: ${total / 1_000_000}ms"
            println "  平均时间: ${avg / 1_000_000}ms"
            println "  中位数: ${median / 1_000_000}ms"
            println "  95分位: ${p95 / 1_000_000}ms"
            println "  99分位: ${p99 / 1_000_000}ms"
            println "  最小时间: ${min / 1_000_000}ms"
            println "  最大时间: ${max / 1_000_000}ms"
            println ""
        }
    }

    void clearProfiles() {
        profiles.clear()
    }
}
```

### 9.2 内存分析

```groovy
@CompileStatic
class MemoryAnalyzer {
    private static final Runtime runtime = Runtime.getRuntime()

    static void printMemoryInfo() {
        def totalMemory = runtime.totalMemory()
        def freeMemory = runtime.freeMemory()
        def usedMemory = totalMemory - freeMemory
        def maxMemory = runtime.maxMemory()

        println "\n=== 内存使用情况 ==="
        println "总内存: ${totalMemory / (1024 * 1024)}MB"
        println "已用内存: ${usedMemory / (1024 * 1024)}MB"
        println "空闲内存: ${freeMemory / (1024 * 1024)}MB"
        println "最大内存: ${maxMemory / (1024 * 1024)}MB"
        println "内存使用率: ${(usedMemory * 100 / maxMemory) as float}%"
    }

    static void forceGC() {
        println "强制执行垃圾回收..."
        System.gc()
        System.runFinalization()
        Thread.sleep(1000)
        printMemoryInfo()
    }

    static void analyzeObjectCreation() {
        def beforeMemory = runtime.totalMemory() - runtime.freeMemory()

        // 创建大量对象
        def objects = []
        100000.times {
            objects.add(new Object())
        }

        def afterMemory = runtime.totalMemory() - runtime.freeMemory()
        def objectSize = (afterMemory - beforeMemory) / 100000

        println "\n对象创建分析:"
        println "创建100,000个对象"
        println "内存增长: ${afterMemory - beforeMemory} bytes"
        println "平均对象大小: ${objectSize} bytes"

        // 清理
        objects.clear()
        System.gc()
    }
}
```

## 10. 最佳实践与性能调优策略

### 10.1 代码优化策略

```groovy
@CompileStatic
class PerformanceBestPractices {
    // 1. 避免在循环中创建对象
    static List<String> optimizeObjectCreation(List<String> inputs) {
        def result = new ArrayList<String>(inputs.size())
        def builder = new StringBuilder()

        inputs.each { input ->
            builder.setLength(0)
            builder.append("Processed: ").append(input)
            result.add(builder.toString())
        }

        return result
    }

    // 2. 使用基本类型而不是包装类型
    static int sumWithPrimitives(int[] numbers) {
        def sum = 0
        for (int i = 0; i < numbers.length; i++) {
            sum += numbers[i]
        }
        return sum
    }

    // 3. 避免不必要的装箱拆箱
    static Integer[] boxingExample(int[] numbers) {
        return numbers.collect { it } as Integer[]
    }

    // 4. 使用静态编译
    @CompileStatic
    static int staticCompiledMethod(int a, int b) {
        return a + b
    }

    // 5. 缓存重用对象
    private static final DateFormat DATE_FORMAT = new SimpleDateFormat("yyyy-MM-dd")

    static String formatDate(Date date) {
        synchronized (DATE_FORMAT) {
            return DATE_FORMAT.format(date)
        }
    }
}
```

### 10.2 性能调优清单

```groovy
class PerformanceTuningChecklist {
    static void runPerformanceAudit() {
        println "=== 性能调优检查清单 ==="

        // 1. 检查内存使用
        MemoryAnalyzer.printMemoryInfo()

        // 2. 检查线程状态
        checkThreadStatus()

        // 3. 检查连接池状态
        checkConnectionPool()

        // 4. 检查缓存效率
        checkCacheEfficiency()

        // 5. 检查GC情况
        checkGarbageCollection()
    }

    private static void checkThreadStatus() {
        println "\n=== 线程状态检查 ==="
        def threadCount = Thread.getAllStackTraces().size()
        def threadMXBean = ManagementFactory.threadMXBean

        println "活动线程数: $threadCount"
        println "峰值线程数: ${threadMXBean.peakThreadCount}"
        println "总启动线程数: ${threadMXBean.totalStartedThreadCount}"
        println "守护线程数: ${Thread.getAllStackTraces().count { it.key.daemon }}"
    }

    private static void checkConnectionPool() {
        println "\n=== 连接池检查 ==="
        // 这里需要根据实际的连接池实现来检查
        println "活动连接数: 需要实现"
        println "空闲连接数: 需要实现"
        println "等待连接数: 需要实现"
    }

    private static void checkCacheEfficiency() {
        println "\n=== 缓存效率检查 ==="
        // 这里需要根据实际的缓存实现来检查
        println "缓存命中率: 需要实现"
        println "缓存大小: 需要实现"
        println "缓存过期时间: 需要实现"
    }

    private static void checkGarbageCollection() {
        println "\n=== 垃圾回收检查 ==="
        def memoryMXBean = ManagementFactory.memoryMXBean

        println "堆内存使用: ${memoryMXBean.heapMemoryUsage.used / (1024 * 1024)}MB"
        println "堆内存最大: ${memoryMXBean.heapMemoryUsage.max / (1024 * 1024)}MB"
        println "非堆内存使用: ${memoryMXBean.nonHeapMemoryUsage.used / (1024 * 1024)}MB"

        def gcMXBeans = ManagementFactory.garbageCollectorMXBeans
        gcMXBeans.each { gcMXBean ->
            println "GC(${gcMXBean.name}): 收集次数=${gcMXBean.collectionCount}, 耗时=${gcMXBean.collectionTime}ms"
        }
    }
}
```

## 11. 总结

Groovy性能优化是一个系统工程，需要从多个维度进行考虑：

1. **编译优化**：使用@CompileStatic和@TypeChecked注解
2. **内存优化**：对象池、缓存、避免不必要的对象创建
3. **集合优化**：选择合适的数据结构，预分配大小
4. **I/O优化**：缓冲区、批量操作、连接池
5. **算法优化**：选择合适的算法和数据结构
6. **并发优化**：合理使用并发工具，避免锁竞争
7. **监控分析**：建立完善的性能监控体系

### 关键优化原则：

1. **测量优先**：在优化前先测量性能瓶颈
2. **针对性优化**：针对真正的性能问题进行优化
3. **保持可读性**：在性能和可读性之间找到平衡
4. **渐进式优化**：逐步优化，每次优化后测试验证
5. **考虑整体影响**：优化一个方面可能影响其他方面

通过本文的深入分析和实践指南，相信你已经掌握了Groovy性能优化的核心技术和最佳实践。在实际项目中，应该根据具体的应用场景和性能需求，选择合适的优化策略，并建立完善的性能监控体系。