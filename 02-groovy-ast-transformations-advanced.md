# Groovy AST转换高级应用：编译时元编程的深度解析

## 引言

AST（Abstract Syntax Tree）转换是Groovy最强大的编译时元编程特性之一。通过AST转换，我们可以在编译期间修改和增强代码结构，实现许多在运行时无法完成的高级功能。本文将深入探讨Groovy AST转换的底层实现原理、高级应用场景以及性能优化策略。

## 1. AST转换基础架构

### 1.1 编译阶段与AST转换时机

Groovy编译过程包含以下阶段：

```
源代码 → 词法分析 → 语法分析 → AST构建 → AST转换 → 字节码生成 → 类文件
```

AST转换在AST构建之后、字节码生成之前执行，这是修改代码结构的最佳时机。

### 1.2 AST转换类型

Groovy支持三种主要的AST转换类型：

1. **全局AST转换**：应用于整个编译单元
2. **局部AST转换**：应用于特定的注解目标
3. **AST转换注解**：通过注解触发的转换

```groovy
// AST转换基础示例
@GroovyASTTransformation(phase = CompilePhase.SEMANTIC_ANALYSIS)
class MyASTTransformation implements ASTTransformation {
    @Override
    void visit(ASTNode[] nodes, SourceUnit sourceUnit) {
        def annotationNode = nodes[0] as AnnotationNode
        def annotatedNode = nodes[1] as AnnotatedNode

        // AST转换逻辑
        performTransformation(annotationNode, annotatedNode, sourceUnit)
    }
}
```

## 2. 深度解析AST节点结构

### 2.1 核心AST节点类型

```groovy
// AST节点层次结构
ASTNode
├── ModuleNode          // 模块节点（整个文件）
├── ClassNode           // 类节点
├── MethodNode          // 方法节点
├── FieldNode           // 字段节点
├── PropertyNode        // 属性节点
├── ConstructorNode     // 构造函数节点
├── Statement           // 语句节点
├── Expression          // 表达式节点
└── AnnotationNode      // 注解节点
```

### 2.2 AST节点操作基础

```groovy
class ASTNodeOperations {
    // 创建方法节点
    static MethodNode createMethodNode(ClassNode classNode, String methodName,
                                      Class returnType, List<Parameter> parameters,
                                      BlockStatement block) {

        def methodNode = new MethodNode(
            methodName,
            ACC_PUBLIC,
            ClassHelper.make(returnType),
            parameters as Parameter[],
            [] as ClassNode[],
            block
        )

        classNode.addMethod(methodNode)
        return methodNode
    }

    // 创建字段节点
    static FieldNode createFieldNode(ClassNode classNode, String fieldName,
                                   Class fieldType, Expression initialValue) {

        def fieldNode = new FieldNode(
            fieldName,
            ACC_PRIVATE,
            ClassHelper.make(fieldType),
            classNode,
            initialValue
        )

        classNode.addField(fieldNode)
        return fieldNode
    }

    // 创建属性节点
    static PropertyNode createPropertyNode(ClassNode classNode, String propName,
                                         Class propType, Expression initialValue) {

        def propNode = new PropertyNode(
            propName,
            ACC_PUBLIC,
            ClassHelper.make(propType),
            classNode,
            initialValue,
            null,  // getter block
            null   // setter block
        )

        classNode.addProperty(propNode)
        return propNode
    }
}
```

## 3. 高级AST转换实现

### 3.1 自动日志记录AST转换

```groovy
@Retention(RetentionPolicy.SOURCE)
@Target([ElementType.METHOD, ElementType.TYPE])
@GroovyASTTransformation(phase = CompilePhase.SEMANTIC_ANALYSIS)
@interface Loggable {}

@GroovyASTTransformation(phase = CompilePhase.SEMANTIC_ANALYSIS)
class LoggingASTTransformation implements ASTTransformation {
    @Override
    void visit(ASTNode[] nodes, SourceUnit sourceUnit) {
        def annotationNode = nodes[0] as AnnotationNode
        def annotatedNode = nodes[1] as AnnotatedNode

        if (annotatedNode instanceof MethodNode) {
            transformMethod(annotatedNode as MethodNode, sourceUnit)
        } else if (annotatedNode instanceof ClassNode) {
            transformClass(annotatedNode as ClassNode, sourceUnit)
        }
    }

    def transformMethod(MethodNode methodNode, SourceUnit sourceUnit) {
        def methodName = methodNode.name
        def className = methodNode.declaringClass.name

        // 创建日志记录代码
        def logEnter = createLogStatement("进入方法: $className.$methodName")
        def logExit = createLogStatement("退出方法: $className.$methodName")

        // 修改方法体
        def originalStatements = methodNode.code.statements
        def newStatements = []

        newStatements.add(logEnter)
        newStatements.addAll(originalStatements)

        // 添加异常处理的日志记录
        def tryBlock = new BlockStatement(newStatements, methodNode.code.variableScope)
        def catchBlock = createCatchBlock(methodNode, sourceUnit)
        def finallyBlock = new BlockStatement([logExit], methodNode.code.variableScope)

        def tryCatchStatement = new TryCatchStatement(
            tryBlock,
            catchBlock,
            finallyBlock
        )

        methodNode.code = new BlockStatement([tryCatchStatement], methodNode.code.variableScope)
    }

    def transformClass(ClassNode classNode, SourceUnit sourceUnit) {
        classNode.methods.each { method ->
            if (!method.isAbstract() && !method.isStatic()) {
                transformMethod(method, sourceUnit)
            }
        }
    }

    def createLogStatement(String message) {
        def argsList = new ArgumentListExpression([
            new ConstantExpression(message)
        ])

        return new ExpressionStatement(
            new MethodCallExpression(
                new VariableExpression("this"),
                "println",
                argsList
            )
        )
    }

    def createCatchBlock(MethodNode methodNode, SourceUnit sourceUnit) {
        def exceptionParameter = new Parameter(
            ClassHelper.make(Exception),
            "ex"
        )

        def logException = new ExpressionStatement(
            new MethodCallExpression(
                new VariableExpression("this"),
                "println",
                new ArgumentListExpression([
                    new BinaryExpression(
                        new ConstantExpression("方法异常: "),
                        Token.newSymbol("+", -1, -1),
                        new PropertyExpression(
                            new VariableExpression("ex"),
                            "message"
                        )
                    )
                ])
            )
        )

        def throwStatement = new ThrowStatement(new VariableExpression("ex"))

        return new CatchStatement(
            exceptionParameter,
            new BlockStatement([logException, throwStatement], null)
        )
    }
}
```

