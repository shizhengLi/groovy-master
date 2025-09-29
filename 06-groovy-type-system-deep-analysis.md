# Groovy类型系统深度剖析：动态与静态的完美结合

## 引言

Groovy的类型系统是其最独特和强大的特性之一。它既支持动态类型的灵活性，又提供了静态类型的安全性和性能优势。本文将深入探讨Groovy类型系统的内部机制、类型推断、静态编译优化以及类型转换的底层实现原理。

## 1. Groovy类型系统基础

### 1.1 类型系统架构

```groovy
// Groovy类型系统层次结构
class TypeSystemHierarchy {
    // 1. 基本类型层次
    static void printTypeHierarchy() {
        println "Groovy类型系统层次结构:"
        println "java.lang.Object"
        println "├── groovy.lang.GroovyObject"
        println "│   ├── groovy.lang.GroovyObjectSupport"
        println "│   └── 所有Groovy类"
        println "├── java.lang.Number"
        println "│   ├── java.lang.Integer"
        println "│   ├── java.lang.Double"
        println "│   ├── java.lang.BigDecimal"
        println "│   └── groovy.lang.IntRange (通过元类)"
        println "├── java.lang.String"
        println "├── java.util.Collection"
        println "│   ├── java.util.List"
        println "│   ├── java.util.Set"
        println "│   └── java.util.Map"
        println "└── java.lang.Object[] (数组)"
    }

    // 2. 类型推断示例
    static def typeInferenceExamples() {
        // Groovy的类型推断
        def dynamicVar = "Hello"  // 推断为String
        def listVar = [1, 2, 3]  // 推断为ArrayList<Integer>
        def mapVar = [a: 1, b: 2] // 推断为LinkedHashMap<String, Integer>

        println "dynamicVar类型: ${dynamicVar.getClass().simpleName}"
        println "listVar类型: ${listVar.getClass().simpleName}"
        println "mapVar类型: ${mapVar.getClass().simpleName}"
    }

    // 3. 类型转换机制
    static def typeConversionMechanism() {
        // 自动类型转换
        def intNum = 42
        def doubleNum = intNum  // int -> double
        def stringNum = intNum.toString()  // int -> String

        println "int: $intNum (${intNum.getClass().simpleName})"
        println "double: $doubleNum (${doubleNum.getClass().simpleName})"
        println "string: $stringNum (${stringNum.getClass().simpleName})"

        // 强制类型转换
        def obj = "Hello"
        def str = obj as String  // 安全转换
        def num = obj as Integer // 会抛出ClassCastException

        println "安全转换: $str"
        try {
            println "强制转换: $num"
        } catch (e) {
            println "强制转换失败: ${e.message}"
        }
    }
}
```

### 1.2 动态类型与静态类型

```groovy
class DynamicVsStatic {
    // 1. 动态类型示例
    static def dynamicTypingExample() {
        def dynamicVar = "Hello"
        println "初始类型: ${dynamicVar.getClass().simpleName}"

        dynamicVar = 42
        println "改变后类型: ${dynamicVar.getClass().simpleName}"

        dynamicVar = [1, 2, 3]
        println "再次改变后类型: ${dynamicVar.getClass().simpleName}"

        // 动态方法调用
        def result = dynamicMethod("动态调用")
        println "动态方法调用结果: $result"
    }

    static def dynamicMethod(arg) {
        return "接收到参数: $arg, 类型: ${arg.getClass().simpleName}"
    }

    // 2. 静态类型示例
    @CompileStatic
    static String staticTypingExample(String input) {
        // 静态类型检查，编译时会验证类型
        return input.toUpperCase() + " (静态类型)"
    }

    // 3. 类型推断示例
    @TypeChecked
    static def typeInferenceExample() {
        def name = "张三"  // 推断为String
        def age = 25       // 推断为Integer
        def scores = [90, 85, 95]  // 推断为List<Integer>

        println "姓名: $name (${name.getClass().simpleName})"
        println "年龄: $age (${age.getClass().simpleName})"
        println "成绩: $scores (${scores.getClass().simpleName})"

        return "类型推断完成"
    }

    // 4. 混合类型示例
    static class MixedTypeContainer {
        def dynamicField = "动态字段"
        String staticField = "静态字段"
        def inferredField = "推断字段"  // 编译时推断为String

        def dynamicMethod(arg) {
            return "动态方法: $arg"
        }

        String staticMethod(String arg) {
            return "静态方法: $arg"
        }

        @CompileStatic
        def compiledMethod(String arg) {
            return "编译方法: $arg"
        }
    }
}
```

## 2. 类型推断机制

### 2.1 编译时类型推断

