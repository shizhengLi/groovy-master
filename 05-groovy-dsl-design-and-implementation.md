# Groovy DSL设计与实现：构建优雅领域特定语言的完整指南

## 引言

领域特定语言（DSL）是Groovy最强大的特性之一。通过DSL，我们可以为特定领域创建简洁、表达力强的语言，让业务代码更加清晰易懂。本文将深入探讨Groovy DSL的设计原理、实现技术和最佳实践，帮助你构建高质量的DSL。

## 1. DSL基础概念

### 1.1 DSL类型与特点

```groovy
// 外部DSL示例（类似SQL）
query {
    select "id", "name", "email"
    from "users"
    where "age > 18"
    orderBy "name"
    limit 10
}

// 内部DSL示例（类似Gradle）
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

// 流畅接口DSL
new StringBuilder()
    .append("Hello")
    .append(" ")
    .append("World")
    .toString()
```

### 1.2 DSL设计原则

```groovy
class DSLDesignPrinciples {
    // 1. 可读性优先
    def readableQuery = sql {
        SELECT("id", "name")
        FROM("users")
        WHERE("age > 18")
    }

    // 2. 领域特定
    def businessRule = businessRules {
        rule "用户年龄验证" {
            when { user -> user.age >= 18 }
            then { "用户已成年" }
        }
    }

    // 3. 最小化语法噪音
    def cleanConfig = config {
        database {
            url "jdbc:mysql://localhost:3306/mydb"
            username "admin"
            password "password"
        }
    }

    // 4. 类型安全
    def typedBuilder = person {
        name "张三"
        age 25
        email "zhangsan@example.com"
    }
}
```

## 2. 基础DSL实现技术

### 2.1 闭包与委托

```groovy
class ClosureAndDelegate {
    // 1. 基本闭包委托
    static class ConfigBuilder {
        def config = [:]

        def database(Closure closure) {
            def databaseConfig = new DatabaseConfig()
            closure.delegate = databaseConfig
            closure.resolveStrategy = Closure.DELEGATE_FIRST
            closure()
            config.database = databaseConfig.config
        }

        def server(Closure closure) {
            def serverConfig = new ServerConfig()
            closure.delegate = serverConfig
            closure.resolveStrategy = Closure.DELEGATE_FIRST
            closure()
            config.server = serverConfig.config
        }
    }

    static class DatabaseConfig {
        def config = [:]

        def url(String url) {
            config.url = url
        }

        def username(String username) {
            config.username = username
        }

        def password(String password) {
            config.password = password
        }
    }

    static class ServerConfig {
        def config = [:]

        def port(int port) {
            config.port = port
        }

        def host(String host) {
            config.host = host
        }
    }

    // 2. 使用Builder模式
    static def config = new ConfigBuilder().with {
        database {
            url "jdbc:mysql://localhost:3306/mydb"
            username "admin"
            password "password"
        }
        server {
            port 8080
            host "localhost"
        }
        config
    }

    // 3. 嵌套闭包
    static class NestedBuilder {
        def build(Closure closure) {
            closure.delegate = this
            closure.resolveStrategy = Closure.DELEGATE_FIRST
            closure()
        }

        def section(String name, Closure closure) {
            println "开始节: $name"
            def sectionBuilder = new SectionBuilder()
            closure.delegate = sectionBuilder
            closure.resolveStrategy = Closure.DELEGATE_FIRST
            closure()
            println "结束节: $name"
        }
    }

    static class SectionBuilder {
        def paragraph(String text) {
            println "  段落: $text"
        }

        def list(List<String> items) {
            items.each { item ->
                println "  - $item"
            }
        }
    }
}
```

### 2.2 方法缺失处理

```groovy
class MethodMissingDSL {
    // 1. 动态方法调用
    static class DynamicQueryBuilder {
        def query = [:]

        def methodMissing(String name, Object args) {
            if (args && args[0] instanceof Closure) {
                // 处理嵌套查询
                def subQuery = new DynamicQueryBuilder()
                args[0].delegate = subQuery
                args[0].resolveStrategy = Closure.DELEGATE_FIRST
                args[0]()
                query[name] = subQuery.query
            } else {
                // 处理简单属性
                query[name] = args ? args[0] : true
            }
        }

        def build() {
            return query
        }
    }

    // 2. 动态属性访问
    static class DynamicConfig {
        def config = [:]

        def propertyMissing(String name) {
            return config[name]
        }

        def propertyMissing(String name, Object value) {
            config[name] = value
        }

        def methodMissing(String name, Object args) {
            if (args && args[0] instanceof Closure) {
                def nestedConfig = new DynamicConfig()
                args[0].delegate = nestedConfig
                args[0].resolveStrategy = Closure.DELEGATE_FIRST
                args[0]()
                config[name] = nestedConfig.config
            } else {
                config[name] = args ? args[0] : true
            }
        }
    }

    // 3. 使用示例
    static def dynamicQuery = new DynamicQueryBuilder().with {
        select "id", "name", "email"
        from "users"
        where {
            age greaterThan 18
            status "active"
        }
        orderBy "name"
        limit 10
        build()
    }

    static def dynamicConfig = new DynamicConfig().with {
        database {
            url "jdbc:mysql://localhost:3306/mydb"
            username "admin"
            password "password"
        }
        server {
            port 8080
            host "localhost"
        }
        config
    }
}
```

## 3. 高级DSL构建技术

### 3.1 Builder模式进阶