### 3.2 自动缓存AST转换

```groovy
@Retention(RetentionPolicy.SOURCE)
@Target(ElementType.METHOD)
@GroovyASTTransformation(phase = CompilePhase.SEMANTIC_ANALYSIS)
@interface Cached {
    int maxSize() default 100
    long expireTime() default 60000 // 1分钟
}

@GroovyASTTransformation(phase = CompilePhase.SEMANTIC_ANALYSIS)
class CachedASTTransformation implements ASTTransformation {
    @Override
    void visit(ASTNode[] nodes, SourceUnit sourceUnit) {
        def annotationNode = nodes[0] as AnnotationNode
        def methodNode = nodes[1] as MethodNode

        def maxSize = getAnnotationValue(annotationNode, "maxSize", 100)
        def expireTime = getAnnotationValue(annotationNode, "expireTime", 60000)

        createCachedMethod(methodNode, maxSize, expireTime, sourceUnit)
    }

    def createCachedMethod(MethodNode originalMethod, int maxSize, long expireTime, SourceUnit sourceUnit) {
        def methodName = originalMethod.name
        def parameters = originalMethod.parameters
        def returnType = originalMethod.returnType

        // 创建缓存字段
        def cacheFieldName = "_cache_${methodName}"
        def cacheField = createCacheField(originalMethod.declaringClass, cacheFieldName)
        originalMethod.declaringClass.addField(cacheField)

        // 创建缓存键生成方法
        def keyGenerator = createKeyGeneratorMethod(originalMethod.declaringClass, methodName, parameters)
        originalMethod.declaringClass.addMethod(keyGenerator)

        // 创建新方法体
        def newMethodBody = createCachedMethodBody(originalMethod, cacheFieldName, maxSize, expireTime)

        // 替换原始方法
        def cachedMethod = new MethodNode(
            methodName,
            originalMethod.modifiers,
            returnType,
            parameters,
            originalMethod.exceptions,
            newMethodBody
        )

        originalMethod.declaringClass.removeMethod(originalMethod)
        originalMethod.declaringClass.addMethod(cachedMethod)
    }

    def createCacheField(ClassNode classNode, String fieldName) {
        def cacheType = new ClassNode("java.util.concurrent.ConcurrentHashMap", 0, null)
        return new FieldNode(
            fieldName,
            ACC_PRIVATE | ACC_STATIC,
            cacheType,
            classNode,
            new ConstructorCallExpression(cacheType, new ArgumentListExpression())
        )
    }

    def createKeyGeneratorMethod(ClassNode classNode, String methodName, Parameter[] parameters) {
        def methodName = "_generateCacheKey_${methodName}"

        def parametersList = parameters.collect { param ->
            new VariableExpression(param.name)
        }

        def keyGeneration = parametersList.inject(new ConstantExpression(methodName)) { acc, param ->
            new BinaryExpression(
                new BinaryExpression(acc, Token.newSymbol("+", -1, -1), new ConstantExpression("_")),
                Token.newSymbol("+", -1, -1),
                new MethodCallExpression(
                    param,
                    "hashCode",
                    new ArgumentListExpression()
                )
            )
        }

        def block = new BlockStatement([
            new ReturnStatement(keyGeneration)
        ], null)

        return new MethodNode(
            methodName,
            ACC_PRIVATE | ACC_STATIC,
            ClassHelper.make(String),
            parameters,
            [] as ClassNode[],
            block
        )
    }

    def createCachedMethodBody(MethodNode originalMethod, String cacheFieldName, int maxSize, long expireTime) {
        def parameters = originalMethod.parameters
        def returnType = originalMethod.returnType

        // 生成缓存键
        def keyGeneratorCall = new MethodCallExpression(
            new ClassExpression(originalMethod.declaringClass),
            "_generateCacheKey_${originalMethod.name}",
            new ArgumentListExpression(parameters.collect { new VariableExpression(it.name) })
        )

        // 获取缓存
        def cacheFieldAccess = new PropertyExpression(
            new ClassExpression(originalMethod.declaringClass),
            cacheFieldName
        )

        // 检查缓存中是否存在
        def containsKeyCall = new MethodCallExpression(
            cacheFieldAccess,
            "containsKey",
            new ArgumentListExpression([keyGeneratorCall])
        )

        // 获取缓存值
        def getCall = new MethodCallExpression(
            cacheFieldAccess,
            "get",
            new ArgumentListExpression([keyGeneratorCall])
        )

        // 创建缓存条目
        def cacheEntryType = new ClassNode("java.util.concurrent.ConcurrentHashMap\$Node", 0, null)
        def cacheEntry = new VariableExpression("cacheEntry", cacheEntryType)

        // 检查是否过期
        def currentTime = new MethodCallExpression(
            new ClassExpression(ClassHelper.make(System)),
            "currentTimeMillis",
            new ArgumentListExpression()
        )

        def timestampAccess = new MethodCallExpression(
            cacheEntry,
            "getTimestamp",
            new ArgumentListExpression()
        )

        def isExpired = new BinaryExpression(
            new BinaryExpression(currentTime, Token.newSymbol("-", -1, -1), timestampAccess),
            Token.newSymbol(">", -1, -1),
            new ConstantExpression(expireTime)
        )

        // 原始方法调用
        def originalMethodCall = new MethodCallExpression(
            new VariableExpression("this"),
            "_original_${originalMethod.name}",
            new ArgumentListExpression(parameters.collect { new VariableExpression(it.name) })
        )

        // 存储到缓存
        def putCall = new MethodCallExpression(
            cacheFieldAccess,
            "put",
            new ArgumentListExpression([
                keyGeneratorCall,
                new ConstructorCallExpression(
                    new ClassNode("CacheEntry", 0, null),
                    new ArgumentListExpression([
                        originalMethodCall,
                        currentTime
                    ])
                )
            ])
        )

        // 创建完整的if-else逻辑
        def ifBlock = new BlockStatement([
            new ExpressionStatement(new DeclarationExpression(
                cacheEntry,
                Token.newSymbol("=", -1, -1),
                getCall
            )),
            new IfStatement(
                new BooleanExpression(isExpired),
                new BlockStatement([
                    new ExpressionStatement(new MethodCallExpression(
                        cacheFieldAccess,
                        "remove",
                        new ArgumentListExpression([keyGeneratorCall])
                    )),
                    new ReturnStatement(putCall)
                ], null),
                new BlockStatement([
                    new ReturnStatement(new MethodCallExpression(cacheEntry, "getValue", new ArgumentListExpression()))
                ], null)
            )
        ], null)

        def elseBlock = new BlockStatement([new ReturnStatement(putCall)], null)

        return new BlockStatement([
            new IfStatement(
                new BooleanExpression(containsKeyCall),
                ifBlock,
                elseBlock
            )
        ], null)
    }

    def getAnnotationValue(AnnotationNode annotationNode, String key, def defaultValue) {
        def value = annotationNode.getMember(key)
        return value?.value ?: defaultValue
    }
}
```