```groovy
class CompileTimeTypeInference {
    // 1. 基本类型推断
    @TypeChecked
    static def basicTypeInference() {
        def intVar = 42           // 推断为Integer
        def doubleVar = 3.14     // 推断为Double
        def stringVar = "Hello"   // 推断为String
        def boolVar = true        // 推断为Boolean

        // 使用推断类型
        def result = intVar + doubleVar  // 推断为Double
        return "基本类型推断: $result (${result.getClass().simpleName})"
    }

    // 2. 集合类型推断
    @TypeChecked
    static def collectionTypeInference() {
        def stringList = ["A", "B", "C"]           // List<String>
        def intSet = [1, 2, 3] as Set             // Set<Integer>
        def stringMap = [key1: "value1", key2: "value2"]  // Map<String, String>

        // 类型安全操作
        def upperCaseList = stringList.collect { it.toUpperCase() }
        def sumSet = intSet.sum()

        return """
            集合类型推断:
            List<String>: $upperCaseList
            Set<Integer> sum: $sumSet
            Map<String, String>: $stringMap
        """
    }

    // 3. 闭包参数类型推断
    @TypeChecked
    static def closureTypeInference() {
        def numbers = [1, 2, 3, 4, 5]

        // 闭包参数类型推断
        def doubled = numbers.collect { it * 2 }  // it推断为Integer
        def filtered = numbers.findAll { it > 3 }  // it推断为Integer

        return """
            闭包类型推断:
            原始: $numbers
            双倍: $doubled
            过滤: $filtered
        """
    }

    // 4. 泛型类型推断
    @TypeChecked
    static def genericTypeInference() {
        def stringList = new ArrayList<String>()
        stringList.add("Hello")
        stringList.add("World")

        def integerList = new ArrayList<>()
        integerList.add(1)
        integerList.add(2)

        return """
            泛型类型推断:
            String List: $stringList
            Integer List: $integerList
        """
    }
}
```

### 2.2 运行时类型推断

```groovy
class RuntimeTypeInference {
    // 1. 运行时类型检查
    static def runtimeTypeChecking() {
        def variables = ["Hello", 42, 3.14, true, [1, 2, 3]]

        variables.each { var ->
            println "变量: $var, 类型: ${var.getClass().simpleName}"

            // 运行时类型检查
            if (var instanceof String) {
                println "  - 是字符串，长度: ${var.length()}"
            } else if (var instanceof Number) {
                println "  - 是数字，平方: ${var * var}"
            } else if (var instanceof List) {
                println "  - 是列表，大小: ${var.size()}"
            }
        }
    }

    // 2. 动态类型转换
    static def dynamicTypeConversion() {
        def conversions = [
            "42" as Integer,
            "3.14" as Double,
            "true" as Boolean,
            42 as String,
            3.14 as Integer
        ]

        conversions.each { converted ->
            println "转换结果: $converted (${converted.getClass().simpleName})"
        }
    }

    // 3. 多态类型处理
    static def polymorphicTypeHandling() {
        def objects = [
            new Circle(5),
            new Rectangle(4, 6),
            new Triangle(3, 4, 5)
        ]

        objects.each { obj ->
            println "形状: ${obj.getClass().simpleName}"
            println "  面积: ${obj.area()}"
            println "  周长: ${obj.perimeter()}"
        }
    }

    // 4. 类型适配器模式
    static class TypeAdapter {
        private static final Map<Class, Closure> adapters = [:]

        static {
            // 注册类型适配器
            adapters[String] = { obj -> obj.toString() }
            adapters[Integer] = { obj -> obj as Integer }
            adapters[Double] = { obj -> obj as Double }
            adapters[Boolean] = { obj -> obj as Boolean }
        }

        static def adapt(Object obj, Class targetType) {
            def adapter = adapters[targetType]
            if (adapter) {
                return adapter.call(obj)
            }
            throw new IllegalArgumentException("不支持的类型转换: ${obj.class} -> $targetType")
        }

        static def registerAdapter(Class targetType, Closure adapter) {
            adapters[targetType] = adapter
        }
    }
}

// 几何形状类
class Circle {
    def radius

    Circle(radius) {
        this.radius = radius
    }

    def area() {
        Math.PI * radius * radius
    }

    def perimeter() {
        2 * Math.PI * radius
    }
}

class Rectangle {
    def width
    def height

    Rectangle(width, height) {
        this.width = width
        this.height = height
    }

    def area() {
        width * height
    }

    def perimeter() {
        2 * (width + height)
    }
}

class Triangle {
    def a, b, c

    Triangle(a, b, c) {
        this.a = a
        this.b = b
        this.c = c
    }

    def area() {
        def s = (a + b + c) / 2
        Math.sqrt(s * (s - a) * (s - b) * (s - c))
    }

    def perimeter() {
        a + b + c
    }
}
```

## 3. 静态编译优化

### 3.1 @CompileStatic注解

```groovy
@CompileStatic
class CompileStaticOptimization {
    // 1. 基本静态编译
    def staticCompiledMethod(String input, int multiplier) {
        // 编译时类型检查和优化
        def result = input * multiplier  // 字符串重复
        return result.toUpperCase()     // 编译时确定方法存在
    }

    // 2. 静态编译的性能优势
    static def performanceComparison() {
        def iterations = 1000000

        // 动态版本
        def dynamicStart = System.currentTimeMillis()
        iterations.times {
            def result = "Hello" * 5
            result.toUpperCase()
        }
        def dynamicEnd = System.currentTimeMillis()

        // 静态版本
        def staticStart = System.currentTimeMillis()
        iterations.times {
            def result = staticCompiledMethod("Hello", 5)
        }
        def staticEnd = System.currentTimeMillis()

        println """
            性能比较 ($iterations 次迭代):
            动态版本: ${dynamicEnd - dynamicStart}ms
            静态版本: ${staticEnd - staticStart}ms
            性能提升: ${((dynamicEnd - dynamicStart) / (staticEnd - staticStart) as float).round(2)}x
        """
    }

    // 3. 静态编译的类型安全
    def typeSafetyExample() {
        def stringList = ["Hello", "World"]

        // 编译时检查
        stringList.each { String item ->
            println item.toUpperCase()  // 编译时知道item是String
        }

        // 以下代码会在编译时报错
        // stringList.each { item ->
        //     println item.nonExistentMethod()  // 编译错误
        // }
    }

    // 4. 静态编译的闭包优化
    def optimizedClosureProcessing(List<String> strings) {
        // 静态编译的闭包具有更好的性能
        def result = strings.collect { String s ->
            s.toUpperCase().reverse()
        }

        return result
    }

    // 5. 静态编译的数学运算优化
    def optimizedMathOperations(List<Integer> numbers) {
        def sum = 0
        for (int i = 0; i < numbers.size(); i++) {
            sum += numbers[i]  // 编译时优化循环和数学运算
        }
        return sum
    }
}
```

