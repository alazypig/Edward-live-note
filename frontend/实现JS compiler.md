# **实现一个最小的JS编译器**

本文实现一个List-Style function call转化为C-Style的compiler。

|**natual language**|**Lisp(prefix expression)**|**C(function call)**|
|---|---|---|
|2 + 2|(add 2 2)|add(2, 2)|
|4 - 2|(substract 4 2)|substract(4, 2)|
|2 + (4 - 2)|(add 2 (substract(4 2))|add(2, substract(4, 2))|

### **Compiler需要做什么工作**

- Parsing：将source code转化为更抽象的代码表示（AST）。
- Transformation：对parsing后的代码，做一些转换操作（静态分析，代码优化）。
- Code Generation：将转换后的代码生成新的所需代码。

### **Parsing 阶段**

Parsing通常分为两个阶段：

- 词法分析（lexical analysis）：获取原始代码并通过称为标记器（tokenizer）（或词法分析器（lexer））将其拆分为标记（Token）。 标记是一组微小的小对象，它们描述了语法的一个孤立部分。 它们可以是数字、标签、标点符号、运算符等等。
- 语法分析（syntax analysis）将Token重新格式化为描述语法的每个部分及其相互关系的表示，称为中间表示（representation）或抽象语法树（Abstract Syntax Tree）。

[JS的在线parse网站](https://esprima.org/demo/parse.html)

现在来看下面的代码：

`(add 2 (subtract 4 2))`

经过词法分析后，得到它的token：

```javascript
[
  { type: 'paren',  value: '('        },
  { type: 'name',   value: 'add'      },
  { type: 'number', value: '2'        },
  { type: 'paren',  value: '('        },
  { type: 'name',   value: 'subtract' },
  { type: 'number', value: '4'        },
  { type: 'number', value: '2'        },
  { type: 'paren',  value: ')'        },
  { type: 'paren',  value: ')'        },
]
```

它的AST长这样：

```javascript
{
  type: 'Program',
  body: [{
    type: 'CallExpression',
    name: 'add',
    params: [{
      type: 'NumberLiteral',
      value: '2',
    }, {
      type: 'CallExpression',
      name: 'subtract',
      params: [{
        type: 'NumberLiteral',
        value: '4',
      }, {
        type: 'NumberLiteral',
        value: '2',
      }]
    }]
  }]
}
```

我们通过AST就可以发现代码执行顺序：

1. 进入程序
2. 函数调用（add）- 进入 Program 的 body 的第一个对象
3. 数字 - 来到 add 函数的 params 的第一个对象
4. 函数调用（substract） - 来到 add 函数的 params 的第二个对象
5. 数字 - 来到 substract 函数的 params 的第一个对象
6. 数字 - 来到 substract 函数的 params 的第二个对象

通过遍历这个AST对象，我们就可以完成我们想要完成的操作。

### **Visitor**

为了遍历AST，我们可以创建一个 visitor 。我们需要针对不同的Token去调用不同的方法，同时为了方便，我们把 node 和他的 parent 也一起交给访问方法。

```javascript
var visitor = {
  NumberLiteral(node, parent) { },
  CallExpression(node, parent) { }
}
```

现在，我们可以来描述AST的基本流程：

1. → Program (enter)
2.       → CallExpression (enter)
3.             → NumberLiteral (enter)
4.             ← NumberLiteral (exit)
5.             → CallExpression (enter)
6.                   → NumberLiteral (enter)
7.                   ← NumberLiteral (exit)
8.                   → NumberLiteral (enter)
9.                   ← NumberLiteral (exit)
10.             ← CallExpression (exit)
11.       ← CallExpression (exit)
12. ← Program (exit)

所以我们需要去改造我们的 visitor：

```javascript
var visitor = {
  NumberLiteral: {
    enter(node, parent) { },
    exit(node, parent) { }
  },
  CallExpression: {
    enter(node, parent) { },
    exit(node, parent) { }
  }
}
```

### **代码生成阶段**

编译器的最后阶段是代码生成。 有时编译器会做一些与转换之类的事情，但在大多数情况下，代码生成只是意味着将我们的 AST 和 String-ify code 取出来。 代码生成器有几种不同的工作方式，一些编译器会重用之前的Token，另一些编译器会创建代码的单独表示，以便它们可以线性地打印节点，但大多数将使用我们刚刚创建的相同 AST。 实际上，我们的代码生成器将知道如何“打印”AST 的所有不同节点类型，并且它将递归调用自身以打印嵌套节点，直到将所有内容输出成一长串目标代码。 现在我们就可以着手写自己的 compiler。

### **Source Code**

**compiler.js** 

```javascript
/**
 * @author Edward <wang.huiyang@outlook.com>
 */
'use strict'

/**
 * 返回所有的 Token
 * @param {string} input - the code
 * @returns {Token[]} tokens
 */
function tokenizer(input) {
  let current = 0 // 目前所在位置
  let tokens = [] // 存放所有 Token 的数组
  const LEN = input.length
  while (current < LEN) {
    let char = input[current] // 拿到当前的字符
    if (char === '(') {
      tokens.push({
        type: 'paren',
        value: '('
      })
      current++
      
      continue
    }
    
    if (char === ')') {
      tokens.push({
        type: 'paren',
        value: ')'
      })
      current++
      
      continue
    }
    const WHITE_SPACE = /\s/
    if (WHITE_SPACE.test(char)) {
      // 不处理空白字符
      current++

      continue

    }
    // (add 1321 12341)
    let NUMBERS = /[0-9]/
    if (NUMBERS.test(char)) {
      let value = ''
      // 需要集中处理
      // eg: 12341，得到1之后，2341需要一次读入
      while (NUMBERS.test(char)) {
        value += char
        char = input[++current]
      }
      tokens.push({
        type: 'number',
        value
      })

      continue
    }
    // (concat "test" "foo")
    if (char === '"') {
      let value = ''
      // 跳过 "
      char = input[++current]
      while (char !== '"') {
        value += char
        char = input[++current]
      }
      // 跳过 "
      char = input[++current]
      tokens.push({
        type: 'string',
        value
      })

      continue
    }
    let LETTERS = /[a-z]/i
    if (LETTERS.test(char)) {
      let value = ''
      while (LETTERS.test(char)) {
        value += char
        char = input[++current]
      }
      tokens.push({
        type: 'name',
        value
      })

      continue
    }
    throw new TypeError(`Can't parse this character: ${char}`)
  }
  return tokens
}