## 4. 复杂AST转换模式

### 4.1 自动实现接口AST转换

```groovy
@Retention(RetentionPolicy.SOURCE)
@Target(ElementType.TYPE)
@GroovyASTTransformation(phase = CompilePhase.SEMANTIC_ANALYSIS)
@interface AutoImplement {
    Class[] interfaces() default []
    boolean lazy() default false
}

@GroovyASTTransformation(phase = CompilePhase.SEMANTIC_ANALYSIS)
class AutoImplementASTTransformation implements ASTTransformation {
    @Override
    void visit(ASTNode[] nodes, SourceUnit sourceUnit) {
        def annotationNode = nodes[0] as AnnotationNode
        def classNode = nodes[1] as ClassNode

        def interfaces = getAnnotationValue(annotationNode, "interfaces", []) as Class[]
        def lazy = getAnnotationValue(annotationNode, "lazy", false) as boolean

        interfaces.each { interfaceClass ->
            implementInterface(classNode, interfaceClass, lazy, sourceUnit)
        }
    }

    def implementInterface(ClassNode classNode, Class interfaceClass, boolean lazy, SourceUnit sourceUnit) {
        def interfaceNode = ClassHelper.make(interfaceClass)
        classNode.addInterface(interfaceNode)

        // 获取接口的所有方法
        def interfaceMethods = interfaceNode.methods.findAll {
            it.isAbstract() && !it.isDefault()
        }

        interfaceMethods.each { method ->
            if (!classNode.hasMethod(method.name, method.parameters)) {
                def implementation = lazy ?
                    createLazyImplementation(method, sourceUnit) :
                    createDefaultImplementation(method, sourceUnit)

                classNode.addMethod(implementation)
            }
        }
    }

    def createDefaultImplementation(MethodNode interfaceMethod, SourceUnit sourceUnit) {
        def methodName = interfaceMethod.name
        def parameters = interfaceMethod.parameters
        def returnType = interfaceMethod.returnType

        def throwStatement = new ThrowStatement(
            new ConstructorCallExpression(
                ClassHelper.make(UnsupportedOperationException),
                new ArgumentListExpression([
                    new ConstantExpression("方法 ${methodName} 尚未实现")
                ])
            )
        )

        return new MethodNode(
            methodName,
            ACC_PUBLIC,
            returnType,
            parameters,
            [] as ClassNode[],
            new BlockStatement([throwStatement], null)
        )
    }

    def createLazyImplementation(MethodNode interfaceMethod, SourceUnit sourceUnit) {
        def methodName = interfaceMethod.name
        def parameters = interfaceMethod.parameters
        def returnType = interfaceMethod.returnType

        // 创建懒加载字段
        def fieldName = "_lazy_${methodName}"
        def fieldNode = new FieldNode(
            fieldName,
            ACC_PRIVATE,
            ClassHelper.make(Closure),
            interfaceMethod.declaringClass,
            null
        )
        interfaceMethod.declaringClass.addField(fieldNode)

        // 创建初始化方法
        def initMethodName = "_init_${methodName}"
        def initMethod = new MethodNode(
            initMethodName,
            ACC_PRIVATE,
            ClassHelper.make(Object),
            parameters,
            [] as ClassNode[],
            new BlockStatement([
                new ReturnStatement(
                    new ConstructorCallExpression(
                        ClassHelper.make(UnsupportedOperationException),
                        new ArgumentListExpression([
                            new ConstantExpression("方法 ${methodName} 尚未实现")
                        ])
                    )
                )
            ], null)
        )
        interfaceMethod.declaringClass.addMethod(initMethod)

        // 创建懒加载方法
        def getCall = new MethodCallExpression(
            new VariableExpression("this"),
            "get${methodName.capitalize()}",
            new ArgumentListExpression()
        )

        def methodCall = new MethodCallExpression(
            getCall,
            "call",
            new ArgumentListExpression(parameters.collect { new VariableExpression(it.name) })
        )

        return new MethodNode(
            methodName,
            ACC_PUBLIC,
            returnType,
            parameters,
            [] as ClassNode[],
            new BlockStatement([new ReturnStatement(methodCall)], null)
        )
    }

    def getAnnotationValue(AnnotationNode annotationNode, String key, def defaultValue) {
        def value = annotationNode.getMember(key)
        return value?.value ?: defaultValue
    }
}
```

### 4.2 自动序列化AST转换