### 3.2 @TypeChecked注解

```groovy
@TypeChecked
class TypeCheckedFeatures {
    // 1. 类型检查但不编译
    def typeCheckedOnly(String input) {
        def length = input.length()  // 编译时检查length()方法存在
        def upper = input.toUpperCase()  // 编译时检查toUpperCase()方法存在
        return "$input -> $upper (长度: $length)"
    }

    // 2. 类型推断与检查结合
    def inferenceWithChecking() {
        def name = "张三"  // 推断为String
        def age = 25       // 推断为Integer
        def hobbies = ["读书", "运动"]  // 推断为List<String>

        // 类型安全操作
        def greeting = "你好，${name}，你今年${age}岁"
        def hobbyCount = hobbies.size()

        return "$greeting，有${hobbyCount}个爱好"
    }

    // 3. 集合操作类型安全
    def safeCollectionOperations() {
        def numbers = [1, 2, 3, 4, 5]  // 推断为List<Integer>

        // 类型安全的集合操作
        def doubled = numbers.collect { it * 2 }  // it推断为Integer
        def filtered = numbers.findAll { it > 3 }  // it推断为Integer
        def sum = numbers.sum()  // 编译时知道sum()方法存在

        return "处理结果: 双倍=$doubled, 过滤=$filtered, 总和=$sum"
    }

    // 4. 自定义类型检查
    @TypeChecked(extensions = TypeCheckingExtensions.class)
    def extendedTypeChecking(Object obj) {
        // 使用扩展的类型检查
        if (obj.isString()) {
            return "字符串: $obj"
        } else if (obj.isNumber()) {
            return "数字: $obj"
        }
        return "其他类型: $obj"
    }

    // 5. 泛型类型检查
    def genericTypeSafety() {
        def stringList = new ArrayList<String>()
        stringList.add("Hello")
        stringList.add("World")

        // 编译时检查类型安全
        stringList.each { String item ->
            println item.toUpperCase()
        }

        return "泛型类型安全检查完成"
    }
}

// 类型检查扩展
class TypeCheckingExtensions extends TypeCheckingExtension {
    @Override
    Object visitMethodCall(MethodCall call) {
        def object = call.objectExpression
        def methodName = call.methodAsString

        // 添加自定义方法检查
        if (methodName == "isString") {
            addStaticType(object, String)
            return call
        } else if (methodName == "isNumber") {
            addStaticType(object, Number)
            return call
        }

        return super.visitMethodCall(call)
    }
}
```

## 4. 类型转换机制

### 4.1 自动类型转换

```groovy
class AutomaticTypeConversion {
    // 1. 数值类型转换
    static def numericConversion() {
        def conversions = [
            "整数到小数: ${42} -> ${42 as Double} (${(42 as Double).getClass().simpleName})",
            "小数到整数: ${3.14} -> ${3.14 as Integer} (${(3.14 as Integer).getClass().simpleName})",
            "整数到大整数: ${42} -> ${42 as BigInteger} (${(42 as BigInteger).getClass().simpleName})",
            "小数到大数: ${3.14} -> ${3.14 as BigDecimal} (${(3.14 as BigDecimal).getClass().simpleName})"
        ]

        return conversions.join("\n")
    }

    // 2. 字符串类型转换
    static def stringConversion() {
        def conversions = [
            "字符串到整数: \"42\" -> ${"42" as Integer}",
            "字符串到小数: \"3.14\" -> ${"3.14" as Double}",
            "字符串到布尔: \"true\" -> ${"true" as Boolean}",
            "整数到字符串: ${42} -> \"${42}\"",
            "布尔到字符串: ${true} -> \"${true}\""
        ]

        return conversions.join("\n")
    }

    // 3. 集合类型转换
    static def collectionConversion() {
        def array = [1, 2, 3] as int[]
        def list = array as List
        def set = list as Set
        def vector = list as Vector

        return """
            集合类型转换:
            数组: ${array.getClass().simpleName} -> $array
            列表: ${list.getClass().simpleName} -> $list
            集合: ${set.getClass().simpleName} -> $set
            向量: ${vector.getClass().simpleName} -> $vector
        """
    }

    // 4. 自定义类型转换
    static def customTypeConversion() {
        def person = [name: "张三", age: 25] as Person
        def map = person as Map

        return """
            自定义类型转换:
            Person -> Map: $map
            Map -> Person: $person
        """
    }
}

// 可转换的Person类
class Person {
    String name
    int age

    // 实现asType方法
    def asType(Class type) {
        if (type == Map) {
            return [name: name, age: age]
        }
        throw new ClassCastException("无法转换为 $type")
    }

    // 实现from方法
    static Person from(Map map) {
        def person = new Person()
        person.name = map.name
        person.age = map.age
        return person
    }

    String toString() {
        "Person(name: $name, age: $age)"
    }
}
```

### 4.2 强制类型转换