```groovy
class AdvancedBuilderDSL {
    // 1. 类型安全的Builder
    static class PersonBuilder {
        private String name
        private Integer age
        private String email
        private AddressBuilder address

        PersonBuilder name(String name) {
            this.name = name
            return this
        }

        PersonBuilder age(int age) {
            this.age = age
            return this
        }

        PersonBuilder email(String email) {
            this.email = email
            return this
        }

        PersonBuilder address(@DelegatesTo(AddressBuilder) Closure closure) {
            this.address = new AddressBuilder()
            closure.delegate = address
            closure.resolveStrategy = Closure.DELEGATE_FIRST
            closure()
            return this
        }

        Person build() {
            if (!name) throw new IllegalStateException("Name is required")
            if (!age) throw new IllegalStateException("Age is required")

            return new Person(
                name: name,
                age: age,
                email: email,
                address: address?.build()
            )
        }
    }

    static class AddressBuilder {
        private String street
        private String city
        private String country

        AddressBuilder street(String street) {
            this.street = street
            return this
        }

        AddressBuilder city(String city) {
            this.city = city
            return this
        }

        AddressBuilder country(String country) {
            this.country = country
            return this
        }

        Address build() {
            return new Address(
                street: street,
                city: city,
                country: country
            )
        }
    }

    static class Person {
        String name
        int age
        String email
        Address address
    }

    static class Address {
        String street
        String city
        String country
    }

    // 2. 使用示例
    static def person = new PersonBuilder()
        .name("张三")
        .age(25)
        .email("zhangsan@example.com")
        .address {
            street "人民路123号"
            city "北京"
            country "中国"
        }
        .build()

    // 3. 集合Builder
    static class CollectionBuilder<T> {
        private List<T> items = []

        CollectionBuilder<T> add(T item) {
            items.add(item)
            return this
        }

        CollectionBuilder<T> addAll(Collection<T> items) {
            this.items.addAll(items)
            return this
        }

        CollectionBuilder<T> withEach(Closure<T> closure) {
            items.each(closure)
            return this
        }

        List<T> build() {
            return new ArrayList<>(items)
        }
    }
}
```

### 3.2 流畅接口设计

```groovy
class FluentInterfaceDSL {
    // 1. 链式调用
    static class QueryBuilder {
        private String selectClause = "*"
        private String fromClause
        private List<String> whereClauses = []
        private String orderByClause
        private Integer limitClause
        private Integer offsetClause

        QueryBuilder select(String... columns) {
            this.selectClause = columns.join(", ")
            return this
        }

        QueryBuilder from(String table) {
            this.fromClause = table
            return this
        }

        QueryBuilder where(String condition) {
            this.whereClauses.add(condition)
            return this
        }

        QueryBuilder and(String condition) {
            this.whereClauses.add("AND $condition")
            return this
        }

        QueryBuilder or(String condition) {
            this.whereClauses.add("OR $condition")
            return this
        }

        QueryBuilder orderBy(String column) {
            this.orderByClause = column
            return this
        }

        QueryBuilder limit(int limit) {
            this.limitClause = limit
            return this
        }

        QueryBuilder offset(int offset) {
            this.offsetClause = offset
            return this
        }

        String build() {
            def sql = new StringBuilder()
            sql.append("SELECT $selectClause FROM $fromClause")

            if (!whereClauses.isEmpty()) {
                sql.append(" WHERE ").append(whereClauses.join(" "))
            }

            if (orderByClause) {
                sql.append(" ORDER BY $orderByClause")
            }

            if (limitClause) {
                sql.append(" LIMIT $limitClause")
            }

            if (offsetClause) {
                sql.append(" OFFSET $offsetClause")
            }

            return sql.toString()
        }
    }

    // 2. 使用示例
    static def sql = new QueryBuilder()
        .select("id", "name", "email")
        .from("users")
        .where("age > 18")
        .and("status = 'active'")
        .orderBy("name")
        .limit(10)
        .build()

    // 3. 状态验证
    static class ValidatorBuilder {
        private Object target
        private List<String> errors = []

        ValidatorBuilder(Object target) {
            this.target = target
        }

        ValidatorBuilder isNotNull(String fieldName) {
            def value = target."$fieldName"
            if (value == null) {
                errors.add("$fieldName 不能为空")
            }
            return this
        }

        ValidatorBuilder isNotEmpty(String fieldName) {
            def value = target."$fieldName"
            if (value == null || value.toString().isEmpty()) {
                errors.add("$fieldName 不能为空")
            }
            return this
        }

        ValidatorBuilder hasMinLength(String fieldName, int length) {
            def value = target."$fieldName"
            if (value == null || value.toString().length() < length) {
                errors.add("$fieldName 长度不能少于 $length 个字符")
            }
            return this
        }

        ValidatorBuilder matches(String fieldName, String pattern) {
            def value = target."$fieldName"
            if (value == null || !value.toString().matches(pattern)) {
                errors.add("$fieldName 格式不正确")
            }
            return this
        }

        List<String> validate() {
            return errors
        }

        boolean isValid() {
            return errors.isEmpty()
        }
    }
}
```

## 4. 领域特定DSL设计

### 4.1 数据库查询DSL

```groovy
class DatabaseQueryDSL {
    // 1. SQL构建器
    static class SQLBuilder {
        private String selectClause = "*"
        private String fromClause
        private List<String> whereClauses = []
        private List<String> joinClauses = []
        private List<String> groupByClauses = []
        private List<String> havingClauses = []
        private List<String> orderByClauses = []
        private Integer limit
        private Integer offset

        SQLBuilder select(String... columns) {
            this.selectClause = columns.join(", ")
            return this
        }

        SQLBuilder from(String table) {
            this.fromClause = table
            return this
        }

        SQLBuilder innerJoin(String table, String on) {
            joinClauses.add("INNER JOIN $table ON $on")
            return this
        }

        SQLBuilder leftJoin(String table, String on) {
            joinClauses.add("LEFT JOIN $table ON $on")
            return this
        }

        SQLBuilder where(String condition) {
            whereClauses.add(condition)
            return this
        }

        SQLBuilder groupBy(String... columns) {
            groupByClauses.addAll(columns)
            return this
        }

        SQLBuilder having(String condition) {
            havingClauses.add(condition)
            return this
        }

        SQLBuilder orderBy(String column, String direction = "ASC") {
            orderByClauses.add("$column $direction")
            return this
        }

        SQLBuilder limit(int limit) {
            this.limit = limit
            return this
        }

        SQLBuilder offset(int offset) {
            this.offset = offset
            return this
        }

        String build() {
            def sql = new StringBuilder()
            sql.append("SELECT $selectClause FROM $fromClause")

            joinClauses.each { sql.append(" $it") }

            if (!whereClauses.isEmpty()) {
                sql.append(" WHERE ").append(whereClauses.join(" AND "))
            }

            if (!groupByClauses.isEmpty()) {
                sql.append(" GROUP BY ").append(groupByClauses.join(", "))
            }

            if (!havingClauses.isEmpty()) {
                sql.append(" HAVING ").append(havingClauses.join(" AND "))
            }

            if (!orderByClauses.isEmpty()) {
                sql.append(" ORDER BY ").append(orderByClauses.join(", "))
            }

            if (limit != null) {
                sql.append(" LIMIT $limit")
            }

            if (offset != null) {
                sql.append(" OFFSET $offset")
            }

            return sql.toString()
        }

        Map<String, Object> buildWithParams() {
            def sql = build()
            def params = []

            // 简化的参数提取逻辑
            whereClauses.each { condition ->
                def matches = condition =~ /\?/
                params.addAll(Collections.nCopies(matches.count(), null))
            }

            return [sql: sql, params: params]
        }
    }

    // 2. 使用示例
    static def query = new SQLBuilder()
        .select("u.id", "u.name", "u.email", "o.order_count")
        .from("users u")
        .leftJoin("orders o", "u.id = o.user_id")
        .where("u.age > ?")
        .and("u.status = ?")
        .groupBy("u.id", "u.name", "u.email")
        .having("COUNT(o.id) > ?")
        .orderBy("u.name")
        .limit(10)
        .buildWithParams()

    // 3. 复杂查询DSL
    static class ComplexQueryBuilder {
        def queries = []
        def currentQuery

        ComplexQueryBuilder query(Closure closure) {
            currentQuery = new SQLBuilder()
            closure.delegate = currentQuery
            closure.resolveStrategy = Closure.DELEGATE_FIRST
            closure()
            queries.add(currentQuery)
            return this
        }

        List<String> buildAll() {
            return queries.collect { it.build() }
        }

        String union() {
            return queries.collect { it.build() }.join(" UNION ")
        }

        String unionAll() {
            return queries.collect { it.build() }.join(" UNION ALL ")
        }
    }
}
```