```groovy
@Retention(RetentionPolicy.SOURCE)
@Target(ElementType.TYPE)
@GroovyASTTransformation(phase = CompilePhase.SEMANTIC_ANALYSIS)
@interface AutoSerializable {
    boolean includeTransient() default false
    String dateFormat() default "yyyy-MM-dd HH:mm:ss"
}

@GroovyASTTransformation(phase = CompilePhase.SEMANTIC_ANALYSIS)
class AutoSerializableASTTransformation implements ASTTransformation {
    @Override
    void visit(ASTNode[] nodes, SourceUnit sourceUnit) {
        def annotationNode = nodes[0] as AnnotationNode
        def classNode = nodes[1] as ClassNode

        def includeTransient = getAnnotationValue(annotationNode, "includeTransient", false)
        def dateFormat = getAnnotationValue(annotationNode, "dateFormat", "yyyy-MM-dd HH:mm:ss")

        addSerializationMethods(classNode, includeTransient, dateFormat, sourceUnit)
    }

    def addSerializationMethods(ClassNode classNode, boolean includeTransient, String dateFormat, SourceUnit sourceUnit) {
        // 实现Serializable接口
        classNode.addInterface(ClassHelper.make(Serializable))

        // 添加writeObject方法
        def writeObjectMethod = createWriteObjectMethod(classNode, includeTransient, dateFormat)
        classNode.addMethod(writeObjectMethod)

        // 添加readObject方法
        def readObjectMethod = createReadObjectMethod(classNode, includeTransient, dateFormat)
        classNode.addMethod(readObjectMethod)

        // 添加序列化ID
        def serialVersionUID = createSerialVersionUID(classNode)
        classNode.addField(serialVersionUID)
    }

    def createWriteObjectMethod(ClassNode classNode, boolean includeTransient, String dateFormat) {
        def parameters = [
            new Parameter(ClassHelper.make(ObjectOutputStream), "out")
        ] as Parameter[]

        def statements = []

        // 获取所有需要序列化的字段
        def fields = classNode.fields.findAll { field ->
            !field.isStatic() && !field.isSynthetic() &&
            (includeTransient || !field.isTransient())
        }

        // 写入每个字段
        fields.each { field ->
            def fieldName = field.name
            def fieldType = field.type

            if (fieldType == ClassHelper.make(Date)) {
                // 日期字段特殊处理
                def dateFormatExpr = new ConstantExpression(dateFormat)
                def simpleDateFormat = new ConstructorCallExpression(
                    new ClassNode("java.text.SimpleDateFormat", 0, null),
                    new ArgumentListExpression([dateFormatExpr])
                )

                def formattedDate = new MethodCallExpression(
                    simpleDateFormat,
                    "format",
                    new ArgumentListExpression([new VariableExpression(fieldName)])
                )

                statements.add(new ExpressionStatement(
                    new MethodCallExpression(
                        new VariableExpression("out"),
                        "writeObject",
                        new ArgumentListExpression([formattedDate])
                    )
                ))
            } else {
                // 普通字段
                statements.add(new ExpressionStatement(
                    new MethodCallExpression(
                        new VariableExpression("out"),
                        "writeObject",
                        new ArgumentListExpression([new VariableExpression(fieldName)])
                    )
                ))
            }
        }

        def block = new BlockStatement(statements, null)
        return new MethodNode(
            "writeObject",
            ACC_PRIVATE,
            ClassHelper.make(Void.TYPE),
            parameters,
            [] as ClassNode[],
            block
        )
    }

    def createReadObjectMethod(ClassNode classNode, boolean includeTransient, String dateFormat) {
        def parameters = [
            new Parameter(ClassHelper.make(ObjectInputStream), "in")
        ] as Parameter[]

        def statements = []

        // 获取所有需要反序列化的字段
        def fields = classNode.fields.findAll { field ->
            !field.isStatic() && !field.isSynthetic() &&
            (includeTransient || !field.isTransient())
        }

        // 读取每个字段
        fields.each { field ->
            def fieldName = field.name
            def fieldType = field.type

            if (fieldType == ClassHelper.make(Date)) {
                // 日期字段特殊处理
                def dateFormatExpr = new ConstantExpression(dateFormat)
                def simpleDateFormat = new ConstructorCallExpression(
                    new ClassNode("java.text.SimpleDateFormat", 0, null),
                    new ArgumentListExpression([dateFormatExpr])
                )

                def readValue = new MethodCallExpression(
                    new VariableExpression("in"),
                    "readObject",
                    new ArgumentListExpression()
                )

                def parsedDate = new MethodCallExpression(
                    simpleDateFormat,
                    "parse",
                    new ArgumentListExpression([readValue])
                )

                statements.add(new ExpressionStatement(
                    new BinaryExpression(
                        new VariableExpression(fieldName),
                        Token.newSymbol("=", -1, -1),
                        parsedDate
                    )
                ))
            } else {
                // 普通字段
                def readValue = new MethodCallExpression(
                    new VariableExpression("in"),
                    "readObject",
                    new ArgumentListExpression()
                )

                statements.add(new ExpressionStatement(
                    new BinaryExpression(
                        new VariableExpression(fieldName),
                        Token.newSymbol("=", -1, -1),
                        new CastExpression(field.type, readValue)
                    )
                ))
            }
        }

        def block = new BlockStatement(statements, null)
        return new MethodNode(
            "readObject",
            ACC_PRIVATE,
            ClassHelper.make(Void.TYPE),
            parameters,
            [] as ClassNode[],
            block
        )
    }

    def createSerialVersionUID(ClassNode classNode) {
        return new FieldNode(
            "serialVersionUID",
            ACC_PRIVATE | ACC_STATIC | ACC_FINAL,
            ClassHelper.make(Long),
            classNode,
            new ConstantExpression(1L)
        )
    }

    def getAnnotationValue(AnnotationNode annotationNode, String key, def defaultValue) {
        def value = annotationNode.getMember(key)
        return value?.value ?: defaultValue
    }
}
```

## 5. AST转换性能优化