/**
 * parse tokens to AST.
 * @param {Tokens} tokens - the tokens.
 */
function parser(tokens) {
  let current = 0
  function run() {
    let token = tokens[current]
    // Number node
    if (token.type === 'number') {
      current++
      return {
        type: 'NumberLiteral',
        value: token.value
      }
    }
    
    // String node
    if (token.type === 'string') {
      current++
      return {
        type: 'StringLiteral',
        value: token.value
      }
    }

    // CallExpression
    if (token.type === 'paren' && token.value === '(') {
      token = tokens[++current]
      let node = {
        type: 'CallExpression',
        name: token.value,
        params: []
      }
      token = tokens[++current]
      
      // focus here
      while ((token.type !== 'paren') || (token.type === 'paren' && token.value !== ')')) {

        node.params.push(run()) // 在这一步的结尾，current指向的是未遍历的token
        token = tokens[current] // 所以需要在这里给token归位
      }
      current++

      return node
    }
    throw new TypeError(token.type)
  }

  const ast = {
    type: 'Program',
    body: []
  }

  while (current < tokens.length) {
    ast.body.push(run())
  }

  return ast
}

/**
 * 使用visitor访问AST的节点。
 * 根据文章中给出的定义，调用方式为：
 * traverse(ast, {
 *   Program: {
 *     enter(node, parent) {},
 *     exit(node, parent) {}
 *   }
 *   // etc.
 * })
 * @param {ASTNode} ast - the ast node.
 * @param {*} visitor - the visitor
 */
function traverse(ast, visitor) {
  function traverseArray(array, parent) {
    array.forEach(child => traverseNode(child, parent))
  }

  function traverseNode(node, parent) {
    let methods = visitor[node.type] // 根据type去拿不同的访问方法
    if (methods && methods.enter) {
      methods.enter(node, parent)
    }

    switch (node.type) {
      case 'Program':
        traverseArray(node.body, node)
        break
      case 'CallExpression':
        traverseArray(node.params, node)
        break
      case 'NumberLiteral':
      case 'StringLiteral':
        break
      default:
        throw new TypeError(node.type)
    }

    if (methods && methods.exit) {
      methods.exit(node, parent)
    }
  }

  traverseNode(ast, null)
}