### 4.2 配置管理DSL

```groovy
class ConfigurationDSL {
    // 1. 配置构建器
    static class ConfigurationBuilder {
        def config = [:]

        def database(@DelegatesTo(DatabaseConfigBuilder) Closure closure) {
            def builder = new DatabaseConfigBuilder()
            closure.delegate = builder
            closure.resolveStrategy = Closure.DELEGATE_FIRST
            closure()
            config.database = builder.build()
        }

        def server(@DelegatesTo(ServerConfigBuilder) Closure closure) {
            def builder = new ServerConfigBuilder()
            closure.delegate = builder
            closure.resolveStrategy = Closure.DELEGATE_FIRST
            closure()
            config.server = builder.build()
        }

        def cache(@DelegatesTo(CacheConfigBuilder) Closure closure) {
            def builder = new CacheConfigBuilder()
            closure.delegate = builder
            closure.resolveStrategy = Closure.DELEGATE_FIRST
            closure()
            config.cache = builder.build()
        }

        def logging(@DelegatesTo(LoggingConfigBuilder) Closure closure) {
            def builder = new LoggingConfigBuilder()
            closure.delegate = builder
            closure.resolveStrategy = Closure.DELEGATE_FIRST
            closure()
            config.logging = builder.build()
        }

        Map build() {
            return config
        }
    }

    static class DatabaseConfigBuilder {
        def config = [:]

        def url(String url) {
            config.url = url
        }

        def username(String username) {
            config.username = username
        }

        def password(String password) {
            config.password = password
        }

        def driver(String driver) {
            config.driver = driver
        }

        def poolSize(int size) {
            config.poolSize = size
        }

        Map build() {
            return config
        }
    }

    static class ServerConfigBuilder {
        def config = [:]

        def port(int port) {
            config.port = port
        }

        def host(String host) {
            config.host = host
        }

        def contextPath(String path) {
            config.contextPath = path
        }

        def ssl(boolean enabled) {
            config.ssl = enabled
        }

        Map build() {
            return config
        }
    }

    static class CacheConfigBuilder {
        def config = [:]

        def type(String type) {
            config.type = type
        }

        def maxSize(int size) {
            config.maxSize = size
        }

        def expireAfterWrite(long seconds) {
            config.expireAfterWrite = seconds
        }

        def expireAfterAccess(long seconds) {
            config.expireAfterAccess = seconds
        }

        Map build() {
            return config
        }
    }

    static class LoggingConfigBuilder {
        def config = [:]

        def level(String level) {
            config.level = level
        }

        def pattern(String pattern) {
            config.pattern = pattern
        }

        def file(String file) {
            config.file = file
        }

        def maxSize(String size) {
            config.maxSize = size
        }

        def maxFiles(int files) {
            config.maxFiles = files
        }

        Map build() {
            return config
        }
    }

    // 2. 使用示例
    static def appConfig = new ConfigurationBuilder().with {
        database {
            url "jdbc:mysql://localhost:3306/myapp"
            username "appuser"
            password "apppassword"
            driver "com.mysql.jdbc.Driver"
            poolSize 20
        }

        server {
            port 8080
            host "0.0.0.0"
            contextPath "/app"
            ssl false
        }

        cache {
            type "redis"
            maxSize 10000
            expireAfterWrite 3600
            expireAfterAccess 1800
        }

        logging {
            level "INFO"
            pattern "%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n"
            file "logs/application.log"
            maxSize "50MB"
            maxFiles 10
        }

        build()
    }

    // 3. 环境特定配置
    static class EnvironmentConfigBuilder {
        def environments = [:]
        def currentEnv

        EnvironmentConfigBuilder environment(String name, Closure closure) {
            def envBuilder = new ConfigurationBuilder()
            closure.delegate = envBuilder
            closure.resolveStrategy = Closure.DELEGATE_FIRST
            closure()
            environments[name] = envBuilder.build()
            return this
        }

        EnvironmentConfigBuilder active(String env) {
            this.currentEnv = env
            return this
        }

        Map getActiveConfig() {
            return environments[currentEnv]
        }

        Map getAllConfigs() {
            return environments
        }
    }
}
```

## 5. 业务规则DSL

### 5.1 规则引擎DSL