```groovy
class CoercionTypeConversion {
    // 1. 使用as操作符
    static def asOperatorExample() {
        def conversions = [
            "字符串转换: \"123\" as Integer -> ${"123" as Integer}",
            "布尔转换: "true" as Boolean -> ${"true" as Boolean}",
            "列表转换: [1, 2, 3] as Set -> ${[1, 2, 3] as Set}",
            "自定义转换: [name: "张三"] as Person -> ${[name: "张三"] as Person}"
        ]

        return conversions.join("\n")
    }

    // 2. 类型转换异常处理
    static def safeTypeConversion() {
        def results = []

        // 安全转换
        results.add("安全转换 - 整数: ${safeConvert("123", Integer)}")
        results.add("安全转换 - 字符串: ${safeConvert(123, String)}")
        results.add("安全转换 - 布尔: ${safeConvert("true", Boolean)}")

        // 危险转换
        results.add("危险转换 - 失败: ${unsafeConvert("abc", Integer)}")

        return results.join("\n")
    }

    // 3. 类型转换链
    static def conversionChain() {
        def original = "123.45"
        def conversions = []

        // 转换链: String -> Double -> Integer -> String
        def step1 = original as Double
        def step2 = step1 as Integer
        def step3 = step2 as String

        conversions.add("原始: $original (${original.getClass().simpleName})")
        conversions.add("步骤1: $step1 (${step1.getClass().simpleName})")
        conversions.add("步骤2: $step2 (${step2.getClass().simpleName})")
        conversions.add("步骤3: $step3 (${step3.getClass().simpleName})")

        return conversions.join("\n")
    }

    // 4. 自定义转换器
    static def customConverters() {
        // 注册自定义转换器
        def converter = new CustomTypeConverter()

        converter.registerConverter(String, Integer) { str ->
            try {
                return Integer.parseInt(str)
            } catch (NumberFormatException e) {
                throw new TypeConversionException("无法将 '$str' 转换为整数")
            }
        }

        converter.registerConverter(String, Date) { str ->
            try {
                return Date.parse("yyyy-MM-dd", str)
            } catch (Exception e) {
                throw new TypeConversionException("无法将 '$str' 转换为日期")
            }
        }

        // 使用自定义转换器
        def converted1 = converter.convert("2023-12-25", Date)
        def converted2 = converter.convert("42", Integer)

        return """
            自定义转换器:
            "2023-12-25" -> $converted1 (${converted1.getClass().simpleName})
            "42" -> $converted2 (${converted2.getClass().simpleName})
        """
    }

    // 安全转换方法
    static def safeConvert(Object value, Class targetType) {
        try {
            return value as targetType
        } catch (Exception e) {
            return "转换失败: ${e.message}"
        }
    }

    // 不安全转换方法
    static def unsafeConvert(Object value, Class targetType) {
        try {
            return value as targetType
        } catch (Exception e) {
            throw e
        }
    }
}

// 自定义类型转换器
class CustomTypeConverter {
    private Map<Class, Closure> converters = [:]

    def registerConverter(Class sourceType, Class targetType, Closure converter) {
        def key = "${sourceType.name}->${targetType.name}"
        converters[key] = converter
    }

    def convert(Object value, Class targetType) {
        def key = "${value.getClass().name}->${targetType.name}"
        def converter = converters[key]

        if (converter) {
            return converter.call(value)
        }

        // 尝试标准转换
        return value as targetType
    }
}

// 类型转换异常
class TypeConversionException extends RuntimeException {
    TypeConversionException(String message) {
        super(message)
    }
}
```

## 5. 泛型类型系统

### 5.1 泛型基础

```groovy
class GenericsBasics {
    // 1. 泛型类定义
    static class Box<T> {
        private T value

        Box(T value) {
            this.value = value
        }

        T getValue() {
            return value
        }

        void setValue(T value) {
            this.value = value
        }

        String toString() {
            "Box<$T>[$value]"
        }
    }

    // 2. 泛型方法
    static <T> T createInstance(Class<T> type) {
        return type.newInstance()
    }

    static <T> List<T> createList(T... items) {
        return Arrays.asList(items)
    }

    // 3. 泛型接口
    static interface Processor<T> {
        T process(T input)
    }

    static class StringProcessor implements Processor<String> {
        @Override
        String process(String input) {
            return input.toUpperCase()
        }
    }

    // 4. 泛型使用示例
    static def genericsExample() {
        def stringBox = new Box<String>("Hello")
        def integerBox = new Box<Integer>(42)

        def stringInstance = createInstance(String)
        def integerList = createList(1, 2, 3, 4, 5)

        def processor = new StringProcessor()
        def processed = processor.process("hello world")

        return """
            泛型示例:
            String Box: $stringBox
            Integer Box: $integerBox
            创建实例: $stringInstance
            整数列表: $integerList
            处理结果: $processed
        """
    }
}
```

### 5.2 高级泛型特性

