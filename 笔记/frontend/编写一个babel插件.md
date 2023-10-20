前置知识：[实现一个最小的JS编译器](http://jira:8100/pages/viewpage.action?pageId=29624700)

因为从0开始实现过于复杂，所以本文使用bebel7内置方法对输入代码进行修改。

babel7内置包有：

- `@babel/parser`：将源码解析成AST。对应parse阶段。
- `@babel/traverse`：遍历AST节点，并调用visitor。对应transform阶段。
- `@babel/generate`：打印AST，生成目标代码和sourcemap。对应generate阶段。
- `@babel/types`：创建、判断AST。
- `@babel/template`：根据模块批量创建AST。
- `@babel/core`：核心引擎，解析配置。
- `@babel/helpers`：转换esNext所需的AST。
- `@babel/helper-*`：操作AST的公共函数。
- `@babel/runtime`：
    - helper：`@bebel/helper-*`的runtime版本。
    - corejs：esNext API的实现。
    - regenerator：async、await实现。

## 如何编写babel插件

1. 去查找我们想修改的Token是属于什么类型的。可以在[其他网站](https://astexplorer.net/)进行查看，或者使用babel/parse。
2. 编写我们的plugin文件。在visitor里面写对应的Token的visite方法。
3. 在`bebel.config.json`中使用我们编写的插件，babel/core会自动去导入执行。
4. 查看输出的代码。

## 代码实现：

```
# 项目结构

babel-demo
│
├─babel.config.json
├─package.json
├─pnpm-lock.yaml
├─dist
├─plugins
│  └babel-plugin-test-1.js
└─src
    └index.js
```

依赖：
```json
{  
  "name": "babel-demo",  
  "version": "1.0.0",  
  "scripts": {  
    "build": "babel src -d dist"  
  },  
  "keywords": [],  
  "author": "Edward Wang<wang.huiyang@outlook.com>",  
  "license": "ISC",  
  "devDependencies": {  
    "@babel/cli": "^7.14.8",  
    "@babel/core": "^7.15.0",  
    "prettier": "^2.3.2"  
  }  
}
```


babel.config.json：
```json
{  
  "plugins": [  
    [  
      "./plugins/babel-plugin-test-1",  
      {  
        "ignore": ["info"]  
      }  
    ]  
  ]  
}
```
plugins/babel-plugin-test-1.js：
```javascript
module.exports = ({ types }) => {  
  return {  
    visitor: {  
      /**  
       * 内置变量  
       */  
      VariableDeclaration(path) {  
        const node = path.node;  
  
        /**  
         * deal with `const`         */        if (node.kind === "const") node.kind = "var";  
  
        // delete comments.  
        delete node.leadingComments;  
        delete node.trailingComments;  
      },  
  
      /**  
       * 二元运算符  
       */  
      BinaryExpression(path) {  
        const node = path.node;  
  
        const equalMap = {  
          "==": "===",  
          "!=": "!==",  
        };  
        node.operator = equalMap[node.operator] || node.operator;  
      },  
  
      /**  
       * 标识符  
       */  
      Identifier(path) {  
        const node = path.node;  
  
        if (node.name === "foo") node.name = "bar";  
      },  
  
      /**  
       * arrow function       */      ArrowFunctionExpression(path) {  
        const node = path.node;  
  
        if (node.type === "ArrowFunctionExpression") {  
          const body = path.get("body").node; // get function body  
  
          if (body.type !== "BlockStatement") {  
            // 如果 arrow function 的 body 不是 block statement，需要特殊处理。  
            // 否则直接将 type 修改为 FunctionExpression的话  
            // const TrueID = () => true;  
            // 编译后的结果为  
            // const TrueID = function () true;  
            const statement = [];  
  
            /**  
             * types: 构造 Token  
             */            statement.push(types.returnStatement(body));  
            node.body = types.blockStatement(statement);  
          }  
  
          node.type = "FunctionExpression";  
        }  
      },  
  
      /**  
       * 表达式  
       */  
      ExpressionStatement(path, { opts: options }) {  
        const node = path.node;  
        const { object, property } = node.expression.callee;  
  
        console.log(options);  
  
        if (object.name === "console") {  
          // deal with console  
  
          // options.ignore 来自 babel.config.json 中的 ignore          const ignore = (options.ignore || []).find(  
            (i) => i === property.name  
          );  
  
          if (!ignore) {  
            path.remove();  
          }  
        }  
  
        // delete comments.  
        delete node.leadingComments;  
        delete node.trailingComments;  
      },  
  
      // .etc ...  
    },  
  };  
};
```
src/index.js：
```javascript
const foo = () => 1;  
  
const b = () => {  
  console.log("a"); // this will be remove after run.  
  const c = foo();  
  console.info("test foo console"); // this will not be remove because it is in the ignore list.  
};  
  
const obj = {};  
  
if (obj.a == obj.c) {  
  console.info("equal");  
}  
  
if (obj.a != obj.c) {  
  console.err("not equal");  
}
```
dist/index.js：
```javascript
var bar = function () {  
  return 1;  
};  
  
var b = function () {  
  var c = bar();  
  console.info("test foo console");  
};  
  
var obj = {};  
  
if (obj.a === obj.c) {  
  console.info("equal");  
}  
  
if (obj.a !== obj.c) {}
```