```groovy
class BusinessRulesDSL {
    // 1. 规则引擎
    static class RuleEngine {
        def rules = []
        def facts = [:]

        RuleEngine rule(String name, Closure closure) {
            def rule = new Rule(name: name)
            closure.delegate = rule
            closure.resolveStrategy = Closure.DELEGATE_FIRST
            closure()
            rules.add(rule)
            return this
        }

        RuleEngine fact(String name, Object value) {
            facts[name] = value
            return this
        }

        RuleEngine facts(Map<String, Object> facts) {
            this.facts.putAll(facts)
            return this
        }

        List<RuleResult> evaluate() {
            def results = []
            rules.each { rule ->
                def result = rule.evaluate(facts)
                if (result.triggered) {
                    results.add(result)
                }
            }
            return results
        }

        List<RuleResult> evaluateFirstMatch() {
            rules.each { rule ->
                def result = rule.evaluate(facts)
                if (result.triggered) {
                    return [result]
                }
            }
            return []
        }
    }

    static class Rule {
        String name
        def conditions = []
        def actions = []

        Rule when(Closure condition) {
            conditions.add(condition)
            return this
        }

        Rule then(Closure action) {
            actions.add(action)
            return this
        }

        RuleResult evaluate(Map<String, Object> facts) {
            def triggered = conditions.every { condition ->
                condition.call(facts)
            }

            if (triggered) {
                def actionResults = []
                actions.each { action ->
                    def result = action.call(facts)
                    actionResults.add(result)
                }

                return new RuleResult(
                    name: name,
                    triggered: true,
                    actionResults: actionResults
                )
            }

            return new RuleResult(name: name, triggered: false)
        }
    }

    static class RuleResult {
        String name
        boolean triggered
        List actionResults = []

        String toString() {
            if (triggered) {
                return "规则 '$name' 触发，结果: $actionResults"
            } else {
                return "规则 '$name' 未触发"
            }
        }
    }

    // 2. 使用示例
    static def ruleEngine = new RuleEngine()
        .fact("userAge", 25)
        .fact("userScore", 750)
        .fact("isVIP", true)
        .fact("purchaseAmount", 1200)

    static def results = ruleEngine
        .rule("VIP用户折扣") {
            when { facts ->
                facts.isVIP == true && facts.purchaseAmount > 1000
            }
            then { facts ->
                def discount = facts.purchaseAmount * 0.15
                return "VIP用户享受15%折扣，节省: ¥$discount"
            }
        }
        .rule("高分用户优惠") {
            when { facts ->
                facts.userScore > 700 && facts.purchaseAmount > 500
            }
            then { facts ->
                def discount = facts.purchaseAmount * 0.10
                return "高分用户享受10%折扣，节省: ¥$discount"
            }
        }
        .rule("成年用户验证") {
            when { facts ->
                facts.userAge >= 18
            }
            then { facts ->
                return "用户已成年，可以购买"
            }
        }
        .evaluate()

    // 3. 复杂业务规则
    static class ComplexRuleEngine {
        def ruleSets = [:]

        ComplexRuleEngine ruleSet(String name, Closure closure) {
            def ruleSet = new RuleEngine()
            closure.delegate = ruleSet
            closure.resolveStrategy = Closure.DELEGATE_FIRST
            closure()
            ruleSets[name] = ruleSet
            return this
        }

        Map<String, List<RuleResult>> evaluateAll(Map<String, Object> facts) {
            def results = [:]
            ruleSets.each { name, ruleSet ->
                ruleSet.facts(facts)
                results[name] = ruleSet.evaluate()
            }
            return results
        }
    }
}
```

### 5.2 工作流DSL

```groovy
class WorkflowDSL {
    // 1. 工作流引擎
    static class WorkflowEngine {
        def name
        def states = [:]
        def transitions = []
        def currentState

        WorkflowEngine(String name) {
            this.name = name
        }

        WorkflowEngine state(String name, Closure closure) {
            def state = new WorkflowState(name: name)
            closure.delegate = state
            closure.resolveStrategy = Closure.DELEGATE_FIRST
            closure()
            states[name] = state

            if (!currentState) {
                currentState = name
            }

            return this
        }

        WorkflowEngine transition(String from, String to, Closure closure) {
            def transition = new WorkflowTransition(from: from, to: to)
            closure.delegate = transition
            closure.resolveStrategy = Closure.DELEGATE_FIRST
            closure()
            transitions.add(transition)
            return this
        }

        WorkflowEngine start() {
            println "工作流 '$name' 开始"
            println "初始状态: $currentState"
            return this
        }

        WorkflowEngine execute(String action, Map context = [:]) {
            def validTransitions = transitions.findAll {
                it.from == currentState && it.action == action
            }

            if (validTransitions.isEmpty()) {
                throw new IllegalStateException("从状态 '$currentState' 执行动作 '$action' 无效")
            }

            def transition = validTransitions.first()

            // 执行前置动作
            transition.beforeActions?.each { it.call(context) }

            // 执行转换
            def fromState = states[currentState]
            fromState.exitActions?.each { it.call(context) }

            currentState = transition.to
            println "状态转换: $transition.from -> $currentState"

            def toState = states[currentState]
            toState.entryActions?.each { it.call(context) }

            // 执行后置动作
            transition.afterActions?.each { it.call(context) }

            return this
        }

        String getCurrentState() {
            return currentState
        }
    }

    static class WorkflowState {
        String name
        def entryActions = []
        def exitActions = []

        WorkflowState onEntry(Closure action) {
            entryActions.add(action)
            return this
        }

        WorkflowState onExit(Closure action) {
            exitActions.add(action)
            return this
        }
    }

    static class WorkflowTransition {
        String from
        String to
        String action
        def beforeActions = []
        def afterActions = []

        WorkflowTransition action(String action) {
            this.action = action
            return this
        }

        WorkflowTransition before(Closure action) {
            beforeActions.add(action)
            return this
        }

        WorkflowTransition after(Closure action) {
            afterActions.add(action)
            return this
        }
    }

    // 2. 订单处理工作流示例
    static def orderWorkflow = new WorkflowEngine("订单处理")
        .state("待支付") {
            onEntry { context ->
                println "订单 ${context.orderId} 进入待支付状态"
                // 发送支付提醒
            }
        }
        .state("已支付") {
            onEntry { context ->
                println "订单 ${context.orderId} 支付完成"
                // 开始处理订单
            }
            onExit { context ->
                println "订单 ${context.orderId} 离开已支付状态"
            }
        }
        .state("已发货") {
            onEntry { context ->
                println "订单 ${context.orderId} 已发货"
                // 发送发货通知
            }
        }
        .state("已完成") {
            onEntry { context ->
                println "订单 ${context.orderId} 已完成"
                // 发送完成通知
            }
        }
        .state("已取消") {
            onEntry { context ->
                println "订单 ${context.orderId} 已取消"
                // 退款处理
            }
        }
        .transition("待支付", "已支付") {
            action "支付"
            before { context ->
                println "验证支付信息..."
            }
            after { context ->
                println "支付成功！"
            }
        }
        .transition("已支付", "已发货") {
            action "发货"
            before { context ->
                println "检查库存..."
            }
            after { context ->
                println "发货成功！"
            }
        }
        .transition("已发货", "已完成") {
            action "确认收货"
            after { context ->
                println "订单完成！"
            }
        }
        .transition("待支付", "已取消") {
            action "取消"
            before { context ->
                println "检查是否可以取消..."
            }
            after { context ->
                println "订单已取消！"
            }
        }
        .transition("已支付", "已取消") {
            action "退款"
            before { context ->
                println "处理退款..."
            }
            after { context ->
                println "退款完成！"
            }
        }

    // 3. 使用工作流
    static def executeOrderWorkflow() {
        def context = [orderId: "ORD-001"]

        orderWorkflow
            .start()
            .execute("支付", context)
            .execute("发货", context)
            .execute("确认收货", context)
    }
}
```

