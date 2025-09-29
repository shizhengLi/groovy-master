# Groovy编译器内部机制

## 目录

1. [引言](#引言)
2. [Groovy编译器架构](#groovy编译器架构)
3. [词法分析器](#词法分析器)
4. [语法分析器](#语法分析器)
5. [AST构建与转换](#ast构建与转换)
6. [代码生成](#代码生成)
7. [编译优化](#编译优化)
8. [静态编译机制](#静态编译机制)
9. [动态编译机制](#动态编译机制)
10. [编译器扩展](#编译器扩展)
11. [性能优化技术](#性能优化技术)
12. [实战案例](#实战案例)
13. [总结](#总结)

## 引言

Groovy编译器是一个复杂而强大的系统，它将Groovy源代码转换为JVM字节码。理解Groovy编译器的内部机制对于深入掌握Groovy语言、进行性能优化和开发编译器扩展都具有重要意义。本章节将深入探讨Groovy编译器的各个组件、工作原理以及优化技术。

### 编译器的重要性

1. **性能优化**：理解编译过程有助于编写更高效的代码
2. **元编程支持**：编译器为Groovy的动态特性提供底层支持
3. **调试支持**：理解编译过程有助于更好地调试Groovy代码
4. **扩展开发**：可以开发自定义的编译器扩展和AST转换

## Groovy编译器架构

### 编译器整体架构

```groovy
class GroovyCompilerArchitecture {

    // 1. 编译器组件概述
    static void demonstrateCompilerComponents() {
        println """
        Groovy Compiler Components:

        1. 词法分析器 (Lexer)
           - 将源代码转换为token序列
           - 处理Groovy特定的语法元素

        2. 语法分析器 (Parser)
           - 构建抽象语法树(AST)
           - 验证语法结构

        3. AST转换器 (AST Transformations)
           - 应用编译时元编程
           - 执行代码转换

        4. 代码生成器 (Code Generator)
           - 生成Java字节码
           - 处理Groovy特有的特性

        5. 优化器 (Optimizer)
           - 应用各种优化策略
           - 生成高效字节码
        """.stripIndent()
    }

    // 2. 编译流程
    static void demonstrateCompilationFlow() {
        def stages = [
            "Source Code" : "原始Groovy源代码",
            "Tokenization" : "词法分析，生成token序列",
            "Parsing" : "语法分析，构建AST",
            "AST Transformations" : "应用AST转换",
            "Semantic Analysis" : "语义分析",
            "Code Generation" : "生成字节码",
            "Optimization" : "优化字节码",
            "Class File" : "最终class文件"
        ]

        println "编译流程："
        stages.each { stage, description ->
            println "  $stage: $description"
        }
    }

    // 3. 编译器配置
    static void demonstrateCompilerConfiguration() {
        def config = [
            "targetBytecode" : "JVM 1.8",
            "debugInfo" : true,
            "optimizationLevel" : "aggressive",
            "staticCompilation" : false,
            "scriptBaseClass" : "groovy.lang.Script",
            "extensions" : ["*.groovy", "*.gvy", "*.gy"],
            "classpath" : [".", "./lib"],
            "sourceEncoding" : "UTF-8",
            "warningLevel" : "all"
        ]

        println "编译器配置："
        config.each { key, value ->
            println "  $key: $value"
        }
    }
}
```

### 编译器核心类

```groovy
class CompilerCoreClasses {

    // 1. 主要编译器类
    static void demonstrateMainCompilerClasses() {
        println """
        主要编译器类：

        1. org.codehaus.groovy.control.CompilerConfiguration
           - 编译器配置管理
           - 编译选项设置

        2. org.codehaus.groovy.control.SourceUnit
           - 源代码单元管理
           - 错误收集和处理

        3. org.codehaus.groovy.control.CompilationUnit
           - 编译单元协调
           - 管理编译过程

        4. org.codehaus.groovy.control.Phases
           - 编译阶段定义
           - 阶段转换控制

        5. org.codehaus.groovy.control.ProcessingUnit
           - 处理单元抽象
           - 统一处理接口
        """.stripIndent()
    }

    // 2. AST相关类
    static void demonstrateASTClasses() {
        println """
        AST相关类：

        1. org.codehaus.groovy.ast.ASTNode
           - AST节点基类
           - 节点属性和方法

        2. org.codehaus.groovy.ast.ModuleNode
           - 模块节点
           - 包含类和脚本

        3. org.codehaus.groovy.ast.ClassNode
           - 类节点
           - 类的定义信息

        4. org.codehaus.groovy.ast.MethodNode
           - 方法节点
           - 方法定义和实现

        5. org.codehaus.groovy.ast.FieldNode
           - 字段节点
           - 字段定义和访问

        6. org.codehaus.groovy.ast.expr.Expression
           - 表达式基类
           - 各种表达式类型
        """.stripIndent()
    }

    // 3. 词法和语法分析类
    static void demonstrateLexerParserClasses() {
        println """
        词法和语法分析类：

        1. org.codehaus.groovy.lexer.GroovyLexer
           - Groovy词法分析器
           - Token生成和管理

        2. org.codehaus.groovy.lexer.GroovyRecognizer
           - Groovy语法识别器
           - 语法规则定义

        3. org.codehaus.groovy.syntax.Reduction
           - 语法归约
           - 语法树构建

        4. org.codehaus.groovy.syntax.Parser
           - 语法分析器
           - AST构建

        5. org.codehaus.groovy.syntax.ASTBuilder
           - AST构建器
           - 节点创建和组装
        """.stripIndent()
    }
}
```

## 词法分析器

### Token定义与处理

```groovy
class LexerImplementation {

    // 1. Token类型定义
    static enum TokenType {
        // 关键字
        CLASS("class"), INTERFACE("interface"), DEF("def"),
        IF("if"), ELSE("else"), FOR("for"), WHILE("while"),
        RETURN("return"), NEW("new"), IMPORT("import"),

        // 标识符和字面量
        IDENTIFIER("identifier"), STRING("string"), NUMBER("number"),
        BOOLEAN("boolean"), NULL("null"),

        // 运算符
        PLUS("+"), MINUS("-"), MULTIPLY("*"), DIVIDE("/"),
        ASSIGN("="), EQUAL("=="), NOT_EQUAL("!="),
        LESS_THAN("<"), GREATER_THAN(">"),

        // 分隔符
        LPAREN("("), RPAREN(")"), LBRACE("{"), RBRACE("}"),
        LBRACKET("["), RBRACKET("]"), SEMICOLON(";"), COMMA(","),

        // 特殊
        EOF("end of file"), WHITESPACE("whitespace"), COMMENT("comment")

        final String description
        TokenType(String description) {
            this.description = description
        }
    }

    // 2. Token类
    static class Token {
        final TokenType type
        final String text
        final int line
        final int column

        Token(TokenType type, String text, int line, int column) {
            this.type = type
            this.text = text
            this.line = line
            this.column = column
        }

        String toString() {
            "Token($type, '$text', at $line:$column)"
        }
    }

    // 3. 简单词法分析器
    static class SimpleLexer {
        private final String input
        private int position = 0
        private int line = 1
        private int column = 1

        SimpleLexer(String input) {
            this.input = input
        }

        Token nextToken() {
            if (position >= input.length()) {
                return new Token(TokenType.EOF, "", line, column)
            }

            def currentChar = input[position]

            // 跳过空白字符
            while (Character.isWhitespace(currentChar)) {
                if (currentChar == '\n') {
                    line++
                    column = 1
                } else {
                    column++
                }
                position++
                if (position >= input.length()) {
                    return new Token(TokenType.EOF, "", line, column)
                }
                currentChar = input[position]
            }

            // 处理不同的token类型
            switch (currentChar) {
                case '+':
                    position++
                    column++
                    return new Token(TokenType.PLUS, "+", line, column - 1)
                case '-':
                    position++
                    column++
                    return new Token(TokenType.MINUS, "-", line, column - 1)
                case '*':
                    position++
                    column++
                    return new Token(TokenType.MULTIPLY, "*", line, column - 1)
                case '/':
                    position++
                    column++
                    return new Token(TokenType.DIVIDE, "/", line, column - 1)
                case '(':
                    position++
                    column++
                    return new Token(TokenType.LPAREN, "(", line, column - 1)
                case ')':
                    position++
                    column++
                    return new Token(TokenType.RPAREN, ")", line, column - 1)
                case '{':
                    position++
                    column++
                    return new Token(TokenType.LBRACE, "{", line, column - 1)
                case '}':
                    position++
                    column++
                    return new Token(TokenType.RBRACE, "}", line, column - 1)
                case '=':
                    position++
                    column++
                    if (position < input.length() && input[position] == '=') {
                        position++
                        column++
                        return new Token(TokenType.EQUAL, "==", line, column - 2)
                    }
                    return new Token(TokenType.ASSIGN, "=", line, column - 1)
            }

            // 处理数字
            if (Character.isDigit(currentChar)) {
                return parseNumber()
            }

            // 处理字符串
            if (currentChar == '"' || currentChar == "'") {
                return parseString()
            }

            // 处理标识符
            if (Character.isLetter(currentChar) || currentChar == '_') {
                return parseIdentifier()
            }

            // 处理注释
            if (currentChar == '/' && position + 1 < input.length()) {
                def nextChar = input[position + 1]
                if (nextChar == '/') {
                    return parseLineComment()
                } else if (nextChar == '*') {
                    return parseBlockComment()
                }
            }

            // 未知字符
            position++
            column++
            new Token(TokenType.IDENTIFIER, currentChar.toString(), line, column - 1)
        }

        private Token parseNumber() {
            def start = position
            while (position < input.length() && Character.isDigit(input[position])) {
                position++
                column++
            }

            // 处理小数点
            if (position < input.length() && input[position] == '.') {
                position++
                column++
                while (position < input.length() && Character.isDigit(input[position])) {
                    position++
                    column++
                }
            }

            def text = input[start..position - 1]
            new Token(TokenType.NUMBER, text, line, column - text.length())
        }

        private Token parseString() {
            def quote = input[position]
            position++
            column++

            def start = position
            def value = new StringBuilder()

            while (position < input.length() && input[position] != quote) {
                def current = input[position]
                if (current == '\\') {
                    position++
                    column++
                    if (position < input.length()) {
                        def escaped = input[position]
                        value.append(handleEscape(escaped))
                        position++
                        column++
                    }
                } else {
                    value.append(current)
                    position++
                    column++
                }
            }

            if (position < input.length()) {
                position++
                column++
            }

            new Token(TokenType.STRING, value.toString(), line, column - value.length() - 2)
        }

        private Token parseIdentifier() {
            def start = position
            while (position < input.length() &&
                   (Character.isLetterOrDigit(input[position]) || input[position] == '_')) {
                position++
                column++
            }

            def text = input[start..position - 1]

            // 检查是否为关键字
            def keyword = TokenType.values().find {
                it.name() == text.toUpperCase() && it.description == text
            }

            if (keyword) {
                return new Token(keyword, text, line, column - text.length())
            }

            new Token(TokenType.IDENTIFIER, text, line, column - text.length())
        }

        private Token parseLineComment() {
            position += 2  // 跳过 '//'
            column += 2

            def start = position
            while (position < input.length() && input[position] != '\n') {
                position++
                column++
            }

            def text = input[start..position - 1]
            new Token(TokenType.COMMENT, text, line, column - text.length())
        }

        private Token parseBlockComment() {
            position += 2  // 跳过 '/*'
            column += 2

            def start = position
            while (position + 1 < input.length() &&
                   !(input[position] == '*' && input[position + 1] == '/')) {
                if (input[position] == '\n') {
                    line++
                    column = 1
                } else {
                    column++
                }
                position++
            }

            def text = input[start..position - 1]
            if (position + 1 < input.length()) {
                position += 2
                column += 2
            }

            new Token(TokenType.COMMENT, text, line, column - text.length())
        }

        private String handleEscape(char escaped) {
            switch (escaped) {
                case 'n': return '\n'
                case 't': return '\t'
                case 'r': return '\r'
                case '"': return '"'
                case "'": return "'"
                case '\\': return '\\'
                default: return escaped.toString()
            }
        }
    }

    static void demonstrateLexer() {
        def code = """
        def add(a, b) {
            // 这是一个简单的加法函数
            return a + b  // 返回两数之和
        }

        def result = add(5, 3)
        println "Result: \${result}"
        """.stripIndent()

        def lexer = new SimpleLexer(code)
        def token = lexer.nextToken()

        println "词法分析结果："
        while (token.type != TokenType.EOF) {
            println "  $token"
            token = lexer.nextToken()
        }
    }
}
```

### 词法分析优化

```groovy
class LexerOptimization {

    // 1. 字符缓冲区优化
    static class OptimizedLexer {
        private final char[] buffer
        private final int bufferSize
        private int position = 0
        private int line = 1
        private int column = 1

        OptimizedLexer(String input) {
            this.buffer = input.toCharArray()
            this.bufferSize = buffer.length
        }

        char currentChar() {
            position < bufferSize ? buffer[position] : '\0'
        }

        char lookahead(int offset) {
            def pos = position + offset
            pos < bufferSize ? buffer[pos] : '\0'
        }

        void advance() {
            if (position < bufferSize) {
                if (buffer[position] == '\n') {
                    line++
                    column = 1
                } else {
                    column++
                }
                position++
            }
        }

        void advance(int count) {
            count.times { advance() }
        }

        boolean hasMore() {
            position < bufferSize
        }

        // 优化的token处理
        Token processToken() {
            skipWhitespace()

            if (!hasMore()) {
                return new Token(TokenType.EOF, "", line, column)
            }

            def current = currentChar()

            // 快速路径：单字符token
            switch (current) {
                case '+': advance(); return createToken(TokenType.PLUS, "+")
                case '-': advance(); return createToken(TokenType.MINUS, "-")
                case '*': advance(); return createToken(TokenType.MULTIPLY, "*")
                case '/':
                    advance()
                    if (currentChar() == '/') {
                        return processLineComment()
                    } else if (currentChar() == '*') {
                        return processBlockComment()
                    }
                    return createToken(TokenType.DIVIDE, "/")
                case '(': advance(); return createToken(TokenType.LPAREN, "(")
                case ')': advance(); return createToken(TokenType.RPAREN, ")")
                case '{': advance(); return createToken(TokenType.LBRACE, "{")
                case '}': advance(); return createToken(TokenType.RBRACE, "}")
            }

            // 复杂token处理
            if (Character.isDigit(current)) {
                return processNumber()
            }

            if (current == '"' || current == "'") {
                return processString()
            }

            if (Character.isLetter(current) || current == '_') {
                return processIdentifier()
            }

            // 未知token
            advance()
            return createToken(TokenType.IDENTIFIER, current.toString())
        }

        private void skipWhitespace() {
            while (hasMore() && Character.isWhitespace(currentChar())) {
                advance()
            }
        }

        private Token createToken(TokenType type, String text) {
            new Token(type, text, line, column - text.length())
        }

        private Token processNumber() {
            def start = position
            def startColumn = column

            // 整数部分
            while (hasMore() && Character.isDigit(currentChar())) {
                advance()
            }

            // 小数部分
            if (hasMore() && currentChar() == '.') {
                advance()
                while (hasMore() && Character.isDigit(currentChar())) {
                    advance()
                }
            }

            // 指数部分
            if (hasMore() && (currentChar() == 'e' || currentChar() == 'E')) {
                advance()
                if (hasMore() && (currentChar() == '+' || currentChar() == '-')) {
                    advance()
                }
                while (hasMore() && Character.isDigit(currentChar())) {
                    advance()
                }
            }

            def text = new String(buffer, start, position - start)
            new Token(TokenType.NUMBER, text, line, startColumn)
        }

        private Token processString() {
            def quote = currentChar()
            advance()
            def start = position
            def startColumn = column
            def value = new StringBuilder()

            while (hasMore() && currentChar() != quote) {
                def current = currentChar()
                if (current == '\\') {
                    advance()
                    if (hasMore()) {
                        value.append(handleEscape(currentChar()))
                        advance()
                    }
                } else {
                    value.append(current)
                    advance()
                }
            }

            if (hasMore()) {
                advance()  // 跳过结束引号
            }

            new Token(TokenType.STRING, value.toString(), line, startColumn)
        }

        private Token processIdentifier() {
            def start = position
            def startColumn = column

            while (hasMore() &&
                   (Character.isLetterOrDigit(currentChar()) || currentChar() == '_')) {
                advance()
            }

            def text = new String(buffer, start, position - start)

            // 检查关键字
            def keyword = TokenType.values().find {
                it.name() == text.toUpperCase() && it.description == text
            }

            if (keyword) {
                return new Token(keyword, text, line, startColumn)
            }

            new Token(TokenType.IDENTIFIER, text, line, startColumn)
        }

        private Token processLineComment() {
            advance(2)  // 跳过 '//'
            def start = position
            def startColumn = column

            while (hasMore() && currentChar() != '\n') {
                advance()
            }

            def text = new String(buffer, start, position - start)
            new Token(TokenType.COMMENT, text, line, startColumn)
        }

        private Token processBlockComment() {
            advance(2)  // 跳过 '/*'
            def start = position
            def startColumn = column

            while (hasMore() && !(currentChar() == '*' && lookahead(1) == '/')) {
                advance()
            }

            def text = new String(buffer, start, position - start)
            if (hasMore()) {
                advance(2)  // 跳过 '*/'
            }

            new Token(TokenType.COMMENT, text, line, startColumn)
        }

        private String handleEscape(char escaped) {
            switch (escaped) {
                case 'n': return '\n'
                case 't': return '\t'
                case 'r': return '\r'
                case '"': return '"'
                case "'": return "'"
                case '\\': return '\\'
                case 'b': return '\b'
                case 'f': return '\f'
                default: return escaped.toString()
            }
        }
    }

    // 2. Token缓存机制
    static class TokenCache {
        private final Map<Integer, Token> cache = [:]
        private final String input
        private final OptimizedLexer lexer

        TokenCache(String input) {
            this.input = input
            this.lexer = new OptimizedLexer(input)
        }

        Token getToken(int position) {
            if (!cache.containsKey(position)) {
                // 重置lexer到指定位置
                // 这里简化处理，实际需要重置lexer状态
                cache[position] = lexer.processToken()
            }
            return cache[position]
        }

        void clear() {
            cache.clear()
        }
    }

    // 3. 批量处理
    static class BatchLexer {
        private final OptimizedLexer lexer
        private final List<Token> batch = []
        private final int batchSize

        BatchLexer(String input, int batchSize = 100) {
            this.lexer = new OptimizedLexer(input)
            this.batchSize = batchSize
        }

        List<Token> nextBatch() {
            batch.clear()
            batchSize.times {
                def token = lexer.processToken()
                batch.add(token)
                if (token.type == TokenType.EOF) {
                    return batch
                }
            }
            batch
        }
    }

    static void demonstrateOptimizedLexer() {
        def code = """
        def calculateTotal(items) {
            def total = 0
            items.each { item ->
                total += item.price * item.quantity
            }
            return total
        }
        """.stripIndent()

        def lexer = new OptimizedLexer(code)
        def token = lexer.processToken()

        println "优化后的词法分析结果："
        while (token.type != TokenType.EOF) {
            println "  $token"
            token = lexer.processToken()
        }
    }
}
```

## 语法分析器

### 语法分析基础

```groovy
class ParserImplementation {

    // 1. 语法树节点
    static class ASTNode {
        String type
        Object value
        List<ASTNode> children = []
        int line
        int column

        ASTNode(String type, Object value = null, int line = 0, int column = 0) {
            this.type = type
            this.value = value
            this.line = line
            this.column = column
        }

        ASTNode addChild(ASTNode child) {
            children.add(child)
            this
        }

        String toString() {
            def result = "($type"
            if (value != null) {
                result += ": $value"
            }
            if (!children.isEmpty()) {
                result += " " + children.join(" ")
            }
            result + ")"
        }
    }

    // 2. 简单语法分析器
    static class SimpleParser {
        private final List<Token> tokens
        private int current = 0

        SimpleParser(List<Token> tokens) {
            this.tokens = tokens
        }

        Token peek() {
            if (current < tokens.size()) {
                return tokens[current]
            }
            null
        }

        Token consume(TokenType expectedType) {
            def token = peek()
            if (token && token.type == expectedType) {
                current++
                return token
            }
            throw new RuntimeException("Expected $expectedType but got ${token?.type}")
        }

        boolean match(TokenType... types) {
            def token = peek()
            token && types.contains(token.type)
        }

        ASTNode parse() {
            parseProgram()
        }

        private ASTNode parseProgram() {
            def program = new ASTNode("Program")

            while (peek() && peek().type != TokenType.EOF) {
                def statement = parseStatement()
                program.addChild(statement)
            }

            program
        }

        private ASTNode parseStatement() {
            def token = peek()

            if (token.type == TokenType.DEF) {
                return parseFunctionDefinition()
            } else if (token.type == TokenType.IF) {
                return parseIfStatement()
            } else if (token.type == TokenType.FOR) {
                return parseForStatement()
            } else if (token.type == TokenType.WHILE) {
                return parseWhileStatement()
            } else if (token.type == TokenType.RETURN) {
                return parseReturnStatement()
            } else {
                return parseExpressionStatement()
            }
        }

        private ASTNode parseFunctionDefinition() {
            consume(TokenType.DEF)
            def nameToken = consume(TokenType.IDENTIFIER)
            consume(TokenType.LPAREN)

            def params = []
            if (!match(TokenType.RPAREN)) {
                params.add(parseParameter())
                while (match(TokenType.COMMA)) {
                    consume(TokenType.COMMA)
                    params.add(parseParameter())
                }
            }

            consume(TokenType.RPAREN)
            consume(TokenType.LBRACE)

            def body = []
            while (!match(TokenType.RBRACE)) {
                body.add(parseStatement())
            }

            consume(TokenType.RBRACE)

            def functionNode = new ASTNode("FunctionDefinition", nameToken.text, nameToken.line, nameToken.column)
            def paramsNode = new ASTNode("Parameters")
            params.each { paramsNode.addChild(it) }
            functionNode.addChild(paramsNode)

            def bodyNode = new ASTNode("FunctionBody")
            body.each { bodyNode.addChild(it) }
            functionNode.addChild(bodyNode)

            functionNode
        }

        private ASTNode parseParameter() {
            def nameToken = consume(TokenType.IDENTIFIER)
            new ASTNode("Parameter", nameToken.text, nameToken.line, nameToken.column)
        }

        private ASTNode parseIfStatement() {
            consume(TokenType.IF)
            consume(TokenType.LPAREN)
            def condition = parseExpression()
            consume(TokenType.RPAREN)
            consume(TokenType.LBRACE)

            def thenBody = []
            while (!match(TokenType.RBRACE)) {
                thenBody.add(parseStatement())
            }

            consume(TokenType.RBRACE)

            def ifNode = new ASTNode("IfStatement")
            ifNode.addChild(condition)

            def thenNode = new ASTNode("ThenBranch")
            thenBody.each { thenNode.addChild(it) }
            ifNode.addChild(thenNode)

            // 可选的else分支
            if (match(TokenType.ELSE)) {
                consume(TokenType.ELSE)
                consume(TokenType.LBRACE)

                def elseBody = []
                while (!match(TokenType.RBRACE)) {
                    elseBody.add(parseStatement())
                }

                consume(TokenType.RBRACE)

                def elseNode = new ASTNode("ElseBranch")
                elseBody.each { elseNode.addChild(it) }
                ifNode.addChild(elseNode)
            }

            ifNode
        }

        private ASTNode parseForStatement() {
            consume(TokenType.FOR)
            consume(TokenType.LPAREN)
            def variable = consume(TokenType.IDENTIFIER)
            consume(TokenType.IN)
            def collection = parseExpression()
            consume(TokenType.RPAREN)
            consume(TokenType.LBRACE)

            def body = []
            while (!match(TokenType.RBRACE)) {
                body.add(parseStatement())
            }

            consume(TokenType.RBRACE)

            def forNode = new ASTNode("ForStatement")
            forNode.addChild(new ASTNode("Variable", variable.text, variable.line, variable.column))
            forNode.addChild(collection)

            def bodyNode = new ASTNode("ForBody")
            body.each { bodyNode.addChild(it) }
            forNode.addChild(bodyNode)

            forNode
        }

        private ASTNode parseWhileStatement() {
            consume(TokenType.WHILE)
            consume(TokenType.LPAREN)
            def condition = parseExpression()
            consume(TokenType.RPAREN)
            consume(TokenType.LBRACE)

            def body = []
            while (!match(TokenType.RBRACE)) {
                body.add(parseStatement())
            }

            consume(TokenType.RBRACE)

            def whileNode = new ASTNode("WhileStatement")
            whileNode.addChild(condition)

            def bodyNode = new ASTNode("WhileBody")
            body.each { bodyNode.addChild(it) }
            whileNode.addChild(bodyNode)

            whileNode
        }

        private ASTNode parseReturnStatement() {
            consume(TokenType.RETURN)
            def expression = parseExpression()
            new ASTNode("ReturnStatement").addChild(expression)
        }

        private ASTNode parseExpressionStatement() {
            def expression = parseExpression()
            new ASTNode("ExpressionStatement").addChild(expression)
        }

        private ASTNode parseExpression() {
            parseLogicalOr()
        }

        private ASTNode parseLogicalOr() {
            def left = parseLogicalAnd()

            while (match(TokenType.OR)) {
                def operator = consume(TokenType.OR)
                def right = parseLogicalAnd()
                left = new ASTNode("BinaryExpression", operator.text, operator.line, operator.column)
                    .addChild(left)
                    .addChild(right)
            }

            left
        }

        private ASTNode parseLogicalAnd() {
            def left = parseEquality()

            while (match(TokenType.AND)) {
                def operator = consume(TokenType.AND)
                def right = parseEquality()
                left = new ASTNode("BinaryExpression", operator.text, operator.line, operator.column)
                    .addChild(left)
                    .addChild(right)
            }

            left
        }

        private ASTNode parseEquality() {
            def left = parseRelational()

            while (match(TokenType.EQUAL, TokenType.NOT_EQUAL)) {
                def operator = consume(peek().type)
                def right = parseRelational()
                left = new ASTNode("BinaryExpression", operator.text, operator.line, operator.column)
                    .addChild(left)
                    .addChild(right)
            }

            left
        }

        private ASTNode parseRelational() {
            def left = parseAdditive()

            while (match(TokenType.LESS_THAN, TokenType.GREATER_THAN)) {
                def operator = consume(peek().type)
                def right = parseAdditive()
                left = new ASTNode("BinaryExpression", operator.text, operator.line, operator.column)
                    .addChild(left)
                    .addChild(right)
            }

            left
        }

        private ASTNode parseAdditive() {
            def left = parseMultiplicative()

            while (match(TokenType.PLUS, TokenType.MINUS)) {
                def operator = consume(peek().type)
                def right = parseMultiplicative()
                left = new ASTNode("BinaryExpression", operator.text, operator.line, operator.column)
                    .addChild(left)
                    .addChild(right)
            }

            left
        }

        private ASTNode parseMultiplicative() {
            def left = parseUnary()

            while (match(TokenType.MULTIPLY, TokenType.DIVIDE)) {
                def operator = consume(peek().type)
                def right = parseUnary()
                left = new ASTNode("BinaryExpression", operator.text, operator.line, operator.column)
                    .addChild(left)
                    .addChild(right)
            }

            left
        }

        private ASTNode parseUnary() {
            if (match(TokenType.MINUS, TokenType.NOT)) {
                def operator = consume(peek().type)
                def operand = parsePrimary()
                return new ASTNode("UnaryExpression", operator.text, operator.line, operator.column)
                    .addChild(operand)
            }

            parsePrimary()
        }

        private ASTNode parsePrimary() {
            def token = peek()

            if (token.type == TokenType.NUMBER) {
                return parseNumberLiteral()
            } else if (token.type == TokenType.STRING) {
                return parseStringLiteral()
            } else if (token.type == TokenType.BOOLEAN) {
                return parseBooleanLiteral()
            } else if (token.type == TokenType.NULL) {
                return parseNullLiteral()
            } else if (token.type == TokenType.IDENTIFIER) {
                return parseIdentifierExpression()
            } else if (token.type == TokenType.LPAREN) {
                return parseParenthesizedExpression()
            } else {
                throw new RuntimeException("Unexpected token: $token")
            }
        }

        private ASTNode parseNumberLiteral() {
            def token = consume(TokenType.NUMBER)
            new ASTNode("NumberLiteral", token.text, token.line, token.column)
        }

        private ASTNode parseStringLiteral() {
            def token = consume(TokenType.STRING)
            new ASTNode("StringLiteral", token.text, token.line, token.column)
        }

        private ASTNode parseBooleanLiteral() {
            def token = consume(TokenType.BOOLEAN)
            new ASTNode("BooleanLiteral", token.text, token.line, token.column)
        }

        private ASTNode parseNullLiteral() {
            consume(TokenType.NULL)
            new ASTNode("NullLiteral", "null")
        }

        private ASTNode parseIdentifierExpression() {
            def token = consume(TokenType.IDENTIFIER)

            if (match(TokenType.LPAREN)) {
                // 函数调用
                consume(TokenType.LPAREN)
                def args = []
                if (!match(TokenType.RPAREN)) {
                    args.add(parseExpression())
                    while (match(TokenType.COMMA)) {
                        consume(TokenType.COMMA)
                        args.add(parseExpression())
                    }
                }
                consume(TokenType.RPAREN)

                def callNode = new ASTNode("FunctionCall", token.text, token.line, token.column)
                args.each { callNode.addChild(it) }
                callNode
            } else {
                // 变量引用
                new ASTNode("VariableReference", token.text, token.line, token.column)
            }
        }

        private ASTNode parseParenthesizedExpression() {
            consume(TokenType.LPAREN)
            def expression = parseExpression()
            consume(TokenType.RPAREN)
            expression
        }
    }

    static void demonstrateParser() {
        def code = """
        def factorial(n) {
            if (n <= 1) {
                return 1
            } else {
                return n * factorial(n - 1)
            }
        }

        def result = factorial(5)
        println "Factorial of 5 is: \${result}"
        """.stripIndent()

        def lexer = new SimpleLexer(code)
        def tokens = []
        def token = lexer.nextToken()

        while (token.type != TokenType.EOF) {
            tokens.add(token)
            token = lexer.nextToken()
        }

        def parser = new SimpleParser(tokens)
        def ast = parser.parse()

        println "语法分析结果："
        println ast
    }
}
```

### 语法分析优化

```groovy
class ParserOptimization {

    // 1. 错误恢复机制
    static class ErrorRecoveryParser {
        private final List<Token> tokens
        private int current = 0
        private final List<String> errors = []

        ErrorRecoveryParser(List<Token> tokens) {
            this.tokens = tokens
        }

        Token peek() {
            current < tokens.size() ? tokens[current] : null
        }

        Token consume(TokenType expectedType) {
            def token = peek()
            if (token && token.type == expectedType) {
                current++
                return token
            }

            def line = token?.line ?: 0
            def column = token?.column ?: 0
            errors.add("Error at $line:$column: Expected $expectedType but got ${token?.type}")

            // 错误恢复：跳过到下一个同步点
            synchronize()

            // 返回一个虚拟token
            new Token(expectedType, "expected_$expectedType", line, column)
        }

        private void synchronize() {
            current++

            def synchronizingTokens = [
                TokenType.CLASS, TokenType.DEF, TokenType.IF, TokenType.ELSE,
                TokenType.FOR, TokenType.WHILE, TokenType.RETURN, TokenType.RBRACE
            ]

            while (current < tokens.size()) {
                if (tokens[current].type in synchronizingTokens) {
                    return
                }
                current++
            }
        }

        List<String> getErrors() {
            errors
        }

        ASTNode parse() {
            errors.clear()
            parseProgram()
        }

        private ASTNode parseProgram() {
            def program = new ASTNode("Program")

            while (peek() && peek().type != TokenType.EOF) {
                try {
                    def statement = parseStatement()
                    program.addChild(statement)
                } catch (Exception e) {
                    errors.add(e.message)
                    synchronize()
                }
            }

            program
        }

        private ASTNode parseStatement() {
            def token = peek()

            if (token.type == TokenType.DEF) {
                return parseFunctionDefinition()
            } else if (token.type == TokenType.IF) {
                return parseIfStatement()
            } else if (token.type == TokenType.FOR) {
                return parseForStatement()
            } else if (token.type == TokenType.WHILE) {
                return parseWhileStatement()
            } else if (token.type == TokenType.RETURN) {
                return parseReturnStatement()
            } else {
                return parseExpressionStatement()
            }
        }

        // 其他方法与SimpleParser类似，但增加了错误处理
        private ASTNode parseFunctionDefinition() {
            try {
                consume(TokenType.DEF)
                def nameToken = consume(TokenType.IDENTIFIER)
                consume(TokenType.LPAREN)

                def params = []
                if (!match(TokenType.RPAREN)) {
                    params.add(parseParameter())
                    while (match(TokenType.COMMA)) {
                        consume(TokenType.COMMA)
                        params.add(parseParameter())
                    }
                }

                consume(TokenType.RPAREN)
                consume(TokenType.LBRACE)

                def body = []
                while (!match(TokenType.RBRACE)) {
                    body.add(parseStatement())
                }

                consume(TokenType.RBRACE)

                def functionNode = new ASTNode("FunctionDefinition", nameToken.text, nameToken.line, nameToken.column)
                def paramsNode = new ASTNode("Parameters")
                params.each { paramsNode.addChild(it) }
                functionNode.addChild(paramsNode)

                def bodyNode = new ASTNode("FunctionBody")
                body.each { bodyNode.addChild(it) }
                functionNode.addChild(bodyNode)

                functionNode
            } catch (Exception e) {
                errors.add("Error parsing function definition: ${e.message}")
                new ASTNode("ErrorFunctionDefinition")
            }
        }

        private ASTNode parseParameter() {
            def nameToken = consume(TokenType.IDENTIFIER)
            new ASTNode("Parameter", nameToken.text, nameToken.line, nameToken.column)
        }

        // 其他方法实现...
    }

    // 2. 递归下降优化
    static class OptimizedRecursiveDescentParser {
        private final List<Token> tokens
        private int current = 0

        OptimizedRecursiveDescentParser(List<Token> tokens) {
            this.tokens = tokens
        }

        // 使用前瞻减少回溯
        Token peek(int offset = 0) {
            def index = current + offset
            index < tokens.size() ? tokens[index] : null
        }

        // 批量消费token
        List<Token> consumeWhile(TokenType... types) {
            def consumed = []
            while (peek()?.type in types) {
                consumed.add(tokens[current++])
            }
            consumed
        }

        // 预测解析器
        boolean predict(String... expectedSequences) {
            def tempCurrent = current

            for (sequence in expectedSequences) {
                def tokens = sequence.split(/\\s+/)
                def match = true

                for (tokenType in tokens) {
                    def actualToken = peek(tempCurrent - current)
                    if (!actualToken || actualToken.type.toString() != tokenType) {
                        match = false
                        break
                    }
                    tempCurrent++
                }

                if (match) {
                    return true
                }
                tempCurrent = current
            }

            false
        }

        ASTNode parseExpression() {
            // 使用预测选择最优解析路径
            if (predict("IDENTIFIER LPAREN")) {
                return parseFunctionCall()
            } else if (predict("IDENTIFIER")) {
                return parseVariableReference()
            } else if (predict("NUMBER")) {
                return parseNumberLiteral()
            } else if (predict("STRING")) {
                return parseStringLiteral()
            } else if (predict("LPAREN")) {
                return parseParenthesizedExpression()
            } else {
                throw new RuntimeException("Unexpected token in expression")
            }
        }

        private ASTNode parseFunctionCall() {
            def identifier = consume(TokenType.IDENTIFIER)
            consume(TokenType.LPAREN)

            def args = []
            if (!match(TokenType.RPAREN)) {
                args.add(parseExpression())
                while (match(TokenType.COMMA)) {
                    consume(TokenType.COMMA)
                    args.add(parseExpression())
                }
            }

            consume(TokenType.RPAREN)

            def callNode = new ASTNode("FunctionCall", identifier.text, identifier.line, identifier.column)
            args.each { callNode.addChild(it) }
            callNode
        }

        private ASTNode parseVariableReference() {
            def identifier = consume(TokenType.IDENTIFIER)
            new ASTNode("VariableReference", identifier.text, identifier.line, identifier.column)
        }

        private ASTNode parseNumberLiteral() {
            def number = consume(TokenType.NUMBER)
            new ASTNode("NumberLiteral", number.text, number.line, number.column)
        }

        private ASTNode parseStringLiteral() {
            def string = consume(TokenType.STRING)
            new ASTNode("StringLiteral", string.text, string.line, string.column)
        }

        private ASTNode parseParenthesizedExpression() {
            consume(TokenType.LPAREN)
            def expression = parseExpression()
            consume(TokenType.RPAREN)
            expression
        }

        private Token consume(TokenType type) {
            def token = peek()
            if (token && token.type == type) {
                current++
                return token
            }
            throw new RuntimeException("Expected $type but got ${token?.type}")
        }

        private boolean match(TokenType... types) {
            peek()?.type in types
        }
    }

    // 3. 语法分析缓存
    static class CachedParser {
        private final List<Token> tokens
        private int current = 0
        private final Map<Integer, ASTNode> parseCache = [:]

        CachedParser(List<Token> tokens) {
            this.tokens = tokens
        }

        ASTNode parseWithCache(int startPosition) {
            if (parseCache.containsKey(startPosition)) {
                // 如果已经解析过，直接返回缓存结果
                current = startPosition
                return parseCache[startPosition]
            }

            // 解析并缓存结果
            def result = parseExpression()
            parseCache[startPosition] = result
            result
        }

        void clearCache() {
            parseCache.clear()
        }

        // 其他方法...
    }

    static void demonstrateOptimizedParser() {
        def code = """
        def calculate(a, b) {
            if (a > b) {
                return a * 2
            } else {
                return b * 3
            }
        }
        """.stripIndent()

        def lexer = new SimpleLexer(code)
        def tokens = []
        def token = lexer.nextToken()

        while (token.type != TokenType.EOF) {
            tokens.add(token)
            token = lexer.nextToken()
        }

        def parser = new OptimizedRecursiveDescentParser(tokens)
        def ast = parser.parseExpression()

        println "优化后的语法分析结果："
        println ast
    }
}
```

## AST构建与转换

### AST节点体系

```groovy
class ASTNodeSystem {

    // 1. AST节点基类
    static abstract class GroovyASTNode {
        int lineNumber
        int columnNumber

        GroovyASTNode(int line = -1, int column = -1) {
            this.lineNumber = line
            this.columnNumber = column
        }

        abstract String getText()

        abstract void visit(GroovyASTVisitor visitor)

        String toString() {
            "${getClass().simpleName}[line:$lineNumber, col:$columnNumber]"
        }
    }

    // 2. 表达式节点
    static abstract class Expression extends GroovyASTNode {
        Expression(int line = -1, int column = -1) {
            super(line, column)
        }
    }

    static class ConstantExpression extends Expression {
        Object value

        ConstantExpression(Object value, int line = -1, int column = -1) {
            super(line, column)
            this.value = value
        }

        String getText() {
            value?.toString() ?: "null"
        }

        void visit(GroovyASTVisitor visitor) {
            visitor.visitConstant(this)
        }
    }

    static class VariableExpression extends Expression {
        String variableName

        VariableExpression(String name, int line = -1, int column = -1) {
            super(line, column)
            this.variableName = name
        }

        String getText() {
            variableName
        }

        void visit(GroovyASTVisitor visitor) {
            visitor.visitVariable(this)
        }
    }

    static class BinaryExpression extends Expression {
        Expression leftExpression
        String operation
        Expression rightExpression

        BinaryExpression(Expression left, String op, Expression right, int line = -1, int column = -1) {
            super(line, column)
            this.leftExpression = left
            this.operation = op
            this.rightExpression = right
        }

        String getText() {
            "(${leftExpression.text} $operation ${rightExpression.text})"
        }

        void visit(GroovyASTVisitor visitor) {
            visitor.visitBinaryExpression(this)
        }
    }

    static class MethodCallExpression extends Expression {
        Expression objectExpression
        String methodName
        List<Expression> arguments

        MethodCallExpression(Expression object, String method, List<Expression> args, int line = -1, int column = -1) {
            super(line, column)
            this.objectExpression = object
            this.methodName = method
            this.arguments = args ?: []
        }

        String getText() {
            "${objectExpression.text}.$methodName(${arguments.collect { it.text }.join(', ')})"
        }

        void visit(GroovyASTVisitor visitor) {
            visitor.visitMethodCall(this)
        }
    }

    // 3. 语句节点
    static abstract class Statement extends GroovyASTNode {
        Statement(int line = -1, int column = -1) {
            super(line, column)
        }
    }

    static class ExpressionStatement extends Statement {
        Expression expression

        ExpressionStatement(Expression expr, int line = -1, int column = -1) {
            super(line, column)
            this.expression = expr
        }

        String getText() {
            expression.text
        }

        void visit(GroovyASTVisitor visitor) {
            visitor.visitExpressionStatement(this)
        }
    }

    static class ReturnStatement extends Statement {
        Expression expression

        ReturnStatement(Expression expr = null, int line = -1, int column = -1) {
            super(line, column)
            this.expression = expr
        }

        String getText() {
            expression ? "return ${expression.text}" : "return"
        }

        void visit(GroovyASTVisitor visitor) {
            visitor.visitReturnStatement(this)
        }
    }

    static class IfStatement extends Statement {
        Expression booleanExpression
        Statement ifBlock
        Statement elseBlock

        IfStatement(Expression condition, Statement ifStmt, Statement elseStmt = null, int line = -1, int column = -1) {
            super(line, column)
            this.booleanExpression = condition
            this.ifBlock = ifStmt
            this.elseBlock = elseStmt
        }

        String getText() {
            def result = "if (${booleanExpression.text}) { ... }"
            if (elseBlock) {
                result += " else { ... }"
            }
            result
        }

        void visit(GroovyASTVisitor visitor) {
            visitor.visitIfStatement(this)
        }
    }

    // 4. 声明节点
    static class ClassNode extends GroovyASTNode {
        String name
        String superClass
        List<MethodNode> methods = []
        List<FieldNode> fields = []
        List<PropertyNode> properties = []

        ClassNode(String name, String superClass = "Object", int line = -1, int column = -1) {
            super(line, column)
            this.name = name
            this.superClass = superClass
        }

        String getText() {
            "class $name extends $superClass { ... }"
        }

        void addMethod(MethodNode method) {
            methods.add(method)
        }

        void addField(FieldNode field) {
            fields.add(field)
        }

        void addProperty(PropertyNode property) {
            properties.add(property)
        }

        void visit(GroovyASTVisitor visitor) {
            visitor.visitClass(this)
        }
    }

    static class MethodNode extends GroovyASTNode {
        String name
        Class returnType
        List<Parameter> parameters = []
        Statement code

        MethodNode(String name, Class returnType, int line = -1, int column = -1) {
            super(line, column)
            this.name = name
            this.returnType = returnType
        }

        String getText() {
            def paramsText = parameters.collect { "${it.type.simpleName} ${it.name}" }.join(', ')
            "$returnType.simpleName $name($paramsText) { ... }"
        }

        void addParameter(Parameter param) {
            parameters.add(param)
        }

        void visit(GroovyASTVisitor visitor) {
            visitor.visitMethod(this)
        }
    }

    static class FieldNode extends GroovyASTNode {
        String name
        Class type
        Expression initialValue

        FieldNode(String name, Class type, Expression initialValue = null, int line = -1, int column = -1) {
            super(line, column)
            this.name = name
            this.type = type
            this.initialValue = initialValue
        }

        String getText() {
            initialValue ? "$type.simpleName $name = ${initialValue.text}" : "$type.simpleName $name"
        }

        void visit(GroovyASTVisitor visitor) {
            visitor.visitField(this)
        }
    }

    static class PropertyNode extends GroovyASTNode {
        String name
        Class type
        Expression getterExpression
        Expression setterExpression

        PropertyNode(String name, Class type, int line = -1, int column = -1) {
            super(line, column)
            this.name = name
            this.type = type
        }

        String getText() {
            "property $type.simpleName $name"
        }

        void visit(GroovyASTVisitor visitor) {
            visitor.visitProperty(this)
        }
    }

    // 5. AST访问者模式
    static interface GroovyASTVisitor {
        void visitConstant(ConstantExpression expr)
        void visitVariable(VariableExpression expr)
        void visitBinaryExpression(BinaryExpression expr)
        void visitMethodCall(MethodCallExpression expr)
        void visitExpressionStatement(ExpressionStatement stmt)
        void visitReturnStatement(ReturnStatement stmt)
        void visitIfStatement(IfStatement stmt)
        void visitClass(ClassNode classNode)
        void visitMethod(MethodNode methodNode)
        void visitField(FieldNode fieldNode)
        void visitProperty(PropertyNode propertyNode)
    }

    // 6. 示例访问者
    static class PrintingVisitor implements GroovyASTVisitor {
        private int indent = 0

        private String getIndent() {
            '  ' * indent
        }

        void visitConstant(ConstantExpression expr) {
            println "${getIndent()}Constant: ${expr.value}"
        }

        void visitVariable(VariableExpression expr) {
            println "${getIndent()}Variable: ${expr.variableName}"
        }

        void visitBinaryExpression(BinaryExpression expr) {
            println "${getIndent()}Binary: ${expr.operation}"
            indent++
            expr.leftExpression.visit(this)
            expr.rightExpression.visit(this)
            indent--
        }

        void visitMethodCall(MethodCallExpression expr) {
            println "${getIndent()}MethodCall: ${expr.methodName}"
            indent++
            expr.objectExpression.visit(this)
            expr.arguments.each { it.visit(this) }
            indent--
        }

        void visitExpressionStatement(ExpressionStatement stmt) {
            println "${getIndent()}ExpressionStatement:"
            indent++
            stmt.expression.visit(this)
            indent--
        }

        void visitReturnStatement(ReturnStatement stmt) {
            println "${getIndent()}ReturnStatement:"
            indent++
            stmt.expression?.visit(this)
            indent--
        }

        void visitIfStatement(IfStatement stmt) {
            println "${getIndent()}IfStatement:"
            indent++
            println "${getIndent()}Condition:"
            stmt.booleanExpression.visit(this)
            println "${getIndent()}IfBlock:"
            stmt.ifBlock.visit(this)
            if (stmt.elseBlock) {
                println "${getIndent()}ElseBlock:"
                stmt.elseBlock.visit(this)
            }
            indent--
        }

        void visitClass(ClassNode classNode) {
            println "${getIndent()}Class: ${classNode.name}"
            indent++
            println "${getIndent()}Fields:"
            classNode.fields.each { it.visit(this) }
            println "${getIndent()}Methods:"
            classNode.methods.each { it.visit(this) }
            indent--
        }

        void visitMethod(MethodNode methodNode) {
            println "${getIndent()}Method: ${methodNode.name}"
            indent++
            println "${getIndent()}Parameters:"
            methodNode.parameters.each { println "${getIndent()}  ${it.type.simpleName} ${it.name}" }
            println "${getIndent()}Code:"
            methodNode.code?.visit(this)
            indent--
        }

        void visitField(FieldNode fieldNode) {
            println "${getIndent()}Field: ${fieldNode.type.simpleName} ${fieldNode.name}"
            indent++
            fieldNode.initialValue?.visit(this)
            indent--
        }

        void visitProperty(PropertyNode propertyNode) {
            println "${getIndent()}Property: ${propertyNode.type.simpleName} ${propertyNode.name}"
        }
    }

    static void demonstrateASTSystem() {
        // 构建一个简单的AST
        def classNode = new ClassNode("MyClass", "Object", 1, 1)

        def fieldNode = new FieldNode("count", int.class, new ConstantExpression(0), 2, 5)
        classNode.addField(fieldNode)

        def methodNode = new MethodNode("add", int.class, 4, 5)
        methodNode.addParameter(new Parameter("a", int.class))
        methodNode.addParameter(new Parameter("b", int.class))

        def returnStmt = new ReturnStatement(
            new BinaryExpression(
                new VariableExpression("a", 5, 10),
                "+",
                new VariableExpression("b", 5, 14),
                5, 12
            ),
            5, 5
        )

        methodNode.code = returnStmt
        classNode.addMethod(methodNode)

        // 使用访问者遍历AST
        def visitor = new PrintingVisitor()
        classNode.visit(visitor)
    }
}
```

### AST转换机制

```groovy
class ASTTransformationMechanism {

    // 1. AST转换基类
    static abstract class ASTTransformation {
        abstract void transform(ASTNode[] nodes, SourceUnit sourceUnit)

        protected void visitNode(ASTNode node, GroovyASTVisitor visitor) {
            node.visit(visitor)
        }

        protected void reportError(String message, ASTNode node, SourceUnit sourceUnit) {
            sourceUnit.addError(new SyntaxErrorMessage(
                message,
                node.lineNumber,
                node.columnNumber
            ))
        }
    }

    // 2. 日志转换示例
    static class LoggingTransformation extends ASTTransformation {
        void transform(ASTNode[] nodes, SourceUnit sourceUnit) {
            nodes.each { node ->
                if (node instanceof ClassNode) {
                    transformClass(node as ClassNode, sourceUnit)
                }
            }
        }

        private void transformClass(ClassNode classNode, SourceUnit sourceUnit) {
            classNode.methods.each { method ->
                if (method.name.startsWith("process")) {
                    addLoggingToMethod(method, sourceUnit)
                }
            }
        }

        private void addLoggingToMethod(MethodNode methodNode, SourceUnit sourceUnit) {
            def originalCode = methodNode.code
            if (!(originalCode instanceof BlockStatement)) {
                return
            }

            // 创建日志语句
            def logExpression = new MethodCallExpression(
                new VariableExpression("log"),
                "info",
                [
                    new ConstantExpression("Entering method: ${methodNode.name}"),
                    new ConstantExpression(new Date())
                ]
            )

            def logStatement = new ExpressionStatement(logExpression)

            // 创建新的代码块
            def newBlock = new BlockStatement()
            newBlock.addStatement(logStatement)
            newBlock.addStatement(originalCode)

            methodNode.code = newBlock
        }
    }

    // 3. 属性注入转换
    static class PropertyInjectionTransformation extends ASTTransformation {
        void transform(ASTNode[] nodes, SourceUnit sourceUnit) {
            nodes.each { node ->
                if (node instanceof ClassNode) {
                    injectProperties(node as ClassNode, sourceUnit)
                }
            }
        }

        private void injectProperties(ClassNode classNode, SourceUnit sourceUnit) {
            // 注入service属性
            def serviceProperty = new PropertyNode("service", Service.class)
            classNode.addProperty(serviceProperty)

            // 注入logger属性
            def loggerProperty = new PropertyNode("logger", Logger.class)
            classNode.addProperty(loggerProperty)

            // 为所有方法添加初始化逻辑
            classNode.methods.each { method ->
                if (method.name != "<init>") {
                    addInitializationToMethod(method, sourceUnit)
                }
            }
        }

        private void addInitializationToMethod(MethodNode methodNode, SourceUnit sourceUnit) {
            def originalCode = methodNode.code
            if (!(originalCode instanceof BlockStatement)) {
                return
            }

            // 创建初始化代码
            def initBlock = new BlockStatement()

            // 初始化logger
            def loggerInit = new ExpressionStatement(
                new BinaryExpression(
                    new VariableExpression("logger"),
                    "=",
                    new MethodCallExpression(
                        new VariableExpression("Logger"),
                        "getLogger",
                        [new VariableExpression("class")]
                    )
                )
            )

            initBlock.addStatement(loggerInit)

            // 添加原有代码
            initBlock.addStatement(originalCode)

            methodNode.code = initBlock
        }
    }

    // 4. 缓存转换
    static class CacheTransformation extends ASTTransformation {
        void transform(ASTNode[] nodes, SourceUnit sourceUnit) {
            nodes.each { node ->
                if (node instanceof MethodNode) {
                    addCachingToMethod(node as MethodNode, sourceUnit)
                }
            }
        }

        private void addCachingToMethod(MethodNode methodNode, SourceUnit sourceUnit) {
            if (methodNode.returnType == void.class) {
                return  // 不缓存void方法
            }

            def originalCode = methodNode.code
            if (!(originalCode instanceof BlockStatement)) {
                return
            }

            // 创建缓存逻辑
            def cacheLogic = createCacheLogic(methodNode, originalCode)

            methodNode.code = cacheLogic
        }

        private BlockStatement createCacheLogic(MethodNode methodNode, Statement originalCode) {
            def cacheKey = "${methodNode.name}_${methodNode.parameters.collect { it.name }.join('_')}"

            def cacheBlock = new BlockStatement()

            // 检查缓存
            def cacheCheck = new IfStatement(
                new MethodCallExpression(
                    new VariableExpression("cache"),
                    "containsKey",
                    [new ConstantExpression(cacheKey)]
                ),
                new ReturnStatement(
                    new MethodCallExpression(
                        new VariableExpression("cache"),
                        "get",
                        [new ConstantExpression(cacheKey)]
                    )
                ),
                null
            )

            cacheBlock.addStatement(cacheCheck)

            // 执行原方法
            cacheBlock.addStatement(originalCode)

            // 存入缓存
            def cacheStore = new ExpressionStatement(
                new MethodCallExpression(
                    new VariableExpression("cache"),
                    "put",
                    [
                        new ConstantExpression(cacheKey),
                        new VariableExpression("result")
                    ]
                )
            )

            cacheBlock.addStatement(cacheStore)

            cacheBlock
        }
    }

    // 5. 事务管理转换
    static class TransactionTransformation extends ASTTransformation {
        void transform(ASTNode[] nodes, SourceUnit sourceUnit) {
            nodes.each { node ->
                if (node instanceof MethodNode) {
                    def methodNode = node as MethodNode
                    if (methodNode.name.startsWith("update") || methodNode.name.startsWith("save")) {
                        addTransactionManagement(methodNode, sourceUnit)
                    }
                }
            }
        }

        private void addTransactionManagement(MethodNode methodNode, SourceUnit sourceUnit) {
            def originalCode = methodNode.code
            if (!(originalCode instanceof BlockStatement)) {
                return
            }

            // 创建事务包装
            def transactionBlock = new BlockStatement()

            // 开始事务
            def beginTx = new ExpressionStatement(
                new MethodCallExpression(
                    new VariableExpression("transactionManager"),
                    "begin",
                    []
                )
            )

            transactionBlock.addStatement(beginTx)

            try {
                // 执行原方法
                transactionBlock.addStatement(originalCode)

                // 提交事务
                def commitTx = new ExpressionStatement(
                    new MethodCallExpression(
                        new VariableExpression("transactionManager"),
                        "commit",
                        []
                    )
                )

                transactionBlock.addStatement(commitTx)

            } catch (Exception e) {
                // 回滚事务
                def rollbackTx = new ExpressionStatement(
                    new MethodCallExpression(
                        new VariableExpression("transactionManager"),
                        "rollback",
                        []
                    )
                )

                transactionBlock.addStatement(rollbackTx)
                throw e
            }

            methodNode.code = transactionBlock
        }
    }

    // 6. 转换注册器
    static class TransformationRegistry {
        private static final Map<String, Class<? extends ASTTransformation>> transformations = [:]

        static {
            registerTransformation("logging", LoggingTransformation)
            registerTransformation("propertyInjection", PropertyInjectionTransformation)
            registerTransformation("cache", CacheTransformation)
            registerTransformation("transaction", TransactionTransformation)
        }

        static void registerTransformation(String name, Class<? extends ASTTransformation> transformationClass) {
            transformations[name] = transformationClass
        }

        static ASTTransformation getTransformation(String name) {
            def transformationClass = transformations[name]
            transformationClass?.newInstance()
        }

        static List<String> getAvailableTransformations() {
            transformations.keySet().toList()
        }
    }

    // 7. 转换执行器
    static class TransformationExecutor {
        static void executeTransformations(SourceUnit sourceUnit, List<String> transformationNames) {
            def ast = sourceUnit.ast

            transformationNames.each { name ->
                def transformation = TransformationRegistry.getTransformation(name)
                if (transformation) {
                    println "Applying transformation: $name"
                    transformation.transform([ast] as ASTNode[], sourceUnit)
                } else {
                    println "Unknown transformation: $name"
                }
            }
        }
    }

    static void demonstrateASTTransformations() {
        // 创建一个简单的类AST
        def classNode = new ClassNode("UserService", "Object", 1, 1)

        def methodNode = new MethodNode("saveUser", void.class, 3, 5)
        methodNode.addParameter(new Parameter("user", User.class))

        def saveCode = new BlockStatement()
        saveCode.addStatement(
            new ExpressionStatement(
                new MethodCallExpression(
                    new VariableExpression("userRepository"),
                    "save",
                    [new VariableExpression("user")]
                )
            )
        )

        methodNode.code = saveCode
        classNode.addMethod(methodNode)

        // 创建模拟的SourceUnit
        def sourceUnit = new MockSourceUnit(classNode)

        // 应用转换
        def transformations = ["logging", "propertyInjection", "transaction"]
        TransformationExecutor.executeTransformations(sourceUnit, transformations)

        // 显示转换后的AST
        def visitor = new PrintingVisitor()
        classNode.visit(visitor)
    }
}
```

### AST优化技术

```groovy
class ASTOptimization {

    // 1. 常量折叠
    static class ConstantFoldingOptimizer {
        static Expression optimize(Expression expr) {
            if (expr instanceof BinaryExpression) {
                return optimizeBinaryExpression(expr as BinaryExpression)
            } else if (expr instanceof UnaryExpression) {
                return optimizeUnaryExpression(expr as UnaryExpression)
            }
            expr
        }

        private static Expression optimizeBinaryExpression(BinaryExpression expr) {
            def left = optimize(expr.leftExpression)
            def right = optimize(expr.rightExpression)

            if (left instanceof ConstantExpression && right instanceof ConstantExpression) {
                def leftValue = (left as ConstantExpression).value
                def rightValue = (right as ConstantExpression).value

                try {
                    def result = evaluateBinaryExpression(leftValue, expr.operation, rightValue)
                    return new ConstantExpression(result, expr.lineNumber, expr.columnNumber)
                } catch (Exception e) {
                    // 计算失败，返回原表达式
                    return new BinaryExpression(left, expr.operation, right, expr.lineNumber, expr.columnNumber)
                }
            }

            return new BinaryExpression(left, expr.operation, right, expr.lineNumber, expr.columnNumber)
        }

        private static Expression optimizeUnaryExpression(UnaryExpression expr) {
            def operand = optimize(expr.expression)

            if (operand instanceof ConstantExpression) {
                def value = (operand as ConstantExpression).value
                try {
                    def result = evaluateUnaryExpression(expr.operation, value)
                    return new ConstantExpression(result, expr.lineNumber, expr.columnNumber)
                } catch (Exception e) {
                    // 计算失败，返回原表达式
                    return new UnaryExpression(expr.operation, operand, expr.lineNumber, expr.columnNumber)
                }
            }

            return new UnaryExpression(expr.operation, operand, expr.lineNumber, expr.columnNumber)
        }

        private static Object evaluateBinaryExpression(Object left, String op, Object right) {
            switch (op) {
                case "+": return left + right
                case "-": return left - right
                case "*": return left * right
                case "/": return left / right
                case "%": return left % right
                case "==": return left == right
                case "!=": return left != right
                case "<": return left < right
                case ">": return left > right
                case "<=": return left <= right
                case ">=": return left >= right
                default: throw new RuntimeException("Unknown operator: $op")
            }
        }

        private static Object evaluateUnaryExpression(String op, Object operand) {
            switch (op) {
                case "-": return -operand
                case "!": return !operand
                case "~": return ~operand
                default: throw new RuntimeException("Unknown unary operator: $op")
            }
        }
    }

    // 2. 死代码消除
    static class DeadCodeEliminator {
        static Statement eliminateDeadCode(Statement stmt) {
            if (stmt instanceof IfStatement) {
                return optimizeIfStatement(stmt as IfStatement)
            } else if (stmt instanceof BlockStatement) {
                return optimizeBlockStatement(stmt as BlockStatement)
            }
            stmt
        }

        private static Statement optimizeIfStatement(IfStatement ifStmt) {
            def condition = optimizeCondition(ifStmt.booleanExpression)

            if (condition instanceof ConstantExpression) {
                def conditionValue = (condition as ConstantExpression).value
                if (conditionValue) {
                    return ifStmt.ifBlock  // 条件为真，只保留if块
                } else {
                    return ifStmt.elseBlock ?: new EmptyStatement()  // 条件为假，保留else块或空语句
                }
            }

            return new IfStatement(condition, ifStmt.ifBlock, ifStmt.elseBlock, ifStmt.lineNumber, ifStmt.columnNumber)
        }

        private static Statement optimizeBlockStatement(BlockStatement blockStmt) {
            def optimizedStatements = blockStmt.statements.collect { eliminateDeadCode(it) }
            def newBlock = new BlockStatement()
            optimizedStatements.each { newBlock.addStatement(it) }
            newBlock
        }

        private static Expression optimizeCondition(Expression expr) {
            ConstantFoldingOptimizer.optimize(expr)
        }
    }

    // 3. 内联优化
    static class Inliner {
        static Expression inlineMethodCalls(Expression expr, MethodCallResolver resolver) {
            if (expr instanceof MethodCallExpression) {
                def methodCall = expr as MethodCallExpression
                def inlinedExpr = tryInlineMethodCall(methodCall, resolver)
                return inlinedExpr ?: expr
            } else if (expr instanceof BinaryExpression) {
                def binaryExpr = expr as BinaryExpression
                def left = inlineMethodCalls(binaryExpr.leftExpression, resolver)
                def right = inlineMethodCalls(binaryExpr.rightExpression, resolver)
                return new BinaryExpression(left, binaryExpr.operation, right, expr.lineNumber, expr.columnNumber)
            }
            expr
        }

        private static Expression tryInlineMethodCall(MethodCallExpression methodCall, MethodCallResolver resolver) {
            def inlinedBody = resolver.getInlinedMethodBody(methodCall.methodName, methodCall.arguments)
            if (inlinedBody) {
                // 创建参数替换
                def parameterReplacer = new ParameterReplacer(
                    methodCall.arguments.collect { it.text },
                    methodCall.lineNumber,
                    methodCall.columnNumber
                )

                return parameterReplacer.replaceParameters(inlinedBody)
            }
            null
        }
    }

    // 4. 参数替换器
    static class ParameterReplacer {
        private final Map<String, Expression> parameterMap = [:]
        private final int lineNumber
        private final int columnNumber

        ParameterReplacer(List<String> parameterNames, int line, int column) {
            this.lineNumber = line
            this.columnNumber = column
            parameterNames.eachWithIndex { name, index ->
                parameterMap[name] = new VariableExpression("arg$index", line, column)
            }
        }

        Expression replaceParameters(Expression expr) {
            if (expr instanceof VariableExpression) {
                def varExpr = expr as VariableExpression
                def replacement = parameterMap[varExpr.variableName]
                return replacement ?: expr
            } else if (expr instanceof BinaryExpression) {
                def binaryExpr = expr as BinaryExpression
                def left = replaceParameters(binaryExpr.leftExpression)
                def right = replaceParameters(binaryExpr.rightExpression)
                return new BinaryExpression(left, binaryExpr.operation, right, expr.lineNumber, expr.columnNumber)
            } else if (expr instanceof MethodCallExpression) {
                def methodCall = expr as MethodCallExpression
                def objectExpr = replaceParameters(methodCall.objectExpression)
                def args = methodCall.arguments.collect { replaceParameters(it) }
                return new MethodCallExpression(objectExpr, methodCall.methodName, args, expr.lineNumber, expr.columnNumber)
            }
            expr
        }
    }

    // 5. 循环优化
    static class LoopOptimizer {
        static Statement optimizeLoops(Statement stmt) {
            if (stmt instanceof ForStatement) {
                return optimizeForStatement(stmt as ForStatement)
            } else if (stmt instanceof WhileStatement) {
                return optimizeWhileStatement(stmt as WhileStatement)
            }
            stmt
        }

        private static Statement optimizeForStatement(ForStatement forStmt) {
            // 检查是否可以转换为for-each循环
            if (isSimpleForLoop(forStmt)) {
                return convertToForeach(forStmt)
            }

            // 优化循环不变量
            def optimizedBody = optimizeLoopInvariants(forStmt.body)
            return new ForStatement(forStmt.variable, forStmt.collection, optimizedBody, forStmt.lineNumber, forStmt.columnNumber)
        }

        private static Statement optimizeWhileStatement(WhileStatement whileStmt) {
            def optimizedCondition = ConstantFoldingOptimizer.optimize(whileStmt.booleanExpression)
            def optimizedBody = optimizeLoopInvariants(whileStmt.body)

            return new WhileStatement(optimizedCondition, optimizedBody, whileStmt.lineNumber, whileStmt.columnNumber)
        }

        private static Statement optimizeLoopInvariants(Statement body) {
            if (body instanceof BlockStatement) {
                def blockStmt = body as BlockStatement
                def invariantStatements = []
                def variantStatements = []

                blockStmt.statements.each { stmt ->
                    if (isLoopInvariant(stmt)) {
                        invariantStatements.add(stmt)
                    } else {
                        variantStatements.add(stmt)
                    }
                }

                // 如果有循环不变量，创建新的代码块结构
                if (!invariantStatements.isEmpty()) {
                    def newBlock = new BlockStatement()
                    invariantStatements.each { newBlock.addStatement(it) }
                    def loopBlock = new BlockStatement()
                    variantStatements.each { loopBlock.addStatement(it) }
                    newBlock.addStatement(loopBlock)
                    return newBlock
                }
            }

            return body
        }

        private static boolean isLoopInvariant(Statement stmt) {
            // 简化的循环不变量检测
            if (stmt instanceof ExpressionStatement) {
                def expr = (stmt as ExpressionStatement).expression
                return containsOnlyConstants(expr)
            }
            false
        }

        private static boolean containsOnlyConstants(Expression expr) {
            if (expr instanceof ConstantExpression) {
                return true
            } else if (expr instanceof BinaryExpression) {
                def binaryExpr = expr as BinaryExpression
                return containsOnlyConstants(binaryExpr.leftExpression) &&
                       containsOnlyConstants(binaryExpr.rightExpression)
            }
            false
        }

        private static boolean isSimpleForLoop(ForStatement forStmt) {
            // 检查是否是简单的迭代循环
            return forStmt.collection instanceof VariableExpression &&
                   forStmt.variable.type == Object.class
        }

        private static Statement convertToForeach(ForStatement forStmt) {
            def collectionExpr = forStmt.collection as VariableExpression
            def varName = forStmt.variable.name

            def foreachStmt = new ForeachStatement(
                new Parameter(varName, Object.class),
                collectionExpr,
                forStmt.body,
                forStmt.lineNumber,
                forStmt.columnNumber
            )

            foreachStmt
        }
    }

    static void demonstrateASTOptimization() {
        // 创建测试表达式
        def expr = new BinaryExpression(
            new BinaryExpression(
                new ConstantExpression(5),
                "+",
                new ConstantExpression(3)
            ),
            "*",
            new ConstantExpression(2)
        )

        println "Original expression: ${expr.text}"

        // 应用常量折叠
        def optimizedExpr = ConstantFoldingOptimizer.optimize(expr)
        println "Optimized expression: ${optimizedExpr.text}"

        // 创建测试方法调用
        def methodCall = new MethodCallExpression(
            new VariableExpression("Math"),
            "max",
            [
                new ConstantExpression(10),
                new ConstantExpression(20)
            ]
        )

        println "Original method call: ${methodCall.text}"

        // 应用内联优化
        def resolver = new SimpleMethodCallResolver()
        def inlinedExpr = Inliner.inlineMethodCalls(methodCall, resolver)
        println "Inlined expression: ${inlinedExpr.text}"
    }
}
```

## 代码生成

### 字节码生成基础

```groovy
class BytecodeGeneration {

    // 1. 字节码指令
    static enum BytecodeInstruction {
        NOP(0x00),
        ICONST_0(0x03),
        ICONST_1(0x04),
        ICONST_2(0x05),
        ICONST_3(0x06),
        ICONST_4(0x07),
        ICONST_5(0x08),
        BIPUSH(0x10),
        SIPUSH(0x11),
        LDC(0x12),
        ISTORE(0x36),
        ILOAD(0x15),
        ASTORE(0x3A),
        ALOAD(0x19),
        IADD(0x60),
        ISUB(0x64),
        IMUL(0x68),
        IDIV(0x6C),
        IRETURN(0xAC),
        ARETURN(0xB0),
        RETURN(0xB1),
        GETSTATIC(0xB2),
        PUTSTATIC(0xB3),
        GETFIELD(0xB4),
        PUTFIELD(0xB5),
        INVOKEVIRTUAL(0xB6),
        INVOKESPECIAL(0xB7),
        INVOKESTATIC(0xB8),
        NEW(0xBB),
        ANEWARRAY(0xBD),
        ARRAYLENGTH(0xBE),
        IFNE(0x9A),
        IFEQ(0x99),
        IFLT(0x9B),
        IFGT(0x9D),
        IF_ICMPEQ(0x9F),
        IF_ICMPNE(0xA0),
        GOTO(0xA7)

        final int opcode

        BytecodeInstruction(int opcode) {
            this.opcode = opcode
        }
    }

    // 2. 字节码生成器
    static class BytecodeGenerator {
        private final ByteArrayOutputStream outputStream = new ByteArrayOutputStream()
        private final DataOutputStream dataOutputStream

        BytecodeGenerator() {
            this.dataOutputStream = new DataOutputStream(outputStream)
        }

        void emitInstruction(BytecodeInstruction instruction) {
            emitByte(instruction.opcode)
        }

        void emitInstruction(BytecodeInstruction instruction, int operand) {
            emitByte(instruction.opcode)
            emitShort(operand)
        }

        void emitInstruction(BytecodeInstruction instruction, int operand1, int operand2) {
            emitByte(instruction.opcode)
            emitByte(operand1)
            emitByte(operand2)
        }

        void emitByte(int value) {
            dataOutputStream.writeByte(value)
        }

        void emitShort(int value) {
            dataOutputStream.writeShort(value)
        }

        void emitInt(int value) {
            dataOutputStream.writeInt(value)
        }

        void emitConstantPoolEntry(ConstantPoolEntry entry) {
            emitShort(entry.index)
        }

        byte[] toByteArray() {
            outputStream.toByteArray()
        }

        void reset() {
            outputStream.reset()
        }
    }

    // 3. 常量池
    static class ConstantPool {
        private final List<ConstantPoolEntry> entries = []

        int addEntry(ConstantPoolEntry entry) {
            def existingIndex = entries.indexOf(entry)
            if (existingIndex >= 0) {
                return existingIndex + 1  // 常量池索引从1开始
            }

            entries.add(entry)
            return entries.size()
        }

        ConstantPoolEntry getEntry(int index) {
            entries[index - 1]
        }

        int size() {
            entries.size()
        }

        byte[] toByteArray() {
            def baos = new ByteArrayOutputStream()
            def dos = new DataOutputStream(baos)

            dos.writeShort(size() + 1)  // 常量池大小

            entries.each { entry ->
                dos.write(entry.tag)
                dos.write(entry.data)
            }

            baos.toByteArray()
        }
    }

    // 4. 常量池条目
    static abstract class ConstantPoolEntry {
        abstract int getTag()
        abstract byte[] getData()

        boolean equals(Object obj) {
            if (this.is(obj)) return true
            if (obj == null || getClass() != obj.getClass()) return false

            def other = obj as ConstantPoolEntry
            this.tag == other.tag && Arrays.equals(this.data, other.data)
        }

        int hashCode() {
            31 * tag + Arrays.hashCode(data)
        }
    }

    static class ClassConstant extends ConstantPoolEntry {
        final String className
        final int nameIndex

        ClassConstant(String className, int nameIndex) {
            this.className = className
            this.nameIndex = nameIndex
        }

        int getTag() { 7 }
        byte[] getData() {
            def baos = new ByteArrayOutputStream()
            def dos = new DataOutputStream(baos)
            dos.writeShort(nameIndex)
            baos.toByteArray()
        }
    }

    static class FieldRefConstant extends ConstantPoolEntry {
        final int classIndex
        final int nameAndTypeIndex

        FieldRefConstant(int classIndex, int nameAndTypeIndex) {
            this.classIndex = classIndex
            this.nameAndTypeIndex = nameAndTypeIndex
        }

        int getTag() { 9 }
        byte[] getData() {
            def baos = new ByteArrayOutputStream()
            def dos = new DataOutputStream(baos)
            dos.writeShort(classIndex)
            dos.writeShort(nameAndTypeIndex)
            baos.toByteArray()
        }
    }

    static class MethodRefConstant extends ConstantPoolEntry {
        final int classIndex
        final int nameAndTypeIndex

        MethodRefConstant(int classIndex, int nameAndTypeIndex) {
            this.classIndex = classIndex
            this.nameAndTypeIndex = nameAndTypeIndex
        }

        int getTag() { 10 }
        byte[] getData() {
            def baos = new ByteArrayOutputStream()
            def dos = new DataOutputStream(baos)
            dos.writeShort(classIndex)
            dos.writeShort(nameAndTypeIndex)
            baos.toByteArray()
        }
    }

    static class NameAndTypeConstant extends ConstantPoolEntry {
        final int nameIndex
        final int descriptorIndex

        NameAndTypeConstant(int nameIndex, int descriptorIndex) {
            this.nameIndex = nameIndex
            this.descriptorIndex = descriptorIndex
        }

        int getTag() { 12 }
        byte[] getData() {
            def baos = new ByteArrayOutputStream()
            def dos = new DataOutputStream(baos)
            dos.writeShort(nameIndex)
            dos.writeShort(descriptorIndex)
            baos.toByteArray()
        }
    }

    static class Utf8Constant extends ConstantPoolEntry {
        final String value

        Utf8Constant(String value) {
            this.value = value
        }

        int getTag() { 1 }
        byte[] getData() {
            def baos = new ByteArrayOutputStream()
            def dos = new DataOutputStream(baos)
            def bytes = value.getBytes("UTF-8")
            dos.writeShort(bytes.length)
            dos.write(bytes)
            baos.toByteArray()
        }
    }

    // 5. 类文件生成器
    static class ClassFileGenerator {
        private final ConstantPool constantPool = new ConstantPool()
        private final BytecodeGenerator codeGenerator = new BytecodeGenerator()
        private final List<MethodInfo> methods = []
        private final List<FieldInfo> fields = []

        private int magic = 0xCAFEBABE
        private int minorVersion = 0
        private int majorVersion = 52  // Java 8

        private int accessFlags = 0x0021  // ACC_PUBLIC | ACC_SUPER
        private int thisClass
        private int superClass
        private int[] interfaces = []

        ClassFileGenerator(String className, String superClassName = "java/lang/Object") {
            this.thisClass = constantPool.addEntry(new ClassConstant(className,
                constantPool.addEntry(new Utf8Constant(className))))
            this.superClass = constantPool.addEntry(new ClassConstant(superClassName,
                constantPool.addEntry(new Utf8Constant(superClassName))))
        }

        void addMethod(String name, String descriptor, int accessFlags, byte[] code) {
            def codeAttribute = new CodeAttribute(code, 0, 0, [])
            def method = new MethodInfo(
                constantPool.addEntry(new Utf8Constant(name)),
                constantPool.addEntry(new Utf8Constant(descriptor)),
                accessFlags,
                [codeAttribute]
            )
            methods.add(method)
        }

        void addField(String name, String descriptor, int accessFlags) {
            def field = new FieldInfo(
                constantPool.addEntry(new Utf8Constant(name)),
                constantPool.addEntry(new Utf8Constant(descriptor)),
                accessFlags,
                []
            )
            fields.add(field)
        }

        byte[] generate() {
            def baos = new ByteArrayOutputStream()
            def dos = new DataOutputStream(baos)

            // 魔数和版本
            dos.writeInt(magic)
            dos.writeShort(minorVersion)
            dos.writeShort(majorVersion)

            // 常量池
            dos.write(constantPool.toByteArray())

            // 访问标志
            dos.writeShort(accessFlags)

            // 类信息
            dos.writeShort(thisClass)
            dos.writeShort(superClass)

            // 接口
            dos.writeShort(interfaces.length)
            interfaces.each { dos.writeShort(it) }

            // 字段
            dos.writeShort(fields.size())
            fields.each { field -> field.write(dos, constantPool) }

            // 方法
            dos.writeShort(methods.size())
            methods.each { method -> method.write(dos, constantPool) }

            // 属性
            dos.writeShort(0)  // 暂时没有类属性

            baos.toByteArray()
        }
    }

    // 6. 方法信息
    static class MethodInfo {
        final int nameIndex
        final int descriptorIndex
        final int accessFlags
        final List<Attribute> attributes

        MethodInfo(int nameIndex, int descriptorIndex, int accessFlags, List<Attribute> attributes) {
            this.nameIndex = nameIndex
            this.descriptorIndex = descriptorIndex
            this.accessFlags = accessFlags
            this.attributes = attributes
        }

        void write(DataOutputStream dos, ConstantPool constantPool) {
            dos.writeShort(accessFlags)
            dos.writeShort(nameIndex)
            dos.writeShort(descriptorIndex)
            dos.writeShort(attributes.size())
            attributes.each { attr -> attr.write(dos, constantPool) }
        }
    }

    // 7. 字段信息
    static class FieldInfo {
        final int nameIndex
        final int descriptorIndex
        final int accessFlags
        final List<Attribute> attributes

        FieldInfo(int nameIndex, int descriptorIndex, int accessFlags, List<Attribute> attributes) {
            this.nameIndex = nameIndex
            this.descriptorIndex = descriptorIndex
            this.accessFlags = accessFlags
            this.attributes = attributes
        }

        void write(DataOutputStream dos, ConstantPool constantPool) {
            dos.writeShort(accessFlags)
            dos.writeShort(nameIndex)
            dos.writeShort(descriptorIndex)
            dos.writeShort(attributes.size())
            attributes.each { attr -> attr.write(dos, constantPool) }
        }
    }

    // 8. 属性基类
    static abstract class Attribute {
        abstract String getName()
        abstract void write(DataOutputStream dos, ConstantPool constantPool)
    }

    // 9. 代码属性
    static class CodeAttribute extends Attribute {
        final byte[] code
        final int maxStack
        final int maxLocals
        final List<Attribute> attributes

        CodeAttribute(byte[] code, int maxStack, int maxLocals, List<Attribute> attributes) {
            this.code = code
            this.maxStack = maxStack
            this.maxLocals = maxLocals
            this.attributes = attributes
        }

        String getName() { "Code" }

        void write(DataOutputStream dos, ConstantPool constantPool) {
            dos.writeShort(maxStack)
            dos.writeShort(maxLocals)
            dos.writeInt(code.length)
            dos.write(code)
            dos.writeShort(0)  // 异常表长度
            dos.writeShort(attributes.size())
            attributes.each { attr -> attr.write(dos, constantPool) }
        }
    }

    static void demonstrateBytecodeGeneration() {
        // 创建一个简单的类
        def generator = new ClassFileGenerator("SimpleClass")

        // 添加字段
        generator.addField("value", "I", 0x0002)  // ACC_PRIVATE

        // 添加构造方法
        def constructorCode = new BytecodeGenerator()
        constructorCode.emitInstruction(BytecodeInstruction.ALOAD, 0)  // this
        constructorCode.emitInstruction(BytecodeInstruction.INVOKESPECIAL,
            generator.constantPool.addEntry(new MethodRefConstant(
                generator.superClass,
                generator.constantPool.addEntry(new NameAndTypeConstant(
                    generator.constantPool.addEntry(new Utf8Constant("<init>")),
                    generator.constantPool.addEntry(new Utf8Constant("()V"))
                ))
            ))
        )
        constructorCode.emitInstruction(BytecodeInstruction.RETURN)

        generator.addMethod("<init>", "()V", 0x0001, constructorCode.toByteArray())

        // 添加简单方法
        def methodCode = new BytecodeGenerator()
        methodCode.emitInstruction(BytecodeInstruction.ALOAD, 0)  // this
        methodCode.emitInstruction(BytecodeInstruction.GETFIELD,
            generator.constantPool.addEntry(new FieldRefConstant(
                generator.thisClass,
                generator.constantPool.addEntry(new NameAndTypeConstant(
                    generator.constantPool.addEntry(new Utf8Constant("value")),
                    generator.constantPool.addEntry(new Utf8Constant("I"))
                ))
            ))
        )
        methodCode.emitInstruction(BytecodeInstruction.IRETURN)

        generator.addMethod("getValue", "()I", 0x0001, methodCode.toByteArray())

        // 生成类文件
        def classFile = generator.generate()

        println "Generated class file size: ${classFile.length} bytes"

        // 保存到文件
        new File("SimpleClass.class").bytes = classFile
        println "Class file saved to SimpleClass.class"
    }
}
```

### 表达式代码生成

```groovy
class ExpressionCodeGeneration {

    // 1. 表达式代码生成器
    static class ExpressionCodeGenerator {
        private final ConstantPool constantPool
        private final BytecodeGenerator bytecodeGenerator
        private final MethodContext methodContext

        ExpressionCodeGenerator(ConstantPool constantPool, MethodContext methodContext) {
            this.constantPool = constantPool
            this.bytecodeGenerator = new BytecodeGenerator()
            this.methodContext = methodContext
        }

        void generateExpression(Expression expr) {
            if (expr instanceof ConstantExpression) {
                generateConstantExpression(expr as ConstantExpression)
            } else if (expr instanceof VariableExpression) {
                generateVariableExpression(expr as VariableExpression)
            } else if (expr instanceof BinaryExpression) {
                generateBinaryExpression(expr as BinaryExpression)
            } else if (expr instanceof MethodCallExpression) {
                generateMethodCallExpression(expr as MethodCallExpression)
            } else {
                throw new RuntimeException("Unsupported expression type: ${expr.class}")
            }
        }

        private void generateConstantExpression(ConstantExpression expr) {
            def value = expr.value

            if (value instanceof Integer) {
                generateIntegerConstant(value as Integer)
            } else if (value instanceof String) {
                generateStringConstant(value as String)
            } else if (value instanceof Boolean) {
                generateBooleanConstant(value as Boolean)
            } else {
                throw new RuntimeException("Unsupported constant type: ${value.class}")
            }
        }

        private void generateIntegerConstant(int value) {
            switch (value) {
                case 0:
                    bytecodeGenerator.emitInstruction(BytecodeInstruction.ICONST_0)
                    break
                case 1:
                    bytecodeGenerator.emitInstruction(BytecodeInstruction.ICONST_1)
                    break
                case 2:
                    bytecodeGenerator.emitInstruction(BytecodeInstruction.ICONST_2)
                    break
                case 3:
                    bytecodeGenerator.emitInstruction(BytecodeInstruction.ICONST_3)
                    break
                case 4:
                    bytecodeGenerator.emitInstruction(BytecodeInstruction.ICONST_4)
                    break
                case 5:
                    bytecodeGenerator.emitInstruction(BytecodeInstruction.ICONST_5)
                    break
                default:
                    if (value >= -128 && value <= 127) {
                        bytecodeGenerator.emitInstruction(BytecodeInstruction.BIPUSH, value)
                    } else if (value >= -32768 && value <= 32767) {
                        bytecodeGenerator.emitInstruction(BytecodeInstruction.SIPUSH, value)
                    } else {
                        def constantIndex = constantPool.addEntry(
                            new IntegerConstant(value)
                        )
                        bytecodeGenerator.emitInstruction(BytecodeInstruction.LDC, constantIndex)
                    }
            }
        }

        private void generateStringConstant(String value) {
            def constantIndex = constantPool.addEntry(
                new StringConstant(value)
            )
            bytecodeGenerator.emitInstruction(BytecodeInstruction.LDC, constantIndex)
        }

        private void generateBooleanConstant(boolean value) {
            bytecodeGenerator.emitInstruction(value ? BytecodeInstruction.ICONST_1 : BytecodeInstruction.ICONST_0)
        }

        private void generateVariableExpression(VariableExpression expr) {
            def variableName = expr.variableName
            def localIndex = methodContext.getLocalVariableIndex(variableName)

            if (localIndex >= 0) {
                bytecodeGenerator.emitInstruction(BytecodeInstruction.ILOAD, localIndex)
            } else {
                def fieldIndex = methodContext.getFieldIndex(variableName)
                if (fieldIndex >= 0) {
                    bytecodeGenerator.emitInstruction(BytecodeInstruction.ALOAD, 0)  // this
                    bytecodeGenerator.emitInstruction(BytecodeInstruction.GETFIELD, fieldIndex)
                } else {
                    throw new RuntimeException("Undefined variable: $variableName")
                }
            }
        }

        private void generateBinaryExpression(BinaryExpression expr) {
            // 生成左操作数
            generateExpression(expr.leftExpression)

            // 生成右操作数
            generateExpression(expr.rightExpression)

            // 生成运算指令
            switch (expr.operation) {
                case "+":
                    bytecodeGenerator.emitInstruction(BytecodeInstruction.IADD)
                    break
                case "-":
                    bytecodeGenerator.emitInstruction(BytecodeInstruction.ISUB)
                    break
                case "*":
                    bytecodeGenerator.emitInstruction(BytecodeInstruction.IMUL)
                    break
                case "/":
                    bytecodeGenerator.emitInstruction(BytecodeInstruction.IDIV)
                    break
                case "%":
                    bytecodeGenerator.emitInstruction(BytecodeInstruction.IREM)
                    break
                case "==":
                    generateComparison(BytecodeInstruction.IF_ICMPEQ, BytecodeInstruction.ICONST_1, BytecodeInstruction.ICONST_0)
                    break
                case "!=":
                    generateComparison(BytecodeInstruction.IF_ICMPNE, BytecodeInstruction.ICONST_1, BytecodeInstruction.ICONST_0)
                    break
                case "<":
                    generateComparison(BytecodeInstruction.IFLT, BytecodeInstruction.ICONST_1, BytecodeInstruction.ICONST_0)
                    break
                case ">":
                    generateComparison(BytecodeInstruction.IFGT, BytecodeInstruction.ICONST_1, BytecodeInstruction.ICONST_0)
                    break
                case "<=":
                    generateComparison(BytecodeInstruction.IF_ICMPLE, BytecodeInstruction.ICONST_1, BytecodeInstruction.ICONST_0)
                    break
                case ">=":
                    generateComparison(BytecodeInstruction.IF_ICMPGE, BytecodeInstruction.ICONST_1, BytecodeInstruction.ICONST_0)
                    break
                default:
                    throw new RuntimeException("Unsupported binary operator: ${expr.operation}")
            }
        }

        private void generateComparison(BytecodeInstruction compareOp, BytecodeInstruction trueValue, BytecodeInstruction falseValue) {
            def endLabel = new Label()

            bytecodeGenerator.emitInstruction(compareOp, endLabel)
            bytecodeGenerator.emitInstruction(falseValue)
            bytecodeGenerator.emitInstruction(BytecodeInstruction.GOTO, endLabel)

            endLabel.mark()
            bytecodeGenerator.emitInstruction(trueValue)
        }

        private void generateMethodCallExpression(MethodCallExpression expr) {
            // 生成对象表达式
            generateExpression(expr.objectExpression)

            // 生成参数
            expr.arguments.each { arg ->
                generateExpression(arg)
            }

            // 生成方法调用指令
            def methodRef = methodContext.getMethodReference(expr.methodName, expr.arguments.size())
            bytecodeGenerator.emitInstruction(BytecodeInstruction.INVOKEVIRTUAL, methodRef)
        }

        byte[] getGeneratedCode() {
            bytecodeGenerator.toByteArray()
        }
    }

    // 2. 方法上下文
    static class MethodContext {
        private final Map<String, Integer> localVariables = [:]
        private final Map<String, Integer> fields = [:]
        private final Map<String, MethodReference> methods = [:]
        private int nextLocalIndex = 0

        MethodContext() {
            // 初始化this引用
            localVariables["this"] = 0
            nextLocalIndex = 1
        }

        int addLocalVariable(String name) {
            def index = nextLocalIndex++
            localVariables[name] = index
            index
        }

        int getLocalVariableIndex(String name) {
            localVariables[name] ?: -1
        }

        void addField(String name, int constantPoolIndex) {
            fields[name] = constantPoolIndex
        }

        int getFieldIndex(String name) {
            fields[name] ?: -1
        }

        void addMethod(String name, String descriptor, int constantPoolIndex) {
            methods[name] = new MethodReference(name, descriptor, constantPoolIndex)
        }

        MethodReference getMethodReference(String name, int argumentCount) {
            def method = methods[name]
            if (method && method.argumentCount == argumentCount) {
                method
            } else {
                throw new RuntimeException("Method not found: $name with $argumentCount arguments")
            }
        }
    }

    // 3. 方法引用
    static class MethodReference {
        final String name
        final String descriptor
        final int constantPoolIndex
        final int argumentCount

        MethodReference(String name, String descriptor, int constantPoolIndex) {
            this.name = name
            this.descriptor = descriptor
            this.constantPoolIndex = constantPoolIndex
            this.argumentCount = descriptor.count { it == '(' ? 1 : 0 } - 1  // 简化的参数计数
        }
    }

    // 4. 标签管理
    static class Label {
        private int offset = -1

        void mark() {
            offset = bytecodeGenerator.getCurrentOffset()
        }

        int getOffset() {
            if (offset == -1) {
                throw new RuntimeException("Label not marked")
            }
            offset
        }
    }

    // 5. 语句代码生成器
    static class StatementCodeGenerator {
        private final ExpressionCodeGenerator exprGenerator
        private final MethodContext methodContext

        StatementCodeGenerator(ConstantPool constantPool, MethodContext methodContext) {
            this.exprGenerator = new ExpressionCodeGenerator(constantPool, methodContext)
            this.methodContext = methodContext
        }

        void generateStatement(Statement stmt) {
            if (stmt instanceof ExpressionStatement) {
                generateExpressionStatement(stmt as ExpressionStatement)
            } else if (stmt instanceof ReturnStatement) {
                generateReturnStatement(stmt as ReturnStatement)
            } else if (stmt instanceof IfStatement) {
                generateIfStatement(stmt as IfStatement)
            } else if (stmt instanceof WhileStatement) {
                generateWhileStatement(stmt as WhileStatement)
            } else {
                throw new RuntimeException("Unsupported statement type: ${stmt.class}")
            }
        }

        private void generateExpressionStatement(ExpressionStatement stmt) {
            exprGenerator.generateExpression(stmt.expression)
            // 如果表达式有值，弹出栈
            exprGenerator.bytecodeGenerator.emitInstruction(BytecodeInstruction.POP)
        }

        private void generateReturnStatement(ReturnStatement stmt) {
            if (stmt.expression) {
                exprGenerator.generateExpression(stmt.expression)
                exprGenerator.bytecodeGenerator.emitInstruction(BytecodeInstruction.IRETURN)
            } else {
                exprGenerator.bytecodeGenerator.emitInstruction(BytecodeInstruction.RETURN)
            }
        }

        private void generateIfStatement(IfStatement stmt) {
            // 生成条件表达式
            exprGenerator.generateExpression(stmt.booleanExpression)

            // 比较结果，跳转到else块
            def elseLabel = new Label()
            def endLabel = new Label()

            exprGenerator.bytecodeGenerator.emitInstruction(BytecodeInstruction.IFEQ, elseLabel)

            // 生成if块
            generateStatement(stmt.ifBlock)
            exprGenerator.bytecodeGenerator.emitInstruction(BytecodeInstruction.GOTO, endLabel)

            // 生成else块
            elseLabel.mark()
            if (stmt.elseBlock) {
                generateStatement(stmt.elseBlock)
            }

            endLabel.mark()
        }

        private void generateWhileStatement(WhileStatement stmt) {
            def startLabel = new Label()
            def endLabel = new Label()

            startLabel.mark()

            // 生成条件表达式
            exprGenerator.generateExpression(stmt.booleanExpression)
            exprGenerator.bytecodeGenerator.emitInstruction(BytecodeInstruction.IFEQ, endLabel)

            // 生成循环体
            generateStatement(stmt.body)
            exprGenerator.bytecodeGenerator.emitInstruction(BytecodeInstruction.GOTO, startLabel)

            endLabel.mark()
        }
    }

    static void demonstrateExpressionCodeGeneration() {
        // 创建常量池和方法上下文
        def constantPool = new ConstantPool()
        def methodContext = new MethodContext()

        // 创建表达式代码生成器
        def exprGenerator = new ExpressionCodeGenerator(constantPool, methodContext)

        // 生成简单表达式: 5 + 3 * 2
        def expr = new BinaryExpression(
            new ConstantExpression(5),
            "+",
            new BinaryExpression(
                new ConstantExpression(3),
                "*",
                new ConstantExpression(2)
            )
        )

        exprGenerator.generateExpression(expr)

        def generatedCode = exprGenerator.getGeneratedCode()
        println "Generated code for '5 + 3 * 2': ${generatedCode.length} bytes"

        // 打印生成的字节码
        printBytecode(generatedCode)
    }

    static void printBytecode(byte[] bytecode) {
        println "Generated bytecode:"
        bytecode.eachWithIndex { byteValue, index ->
            printf "%02X ", byteValue
            if ((index + 1) % 16 == 0) {
                println()
            }
        }
        println()
    }
}
```

### 代码生成优化

```groovy
class CodeGenerationOptimization {

    // 1. 寄存器分配器
    static class RegisterAllocator {
        private final int maxLocals
        private final BitSet usedLocals = new BitSet()
        private final Stack<Integer> freeLocals = new Stack<>()

        RegisterAllocator(int maxLocals) {
            this.maxLocals = maxLocals
        }

        int allocateRegister() {
            if (!freeLocals.isEmpty()) {
                return freeLocals.pop()
            }

            for (int i = 0; i < maxLocals; i++) {
                if (!usedLocals.get(i)) {
                    usedLocals.set(i)
                    return i
                }
            }

            throw new RuntimeException("No available registers")
        }

        void freeRegister(int register) {
            if (usedLocals.get(register)) {
                usedLocals.clear(register)
                freeLocals.push(register)
            }
        }

        void optimizeRegisterUsage(List<Statement> statements) {
            // 分析变量生命周期
            def liveRanges = analyzeLiveRanges(statements)

            // 应用寄存器分配算法
            applyRegisterAllocation(liveRanges)
        }

        private Map<String, List<Integer>> analyzeLiveRanges(List<Statement> statements) {
            def liveRanges = [:]

            statements.eachWithIndex { stmt, index ->
                def usedVariables = extractUsedVariables(stmt)
                usedVariables.each { variable ->
                    if (!liveRanges.containsKey(variable)) {
                        liveRanges[variable] = []
                    }
                    liveRanges[variable] << index
                }
            }

            liveRanges
        }

        private List<String> extractUsedVariables(Statement stmt) {
            def variables = []

            if (stmt instanceof ExpressionStatement) {
                variables.addAll(extractVariablesFromExpression(stmt.expression))
            } else if (stmt instanceof ReturnStatement) {
                if (stmt.expression) {
                    variables.addAll(extractVariablesFromExpression(stmt.expression))
                }
            }

            variables
        }

        private List<String> extractVariablesFromExpression(Expression expr) {
            def variables = []

            if (expr instanceof VariableExpression) {
                variables.add(expr.variableName)
            } else if (expr instanceof BinaryExpression) {
                variables.addAll(extractVariablesFromExpression(expr.leftExpression))
                variables.addAll(extractVariablesFromExpression(expr.rightExpression))
            } else if (expr instanceof MethodCallExpression) {
                variables.addAll(extractVariablesFromExpression(expr.objectExpression))
                expr.arguments.each { arg ->
                    variables.addAll(extractVariablesFromExpression(arg))
                }
            }

            variables
        }

        private void applyRegisterAllocation(Map<String, List<Integer>> liveRanges) {
            // 简化的寄存器分配算法
            def variablesByFirstUse = liveRanges.collectMany { variable, ranges ->
                [[variable: variable, firstUse: ranges.min()]]
            }.sort { it.firstUse }

            def allocatedRegisters = [:]

            variablesByFirstUse.each { entry ->
                def variable = entry.variable
                def firstUse = entry.firstUse

                // 检查是否有可用的寄存器
                def availableRegister = findAvailableRegister(variable, firstUse, allocatedRegisters)

                if (availableRegister != null) {
                    allocatedRegisters[variable] = availableRegister
                }
            }

            // 应用寄存器分配结果
            allocatedRegisters.each { variable, register ->
                println "Assigned variable '$variable' to register $register"
            }
        }

        private Integer findAvailableRegister(String variable, int firstUse, Map<String, Integer> allocatedRegisters) {
            // 简化的寄存器查找算法
            def usedRegisters = allocatedRegisters.values().toSet()

            for (int i = 1; i < maxLocals; i++) {  // 跳过this引用
                if (!usedRegisters.contains(i)) {
                    return i
                }
            }

            null
        }
    }

    // 2. 死代码消除
    static class DeadCodeEliminator {
        static List<Statement> eliminateDeadCode(List<Statement> statements) {
            def optimizedStatements = []
            def liveVariables = analyzeLiveVariables(statements)

            statements.reverseEach { stmt ->
                if (isStatementLive(stmt, liveVariables)) {
                    optimizedStatements.add(0, stmt)
                }
            }

            optimizedStatements
        }

        private static boolean isStatementLive(Statement stmt, Set<String> liveVariables) {
            if (stmt instanceof ExpressionStatement) {
                // 检查表达式是否有副作用
                return hasSideEffects(stmt.expression) ||
                       usesLiveVariables(stmt.expression, liveVariables)
            } else if (stmt instanceof ReturnStatement) {
                return true  // return语句总是活的
            } else if (stmt instanceof IfStatement) {
                return isStatementLive(stmt.ifBlock, liveVariables) ||
                       (stmt.elseBlock && isStatementLive(stmt.elseBlock, liveVariables))
            } else if (stmt instanceof WhileStatement) {
                return isStatementLive(stmt.body, liveVariables)
            }

            true
        }

        private static boolean hasSideEffects(Expression expr) {
            if (expr instanceof MethodCallExpression) {
                return true  // 方法调用可能有副作用
            } else if (expr instanceof BinaryExpression) {
                return hasSideEffects(expr.leftExpression) ||
                       hasSideEffects(expr.rightExpression)
            } else if (expr instanceof VariableExpression) {
                return false
            } else if (expr instanceof ConstantExpression) {
                return false
            }

            false
        }

        private static boolean usesLiveVariables(Expression expr, Set<String> liveVariables) {
            if (expr instanceof VariableExpression) {
                return liveVariables.contains(expr.variableName)
            } else if (expr instanceof BinaryExpression) {
                return usesLiveVariables(expr.leftExpression, liveVariables) ||
                       usesLiveVariables(expr.rightExpression, liveVariables)
            } else if (expr instanceof MethodCallExpression) {
                return usesLiveVariables(expr.objectExpression, liveVariables) ||
                       expr.arguments.any { usesLiveVariables(it, liveVariables) }
            }

            false
        }

        private static Set<String> analyzeLiveVariables(List<Statement> statements) {
            def liveVariables = new HashSet<>()

            statements.reverseEach { stmt ->
                if (stmt instanceof ExpressionStatement) {
                    updateLiveVariables(stmt.expression, liveVariables)
                } else if (stmt instanceof ReturnStatement) {
                    if (stmt.expression) {
                        updateLiveVariables(stmt.expression, liveVariables)
                    }
                } else if (stmt instanceof IfStatement) {
                    updateLiveVariables(stmt.booleanExpression, liveVariables)
                    if (stmt.elseBlock) {
                        updateLiveVariables(stmt.elseBlock, liveVariables)
                    }
                } else if (stmt instanceof WhileStatement) {
                    updateLiveVariables(stmt.booleanExpression, liveVariables)
                    updateLiveVariables(stmt.body, liveVariables)
                }
            }

            liveVariables
        }

        private static void updateLiveVariables(Expression expr, Set<String> liveVariables) {
            if (expr instanceof VariableExpression) {
                liveVariables.add(expr.variableName)
            } else if (expr instanceof BinaryExpression) {
                updateLiveVariables(expr.leftExpression, liveVariables)
                updateLiveVariables(expr.rightExpression, liveVariables)
            } else if (expr instanceof MethodCallExpression) {
                updateLiveVariables(expr.objectExpression, liveVariables)
                expr.arguments.each { arg ->
                    updateLiveVariables(arg, liveVariables)
                }
            }
        }
    }

    // 3. 常量传播
    static class ConstantPropagator {
        static Expression propagateConstants(Expression expr, Map<String, Object> constantValues) {
            if (expr instanceof VariableExpression) {
                def varExpr = expr as VariableExpression
                if (constantValues.containsKey(varExpr.variableName)) {
                    return new ConstantExpression(constantValues[varExpr.variableName])
                }
                return expr
            } else if (expr instanceof BinaryExpression) {
                def binaryExpr = expr as BinaryExpression
                def left = propagateConstants(binaryExpr.leftExpression, constantValues)
                def right = propagateConstants(binaryExpr.rightExpression, constantValues)

                if (left instanceof ConstantExpression && right instanceof ConstantExpression) {
                    def leftValue = (left as ConstantExpression).value
                    def rightValue = (right as ConstantExpression).value

                    try {
                        def result = evaluateConstantExpression(leftValue, binaryExpr.operation, rightValue)
                        return new ConstantExpression(result)
                    } catch (Exception e) {
                        // 计算失败，返回原表达式
                    }
                }

                return new BinaryExpression(left, binaryExpr.operation, right)
            }

            expr
        }

        private static Object evaluateConstantExpression(Object left, String operation, Object right) {
            switch (operation) {
                case "+": return left + right
                case "-": return left - right
                case "*": return left * right
                case "/": return left / right
                case "%": return left % right
                case "==": return left == right
                case "!=": return left != right
                case "<": return left < right
                case ">": return left > right
                case "<=": return left <= right
                case ">=": return left >= right
                default: throw new RuntimeException("Unknown operation: $operation")
            }
        }

        static Map<String, Object> analyzeConstants(List<Statement> statements) {
            def constantValues = [:]

            statements.each { stmt ->
                if (stmt instanceof ExpressionStatement) {
                    def expr = stmt.expression
                    if (expr instanceof BinaryExpression && expr.operation == "=") {
                        def left = expr.leftExpression
                        def right = expr.rightExpression

                        if (left instanceof VariableExpression && right instanceof ConstantExpression) {
                            def varName = (left as VariableExpression).variableName
                            def constValue = (right as ConstantExpression).value
                            constantValues[varName] = constValue
                        }
                    }
                }
            }

            constantValues
        }
    }

    // 4. 循环优化
    static class LoopOptimizer {
        static Statement optimizeLoops(Statement stmt) {
            if (stmt instanceof ForStatement) {
                return optimizeForStatement(stmt as ForStatement)
            } else if (stmt instanceof WhileStatement) {
                return optimizeWhileStatement(stmt as WhileStatement)
            }
            stmt
        }

        private static Statement optimizeForStatement(ForStatement forStmt) {
            // 检查是否可以转换为while循环
            if (isSimpleForLoop(forStmt)) {
                return convertToWhileLoop(forStmt)
            }

            // 优化循环不变量
            def optimizedBody = optimizeLoopInvariants(forStmt.body)
            return new ForStatement(forStmt.variable, forStmt.collection, optimizedBody)
        }

        private static Statement optimizeWhileStatement(WhileStatement whileStmt) {
            def optimizedBody = optimizeLoopInvariants(whileStmt.body)
            return new WhileStatement(whileStmt.booleanExpression, optimizedBody)
        }

        private static Statement optimizeLoopInvariants(Statement body) {
            if (body instanceof BlockStatement) {
                def blockStmt = body as BlockStatement
                def invariantStatements = []
                def variantStatements = []

                blockStmt.statements.each { stmt ->
                    if (isLoopInvariant(stmt)) {
                        invariantStatements.add(stmt)
                    } else {
                        variantStatements.add(stmt)
                    }
                }

                // 如果有循环不变量，创建新的代码块结构
                if (!invariantStatements.isEmpty()) {
                    def newBlock = new BlockStatement()
                    invariantStatements.each { newBlock.addStatement(it) }
                    def loopBlock = new BlockStatement()
                    variantStatements.each { loopBlock.addStatement(it) }
                    newBlock.addStatement(loopBlock)
                    return newBlock
                }
            }

            return body
        }

        private static boolean isLoopInvariant(Statement stmt) {
            if (stmt instanceof ExpressionStatement) {
                def expr = stmt.expression
                return containsOnlyConstants(expr)
            }
            false
        }

        private static boolean containsOnlyConstants(Expression expr) {
            if (expr instanceof ConstantExpression) {
                return true
            } else if (expr instanceof BinaryExpression) {
                def binaryExpr = expr as BinaryExpression
                return containsOnlyConstants(binaryExpr.leftExpression) &&
                       containsOnlyConstants(binaryExpr.rightExpression)
            }
            false
        }

        private static boolean isSimpleForLoop(ForStatement forStmt) {
            // 简化的简单循环检测
            return forStmt.collection instanceof VariableExpression
        }

        private static Statement convertToWhileLoop(ForStatement forStmt) {
            def iteratorVariable = new VariableExpression("iterator")
            def hasNextCall = new MethodCallExpression(
                forStmt.collection,
                "hasNext",
                []
            )
            def nextCall = new MethodCallExpression(
                forStmt.collection,
                "next",
                []
            )
            def assignment = new BinaryExpression(
                new VariableExpression(forStmt.variable.name),
                "=",
                nextCall
            )

            def whileBody = new BlockStatement()
            whileBody.addStatement(new ExpressionStatement(assignment))
            whileBody.addStatement(forStmt.body)

            new WhileStatement(hasNextCall, whileBody)
        }
    }

    static void demonstrateCodeGenerationOptimization() {
        // 创建测试语句
        def statements = [
            new ExpressionStatement(
                new BinaryExpression(
                    new VariableExpression("x"),
                    "=",
                    new ConstantExpression(10)
                )
            ),
            new ExpressionStatement(
                new BinaryExpression(
                    new VariableExpression("y"),
                    "=",
                    new ConstantExpression(20)
                )
            ),
            new ExpressionStatement(
                new BinaryExpression(
                    new VariableExpression("result"),
                    "=",
                    new BinaryExpression(
                        new VariableExpression("x"),
                        "+",
                        new VariableExpression("y")
                    )
                )
            ),
            new ReturnStatement(
                new VariableExpression("result")
            )
        ]

        // 应用常量传播
        def constantValues = ConstantPropagator.analyzeConstants(statements)
        println "Detected constant values: $constantValues"

        def optimizedStatements = statements.collect { stmt ->
            if (stmt instanceof ExpressionStatement) {
                def optimizedExpr = ConstantPropagator.propagateConstants(
                    stmt.expression, constantValues
                )
                new ExpressionStatement(optimizedExpr)
            } else {
                stmt
            }
        }

        println "Optimized statements:"
        optimizedStatements.each { stmt ->
            println "  $stmt"
        }

        // 应用死代码消除
        def liveStatements = DeadCodeEliminator.eliminateDeadCode(optimizedStatements)
        println "Live statements after dead code elimination:"
        liveStatements.each { stmt ->
            println "  $stmt"
        }

        // 应用寄存器分配
        def allocator = new RegisterAllocator(10)
        allocator.optimizeRegisterUsage(liveStatements)
    }
}
```

## 总结

Groovy编译器内部机制是一个复杂而强大的系统，它将Groovy源代码转换为高效的JVM字节码。通过本章节的学习，我们深入了解了：

### 核心组件总结

1. **词法分析器**：将源代码转换为token序列，处理Groovy特有的语法元素
2. **语法分析器**：构建抽象语法树，验证语法结构，支持错误恢复
3. **AST系统**：提供完整的AST节点体系，支持访问者模式
4. **AST转换**：实现编译时元编程，支持各种代码转换
5. **代码生成**：生成高效字节码，支持各种JVM特性

### 关键技术要点

1. **词法分析优化**：字符缓冲区、Token缓存、批量处理
2. **语法分析增强**：错误恢复、递归下降优化、语法分析缓存
3. **AST操作**：节点操作、访问者模式、转换机制
4. **代码生成**：字节码指令、常量池、方法生成
5. **性能优化**：常量折叠、死代码消除、寄存器分配

### 实际应用价值

1. **性能调优**：理解编译过程有助于编写更高效的代码
2. **元编程开发**：可以开发自定义的AST转换和编译器扩展
3. **调试支持**：理解编译过程有助于更好地调试Groovy代码
4. **工具开发**：可以开发代码分析、优化和转换工具

通过深入理解Groovy编译器内部机制，开发者可以充分利用Groovy的动态特性，同时获得接近Java的性能。编译器的优化技术和AST转换机制为Groovy提供了强大的扩展能力，使其成为JVM上最灵活的编程语言之一。