### 5.1 AST节点缓存机制

```groovy
class ASTNodeCache {
    private static final Map<String, ASTNode> nodeCache = new ConcurrentHashMap<>()

    static ASTNode getCachedNode(String key, Closure<ASTNode> nodeCreator) {
        return nodeCache.computeIfAbsent(key) { nodeCreator.call() }
    }

    static void clearCache() {
        nodeCache.clear()
    }

    static int getCacheSize() {
        return nodeCache.size()
    }
}

class OptimizedASTTransformation implements ASTTransformation {
    @Override
    void visit(ASTNode[] nodes, SourceUnit sourceUnit) {
        def cacheKey = generateCacheKey(nodes[1])

        // 使用缓存的AST节点
        def cachedNode = ASTNodeCache.getCachedNode(cacheKey) {
            createOptimizedASTNode(nodes[0], nodes[1], sourceUnit)
        }

        applyOptimizedNode(cachedNode, nodes[1], sourceUnit)
    }

    def generateCacheKey(AnnotatedNode annotatedNode) {
        def keyBuilder = new StringBuilder()

        if (annotatedNode instanceof ClassNode) {
            keyBuilder.append("CLASS_")
            keyBuilder.append(annotatedNode.name)
        } else if (annotatedNode instanceof MethodNode) {
            keyBuilder.append("METHOD_")
            keyBuilder.append(annotatedNode.declaringClass.name)
            keyBuilder.append("_")
            keyBuilder.append(annotatedNode.name)
        }

        return keyBuilder.toString()
    }

    def createOptimizedASTNode(AnnotationNode annotationNode, AnnotatedNode annotatedNode, SourceUnit sourceUnit) {
        // 创建优化的AST节点
        // 这里实现具体的AST节点创建逻辑
        return createTransformationNode(annotationNode, annotatedNode, sourceUnit)
    }

    def applyOptimizedNode(ASTNode optimizedNode, AnnotatedNode targetNode, SourceUnit sourceUnit) {
        // 应用优化的AST节点到目标节点
        // 这里实现具体的节点应用逻辑
    }

    def createTransformationNode(AnnotationNode annotationNode, AnnotatedNode annotatedNode, SourceUnit sourceUnit) {
        // 实现具体的AST节点创建逻辑
        return new BlockStatement([], null)
    }
}
```

### 5.2 编译时条件优化

```groovy
@Retention(RetentionPolicy.SOURCE)
@Target([ElementType.METHOD, ElementType.TYPE])
@GroovyASTTransformation(phase = CompilePhase.SEMANTIC_ANALYSIS)
@interface Conditional {
    String condition() default ""
    boolean debug() default false
}

@GroovyASTTransformation(phase = CompilePhase.SEMANTIC_ANALYSIS)
class ConditionalASTTransformation implements ASTTransformation {
    @Override
    void visit(ASTNode[] nodes, SourceUnit sourceUnit) {
        def annotationNode = nodes[0] as AnnotationNode
        def annotatedNode = nodes[1] as AnnotatedNode

        def condition = getAnnotationValue(annotationNode, "condition", "")
        def debug = getAnnotationValue(annotationNode, "debug", false)

        if (evaluateCondition(condition, sourceUnit)) {
            applyTransformation(annotationNode, annotatedNode, sourceUnit, debug)
        } else {
            if (annotatedNode instanceof MethodNode) {
                removeMethod(annotatedNode as MethodNode)
            }
        }
    }

    def evaluateCondition(String condition, SourceUnit sourceUnit) {
        if (!condition) return true

        try {
            // 这里可以集成更复杂的条件评估逻辑
            return evaluateCompileTimeCondition(condition, sourceUnit)
        } catch (Exception e) {
            sourceUnit.addError(new SyntaxException(
                "条件表达式解析错误: ${e.message}",
                null, null
            ))
            return false
        }
    }

    def evaluateCompileTimeCondition(String condition, SourceUnit sourceUnit) {
        // 简单的条件表达式解析
        if (condition.contains("==")) {
            def parts = condition.split("==")
            def left = parts[0].trim()
            def right = parts[1].trim()
            return left == right
        } else if (condition.contains("!=")) {
            def parts = condition.split("!=")
            def left = parts[0].trim()
            def right = parts[1].trim()
            return left != right
        }

        // 默认返回true
        return true
    }

    def applyTransformation(AnnotationNode annotationNode, AnnotatedNode annotatedNode, SourceUnit sourceUnit, boolean debug) {
        if (debug) {
            def debugStatement = new ExpressionStatement(
                new MethodCallExpression(
                    new VariableExpression("System.out"),
                    "println",
                    new ArgumentListExpression([
                        new ConstantExpression("条件编译: ${annotatedNode instanceof MethodNode ? (annotatedNode as MethodNode).name : (annotatedNode as ClassNode).name}")
                    ])
                )
            )

            if (annotatedNode instanceof MethodNode) {
                def methodNode = annotatedNode as MethodNode
                def statements = methodNode.code.statements
                statements.add(0, debugStatement)
            }
        }
    }

    def removeMethod(MethodNode methodNode) {
        methodNode.declaringClass.removeMethod(methodNode)
    }

    def getAnnotationValue(AnnotationNode annotationNode, String key, def defaultValue) {
        def value = annotationNode.getMember(key)
        return value?.value ?: defaultValue
    }
}
```

## 6. AST转换调试与测试

### 6.1 AST转换调试工具