## 6. 测试DSL

### 6.1 单元测试DSL

```groovy
class TestingDSL {
    // 1. 测试构建器
    static class TestBuilder {
        def testName
        def setupActions = []
        def testActions = []
        def cleanupActions = []
        def expectedResults = []

        TestBuilder name(String name) {
            this.testName = name
            return this
        }

        TestBuilder setup(Closure action) {
            setupActions.add(action)
            return this
        }

        TestBuilder when(Closure action) {
            testActions.add(action)
            return this
        }

        TestBuilder then(Closure action) {
            expectedResults.add(action)
            return this
        }

        TestBuilder cleanup(Closure action) {
            cleanupActions.add(action)
            return this
        }

        void run() {
            println "运行测试: $testName"

            try {
                // 执行setup
                setupActions.each { action ->
                    action.call()
                }

                // 执行测试
                def results = testActions.collect { action ->
                    action.call()
                }

                // 验证结果
                expectedResults.eachWithIndex { validator, index ->
                    def result = results[index]
                    def isValid = validator.call(result)
                    assert isValid, "验证失败: 结果不匹配"
                }

                println "测试 '$testName' 通过"
            } catch (Exception e) {
                println "测试 '$testName' 失败: ${e.message}"
                throw e
            } finally {
                // 执行cleanup
                cleanupActions.each { action ->
                    try {
                        action.call()
                    } catch (Exception e) {
                        println "Cleanup执行失败: ${e.message}"
                    }
                }
            }
        }
    }

    // 2. 测试套件
    static class TestSuite {
        def tests = []
        def beforeActions = []
        def afterActions = []

        TestSuite beforeAll(Closure action) {
            beforeActions.add(action)
            return this
        }

        TestSuite afterAll(Closure action) {
            afterActions.add(action)
            return this
        }

        TestSuite test(String name, Closure closure) {
            def builder = new TestBuilder(name: name)
            closure.delegate = builder
            closure.resolveStrategy = Closure.DELEGATE_FIRST
            tests.add(builder)
            return this
        }

        void runAll() {
            println "运行测试套件，共 ${tests.size()} 个测试"

            try {
                // 执行beforeAll
                beforeActions.each { action ->
                    action.call()
                }

                // 运行所有测试
                tests.each { test ->
                    test.run()
                }

                println "所有测试完成"
            } finally {
                // 执行afterAll
                afterActions.each { action ->
                    try {
                        action.call()
                    } catch (Exception e) {
                        println "AfterAll执行失败: ${e.message}"
                    }
                }
            }
        }
    }

    // 3. 使用示例
    static def testSuite = new TestSuite()
        .beforeAll {
            println "初始化测试环境..."
            // 初始化数据库连接等
        }
        .afterAll {
            println "清理测试环境..."
            // 清理资源
        }
        .test("字符串处理测试") {
            setup {
                println "设置测试数据..."
            }
            when {
                "hello world".toUpperCase()
            }
            then { result ->
                result == "HELLO WORLD"
            }
            cleanup {
                println "清理测试数据..."
            }
        }
        .test("列表操作测试") {
            setup {
                println "准备测试列表..."
            }
            when {
                [1, 2, 3].collect { it * 2 }
            }
            then { result ->
                result == [2, 4, 6]
            }
        }

    // 4. Mock DSL
    static class MockBuilder {
        def targetClass
        def behaviors = [:]

        MockBuilder(Class targetClass) {
            this.targetClass = targetClass
        }

        MockBuilder method(String name, Closure behavior) {
            behaviors[name] = behavior
            return this
        }

        def build() {
            def proxy = Proxy.newProxyInstance(
                targetClass.classLoader,
                [targetClass] as Class[],
                { Object proxy, Method method, Object[] args ->
                    def behavior = behaviors[method.name]
                    if (behavior) {
                        return behavior.call(args)
                    }
                    throw new UnsupportedOperationException("Method ${method.name} not mocked")
                }
            )
            return proxy
        }
    }
}
```

## 7. 高级DSL特性

### 7.1 类型安全的DSL