/**
 * 将ast转换为所需要的ast节点。
 * @param {*} ast - the ast node.
 */
function transformer(ast) {
  const newAst = {
    type: 'Program',
    body: []
  }
  
  ast.__context = newAst.body
  traverse(ast, {
    NumberLiteral: {
      enter(node, parent) {
        parent.__context.push({
          type: 'NumberLiteral',
          value: node.value
        })
      }
    },
    StringLiteral: {
      enter(node, parent) {
        parent.__context.push({
          type: 'StringLiteral',
          value: node.value
        })
      }
    },
    CallExpression: {
      enter(node, parent) {
        let expression = {
          type: 'CallExpression',
          callee: {
            type: 'Identifier',
            name: node.name
          },
          arguments: []
        }
        node.__context = expression.arguments
        // 如果parent不是函数调用，代表是一个表达式块
        if (parent.type !== 'CallExpression') {
          expression = {
            type: 'ExpressionStatement',
            expression: expression
          }
        }
        parent.__context.push(expression)
      }
    }
  })
  return newAst
}

/**
 * 代码生成器
 * @param {*} node
 */
function codeGenerator(node) {
  switch (node.type) {
    case 'Program':
      return node.body.map(codeGenerator).join('\n')
    case 'ExpressionStatement':
      return `${codeGenerator(node.expression)};`
    case 'CallExpression':
      return `${codeGenerator(node.callee)}(${node.arguments.map(codeGenerator).join(',')})`
    case 'Identifier':
      return node.name
    case 'NumberLiteral':
      return node.value
    case 'StringLiteral':
      return `"${node.value}"`
    default:
      throw new TypeError(node.type)
  }
}

function compiler(input) {
  const tokens = tokenizer(input)
  const ast = parser(tokens)
  const newAst = transformer(ast)
  const output = codeGenerator(newAst)
  return output
}

module.exports = {
  tokenizer,
  parser,
  traverse,
  transformer,
  codeGenerator,
  compiler
}
```

**test.js** 

```javascript
const {
  tokenizer,
  parser,
  transformer,
  codeGenerator,
  compiler
} = require('./compiler.js')

const assert = require('assert')
const input = '(add 2 (subtract 4 2))'
const output = 'add(2,subtract(4,2));'
const tokens = [
  { type: 'paren', value: '(' },
  { type: 'name', value: 'add' },
  { type: 'number', value: '2' },
  { type: 'paren', value: '(' },
  { type: 'name', value: 'subtract' },
  { type: 'number', value: '4' },
  { type: 'number', value: '2' },
  { type: 'paren', value: ')' },
  { type: 'paren', value: ')' }
];

const ast = {
  type: 'Program',
  body: [{
    type: 'CallExpression',
    name: 'add',
    params: [{
      type: 'NumberLiteral',
      value: '2'
    }, {
      type: 'CallExpression',
      name: 'subtract',
      params: [{
        type: 'NumberLiteral',
        value: '4'
      }, {
        type: 'NumberLiteral',
        value: '2'
      }]
    }]
  }]
};

const newAst = {
  type: 'Program',
  body: [{
    type: 'ExpressionStatement',
    expression: {
      type: 'CallExpression',
      callee: {
        type: 'Identifier',
        name: 'add'
      },
      arguments: [{
        type: 'NumberLiteral',
        value: '2'
      }, {
        type: 'CallExpression',
        callee: {
          type: 'Identifier',
          name: 'subtract'
        },
        arguments: [{
          type: 'NumberLiteral',
          value: '4'
        }, {
          type: 'NumberLiteral',
          value: '2'
        }]
      }]
    }
  }]
};

assert.deepStrictEqual(tokenizer(input), tokens, 'Parse token incorrect!')
assert.deepStrictEqual(parser(tokens), ast, 'Parser token to ast incorrect!')
assert.deepStrictEqual(transformer(ast), newAst, 'Transform ast incorrect!')
assert.deepStrictEqual(codeGenerator(newAst), output, 'CodeGenerator incorrect!')
assert.deepStrictEqual(compiler(input), output, 'Compiler incorrect!')

console.log('Pass!');
```

### **结语**

JS中许多的中间件、代码转化、优化、埋点方案都是通过转化AST实现的。不同的是不需要自己去实现compiler，业界有一套规范的compiler API。你只需要使用他们的api，去转化成自己所需的AST，就可以得到自己想要的代码。  
例如：自己动手实现一个babel插件？