```groovy
class ASTTransformationDebugger {
    private SourceUnit sourceUnit
    private PrintWriter output

    ASTTransformationDebugger(SourceUnit sourceUnit, PrintWriter output) {
        this.sourceUnit = sourceUnit
        this.output = output
    }

    def printAST(ASTNode node, int indent = 0) {
        def prefix = "  " * indent
        output.println("${prefix}${node.class.simpleName}: ${getNodeInfo(node)}")

        // 打印子节点
        node.getClass().methods.each { method ->
            if (method.name.startsWith("get") && method.returnType == List) {
                try {
                    def children = method.invoke(node)
                    if (children) {
                        output.println("${prefix}  ${method.name[3..-1]}:")
                        children.each { child ->
                            printAST(child, indent + 2)
                        }
                    }
                } catch (Exception e) {
                    // 忽略访问异常
                }
            }
        }
    }

    def getNodeInfo(ASTNode node) {
        switch (node) {
            case ClassNode:
                return "Class: ${(node as ClassNode).name}"
            case MethodNode:
                def method = node as MethodNode
                return "Method: ${method.name}(${method.parameters.collect { it.type.name }.join(', ')})"
            case FieldNode:
                def field = node as FieldNode
                return "Field: ${field.name} (${field.type.name})"
            case AnnotationNode:
                def annotation = node as AnnotationNode
                return "Annotation: @${annotation.classNode.name}"
            default:
                return node.toString()
        }
    }

    def validateAST(ASTNode node) {
        def errors = []

        validateNodeStructure(node, errors)
        validateMethodSignatures(node, errors)
        validateTypeCompatibility(node, errors)

        if (errors) {
            output.println("AST验证错误:")
            errors.each { error ->
                output.println("  - $error")
            }
        } else {
            output.println("AST验证通过")
        }

        return errors.isEmpty()
    }

    def validateNodeStructure(ASTNode node, List<String> errors) {
        // 验证节点结构的完整性
        if (node instanceof MethodNode) {
            def method = node as MethodNode
            if (!method.returnType) {
                errors.add("方法 ${method.name} 缺少返回类型")
            }
            if (!method.parameters) {
                // 参数可以为空
            }
        }
    }

    def validateMethodSignatures(ASTNode node, List<String> errors) {
        if (node instanceof ClassNode) {
            def classNode = node as ClassNode
            def methodSignatures = [:]

            classNode.methods.each { method ->
                def signature = "${method.name}(${method.parameters.collect { it.type.name }.join(', ')})"
                if (methodSignatures.containsKey(signature)) {
                    errors.add("重复的方法签名: $signature")
                }
                methodSignatures[signature] = method
            }
        }
    }

    def validateTypeCompatibility(ASTNode node, List<String> errors) {
        // 验证类型兼容性
        if (node instanceof MethodNode) {
            def method = node as MethodNode
            if (method.returnType && method.returnType.name == "void") {
                // 检查方法体是否包含return语句
                if (method.code && containsReturnStatement(method.code)) {
                    errors.add("void方法 ${method.name} 包含return语句")
                }
            }
        }
    }

    def containsReturnStatement(Statement statement) {
        if (statement instanceof ReturnStatement) {
            return true
        }

        if (statement instanceof BlockStatement) {
            return statement.statements.any { containsReturnStatement(it) }
        }

        return false
    }
}
```

### 6.2 AST转换单元测试

```groovy
class ASTTransformationTest {
    def testTransformation(Class<? extends ASTTransformation> transformationClass, String sourceCode) {
        def sourceUnit = new SourceUnit("TestScript", sourceCode, new CompilerConfiguration(), null, null)
        def ast = new AstBuilder().buildFromString(sourceCode)

        def transformation = transformationClass.newInstance()
        def nodes = ast.findAll { it instanceof AnnotatedNode }

        nodes.each { node ->
            if (node instanceof AnnotatedNode) {
                def annotation = node.annotations.find { it.classNode.name == transformationClass.simpleName.replace("ASTTransformation", "") }
                if (annotation) {
                    transformation.visit([annotation, node] as ASTNode[], sourceUnit)
                }
            }
        }

        return sourceUnit
    }

    def assertMethodAdded(ClassNode classNode, String methodName, int parameterCount) {
        def method = classNode.getMethod(methodName, new Parameter[parameterCount])
        assert method != null : "方法 $methodName 未找到"
        return method
    }

    def assertFieldAdded(ClassNode classNode, String fieldName) {
        def field = classNode.fields.find { it.name == fieldName }
        assert field != null : "字段 $fieldName 未找到"
        return field
    }

    def assertCodeContains(Statement statement, String expectedCode) {
        def code = statement.toString()
        assert code.contains(expectedCode) : "代码中未包含: $expectedCode"
    }

    def testAutoImplementTransformation() {
        def sourceCode = '''
            @AutoImplement(interfaces = [Runnable.class])
            class TestClass {
                // 自动实现Runnable接口
            }
        '''

        def sourceUnit = testTransformation(AutoImplementASTTransformation, sourceCode)

        def classNode = sourceUnit.AST.classes.find { it.name == "TestClass" }
        assert classNode != null : "TestClass 未找到"

        def runMethod = assertMethodAdded(classNode, "run", 0)
        assert runMethod.exceptions.any { it.name == "UnsupportedOperationException" }
    }

    def testCachedTransformation() {
        def sourceCode = '''
            class CacheService {
                @Cached(maxSize = 100, expireTime = 60000)
                def expensiveOperation(String input) {
                    Thread.sleep(1000)
                    return "Processed: $input"
                }
            }
        '''

        def sourceUnit = testTransformation(CachedASTTransformation, sourceCode)

        def classNode = sourceUnit.AST.classes.find { it.name == "CacheService" }
        assert classNode != null : "CacheService 未找到"

        def cacheField = assertFieldAdded(classNode, "_cache_expensiveOperation")
        assert cacheField.type.name == "java.util.concurrent.ConcurrentHashMap"
    }
}
```

## 7. 高级AST转换应用案例

### 7.1 自动生成Builder模式