```groovy
class AdvancedGenerics {
    // 1. 通配符
    static def wildcardExample() {
        def stringList = ["Hello", "World"]
        def integerList = [1, 2, 3]

        // 使用通配符
        printList(stringList)
        printList(integerList)

        return "通配符示例完成"
    }

    static def printList(List<?> list) {
        println "列表内容: $list"
    }

    // 2. 有界通配符
    static def boundedWildcardExample() {
        def numbers = [1, 2, 3, 4, 5]
        def sum = sumOfList(numbers)

        return "有界通配符示例: 数字列表和为 $sum"
    }

    static def sumOfList(List<? extends Number> list) {
        def sum = 0.0
        list.each { sum += it.doubleValue() }
        return sum
    }

    // 3. 泛型数组
    static def genericArrayExample() {
        def stringArray = createArray(String, 3)
        stringArray[0] = "Hello"
        stringArray[1] = "World"
        stringArray[2] = "Groovy"

        return "泛型数组: ${Arrays.toString(stringArray)}"
    }

    static <T> T[] createArray(Class<T> type, int size) {
        return Arrays.copyOf(type.arrayType, size) as T[]
    }

    // 4. 泛型类型推断
    @TypeChecked
    static def typeInferenceExample() {
        // 编译器会推断类型
        def stringList = new ArrayList<>()  // 推断为ArrayList<String>
        stringList.add("Hello")
        stringList.add("World")

        def numberList = Arrays.asList(1, 2, 3)  // 推断为List<Integer>

        return """
            泛型类型推断:
            String List: $stringList
            Number List: $numberList
        """
    }

    // 5. 泛型约束
    static <T extends Comparable<T>> T max(T[] array) {
        if (array == null || array.length == 0) {
            return null
        }

        def max = array[0]
        for (int i = 1; i < array.length; i++) {
            if (array[i].compareTo(max) > 0) {
                max = array[i]
            }
        }
        return max
    }

    // 6. 泛型工厂模式
    static class GenericFactory<T> {
        private Class<T> type

        GenericFactory(Class<T> type) {
            this.type = type
        }

        T createInstance() {
            return type.newInstance()
        }

        List<T> createList(int size) {
            def list = new ArrayList<T>(size)
            size.times {
                list.add(createInstance())
            }
            return list
        }
    }

    // 7. 泛型工具类
    static class GenericUtils {
        static <T> void swap(List<T> list, int i, int j) {
            def temp = list[i]
            list[i] = list[j]
            list[j] = temp
        }

        static <T> List<T> filter(List<T> list, Predicate<T> predicate) {
            def result = new ArrayList<T>()
            list.each { item ->
                if (predicate.test(item)) {
                    result.add(item)
                }
            }
            return result
        }

        static <T, R> List<R> map(List<T> list, Function<T, R> mapper) {
            def result = new ArrayList<R>(list.size())
            list.each { item ->
                result.add(mapper.apply(item))
            }
            return result
        }
    }
}

// 函数式接口
interface Predicate<T> {
    boolean test(T t)
}

interface Function<T, R> {
    R apply(T t)
}
```

## 6. 类型注解与元编程

### 6.1 类型注解

```groovy
@TypeChecked
class TypeAnnotations {
    // 1. 自定义类型注解
    @Retention(RetentionPolicy.RUNTIME)
    @Target([ElementType.FIELD, ElementType.PARAMETER, ElementType.METHOD])
    @interface NotNull {
        String message() default "值不能为null"
    }

    @Retention(RetentionPolicy.RUNTIME)
    @Target([ElementType.PARAMETER])
    @interface Min {
        int value() default 0
        String message() default "值不能小于 {value}"
    }

    @Retention(RetentionPolicy.RUNTIME)
    @Target([ElementType.PARAMETER])
    @interface Max {
        int value() default Integer.MAX_VALUE
        String message() default "值不能大于 {value}"
    }

    // 2. 使用类型注解
    def validateUser(
        @NotNull String name,
        @Min(0) int age,
        @Max(150) int maxAge
    ) {
        // 运行时验证
        def annotations = this.class.getMethod("validateUser", String, int, int).parameterAnnotations

        // 验证NotNull
        def nameAnnotation = annotations[0].find { it instanceof NotNull }
        if (nameAnnotation && name == null) {
            throw new IllegalArgumentException(nameAnnotation.message)
        }

        // 验证Min
        def ageAnnotation = annotations[1].find { it instanceof Min }
        if (ageAnnotation && age < (ageAnnotation as Min).value()) {
            throw new IllegalArgumentException((ageAnnotation as Min).message())
        }

        // 验证Max
        def maxAgeAnnotation = annotations[2].find { it instanceof Max }
        if (maxAgeAnnotation && maxAge > (maxAgeAnnotation as Max).value()) {
            throw new IllegalArgumentException((maxAgeAnnotation as Max).message())
        }

        return "用户验证通过: $name, $age 岁"
    }

    // 3. 编译时类型检查
    @CompileStatic
    def compileTimeCheck(String input) {
        // 编译时会检查input是否为String
        return input.length() > 0 ? input : "空字符串"
    }

    // 4. 运行时类型验证
    def runtimeValidation(Object obj) {
        // 运行时类型检查
        if (!(obj instanceof String)) {
            throw new IllegalArgumentException("参数必须是String类型")
        }

        def str = obj as String
        return "验证通过: $str"
    }

    // 5. 类型安全的构建器
    static class SafeBuilder<T> {
        private Class<T> type
        private Map<String, Object> properties = [:]

        SafeBuilder(Class<T> type) {
            this.type = type
        }

        SafeBuilder<T> setProperty(String name, Object value) {
            // 验证属性是否存在
            if (!type.metaClass.properties.any { it.name == name }) {
                throw new IllegalArgumentException("属性 $name 不存在于类型 $type")
            }

            properties[name] = value
            return this
        }

        T build() {
            def instance = type.newInstance()
            properties.each { name, value ->
                instance."$name" = value
            }
            return instance
        }
    }
}
```

### 6.2 类型安全的元编程