```groovy
class TypeSafeDSL {
    // 1. 类型安全的构建器
    static class SafePersonBuilder {
        private String name
        private Integer age
        private String email
        private SafeAddressBuilder address

        SafePersonBuilder name(String name) {
            this.name = name
            return this
        }

        SafePersonBuilder age(int age) {
            this.age = age
            return this
        }

        SafePersonBuilder email(String email) {
            this.email = email
            return this
        }

        SafePersonBuilder address(@DelegatesTo(SafeAddressBuilder) Closure closure) {
            this.address = new SafeAddressBuilder()
            closure.delegate = address
            closure.resolveStrategy = Closure.DELEGATE_FIRST
            closure()
            return this
        }

        SafePerson build() {
            return new SafePerson(
                name: Objects.requireNonNull(name, "Name cannot be null"),
                age: Objects.requireNonNull(age, "Age cannot be null"),
                email: email,
                address: address?.build()
            )
        }
    }

    static class SafeAddressBuilder {
        private String street
        private String city
        private String country

        SafeAddressBuilder street(String street) {
            this.street = Objects.requireNonNull(street, "Street cannot be null")
            return this
        }

        SafeAddressBuilder city(String city) {
            this.city = Objects.requireNonNull(city, "City cannot be null")
            return this
        }

        SafeAddressBuilder country(String country) {
            this.country = Objects.requireNonNull(country, "Country cannot be null")
            return this
        }

        SafeAddress build() {
            return new SafeAddress(
                street: street,
                city: city,
                country: country
            )
        }
    }

    static class SafePerson {
        final String name
        final int age
        final String email
        final SafeAddress address

        SafePerson(Map params) {
            this.name = params.name
            this.age = params.age
            this.email = params.email
            this.address = params.address
        }
    }

    static class SafeAddress {
        final String street
        final String city
        final String country

        SafeAddress(Map params) {
            this.street = params.street
            this.city = params.city
            this.country = params.country
        }
    }

    // 2. 类型安全的查询DSL
    static class SafeQueryBuilder<T> {
        private Class<T> entityClass
        private List<String> selectedFields = []
        private List<String> conditions = []
        private List<String> orderBy = []
        private Integer limit
        private Integer offset

        SafeQueryBuilder(Class<T> entityClass) {
            this.entityClass = entityClass
        }

        SafeQueryBuilder<T> select(String... fields) {
            selectedFields.addAll(fields)
            return this
        }

        SafeQueryBuilder<T> where(String condition) {
            conditions.add(condition)
            return this
        }

        SafeQueryBuilder<T> orderBy(String field, String direction = "ASC") {
            orderBy.add("$field $direction")
            return this
        }

        SafeQueryBuilder<T> limit(int limit) {
            this.limit = limit
            return this
        }

        SafeQueryBuilder<T> offset(int offset) {
            this.offset = offset
            return this
        }

        String build() {
            def tableName = entityClass.simpleName.toLowerCase()
            def fields = selectedFields.isEmpty() ? "*" : selectedFields.join(", ")

            def sql = new StringBuilder()
            sql.append("SELECT $fields FROM $tableName")

            if (!conditions.isEmpty()) {
                sql.append(" WHERE ").append(conditions.join(" AND "))
            }

            if (!orderBy.isEmpty()) {
                sql.append(" ORDER BY ").append(orderBy.join(", "))
            }

            if (limit != null) {
                sql.append(" LIMIT $limit")
            }

            if (offset != null) {
                sql.append(" OFFSET $offset")
            }

            return sql.toString()
        }
    }
}
```

### 7.2 编译时验证

```groovy
@CompileStatic
class CompileTimeValidationDSL {
    // 1. 编译时验证的配置DSL
    static class ValidatedConfigBuilder {
        private Map<String, Object> config = [:]

        ValidatedConfigBuilder database(@DelegatesTo(ValidatedDatabaseConfig) Closure closure) {
            def builder = new ValidatedDatabaseConfig()
            closure.delegate = builder
            closure.resolveStrategy = Closure.DELEGATE_FIRST
            closure()
            config.database = builder.build()
            return this
        }

        ValidatedConfigBuilder server(@DelegatesTo(ValidatedServerConfig) Closure closure) {
            def builder = new ValidatedServerConfig()
            closure.delegate = builder
            closure.resolveStrategy = Closure.DELEGATE_FIRST
            closure()
            config.server = builder.build()
            return this
        }

        Map<String, Object> build() {
            return config
        }
    }

    static class ValidatedDatabaseConfig {
        private String url
        private String username
        private String password
        private int poolSize = 10

        ValidatedDatabaseConfig url(String url) {
            if (!url?.startsWith("jdbc:")) {
                throw new IllegalArgumentException("Database URL must start with 'jdbc:'")
            }
            this.url = url
            return this
        }

        ValidatedDatabaseConfig username(String username) {
            if (!username?.trim()) {
                throw new IllegalArgumentException("Username cannot be empty")
            }
            this.username = username
            return this
        }

        ValidatedDatabaseConfig password(String password) {
            if (!password?.trim()) {
                throw new IllegalArgumentException("Password cannot be empty")
            }
            this.password = password
            return this
        }

        ValidatedDatabaseConfig poolSize(int size) {
            if (size <= 0) {
                throw new IllegalArgumentException("Pool size must be positive")
            }
            this.poolSize = size
            return this
        }

        Map<String, Object> build() {
            if (!url) throw new IllegalStateException("Database URL is required")
            if (!username) throw new IllegalStateException("Username is required")
            if (!password) throw new IllegalStateException("Password is required")

            return [
                url: url,
                username: username,
                password: password,
                poolSize: poolSize
            ]
        }
    }

    static class ValidatedServerConfig {
        private int port = 8080
        private String host = "localhost"

        ValidatedServerConfig port(int port) {
            if (port < 1 || port > 65535) {
                throw new IllegalArgumentException("Port must be between 1 and 65535")
            }
            this.port = port
            return this
        }

        ValidatedServerConfig host(String host) {
            if (!host?.trim()) {
                throw new IllegalArgumentException("Host cannot be empty")
            }
            this.host = host
            return this
        }

        Map<String, Object> build() {
            return [
                port: port,
                host: host
            ]
        }
    }
}
```

## 8. DSL最佳实践

### 8.1 设计模式应用