```groovy
@Retention(RetentionPolicy.SOURCE)
@Target(ElementType.TYPE)
@GroovyASTTransformation(phase = CompilePhase.SEMANTIC_ANALYSIS)
interface Builder {}

@GroovyASTTransformation(phase = CompilePhase.SEMANTIC_ANALYSIS)
class BuilderASTTransformation implements ASTTransformation {
    @Override
    void visit(ASTNode[] nodes, SourceUnit sourceUnit) {
        def annotationNode = nodes[0] as AnnotationNode
        def classNode = nodes[1] as ClassNode

        createBuilderClass(classNode, sourceUnit)
    }

    def createBuilderClass(ClassNode originalClass, SourceUnit sourceUnit) {
        def builderClassName = "${originalClass.name}Builder"
        def builderClass = new ClassNode(
            builderClassName,
            ACC_PUBLIC,
            ClassHelper.OBJECT_TYPE,
            [] as ClassNode[],
            null
        )

        // 添加字段
        originalClass.properties.each { property ->
            def fieldNode = new FieldNode(
                property.name,
                ACC_PRIVATE,
                property.type,
                builderClass,
                null
            )
            builderClass.addField(fieldNode)

            // 添加setter方法
            def setterMethod = createSetterMethod(builderClass, property.name, property.type)
            builderClass.addMethod(setterMethod)
        }

        // 添加build方法
        def buildMethod = createBuildMethod(builderClass, originalClass)
        builderClass.addMethod(buildMethod)

        // 添加构造函数
        def constructor = createConstructor(builderClass)
        builderClass.addMethod(constructor)

        // 将builder类添加到源单元
        sourceUnit.AST.addClass(builderClass)
    }

    def createSetterMethod(ClassNode builderClass, String propertyName, ClassNode propertyType) {
        def methodName = "with${propertyName.capitalize()}"
        def parameter = new Parameter(propertyType, propertyName)

        def setStatement = new ExpressionStatement(
            new BinaryExpression(
                new VariableExpression(propertyName),
                Token.newSymbol("=", -1, -1),
                new VariableExpression(propertyName)
            )
        )

        def returnStatement = new ReturnStatement(
            new VariableExpression("this")
        )

        def block = new BlockStatement([setStatement, returnStatement], null)

        return new MethodNode(
            methodName,
            ACC_PUBLIC,
            builderClass,
            [parameter] as Parameter[],
            [] as ClassNode[],
            block
        )
    }

    def createBuildMethod(ClassNode builderClass, ClassNode originalClass) {
        def constructorCall = new ConstructorCallExpression(
            originalClass,
            new ArgumentListExpression()
        )

        // 为每个属性设置值
        def setProperties = originalClass.properties.collect { property ->
            new ExpressionStatement(
                new BinaryExpression(
                    new PropertyExpression(
                        constructorCall,
                        property.name
                    ),
                    Token.newSymbol("=", -1, -1),
                    new VariableExpression(property.name)
                )
            )
        }

        def block = new BlockStatement(
            [new ExpressionStatement(constructorCall)] + setProperties + [new ReturnStatement(constructorCall)],
            null
        )

        return new MethodNode(
            "build",
            ACC_PUBLIC,
            originalClass,
            [] as Parameter[],
            [] as ClassNode[],
            block
        )
    }

    def createConstructor(ClassNode builderClass) {
        return new MethodNode(
            "<init>",
            ACC_PUBLIC,
            ClassHelper.make(Void.TYPE),
            [] as Parameter[],
            [] as ClassNode[],
            new BlockStatement([
                new ExpressionStatement(
                    new ConstructorCallExpression(
                        ClassHelper.OBJECT_TYPE,
                        new ArgumentListExpression()
                    )
                )
            ], null)
        )
    }
}
```

### 7.2 自动验证AST转换