```groovy
@CompileStatic
class TypeSafeMetaprogramming {
    // 1. 类型安全的方法添加
    static def addTypeSafeMethod(Class type, String methodName, Closure method) {
        def metaClass = type.metaClass

        // 使用静态编译的元类
        metaClass."$methodName" = { Object[] args ->
            // 类型检查
            def paramTypes = method.parameterTypes
            if (args.size() != paramTypes.size()) {
                throw new IllegalArgumentException("参数数量不匹配")
            }

            args.eachWithIndex { arg, index ->
                if (!paramTypes[index].isAssignableFrom(arg.getClass())) {
                    throw new IllegalArgumentException("参数 ${index + 1} 类型不匹配")
                }
            }

            // 调用方法
            method.call(args)
        }
    }

    // 2. 类型安全的属性添加
    static def addTypeSafeProperty(Class type, String propName, Class propType, Object initialValue = null) {
        def metaClass = type.metaClass

        // 添加getter
        metaClass."getPropName" = { ->
            // 确保属性存在
            if (!hasProperty(propName)) {
                setProperty(propName, initialValue)
            }
            return getProperty(propName)
        }

        // 添加setter
        metaClass."setPropName" = { Object value ->
            // 类型检查
            if (!propType.isAssignableFrom(value.getClass())) {
                throw new IllegalArgumentException("属性 $propName 类型必须是 $propType")
            }
            setProperty(propName, value)
        }
    }

    // 3. 类型安全的动态代理
    static def createTypeSafeProxy(Class interfaceType, Closure implementation) {
        return Proxy.newProxyInstance(
            interfaceType.classLoader,
            [interfaceType] as Class[],
            { Object proxy, Method method, Object[] args ->
                // 参数类型检查
                def paramTypes = method.parameterTypes
                args.eachWithIndex { arg, index ->
                    if (!paramTypes[index].isAssignableFrom(arg.getClass())) {
                        throw new IllegalArgumentException("参数 ${index + 1} 类型不匹配")
                    }
                }

                // 执行实现
                implementation.call(method.name, args)
            }
        )
    }

    // 4. 类型安全的AST转换
    @GroovyASTTransformation(phase = CompilePhase.SEMANTIC_ANALYSIS)
    static class TypeSafeASTTransformation implements ASTTransformation {
        @Override
        void visit(ASTNode[] nodes, SourceUnit sourceUnit) {
            def annotationNode = nodes[0] as AnnotationNode
            def annotatedNode = nodes[1] as AnnotatedNode

            if (annotatedNode instanceof MethodNode) {
                addTypeCheckingToMethod(annotatedNode as MethodNode, sourceUnit)
            }
        }

        def addTypeCheckingToMethod(MethodNode methodNode, SourceUnit sourceUnit) {
            // 添加类型检查代码
            def statements = methodNode.code.statements

            methodNode.parameters.each { param ->
                def typeCheck = new IfStatement(
                    new BooleanExpression(
                        new BinaryExpression(
                            new VariableExpression(param),
                            Token.newSymbol("==", -1, -1),
                            new ConstantExpression(null)
                        )
                    ),
                    new BlockStatement([
                        new ThrowStatement(
                            new ConstructorCallExpression(
                                ClassHelper.make(IllegalArgumentException),
                                new ArgumentListExpression([
                                    new ConstantExpression("参数 ${param.name} 不能为null")
                                ])
                            )
                        )
                    ], null),
                    null
                )
                statements.add(0, typeCheck)
            }

            methodNode.code = new BlockStatement(statements, methodNode.code.variableScope)
        }
    }

    // 5. 类型安全的反射工具
    static class TypeSafeReflection {
        static def invokeMethod(Object target, String methodName, Object[] args, Class[] paramTypes) {
            def method = target.class.getMethod(methodName, paramTypes)

            // 参数类型检查
            args.eachWithIndex { arg, index ->
                if (!paramTypes[index].isAssignableFrom(arg.getClass())) {
                    throw new IllegalArgumentException("参数 ${index + 1} 类型不匹配")
                }
            }

            return method.invoke(target, args)
        }

        static def setFieldValue(Object target, String fieldName, Object value, Class fieldType) {
            def field = target.class.getDeclaredField(fieldName)
            field.accessible = true

            // 类型检查
            if (!fieldType.isAssignableFrom(value.getClass())) {
                throw new IllegalArgumentException("字段 $fieldName 类型必须是 $fieldType")
            }

            field.set(target, value)
        }

        static def getFieldValue(Object target, String fieldName, Class fieldType) {
            def field = target.class.getDeclaredField(fieldName)
            field.accessible = true

            def value = field.get(target)
            if (!fieldType.isAssignableFrom(value.getClass())) {
                throw new IllegalArgumentException("字段 $fieldName 类型不是 $fieldType")
            }

            return value
        }
    }
}
```

## 7. 类型系统性能优化

### 7.1 编译时优化

```groovy
@CompileStatic
class TypeSystemOptimization {
    // 1. 类型推断优化
    static def optimizedInference() {
        // 编译时类型推断
        def stringList = ["A", "B", "C"]  // 推断为List<String>
        def result = stringList.collect { it.toUpperCase() }  // 编译时优化

        return result
    }

    // 2. 方法内联优化
    static def inlineOptimization() {
        def numbers = [1, 2, 3, 4, 5]
        def sum = 0

        // 编译器可能会内联这个循环
        for (int i = 0; i < numbers.size(); i++) {
            sum += numbers[i]
        }

        return sum
    }

    // 3. 常量折叠优化
    static def constantFolding() {
        // 编译时会计算常量表达式
        def result = 2 + 3 * 4  // 编译时计算为14
        return result
    }

    // 4. 死代码消除
    static def deadCodeElimination() {
        def x = 10
        def y = 20

        if (false) {
            // 这段代码会被编译器消除
            println "这行代码永远不会执行"
        }

        return x + y
    }

    // 5. 循环优化
    static def loopOptimization() {
        def numbers = [1, 2, 3, 4, 5]
        def result = new int[numbers.size()]

        // 编译器优化循环边界检查
        for (int i = 0; i < numbers.size(); i++) {
            result[i] = numbers[i] * 2
        }

        return result
    }
}
```