```groovy
class DSLBestPractices {
    // 1. 使用建造者模式
    static class ValidatedUserBuilder {
        private String name
        private Integer age
        private String email
        private List<String> roles = []

        ValidatedUserBuilder name(String name) {
            if (!name?.trim()) {
                throw new IllegalArgumentException("Name cannot be empty")
            }
            this.name = name.trim()
            return this
        }

        ValidatedUserBuilder age(int age) {
            if (age < 0 || age > 150) {
                throw new IllegalArgumentException("Age must be between 0 and 150")
            }
            this.age = age
            return this
        }

        ValidatedUserBuilder email(String email) {
            if (!email?.matches(/^[A-Za-z0-9+_.-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$/)) {
                throw new IllegalArgumentException("Invalid email format")
            }
            this.email = email
            return this
        }

        ValidatedUserBuilder role(String role) {
            if (!role?.trim()) {
                throw new IllegalArgumentException("Role cannot be empty")
            }
            this.roles.add(role.trim())
            return this
        }

        ValidatedUserBuilder roles(String... roles) {
            roles.each { role ->
                if (!role?.trim()) {
                    throw new IllegalArgumentException("Role cannot be empty")
                }
                this.roles.add(role.trim())
            }
            return this
        }

        User build() {
            if (!name) throw new IllegalStateException("Name is required")
            if (!age) throw new IllegalStateException("Age is required")
            if (!email) throw new IllegalStateException("Email is required")

            return new User(
                name: name,
                age: age,
                email: email,
                roles: roles.isEmpty() ? ["USER"] : roles
            )
        }
    }

    static class User {
        String name
        int age
        String email
        List<String> roles
    }

    // 2. 使用工厂模式
    static class UserFactory {
        static ValidatedUserBuilder create() {
            return new ValidatedUserBuilder()
        }

        static ValidatedUserBuilder createAdmin() {
            return new ValidatedUserBuilder().roles("ADMIN")
        }

        static ValidatedUserBuilder createWithDefaults(String name, int age) {
            return new ValidatedUserBuilder()
                .name(name)
                .age(age)
                .email("${name.toLowerCase()}@example.com")
        }
    }

    // 3. 使用策略模式
    static class ValidationStrategy {
        private List<Closure<Boolean>> validators = []

        ValidationStrategy addValidator(Closure<Boolean> validator) {
            validators.add(validator)
            return this
        }

        boolean validate(Object value) {
            return validators.every { validator -> validator.call(value) }
        }
    }

    static class CommonValidators {
        static ValidationStrategy notEmpty() {
            return new ValidationStrategy()
                .addValidator { value -> value != null && !value.toString().isEmpty() }
        }

        static ValidationStrategy minLength(int length) {
            return new ValidationStrategy()
                .addValidator { value -> value.toString().length() >= length }
        }

        static ValidationStrategy maxLength(int length) {
            return new ValidationStrategy()
                .addValidator { value -> value.toString().length() <= length }
        }

        static ValidationStrategy emailFormat() {
            return new ValidationStrategy()
                .addValidator { value ->
                    value.toString().matches(/^[A-Za-z0-9+_.-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$/)
                }
        }
    }
}
```

### 8.2 性能优化

```groovy
class DSLPerformanceOptimization {
    // 1. 缓存构建器实例
    static class CachedQueryBuilder {
        private static final Map<String, CachedQueryBuilder> cache = new ConcurrentHashMap<>()

        private String queryTemplate
        private List<Closure> conditions = []

        static CachedQueryBuilder forTemplate(String template) {
            return cache.computeIfAbsent(template) { key ->
                new CachedQueryBuilder(queryTemplate: key)
            }
        }

        CachedQueryBuilder where(Closure condition) {
            conditions.add(condition)
            return this
        }

        String build(Map<String, Object> params) {
            def sql = queryTemplate
            conditions.each { condition ->
                sql += " AND " + condition.call(params)
            }
            return sql
        }
    }

    // 2. 延迟初始化
    static class LazyConfigBuilder {
        private Map<String, Object> config = [:]
        private Map<String, Closure> lazyInitializers = [:]

        LazyConfigBuilder property(String name, Object value) {
            config[name] = value
            return this
        }

        LazyConfigBuilder lazyProperty(String name, Closure initializer) {
            lazyInitializers[name] = initializer
            return this
        }

        Object getProperty(String name) {
            if (config.containsKey(name)) {
                return config[name]
            }

            if (lazyInitializers.containsKey(name)) {
                def value = lazyInitializers[name].call()
                config[name] = value
                lazyInitializers.remove(name)
                return value
            }

            throw new MissingPropertyException(name, this.class)
        }

        Map<String, Object> build() {
            // 初始化所有延迟属性
            lazyInitializers.each { name, initializer ->
                config[name] = initializer.call()
            }
            lazyInitializers.clear()

            return new HashMap<>(config)
        }
    }

    // 3. 批量操作优化
    static class BatchOperationBuilder {
        private List<Closure> operations = []
        private int batchSize = 100

        BatchOperationBuilder batchSize(int size) {
            this.batchSize = size
            return this
        }

        BatchOperationBuilder add(Closure operation) {
            operations.add(operation)
            return this
        }

        List<Object> execute() {
            def results = []
            def batch = []

            operations.each { operation ->
                batch.add(operation)
                if (batch.size() >= batchSize) {
                    results.addAll(executeBatch(batch))
                    batch.clear()
                }
            }

            if (!batch.isEmpty()) {
                results.addAll(executeBatch(batch))
            }

            return results
        }

        private List<Object> executeBatch(List<Closure> batch) {
            println "执行批量操作，大小: ${batch.size()}"
            return batch.collect { operation ->
                try {
                    return operation.call()
                } catch (Exception e) {
                    println "批量操作失败: ${e.message}"
                    return null
                }
            }
        }
    }
}
```

## 9. 实际应用案例

### 9.1 构建工具DSL

```groovy
class BuildToolDSL {
    // 1. 简单的构建DSL
    static class ProjectBuilder {
        def name
        def version = "1.0.0"
        def dependencies = []
        def tasks = [:]
        def repositories = []

        ProjectBuilder name(String name) {
            this.name = name
            return this
        }

        ProjectBuilder version(String version) {
            this.version = version
            return this
        }

        ProjectBuilder dependencies(Closure closure) {
            def depBuilder = new DependencyBuilder()
            closure.delegate = depBuilder
            closure.resolveStrategy = Closure.DELEGATE_FIRST
            closure()
            this.dependencies = depBuilder.dependencies
            return this
        }

        ProjectBuilder repositories(Closure closure) {
            def repoBuilder = new RepositoryBuilder()
            closure.delegate = repoBuilder
            closure.resolveStrategy = Closure.DELEGATE_FIRST
            closure()
            this.repositories = repoBuilder.repositories
            return this
        }

        ProjectBuilder task(String name, Closure closure) {
            tasks[name] = closure
            return this
        }

        void build() {
            println "构建项目: $name"
            println "版本: $version"
            println "依赖: $dependencies"
            println "仓库: $repositories"

            // 执行默认任务
            if (tasks.containsKey("build")) {
                tasks["build"].call()
            }
        }
    }

    static class DependencyBuilder {
        def dependencies = []

        DependencyBuilder implementation(String dependency) {
            dependencies.add([type: "implementation", value: dependency])
            return this
        }

        DependencyBuilder testImplementation(String dependency) {
            dependencies.add([type: "testImplementation", value: dependency])
            return this
        }

        DependencyBuilder compileOnly(String dependency) {
            dependencies.add([type: "compileOnly", value: dependency])
            return this
        }
    }

    static class RepositoryBuilder {
        def repositories = []

        RepositoryBuilder mavenCentral() {
            repositories.add([type: "maven", url: "https://repo.maven.apache.org/maven2"])
            return this
        }

        RepositoryBuilder mavenLocal() {
            repositories.add([type: "maven", url: "file:///${System.getProperty('user.home')}/.m2/repository"])
            return this
        }

        RepositoryBuilder url(String url) {
            repositories.add([type: "maven", url: url])
            return this
        }
    }

    // 2. 使用示例
    static def project = new ProjectBuilder()
        .name("MyApp")
        .version("1.0.0")
        .dependencies {
            implementation "org.springframework.boot:spring-boot-starter-web:2.7.0"
            testImplementation "org.springframework.boot:spring-boot-starter-test:2.7.0"
            compileOnly "org.projectlombok:lombok:1.18.24"
        }
        .repositories {
            mavenCentral()
            mavenLocal()
        }
        .task("build") {
            println "编译项目..."
        }
        .task("test") {
            println "运行测试..."
        }
        .task("clean") {
            println "清理构建文件..."
        }
        .build()
}
```