```groovy
@Retention(RetentionPolicy.SOURCE)
@Target([ElementType.FIELD, ElementType.PARAMETER])
@GroovyASTTransformation(phase = CompilePhase.SEMANTIC_ANALYSIS)
@interface Validate {
    String pattern() default ""
    int min() default Integer.MIN_VALUE
    int max() default Integer.MAX_VALUE
    boolean required() default true
}

@GroovyASTTransformation(phase = CompilePhase.SEMANTIC_ANALYSIS)
class ValidateASTTransformation implements ASTTransformation {
    @Override
    def visit(ASTNode[] nodes, SourceUnit sourceUnit) {
        def annotationNode = nodes[0] as AnnotationNode
        def annotatedNode = nodes[1] as AnnotatedNode

        if (annotatedNode instanceof FieldNode) {
            addValidationToField(annotatedNode as FieldNode, annotationNode, sourceUnit)
        } else if (annotatedNode instanceof Parameter) {
            addValidationToParameter(annotatedNode as Parameter, annotationNode, sourceUnit)
        }
    }

    def addValidationToField(FieldNode fieldNode, AnnotationNode annotationNode, SourceUnit sourceUnit) {
        def className = fieldNode.declaringClass.name
        def fieldName = fieldNode.name
        def fieldType = fieldNode.type

        // 创建验证方法
        def validateMethod = createFieldValidationMethod(fieldNode, annotationNode, sourceUnit)
        fieldNode.declaringClass.addMethod(validateMethod)

        // 修改setter方法
        modifySetterMethod(fieldNode, annotationNode, sourceUnit)
    }

    def addValidationToParameter(Parameter parameter, AnnotationNode annotationNode, SourceUnit sourceUnit) {
        def methodNode = findMethodContainingParameter(parameter, sourceUnit)
        if (methodNode) {
            addParameterValidation(methodNode, parameter, annotationNode, sourceUnit)
        }
    }

    def createFieldValidationMethod(FieldNode fieldNode, AnnotationNode annotationNode, SourceUnit sourceUnit) {
        def methodName = "validate${fieldNode.name.capitalize()}"
        def parameter = new Parameter(fieldNode.type, "value")

        def validationStatements = createValidationStatements(
            new VariableExpression("value"),
            annotationNode,
            fieldNode.name,
            sourceUnit
        )

        def block = new BlockStatement(validationStatements, null)

        return new MethodNode(
            methodName,
            ACC_PRIVATE,
            ClassHelper.make(Void.TYPE),
            [parameter] as Parameter[],
            [] as ClassNode[],
            block
        )
    }

    def createValidationStatements(Expression valueExpression, AnnotationNode annotationNode,
                                 String fieldName, SourceUnit sourceUnit) {
        def statements = []
        def pattern = getAnnotationValue(annotationNode, "pattern", "")
        def min = getAnnotationValue(annotationNode, "min", Integer.MIN_VALUE)
        def max = getAnnotationValue(annotationNode, "max", Integer.MAX_VALUE)
        def required = getAnnotationValue(annotationNode, "required", true)

        if (required) {
            def nullCheck = new IfStatement(
                new BooleanExpression(
                    new BinaryExpression(
                        valueExpression,
                        Token.newSymbol("==", -1, -1),
                        new ConstantExpression(null)
                    )
                ),
                new BlockStatement([
                    new ThrowStatement(
                        new ConstructorCallExpression(
                            ClassHelper.make(IllegalArgumentException),
                            new ArgumentListExpression([
                                new ConstantExpression("$fieldName 不能为空")
                            ])
                        )
                    )
                ], null),
                null
            )
            statements.add(nullCheck)
        }

        if (pattern) {
            def patternCheck = new IfStatement(
                new BooleanExpression(
                    new BinaryExpression(
                        new MethodCallExpression(
                            valueExpression,
                            "matches",
                            new ArgumentListExpression([new ConstantExpression(pattern)])
                        ),
                        Token.newSymbol("==", -1, -1),
                        new ConstantExpression(false)
                    )
                ),
                new BlockStatement([
                    new ThrowStatement(
                        new ConstructorCallExpression(
                            ClassHelper.make(IllegalArgumentException),
                            new ArgumentListExpression([
                                new ConstantExpression("$fieldName 格式不正确，需要匹配: $pattern")
                            ])
                        )
                    )
                ], null),
                null
            )
            statements.add(patternCheck)
        }

        if (min != Integer.MIN_VALUE) {
            def minCheck = new IfStatement(
                new BooleanExpression(
                    new BinaryExpression(
                        new BinaryExpression(
                            valueExpression,
                            Token.newSymbol("<", -1, -1),
                            new ConstantExpression(min)
                        ),
                        Token.newSymbol("||", -1, -1),
                        new BinaryExpression(
                            valueExpression,
                            Token.newSymbol("==", -1, -1),
                            new ConstantExpression(null)
                        )
                    )
                ),
                new BlockStatement([
                    new ThrowStatement(
                        new ConstructorCallExpression(
                            ClassHelper.make(IllegalArgumentException),
                            new ArgumentListExpression([
                                new ConstantExpression("$fieldName 不能小于 $min")
                            ])
                        )
                    )
                ], null),
                null
            )
            statements.add(minCheck)
        }

        if (max != Integer.MAX_VALUE) {
            def maxCheck = new IfStatement(
                new BooleanExpression(
                    new BinaryExpression(
                        new BinaryExpression(
                            valueExpression,
                            Token.newSymbol(">", -1, -1),
                            new ConstantExpression(max)
                        ),
                        Token.newSymbol("||", -1, -1),
                        new BinaryExpression(
                            valueExpression,
                            Token.newSymbol("==", -1, -1),
                            new ConstantExpression(null)
                        )
                    )
                ),
                new BlockStatement([
                    new ThrowStatement(
                        new ConstructorCallExpression(
                            ClassHelper.make(IllegalArgumentException),
                            new ArgumentListExpression([
                                new ConstantExpression("$fieldName 不能大于 $max")
                            ])
                        )
                    )
                ], null),
                null
            )
            statements.add(maxCheck)
        }

        return statements
    }

    def modifySetterMethod(FieldNode fieldNode, AnnotationNode annotationNode, SourceUnit sourceUnit) {
        def setterName = "set${fieldNode.name.capitalize()}"
        def setterMethod = fieldNode.declaringClass.getMethod(setterName, [fieldNode.type] as Parameter[])

        if (setterMethod) {
            def validationCall = new MethodCallExpression(
                new VariableExpression("this"),
                "validate${fieldNode.name.capitalize()}",
                new ArgumentListExpression([new VariableExpression(setterMethod.parameters[0].name)])
            )

            def originalStatements = setterMethod.code.statements
            def newStatements = [new ExpressionStatement(validationCall)] + originalStatements

            setterMethod.code = new BlockStatement(newStatements, setterMethod.code.variableScope)
        }
    }

    def addParameterValidation(MethodNode methodNode, Parameter parameter,
                               AnnotationNode annotationNode, SourceUnit sourceUnit) {
        def validationStatements = createValidationStatements(
            new VariableExpression(parameter.name),
            annotationNode,
            parameter.name,
            sourceUnit
        )

        def originalStatements = methodNode.code.statements
        def newStatements = validationStatements + originalStatements

        methodNode.code = new BlockStatement(newStatements, methodNode.code.variableScope)
    }

    def findMethodContainingParameter(Parameter parameter, SourceUnit sourceUnit) {
        return sourceUnit.AST.classes*.methods.flatten().find { method ->
            method.parameters.any { it.name == parameter.name }
        }
    }

    def getAnnotationValue(AnnotationNode annotationNode, String key, def defaultValue) {
        def value = annotationNode.getMember(key)
        return value?.value ?: defaultValue
    }
}
```

## 8. 总结

Groovy AST转换是一项极其强大的编译时元编程技术，通过本文的深入解析，我们了解了：

1. **AST转换基础**：理解AST节点的结构和操作方式
2. **高级转换模式**：掌握复杂的AST转换实现技巧
3. **性能优化**：学会如何优化AST转换的性能
4. **调试与测试**：建立完善的AST转换测试体系
5. **实际应用**：通过实际案例展示AST转换的强大能力

AST转换的核心优势在于：

- **编译时优化**：在编译期间完成代码转换，零运行时开销
- **代码生成**：自动生成重复性代码，提高开发效率
- **类型安全**：在编译时确保类型安全
- **性能优化**：实现传统反射无法达到的性能

掌握AST转换技术将使你能够构建出高度优化、类型安全且功能强大的Groovy应用程序。在实际项目中，建议根据具体需求设计合适的AST转换策略，并配合完善的测试体系确保代码质量。