### 7.2 运行时优化

```groovy
class RuntimeOptimization {
    // 1. 类型缓存优化
    private static final Map<Class, List<Field>> fieldCache = new ConcurrentHashMap<>()

    static List<Field> getCachedFields(Class clazz) {
        return fieldCache.computeIfAbsent(clazz) { key ->
            key.declaredFields.toList()
        }
    }

    // 2. 方法调用缓存
    private static final Map<String, Method> methodCache = new ConcurrentHashMap<>()

    static Method getCachedMethod(Class clazz, String methodName, Class[] paramTypes) {
        def key = "${clazz.name}.$methodName(${paramTypes*.name.join(',')})"
        return methodCache.computeIfAbsent(key) { k ->
            try {
                return clazz.getMethod(methodName, paramTypes)
            } catch (NoSuchMethodException e) {
                throw new RuntimeException("方法未找到: $k", e)
            }
        }
    }

    // 3. 类型检查优化
    static def optimizedTypeCheck(Object obj, Class expectedType) {
        // 使用instanceof操作符进行快速类型检查
        if (expectedType.isInstance(obj)) {
            return true
        }

        // 处理基本类型
        if (expectedType == int.class && obj instanceof Integer) {
            return true
        }
        if (expectedType == long.class && obj instanceof Long) {
            return true
        }
        if (expectedType == double.class && obj instanceof Double) {
            return true
        }

        return false
    }

    // 4. 反射调用优化
    static def optimizedReflectionCall(Object target, String methodName, Object[] args) {
        def paramTypes = args*.class as Class[]
        def method = getCachedMethod(target.class, methodName, paramTypes)

        try {
            return method.invoke(target, args)
        } catch (Exception e) {
            throw new RuntimeException("反射调用失败", e)
        }
    }

    // 5. 集合操作优化
    static def optimizedCollectionOperations(List<?> list) {
        // 使用类型信息优化集合操作
        if (list.empty) {
            return []
        }

        def firstElement = list[0]
        if (firstElement instanceof String) {
            return optimizedStringOperations(list as List<String>)
        } else if (firstElement instanceof Number) {
            return optimizedNumberOperations(list as List<Number>)
        }

        return list
    }

    private static List<String> optimizedStringOperations(List<String> strings) {
        // 针对字符串列表的优化操作
        return strings.collect { it.toUpperCase() }
    }

    private static List<Number> optimizedNumberOperations(List<Number> numbers) {
        // 针对数字列表的优化操作
        return numbers.collect { it.doubleValue() * 2 }
    }

    // 6. 内存优化
    static def memoryOptimizedProcessing(List<?> data) {
        // 使用迭代器减少内存使用
        def iterator = data.iterator()
        def result = new ArrayList(data.size())

        while (iterator.hasNext()) {
            def item = iterator.next()
            result.add(processItem(item))
        }

        return result
    }

    private static Object processItem(Object item) {
        // 处理单个项目的逻辑
        return item.toString().toUpperCase()
    }
}
```

## 8. 类型系统最佳实践

### 8.1 类型选择策略

```groovy
class TypeSelectionStrategy {
    // 1. 动态类型使用场景
    static def useDynamicTypes() {
        def scriptEngine = new GroovyScriptEngine()

        // 动态类型适合脚本和配置
        def config = [
            database: "mysql",
            port: 3306,
            debug: true
        ]

        // 动态类型适合快速原型
        def quickPrototype = { input ->
            "处理结果: ${input?.toString()?.toUpperCase()}"
        }

        return "动态类型示例完成"
    }

    // 2. 静态类型使用场景
    @CompileStatic
    static def useStaticTypes() {
        // 静态类型适合性能敏感的代码
        def numbers = [1, 2, 3, 4, 5] as List<Integer>

        def sum = 0
        for (int i = 0; i < numbers.size(); i++) {
            sum += numbers[i]
        }

        return "静态类型计算结果: $sum"
    }

    // 3. 混合类型使用策略
    static def mixedTypeStrategy() {
        // 外层使用动态类型提供灵活性
        def processData = { List<?> data ->
            // 内层使用静态类型提供性能
            return staticProcessData(data)
        }

        def result = processData([1, 2, 3, 4, 5])
        return "混合类型处理结果: $result"
    }

    @CompileStatic
    private static List<Integer> staticProcessData(List<?> data) {
        def result = new ArrayList<Integer>(data.size())
        data.each { item ->
            if (item instanceof Number) {
                result.add(item.intValue() * 2)
            }
        }
        return result
    }

    // 4. 类型推断最佳实践
    @TypeChecked
    static def typeInferenceBestPractices() {
        // 明确的类型声明
        String name = "张三"
        int age = 25
        List<String> hobbies = ["读书", "运动"]

        // 合理的类型推断
        def greeting = "你好，${name}"  // 推断为String
        def ageInMonths = age * 12     // 推断为Integer
        def hobbyCount = hobbies.size() // 推断为Integer

        return """
            类型推断最佳实践:
            姓名: $name
            年龄: $age
            年龄(月): $ageInMonths
            爱好数量: $hobbyCount
        """
    }

    // 5. 泛型使用策略
    static def genericsStrategy() {
        // 使用泛型提供类型安全
        def stringList = new ArrayList<String>()
        def numberMap = new HashMap<String, Integer>()

        // 使用泛型方法
        def reversedList = reverseList(Arrays.asList("A", "B", "C"))
        def sum = sumList(Arrays.asList(1, 2, 3, 4, 5))

        return """
            泛型策略:
            反转列表: $reversedList
            列表和: $sum
        """
    }

    static <T> List<T> reverseList(List<T> list) {
        def result = new ArrayList<T>(list)
        Collections.reverse(result)
        return result
    }

    static <T extends Number> double sumList(List<T> list) {
        def sum = 0.0
        list.each { sum += it.doubleValue() }
        return sum
    }
}
```