### 9.2 API测试DSL

```groovy
class APITestingDSL {
    // 1. API测试DSL
    static class APITestBuilder {
        private String baseUrl
        private Map<String, String> headers = [:]
        private Map<String, Object> queryParams = [:]
        private Object body
        private List<Closure> assertions = []

        APITestBuilder baseUrl(String url) {
            this.baseUrl = url
            return this
        }

        APITestBuilder header(String name, String value) {
            headers[name] = value
            return this
        }

        APITestBuilder headers(Map<String, String> headers) {
            this.headers.putAll(headers)
            return this
        }

        APITestBuilder param(String name, Object value) {
            queryParams[name] = value
            return this
        }

        APITestBuilder params(Map<String, Object> params) {
            queryParams.putAll(params)
            return this
        }

        APITestBuilder body(Object body) {
            this.body = body
            return this
        }

        APITestBuilder jsonBody(Map<String, Object> json) {
            this.body = groovy.json.JsonOutput.toJson(json)
            header("Content-Type", "application/json")
            return this
        }

        APITestBuilder expect(Closure assertion) {
            assertions.add(assertion)
            return this
        }

        APITestResult get(String path) {
            return executeRequest("GET", path)
        }

        APITestResult post(String path) {
            return executeRequest("POST", path)
        }

        APITestResult put(String path) {
            return executeRequest("PUT", path)
        }

        APITestResult delete(String path) {
            return executeRequest("DELETE", path)
        }

        private APITestResult executeRequest(String method, String path) {
            def url = buildUrl(path)
            println "执行 $method 请求: $url"

            // 模拟HTTP请求
            def response = simulateHttpRequest(method, url)

            // 执行断言
            def assertionResults = []
            assertions.each { assertion ->
                try {
                    def result = assertion.call(response)
                    assertionResults.add([success: true, message: "断言通过"])
                } catch (AssertionError e) {
                    assertionResults.add([success: false, message: e.message])
                }
            }

            return new APITestResult(
                response: response,
                assertions: assertionResults
            )
        }

        private String buildUrl(String path) {
            def url = baseUrl + path
            if (!queryParams.isEmpty()) {
                def queryString = queryParams.collect { k, v -> "$k=$v" }.join("&")
                url += "?" + queryString
            }
            return url
        }

        private Map simulateHttpRequest(String method, String url) {
            // 模拟HTTP响应
            return [
                status: 200,
                headers: ["Content-Type": "application/json"],
                body: '{"message": "Success", "data": {"id": 1, "name": "Test"}}'
            ]
        }
    }

    static class APITestResult {
        Map response
        List assertions

        boolean isSuccess() {
            return assertions.every { it.success }
        }

        void printReport() {
            println "API测试结果:"
            println "  状态码: ${response.status}"
            println "  断言结果:"
            assertions.each { assertion ->
                println "    ${assertion.success ? "✓" : "✗"} ${assertion.message}"
            }
        }
    }

    // 2. 使用示例
    static def apiTest = new APITestBuilder()
        .baseUrl("https://api.example.com")
        .header("Authorization", "Bearer token123")
        .param("page", 1)
        .param("limit", 10)
        .get("/users")
        .expect { response -> response.status == 200 }
        .expect { response -> response.body.contains("users") }

    static def postTest = new APITestBuilder()
        .baseUrl("https://api.example.com")
        .header("Authorization", "Bearer token123")
        .jsonBody([
            name: "New User",
            email: "newuser@example.com"
        ])
        .post("/users")
        .expect { response -> response.status == 201 }
        .expect { response -> response.body.contains("New User") }
}
```

## 10. 总结

Groovy DSL是一项非常强大的技术，能够为特定领域创建简洁、表达力强的语言。通过本文的深入探讨，我们掌握了：

1. **DSL基础**：理解DSL的类型和设计原则
2. **构建技术**：掌握闭包委托、方法缺失处理等核心技术
3. **高级特性**：类型安全、编译时验证、性能优化
4. **实际应用**：配置管理、业务规则、工作流、测试等领域
5. **最佳实践**：设计模式、性能优化、错误处理

### DSL设计的关键原则：

1. **可读性优先**：DSL应该像自然语言一样易于理解
2. **领域特定**：专注于特定领域的问题和概念
3. **类型安全**：在编译时提供类型检查和验证
4. **性能优化**：避免不必要的对象创建和内存消耗
5. **错误处理**：提供清晰的错误信息和调试支持

### 成功DSL的要素：

1. **简洁的语法**：减少语法噪音，提高可读性
2. **丰富的功能**：覆盖领域内的主要需求
3. **良好的文档**：提供完整的使用指南和示例
4. **工具支持**：IDE支持、语法高亮、自动补全
5. **社区支持**：活跃的社区和丰富的第三方库

通过合理运用Groovy的DSL技术，我们可以为特定领域创建出既强大又易用的语言，大大提高开发效率和代码质量。在实际项目中，建议根据具体需求选择合适的DSL设计模式，并注重用户体验和长期维护性。