### 8.2 类型安全模式

```groovy
@CompileStatic
class TypeSafetyPatterns {
    // 1. 空安全模式
    static def nullSafetyPatterns() {
        // 使用安全导航操作符
        def user = [name: "张三", address: [city: "北京"]]
        def city = user?.address?.city ?: "未知城市"

        // 使用Elvis操作符
        def displayName = user?.name ?: "匿名用户"

        // 使用@NotNull注解
        def result = processNonNullString("Hello")

        return "空安全模式: $city, $displayName, $result"
    }

    // 使用NotNull注解的方法
    static String processNonNullString(@NotNull String input) {
        return input.toUpperCase()
    }

    // 2. 类型断言模式
    static def typeAssertionPatterns() {
        def objects = ["Hello", 42, 3.14, true]

        def results = objects.collect { obj ->
            // 类型断言
            if (obj instanceof String) {
                "字符串: ${obj.toUpperCase()}"
            } else if (obj instanceof Number) {
                "数字: ${obj * 2}"
            } else if (obj instanceof Boolean) {
                "布尔: ${!obj}"
            } else {
                "其他类型: $obj"
            }
        }

        return results.join("\n")
    }

    // 3. 类型转换模式
    static def typeConversionPatterns() {
        def conversions = [
            "字符串到数字: ${safeStringToNumber("123")}",
            "数字到字符串: ${safeNumberToString(42)}",
            "布尔到字符串: ${safeBooleanToString(true)}",
            "日期到字符串: ${safeDateToString(new Date())}"
        ]

        return conversions.join("\n")
    }

    // 4. 集合类型安全模式
    static def collectionTypeSafety() {
        // 类型安全的集合操作
        def stringList = Arrays.asList("A", "B", "C") as List<String>
        def numberList = Arrays.asList(1, 2, 3) as List<Integer>

        def upperCaseList = safeTransformList(stringList) { String s ->
            s.toUpperCase()
        }

        def doubledList = safeTransformList(numberList) { Integer n ->
            n * 2
        }

        return """
            集合类型安全:
            原始字符串: $stringList
            转换后: $upperCaseList
            原始数字: $numberList
            转换后: $doubledList
        """
    }

    // 5. 方法重载模式
    static def methodOverloadingPatterns() {
        // 重载方法提供类型安全
        def result1 = process("Hello")        // 调用字符串版本
        def result2 = process(42)            // 调用数字版本
        def result3 = process([1, 2, 3])     // 调用集合版本

        return """
            方法重载:
            字符串: $result1
            数字: $result2
            集合: $result3
        """
    }

    // 辅助方法
    static def safeStringToNumber(String str) {
        try {
            return Integer.parseInt(str)
        } catch (NumberFormatException e) {
            return 0
        }
    }

    static def safeNumberToString(Number num) {
        return num.toString()
    }

    static def safeBooleanToString(Boolean bool) {
        return bool ? "true" : "false"
    }

    static def safeDateToString(Date date) {
        return date.format("yyyy-MM-dd")
    }

    static <T, R> List<R> safeTransformList(List<T> list, Closure<R> transformer) {
        def result = new ArrayList<R>(list.size())
        list.each { item ->
            result.add(transformer.call(item))
        }
        return result
    }

    static String process(String input) {
        return "处理字符串: ${input.toUpperCase()}"
    }

    static String process(Number input) {
        return "处理数字: ${input * 2}"
    }

    static String process(List<?> input) {
        return "处理集合: ${input.size()} 个元素"
    }
}
```

## 9. 总结

Groovy的类型系统是其在动态性和静态性之间找到完美平衡的关键。通过本文的深入分析，我们了解了：

1. **类型系统架构**：理解了Groovy类型系统的层次结构和设计理念
2. **类型推断机制**：掌握了编译时和运行时类型推断的工作原理
3. **静态编译优化**：学会了使用@CompileStatic和@TypeChecked提升性能
4. **类型转换机制**：深入理解了自动转换和强制转换的底层实现
5. **泛型系统**：掌握了泛型的使用和高级特性
6. **类型注解**：学会了使用类型注解增强类型安全
7. **性能优化**：了解了类型系统的各种优化策略

### 关键要点：

1. **灵活性vs性能**：在动态类型和静态类型之间找到合适的平衡点
2. **类型安全**：使用静态编译和类型检查提高代码质量
3. **性能优化**：在关键路径上使用静态编译获得更好的性能
4. **代码可读性**：合理的类型注解提高代码的可读性和可维护性
5. **最佳实践**：根据具体场景选择合适的类型策略

掌握Groovy类型系统的深度知识将帮助你在实际项目中做出更好的技术决策，构建出既灵活又高效的应用程序。通过合理运用Groovy的类型系统特性，你可以充分发挥JVM的性能优势，同时保持Groovy的简洁和灵活。