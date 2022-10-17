> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.52pojie.cn](https://www.52pojie.cn/thread-1700036-1-1.html)

> [md]## 用 Babel 解析 AST 处理 OB 混淆 JS 代码（一）：搭环境（含 IDEA 配置 eslint 踩坑记录）### 依赖 - Windows10、IDEA、yarn```bashyarn add shel......

![](https://avatar.52pojie.cn/data/avatar/001/90/61/77_avatar_middle.jpg)hans7 _ 本帖最后由 hans7 于 2022-10-17 01:09 编辑_  

用 Babel 解析 AST 处理 OB 混淆 JS 代码（一）：搭环境（含 IDEA 配置 eslint 踩坑记录）
-----------------------------------------------------------

### 依赖

*   Windows10、IDEA、yarn

```
yarn add shelljs -D
# 如果有现成的package.json，只运行这个就行
yarn
```

`package.json`指定的依赖如下：

```
"devDependencies": {
    "@babel/preset-typescript": "^7.18.6",
    "@types/babel__core": "^7.1.19",
    "@types/babel__traverse": "^7.18.0",
    "@types/node": "^18.7.13",
    "@typescript-eslint/eslint-plugin": "^5.35.1",
    "@typescript-eslint/parser": "^5.35.1",
    "eslint": "8.22.0",
    "shelljs": "^0.8.5"
  },
  "dependencies": {
    "@babel/core": "^7.18.13",
    "@babel/parser": "^7.18.13",
    "@babel/preset-env": "^7.18.10",
    "@babel/traverse": "^7.18.13",
    "typescript": "^4.7.4"
  }
```

【52pojie】用 Babel 解析 AST 处理 OB 混淆 JS 代码（一）：[https://www.52pojie.cn/thread-1700036-1-1.html](https://www.52pojie.cn/thread-1700036-1-1.html)

【52pojie】用 Babel 解析 AST 处理 OB 混淆 JS 代码（二）：[https://www.52pojie.cn/thread-1700038-1-1.html](https://www.52pojie.cn/thread-1700038-1-1.html)

【52pojie】用 Babel 解析 AST 处理 OB 混淆 JS 代码（三）：[https://www.52pojie.cn/thread-1700050-1-1.html](https://www.52pojie.cn/thread-1700050-1-1.html)

【52pojie】用 Babel 解析 AST 处理 OB 混淆 JS 代码（四）：[https://www.52pojie.cn/thread-1700068-1-1.html](https://www.52pojie.cn/thread-1700068-1-1.html)

本系列所有代码都基于 GitHub 仓库：[https://github.com/Hans774882968/control-flow-flattening-remove-public](https://github.com/Hans774882968/control-flow-flattening-remove-public)

**作者：[hans774882968](https://blog.csdn.net/hans774882968) 以及 [hans774882968](https://juejin.cn/user/1464964842528888) 以及 [hans774882968](https://www.52pojie.cn/home.php?mod=space&uid=1906177)**

### 引言

> AST（Abstract Syntax Tree，抽象语法树），简称语法树（Syntax Tree），是源代码的抽象语法结构的树状表现形式，树上的每个节点都表示源代码中的一种结构。语法树不是某一种编程语言独有的，JavaScript、Python、Java、Golang 等几乎所有编程语言都有语法树。

AST 的用途很广：IDE 的语法高亮、代码检查、格式化、压缩、转译等，使用 AST 来处理源代码都是最方便的。又比如`ES5`和`ES6`语法有不少差异，为了向后兼容，在实际应用中需要进行转换，这个场景用 AST 也是最方便的。AST 并不是为了逆向而生，但做逆向学会了 AST，在解混淆时会更方便。

原本只打算用 AST 来去除 JS 代码的控制流平坦化，但发现只有先熟悉 AST 的相关操作，才能更好地完成这个目标。索性我把一篇 blog 拆成一个系列，来讲清楚所有相关知识。相信在看到这个系列以后，大家都会感慨 AST 真简单！如果和我一样鶸，就把这些代码跑一遍，很快就能学会！

### 技术选型

1.  shelljs：在 nodejs 中执行 cmd 命令。
2.  Babel：解析 AST，修改 AST 并重新生成代码。在 [astexplorer](https://astexplorer.net/) 中还展示了`recast`等包，它们都能做这件事。
3.  eslint：检查代码格式是否符合规范，并尝试自动 format 代码。
4.  TypeScript

这里只有`Babel`是必须的，但为了在提升开发效率的同时保证代码的健壮，还是建议把 TS、eslint 配好。

#### 为什么要用 TypeScript

1.  Babel 的官方文档语焉不详，TypeScript 的类型提示结合 IDE 是更好的文档。
2.  写类型守卫的过程是在倒逼自己去思考各种边界情况，写出更健壮的代码。

### IDEA 配置 eslint 踩坑记录

每次配置 eslint，eslint 都有新的方式来折磨我，我愿称之为 yyds。

#### eslint 报错：TypeError: this.options.parse is not a function

IDEA 配置的 eslint 不要大于等于**`8.23.0`**，否则你会遇到这个错误：

```
TypeError: this.options.parse is not a function
```

那么我们的`package.json`得这么写：`"eslint": "8.22.0"`。

#### eslint 报 cliEngine 的错

打开`<IDEA安装目录>\plugins\JavaScriptLanguage\languageService\eslint\bin\eslint-plugin.js`。

```
// 旧版用
this.cliEngine = require(this.basicPath + "lib/cli-engine");
// 新版用
this.cliEngine = require(this.basicPath + "lib/cli-engine").CLIEngine;
```

当然有兼容的写法：`?.`即可，但 es2020 很可能不支持，自己 polyfill 一下就行。

#### IDEA 配置自动 format

`yarn add`之后，要根据参考链接 3 来配置：

1.  `Languages & Frameworks -> JavaScript -> Code Quality Tools -> ESLint`，勾选 Enable，然后填相关的字段。
2.  `设置 -> ESLint Settings`，勾选 Enable，然后填`Path to eslint bin`，勾选`Auto fix errors`等字段。
3.  打开`设置 -> Keymap`搜索`Fix ESLint Problems`，配置快捷键。

看到`JS / TS`代码标红，并且能按快捷键 format 代码就成功了。总之能配置的都配置一下，免得它老不生效……

> 后续：呵呵呵这次 IDEA 叕不能显示 TypeScript 的 eslint 错误了，明明啥都装了…… 幸好还能通过`npm run lint`来 format。不得不说 **eslint 永远得神**……

### 动态指定执行命令：用 npm scripts+nodejs 脚本解决

希望实现：在项目根目录输入命令`npm run cff <fname>`，自动执行`tsc && node src/<fname>.js`。

这方面资料少得可怜，参考链接 1 已经是能找到的里面最好的了。

根据参考链接 1，尝试过在`package.json`里加`fname`属性，然后读取`%npm_package_fname%`，但发现读不到值，因为必须放到`package.json`的`config`属性里；也尝试过在`package.json`的`config`对象里加自定义属性`fname`，这次`%npm_package_fname%`能读到值但无法修改。于是我们只能用最麻烦但最灵活的方案了：

用 nodejs 写个脚本，然后用 npm scripts 包装一下。

放在项目根目录下的`cff.js`：

```
const process = require('process');
const shell = require('shelljs');

const args = process.argv.slice(2);
if (!args.length) {
  console.log('Usage: npm run cff <file_name>');
  process.exit(0);
}
const fname = args[0];
shell.exec(`tsc && node src/${fname}.js`);
```

依赖：

```
yarn add shelljs -D
```

给一个 demo（`src/check_pass_demo.ts`）最简单的代码：

```
import fs from 'fs';

function getFile (path: string) {
  return fs.readFileSync(path, 'utf-8');
}

const jsCode = getFile('src/inputs/check_pass_demo.js'); // 运行者不是自己，所以要相对于项目根目录
console.log(jsCode.substring(0, 60));
```

运行命令：

```
npm run cff check_pass_demo
```

用 Babel 解析 AST 处理 OB 混淆 JS 代码（二）：一些通用的基本操作
------------------------------------------

### 引言

在开始用 AST 来进行 JS 代码的修改之前，我们先通过一些例子看 AST 的形态。在 [astexplorer](https://astexplorer.net/) 中选择编译器`@babel/parser`，输入一行代码：`obj['x'](1)`。

```
ExpressionStatement  {
  type: "ExpressionStatement"
  start: 0
  end: 11
  loc: {start, end, filename, identifierName}
  expression: CallExpression  {
    type: "CallExpression"
    start: 0
    end: 11
    loc: {start, end, filename, identifierName}
    callee: MemberExpression  {
      type: "MemberExpression"
      start: 0
      end: 8
      loc: {start, end, filename, identifierName}
      object: Identifier  = $node {
        type: "Identifier"
        start: 0
        end: 3
        loc: {start, end, filename, identifierName}
        name: "obj"
      }
      computed: true
      property: StringLiteral  {
        type: "StringLiteral"
        start: 4
        end: 7
        loc: {start, end, filename, identifierName}
        extra: {rawValue, raw}
        value: "x"
      }
    }
    arguments:  [
      NumericLiteral  {
        type: "NumericLiteral"
        start: 9
        end: 10
        loc: {start, end, filename, identifierName}
        extra: {rawValue, raw}
        value: 1
      }
    ]
  }
}
```

归纳出一些特征：

1.  因为对象是先取属性，再进行调用，所以 AST 是`CallExpression`在`MemberExpression`的上面。
2.  `obj`变量名对应`Identifier`，常量串对应`StringLiteral`，数字对应`NumericLiteral`。
3.  `CallExpression`主要关注`callee`和`arguments`属性，分别表示被调用的函数和参数列表。
4.  `MemberExpression`主要关注`object`、`property`和`computed`属性，分别表示对象，属性和是否是计算属性。`Dot Notation`和`Array Notation`的`computed`分别为`false`和`true`。

我们需要不断地看 AST，归纳出特征，才能写出正确的代码。

### 写 AST 处理代码的套路

接下来看看 AST 处理代码的骨架：

```
import generator from '@babel/generator';
import traverse from '@babel/traverse';
import {
  Node,
  isIdentifier,
  isMemberExpression,
  // ...
} from '@babel/types';

const ast = parser.parse(jsCode);
traverse(ast, {
  // 在递归遍历子树之前，对是Identifier的节点进行修改
  Identifier (path: NodePath<Identifier>) {...}
});
// 省略多个traverse
traverse(ast, {
  // 是某种类型的节点，则调用对应的函数进行修改
  NumericLiteral (path) {...},
  StringLiteral (path) {...}
});
const { code } = generator(ast);
```

`traverse`用 dfs 遍历 AST，并在遍历前后提供钩子给我们，用于修改 AST 的节点。

我们需要知道关于节点的一些知识。所有的节点都是`Node`，用 IEDA 点击`Node`查看类型定义：

```
export type Node = Accessor | AnyTypeAnnotation | ArgumentPlaceholder | ArrayExpression | ArrayPattern | ArrayTypeAnnotation | ArrowFunctionExpression | AssignmentExpression | AssignmentPattern | AwaitExpression | BigIntLiteral | Binary | BinaryExpression | BindExpression | Block | BlockParent | BlockStatement | BooleanLiteral | BooleanLiteralTypeAnnotation | BooleanTypeAnnotation | BreakStatement | CallExpression | CatchClause | Class | ClassAccessorProperty | ClassBody | ClassDeclaration | ClassExpression | ClassImplements | ClassMethod | ClassPrivateMethod | ClassPrivateProperty | ClassProperty | CompletionStatement | Conditional | ConditionalExpression | ContinueStatement | DebuggerStatement | DecimalLiteral | Declaration | DeclareClass | DeclareExportAllDeclaration | DeclareExportDeclaration | DeclareFunction | DeclareInterface | DeclareModule | DeclareModuleExports | DeclareOpaqueType | DeclareTypeAlias | DeclareVariable | DeclaredPredicate | Decorator | Directive | DirectiveLiteral | DoExpression | DoWhileStatement | EmptyStatement | EmptyTypeAnnotation | EnumBody | EnumBooleanBody | EnumBooleanMember | EnumDeclaration | EnumDefaultedMember | EnumMember | EnumNumberBody | EnumNumberMember | EnumStringBody | EnumStringMember | EnumSymbolBody | ExistsTypeAnnotation | ExportAllDeclaration | ExportDeclaration | ExportDefaultDeclaration | ExportDefaultSpecifier | ExportNamedDeclaration | ExportNamespaceSpecifier | ExportSpecifier | Expression | ExpressionStatement | ExpressionWrapper | File | Flow | FlowBaseAnnotation | FlowDeclaration | FlowPredicate | FlowType | For | ForInStatement | ForOfStatement | ForStatement | ForXStatement | Function | FunctionDeclaration | FunctionExpression | FunctionParent | FunctionTypeAnnotation | FunctionTypeParam | GenericTypeAnnotation | Identifier | IfStatement | Immutable | Import | ImportAttribute | ImportDeclaration | ImportDefaultSpecifier | ImportNamespaceSpecifier | ImportSpecifier | IndexedAccessType | InferredPredicate | InterfaceDeclaration | InterfaceExtends | InterfaceTypeAnnotation | InterpreterDirective | IntersectionTypeAnnotation | JSX | JSXAttribute | JSXClosingElement | JSXClosingFragment | JSXElement | JSXEmptyExpression | JSXExpressionContainer | JSXFragment | JSXIdentifier | JSXMemberExpression | JSXNamespacedName | JSXOpeningElement | JSXOpeningFragment | JSXSpreadAttribute | JSXSpreadChild | JSXText | LVal | LabeledStatement | Literal | LogicalExpression | Loop | MemberExpression | MetaProperty | Method | Miscellaneous | MixedTypeAnnotation | ModuleDeclaration | ModuleExpression | ModuleSpecifier | NewExpression | Noop | NullLiteral | NullLiteralTypeAnnotation | NullableTypeAnnotation | NumberLiteral | NumberLiteralTypeAnnotation | NumberTypeAnnotation | NumericLiteral | ObjectExpression | ObjectMember | ObjectMethod | ObjectPattern | ObjectProperty | ObjectTypeAnnotation | ObjectTypeCallProperty | ObjectTypeIndexer | ObjectTypeInternalSlot | ObjectTypeProperty | ObjectTypeSpreadProperty | OpaqueType | OptionalCallExpression | OptionalIndexedAccessType | OptionalMemberExpression | ParenthesizedExpression | Pattern | PatternLike | PipelineBareFunction | PipelinePrimaryTopicReference | PipelineTopicExpression | Placeholder | Private | PrivateName | Program | Property | Pureish | QualifiedTypeIdentifier | RecordExpression | RegExpLiteral | RegexLiteral | RestElement | RestProperty | ReturnStatement | Scopable | SequenceExpression | SpreadElement | SpreadProperty | Standardized | Statement | StaticBlock | StringLiteral | StringLiteralTypeAnnotation | StringTypeAnnotation | Super | SwitchCase | SwitchStatement | SymbolTypeAnnotation | TSAnyKeyword | TSArrayType | TSAsExpression | TSBaseType | TSBigIntKeyword | TSBooleanKeyword | TSCallSignatureDeclaration | TSConditionalType | TSConstructSignatureDeclaration | TSConstructorType | TSDeclareFunction | TSDeclareMethod | TSEntityName | TSEnumDeclaration | TSEnumMember | TSExportAssignment | TSExpressionWithTypeArguments | TSExternalModuleReference | TSFunctionType | TSImportEqualsDeclaration | TSImportType | TSIndexSignature | TSIndexedAccessType | TSInferType | TSInstantiationExpression | TSInterfaceBody | TSInterfaceDeclaration | TSIntersectionType | TSIntrinsicKeyword | TSLiteralType | TSMappedType | TSMethodSignature | TSModuleBlock | TSModuleDeclaration | TSNamedTupleMember | TSNamespaceExportDeclaration | TSNeverKeyword | TSNonNullExpression | TSNullKeyword | TSNumberKeyword | TSObjectKeyword | TSOptionalType | TSParameterProperty | TSParenthesizedType | TSPropertySignature | TSQualifiedName | TSRestType | TSStringKeyword | TSSymbolKeyword | TSThisType | TSTupleType | TSType | TSTypeAliasDeclaration | TSTypeAnnotation | TSTypeAssertion | TSTypeElement | TSTypeLiteral | TSTypeOperator | TSTypeParameter | TSTypeParameterDeclaration | TSTypeParameterInstantiation | TSTypePredicate | TSTypeQuery | TSTypeReference | TSUndefinedKeyword | TSUnionType | TSUnknownKeyword | TSVoidKeyword | TaggedTemplateExpression | TemplateElement | TemplateLiteral | Terminatorless | ThisExpression | ThisTypeAnnotation | ThrowStatement | TopicReference | TryStatement | TupleExpression | TupleTypeAnnotation | TypeAlias | TypeAnnotation | TypeCastExpression | TypeParameter | TypeParameterDeclaration | TypeParameterInstantiation | TypeScript | TypeofTypeAnnotation | UnaryExpression | UnaryLike | UnionTypeAnnotation | UpdateExpression | UserWhitespacable | V8IntrinsicIdentifier | VariableDeclaration | VariableDeclarator | Variance | VoidTypeAnnotation | While | WhileStatement | WithStatement | YieldExpression;
```

`Node`包含了`MemberExpression`、`Identifier`和`StringLiteral`等。

对于`NodePath`我们暂时不需要知道太多，只需要知道：

1.  `path.replaceInline(Nodes extends Node | readonly Node[])`（可以传`Node[]`）、`path.replaceWith(Node)`等方法可以替换当前节点。
2.  `path.remove()`可以删除当前节点。
3.  `path.node`可以获取`NodePath`对应的节点。

我们写 AST 处理代码的流程一般是：

1.  通过 AST 的相关属性来匹配特征，找到要修改的节点所对应的`NodePath`。这部分代码占了绝大部分。这一步需要大量使用” 类型守卫 “的技巧来保证我们的代码考虑到了各种边界条件。
2.  调用`path.replaceInline`等方法来修改 AST。
3.  删除对修改后的 AST 无用的节点，如未用到的变量和函数声明。

这个工作最困难的地方在于，我们需要不停地观看 [astexplorer](https://astexplorer.net/) 给出的 AST，来调整代码。最后再强调一次为什么要用 TS：

1.  Babel 的官方文档语焉不详，TypeScript 的类型提示结合 IDE 是更好的文档。
2.  写类型守卫的过程是在倒逼自己去思考各种边界情况，写出更健壮的代码。

接下来给出几个有通用性的操作的 demo，来迅速入门。

### 还原不直观的编码字符串或数值

对于数字，希望把`0x14`等变成 10 进制；对于常量串，希望把`'\u', '\x'`恢复成可见字符。参考链接 6 的代码处理得很不错。`src/translate_literal.ts`：

```
import traverse from '@babel/traverse';
import { stringLiteral, Node } from '@babel/types';

export function translateLiteral (ast: Node) {
  traverse(ast, {
    NumericLiteral (path) {
      const node = path.node;
      // 直接去除node.extra即可
      if (node.extra && /^0[obx]/i.test(node.extra.raw as string)) {
        node.extra = undefined;
      }
    },
    StringLiteral (path) {
      const node = path.node;
      if (node.extra && /\\[ux]/gi.test(node.extra.raw as string)) {
        let nodeValue = '';
        try {
          nodeValue = decodeURIComponent(escape(node.value));
        } catch (error) {
          nodeValue = node.value;
        }
        path.replaceWith(stringLiteral(nodeValue));
        path.node.extra = {
          'raw': JSON.stringify(nodeValue),
          'rawValue': nodeValue
        };
      }
    }
  });
}

// 调用：translateLiteral(ast);
```

### Babel 实现变量重命名

我们设计了一个简单的变量重命名方案：先遍历一次 AST，收集所有” 可以重命名的 “变量，再给出新名字（形如`v1, v2, ...`），最后再遍历一次 AST 进行替换。

*   为了提供最大的灵活性，我们设计了一个`canReplace`函数，让调用者自己决定哪些变量是参与替换的。
*   我们设计了一个`renameMap`，允许调用者给出期望的变量重命名方案，提高可读性。

注意：对于全局变量与局部变量存在同名的情况，这段代码可能是有问题的。希望能基于作用域进行完善。

```
import traverse, { NodePath } from '@babel/traverse';
import { Identifier, Node } from '@babel/types';

// 对于全局变量与局部变量同名的情况，这段代码可能是有问题的
export function renameVars (
  ast: Node,
  canReplace: (name: string) => boolean = () => {return true;},
  renameMap: {[key: string]: string} = {}
) {
  const names = new Set<string>();
  traverse(ast, {
    Identifier (path: NodePath<Identifier>) {
      const oldName = path.node.name;
      if (!canReplace(oldName)) return;
      names.add(oldName);
    }
  });
  let i = 0;
  names.forEach((name) => {
    if (!Object.getOwnPropertyDescriptor(renameMap, name)) {
      renameMap[name] = `v${++i}`;
    }
  });
  traverse(ast, {
    Identifier (path: NodePath<Identifier>) {
      const oldName = path.node.name;
      if (!canReplace(oldName)) return;
      path.node.name = renameMap[oldName];
    }
  });
}

// 调用
renameVars(
  ast,
  (name: string) => name.substring(0, 3) === '_0x',
  {
    enc: 'enc', _0x263396: 'i', _0x13adf6: 'out'
  }
);
```

### Babel MemberExpression Array Notation 转 Dot Notation

前文提到，`Dot Notation`和`Array Notation`的`computed`分别为`false`和`true`。因此代码会很简单。

```
import traverse, { NodePath } from '@babel/traverse';
import { identifier, Node, MemberExpression } from '@babel/types';

// console['log']() 变 console.log()
// computed 属性如果为 false，是表示 . 来引用成员
// computed 属性为 true，则是 [] 来引用成员
export function memberExpComputedToFalse (ast: Node) {
  traverse(ast, {
    MemberExpression (path: NodePath<MemberExpression>) {
      // path.get('property')获取到的是一个NodePath类型
      const propertyPath = path.get('property');
      if (!propertyPath.isStringLiteral()) return;
      const val = propertyPath.node.value;
      path.node.computed = false;
      propertyPath.replaceWith(identifier(val));
    }
  });
}
```

用 Babel 解析 AST 处理 OB 混淆 JS 代码（三）：处理 Strings Transformations（全网首创）
-----------------------------------------------------------------

### 引言

[这个网站](https://obfuscator.io/)就是[开源项目](https://github.com/javascript-obfuscator/javascript-obfuscator) `javascript-obfuscator`（简称 OB）的 Web UI。它提供了一个`Strings Transformations`选项用于隐藏常量串。据我所知，还没有给出`Strings Transformations`解决方案的 blog，因此可谓全网首创。我们勾选`String Array, String Array Rotate, String Array Shuffle`这 3 个选项，观察一下生成的代码的特征：

```
(function(_0x1f23fa, _0x502274) {
    var _0x1841e6 = _0x546b,
        _0x54332a = _0x1f23fa();
    while ([]) {
        try {
            var _0x37b83c = -parseInt(_0x1841e6(0x72)) / 0x1 + parseInt(_0x1841e6(0x73)) / 0x2 * (-parseInt(_0x1841e6(0x7c)) / 0x3) + parseInt(_0x1841e6(0x88)) / 0x4 * (parseInt(_0x1841e6(0x89)) / 0x5) + -parseInt(_0x1841e6(0x71)) / 0x6 + parseInt(_0x1841e6(0x6c)) / 0x7 * (-parseInt(_0x1841e6(0x85)) / 0x8) + -parseInt(_0x1841e6(0x82)) / 0x9 + -parseInt(_0x1841e6(0x7e)) / 0xa * (-parseInt(_0x1841e6(0x78)) / 0xb);
            if (_0x37b83c === _0x502274) break;
            else _0x54332a['push'](_0x54332a['shift']());
        } catch (_0x258ebb) {
            _0x54332a['push'](_0x54332a['shift']());
        }
    }
}(_0x3ddf, 0x20d95));

function _0x546b(_0x280dd3, _0x383a2d) {
    var _0x3ddf54 = _0x3ddf();
    return _0x546b = function(_0x546b3f, _0x142ae2) {
        _0x546b3f = _0x546b3f - 0x6c;
        var _0x233a8a = _0x3ddf54[_0x546b3f];
        return _0x233a8a;
    }, _0x546b(_0x280dd3, _0x383a2d);
}

function _0x3ddf() {
    var _0x45c37a = ['30037Sxrenc', 'error!', 'len\x20error', 'XmvLm', 'Orz..', '1159374JpqDju', '267734qPEpMO', '364750QkecUn', 'shrai', 'length', 'KUTlo', 'Vwtjq', '99juDGtv', 'FhQZn', 'charCodeAt', 'FdUfK', '3tSVDal', 'Ajnur', '874980MJshmD', 'KclRu', 'Fhqhk', 'charAt', '187074oiwMPp', 'PjAeQ', 'ewhZd', '328PNtXbI', 'congratulation!', 'DpUmp', '57576xxZPaZ', '65fmhmYN', 'ualDk', 'RHSOY', 'log'];
    _0x3ddf = function() {
        return _0x45c37a;
    };
    return _0x3ddf();
}
```

可知：

*   有一个自执行函数和两个函数。这 3 个函数会随机换位置，干扰你的分析。
*   有常量串数组的函数`_0x3ddf`，利用**闭包**来给出常量串数组，记为`sl`。
*   `_0x546b`函数仅仅相当于`(idx) => sl[idx - 0x6c]`。
*   自执行函数可以进行常量串数组的`shuffle`和`rotate`。

再看常量串的获取方式：`_0x583af1(0x74)`。因此我们的目标就是把这种函数调用恢复为常量串。开工！

首先，每个函数开头都有`var _0x583af1 = _0x546b`这样的定义，因此我们需要识别实际上等于`_0x546b`的变量。相关代码：

```
// 如果常量表不止1处，则此代码不正确  
  const stringLiteralFuncs = ['_0x546b'];
  // 收集与常量串隐藏有关的变量
  traverse(ast, {
    VariableDeclarator (path) {
      const vaNode = path.node;
      if (!isIdentifier(vaNode.init) || !isIdentifier(vaNode.id)) return;
      if (stringLiteralFuncs.includes(vaNode.init.name)) {
        stringLiteralFuncs.push(vaNode.id.name);
      }
    }
  });
```

接下来需要拿到最终的常量串数组。暂时没找到优雅的方式，只能先用一个妥协方案：

1.  因为常量串数组的最终形态是固定的，所以我们首先直接在浏览器控制台运行一下上面那段代码，再输入`_0x3ddf()`拿到常量串数组的最终形态，然后把它硬编码进代码里。
2.  获取常量串的函数，我们设计为自行实现，即**硬编码**进代码里。

获取常量串的相关代码（直接展示了函数`restoreStringLiteral`如何调用）：

```
restoreStringLiteral(ast, (idx: number) => {
  return ['30037Sxrenc', 'error!', 'len\x20error', 'XmvLm', 'Orz..', '1159374JpqDju', '267734qPEpMO', '364750QkecUn', 'shrai', 'length', 'KUTlo', 'Vwtjq', '99juDGtv', 'FhQZn', 'charCodeAt', 'FdUfK', '3tSVDal', 'Ajnur', '874980MJshmD', 'KclRu', 'Fhqhk', 'charAt', '187074oiwMPp', 'PjAeQ', 'ewhZd', '328PNtXbI', 'congratulation!', 'DpUmp', '57576xxZPaZ', '65fmhmYN', 'ualDk', 'RHSOY', 'log'][idx - 108];
});
// 调用：getStringArr(idx)
```

最后，只需要`path.replaceWith(stringLiteral(getStringArr(idx)))`完成节点的替换。

完整的相关代码：

```
function restoreStringLiteral (ast: Node, getStringArr: (idx: number) => string) {
  // 如果常量表不止1处，则此代码不正确
  const stringLiteralFuncs = ['_0x546b'];
  // 收集与常量串隐藏有关的变量
  traverse(ast, {
    VariableDeclarator (path) {
      const vaNode = path.node;
      if (!isIdentifier(vaNode.init) || !isIdentifier(vaNode.id)) return;
      if (stringLiteralFuncs.includes(vaNode.init.name)) {
        stringLiteralFuncs.push(vaNode.id.name);
      }
    }
  });
  traverse(ast, {
    CallExpression (path) {
      const cNode = path.node;
      if (!isIdentifier(cNode.callee)) return;
      const varName = cNode.callee.name;
      if (!stringLiteralFuncs.includes(varName)) return;
      if (cNode.arguments.length !== 1 || !isNumericLiteral(cNode.arguments[0])) return;
      const idx = cNode.arguments[0].value;
      path.replaceWith(stringLiteral(getStringArr(idx)));
    }
  });
}
restoreStringLiteral(ast, (idx: number) => {
  return ['30037Sxrenc', 'error!', 'len\x20error', 'XmvLm', 'Orz..', '1159374JpqDju', '267734qPEpMO', '364750QkecUn', 'shrai', 'length', 'KUTlo', 'Vwtjq', '99juDGtv', 'FhQZn', 'charCodeAt', 'FdUfK', '3tSVDal', 'Ajnur', '874980MJshmD', 'KclRu', 'Fhqhk', 'charAt', '187074oiwMPp', 'PjAeQ', 'ewhZd', '328PNtXbI', 'congratulation!', 'DpUmp', '57576xxZPaZ', '65fmhmYN', 'ualDk', 'RHSOY', 'log'][idx - 108];
});
```

TODO：找到一种避免硬编码的方式！

用 Babel 解析 AST 处理 OB 混淆 JS 代码（四）：处理控制流平坦化
-----------------------------------------

### 引言

控制流平坦化通过引入状态机与循环，破坏代码上下文之间的阅读连续性和代码块之间的关联性，将若干个分散的小整体整合成一个巨大的循环体。实现方式是将代码块之间的原有顺序关系打断，改为由一个分发器来控制代码块的跳转。特点：

*   无法还原成原来具体的函数。
*   无法使用以函数为单位的调试方法，大幅度增加调试难度。
*   降低代码运行效率，提高爬虫运行时执行 JS 的资源成本。
*   可根据 JS 运行时检测到的某些因素自由跳转到蜜罐或跳出代码执行。

所有教程都没有提及的是：控制流平坦化实际上至少有两种。第一种是语句级别的，用于打乱语序。第二种是表达式级别的，用于替换双目运算符、逻辑运算符和常量等。我们将尽力为 [OB 网站](https://obfuscator.io/) 提供的两种控制流平坦化提供解决方案。

### 去除基于 switch 语句的控制流平坦化：先来解析一个简单的 demo

这个 demo 来自参考链接 4。待解析文件`src/inputs/hw.js`：

```
var arr = '3,0,1,2,4'.split(',');
var x = 0;
var cnt = 0;
while (true) {
  switch (arr[cnt++]) {
    case '0':
      console.log('case 0');
      x += 5;
      continue;
    case '1':
      console.log('case 1');
      x += 4;
      continue;
    case '2':
      console.log('case 2');
      x += 3;
      continue;
    case '3':
      console.log('case 3');
      x += 2;
      continue;
    case '4':
      console.log('case 4');
      x += 1;
      continue;

  }
  break;
}
```

#### 思路

1.  获取`arr`运行时的值（是个定值）。
2.  用 Babel 读取每一个`case`的 body，具体取哪个`case`用`arr`确定。这里的 body 是`Statement[]`。
3.  把上面的所有 body 拼接起来，得所求，类型仍为`Statement[]`。调用`path.replaceInline(Statement[])`来获取去除了控制流平坦化的代码。

`src/hw.ts`的大多数代码都只是做第一步，因为考虑到源代码可能会变，希望有一定通用性。为了方便，也可以选择直接硬编码第一步的结果。因此代码的骨架如下：

```
const jsCode = getFile('src/inputs/hw.js');
const ast = parser.parse(jsCode);
const decodeWhileOpts = {
  WhileStatement (path: NodePath<WhileStatement>) {
    const { body } = path.node;
    const switchNode = (body as BlockStatement).body[0];
    if (!isSwitchStatement(switchNode)) return;
    const { discriminant, cases } = switchNode;
    // 省略第一步的代码...
    const replaceBody = arrVal.reduce((replaceBody, index) => {
      const caseBody = cases[+index].consequent;
      if (isContinueStatement(caseBody[caseBody.length - 1])) {
        caseBody.pop();
      }
      return replaceBody.concat(caseBody);
    }, [] as Statement[]);
    path.replaceInline(replaceBody);
  }
};
traverse(ast, decodeWhileOpts);
const { code } = generator(ast);
writeOutputToFile('hw_out.js', code);
```

这里偷懒了一下，直接用`cases[+index]`来取具体的`case`了，实际情况很可能要写额外的代码获取`cases[index].test.value`。

完整代码看`src/hw.ts`。注意：

1.  我们在项目根目录用`npm run cff hw`来运行`src/hw.ts`，所以读写文件要相对于项目根目录。

### 去除基于 switch 语句的控制流平坦化：更综合的 demo

这个 demo 和上一个 demo 难度一样，但结合了常量串隐藏。准备以下程序：

```
function enc (inp) {
  var i = 0;
  i += -1;
  var out = '';
  i += 1;
  for (;i < inp.length;++i) {
    var v = 0;
    if (i & 1) v = 0x33;
    else v = 0x31;
    out += String.fromCharCode(inp[i].charCodeAt() ^ v);
  }
  return out;
}
if (enc('flag{hans}') === 'W_PTJ[P]BN') console.log('pass');
else console.log('try again');
```

在 [OB 网站](https://obfuscator.io/)勾选`Control Flow Flattening`，`Control Flow Flattening Threshold`选择 1，`String Transformations`勾选`String Array, String Array Rotate, String Array Shuffle`，`String Array Threshold`选择 1。得以下代码：

```
var _0x47f9f1 = _0x27c4;
(function (_0x47124a, _0x19f73e) {
  var _0x3b6574 = _0x27c4,
    _0x2c307d = _0x47124a();
  while ([]) {
    try {
      var _0x585cd6 = parseInt(_0x3b6574(0x95)) / 0x1 * (parseInt(_0x3b6574(0x8f)) / 0x2) + -parseInt(_0x3b6574(0x97)) / 0x3 * (parseInt(_0x3b6574(0x9d)) / 0x4) + -parseInt(_0x3b6574(0x89)) / 0x5 + -parseInt(_0x3b6574(0x98)) / 0x6 + -parseInt(_0x3b6574(0x8d)) / 0x7 * (-parseInt(_0x3b6574(0x94)) / 0x8) + parseInt(_0x3b6574(0x96)) / 0x9 * (parseInt(_0x3b6574(0xa1)) / 0xa) + parseInt(_0x3b6574(0x92)) / 0xb;
      if (_0x585cd6 === _0x19f73e) break;
      else _0x2c307d['push'](_0x2c307d['shift']());
    } catch (_0x28b17f) {
      _0x2c307d['push'](_0x2c307d['shift']());
    }
  }
}(_0x379e, 0xdbab3));

function _0x27c4 (_0x122105, _0x24f040) {
  var _0x379e52 = _0x379e();
  return _0x27c4 = function (_0x27c4d4, _0x569919) {
    _0x27c4d4 = _0x27c4d4 - 0x89;
    var _0x5dfb85 = _0x379e52[_0x27c4d4];
    return _0x5dfb85;
  }, _0x27c4(_0x122105, _0x24f040);
}

function _0x379e () {
  var _0x3ed6e2 = ['1914456NQDFwp', '1xRwaZJ', '36ZbcbZP', '3gJgrjU', '8162226GwaJpl', '3|4|2|0|5|1', 'split', 'charCodeAt', 'pass', '6278120IHpVNF', 'W_PTJ[P]BN', 'length', 'fromCharCode', '939280gOLaZV', '661835nuUXrL', 'dKifE', 'try\x20again', 'log', '7aEbwep', 'awvtQ', '2804302XtaWgC', 'rmnID', 'flag{hans}', '21393471OyFTzd', 'lXUhG'];
  _0x379e = function () {
    return _0x3ed6e2;
  };
  return _0x379e();
}

function enc (_0x3bf54e) {
  var _0x55bea2 = _0x27c4,
    _0x550d17 = {
      'dKifE': _0x55bea2(0x99),
      'lXUhG': function (_0x7a78d6, _0x13ee42) {
        return _0x7a78d6 < _0x13ee42;
      },
      'rmnID': function (_0x28f0fb, _0x77896d) {
        return _0x28f0fb & _0x77896d;
      },
      'awvtQ': function (_0x26b565, _0x3ffc0b) {
        return _0x26b565 ^ _0x3ffc0b;
      }
    },
    _0x31ce85 = _0x550d17[_0x55bea2(0x8a)][_0x55bea2(0x9a)]('|'),
    _0x1ffdde = 0x0;
  while ([]) {
    switch (_0x31ce85[_0x1ffdde++]) {
      case '0':
        _0x263396 += 0x1;
        continue;
      case '1':
        return _0x13adf6;
      case '2':
        var _0x13adf6 = '';
        continue;
      case '3':
        var _0x263396 = 0x0;
        continue;
      case '4':
        _0x263396 += -0x1;
        continue;
      case '5':
        for (; _0x550d17[_0x55bea2(0x93)](_0x263396, _0x3bf54e[_0x55bea2(0x9f)]); ++_0x263396) {
          var _0x494484 = 0x0;
          if (_0x550d17[_0x55bea2(0x90)](_0x263396, 0x1)) _0x494484 = 0x33;
          else _0x494484 = 0x31;
          _0x13adf6 += String[_0x55bea2(0xa0)](_0x550d17[_0x55bea2(0x8e)](_0x3bf54e[_0x263396][_0x55bea2(0x9b)](), _0x494484));
        }
        continue;
    }
    break;
  }
}
if (enc(_0x47f9f1(0x91)) === _0x47f9f1(0x9e)) console[_0x47f9f1(0x8c)](_0x47f9f1(0x9c));
else console[_0x47f9f1(0x8c)](_0x47f9f1(0x8b));
```

#### 产生基于 switch 语句的控制流平坦化的条件

网上众多 blog 都没提到的：基于 switch 语句的控制流平坦化不总是能产生，需要一定条件。

1.  所有相关变量必须是`var`声明，否则不能产生。
2.  语句要足够多。

#### 思路

我们可以看到这里产生了一个基于 switch 语句的控制流平坦化。`_0x31ce85`变量就是字符串`'3|4|2|0|5|1'`，`_0x1ffdde`是单纯的自增变量。为了方便地在代码中拿到`_0x31ce85`的值，我们需要**先去除`Strings Transformations`**（常量串隐藏，可参考本系列的上一篇《[用 Babel 解析 AST 处理 OB 混淆 JS 代码（三）》](https://www.52pojie.cn/thread-1700050-1-1.html)）。

虽然难度一样，但是这一节我们提供更加完善的代码（~其实是懒得整理了 qwq~）。我们上一节没有删除控制流平坦化的相关变量，因为比较麻烦。参考链接 7 提供了一种不错的写法，能够在不硬编码的前提下方便地删除控制流平坦化的相关变量。它先使用`path.scope.getBinding(varName: string)`来获取当前作用域的变量名的`Binding`，然后调用`Binding.path.remove()`删除变量声明。`Binding`更具体的用法可参考：[https://juejin.cn/post/7113800415057018894](https://juejin.cn/post/7113800415057018894)。

删除控制流平坦化相关变量绑定的节点的相关代码：

```
const arrayName = discriminant.object.name;
const bindingArray = path.scope.getBinding(arrayName);
if (!bindingArray) return;
const autoIncrementName = discriminant.property.argument.name;
const bindingAutoIncrement = path.scope.getBinding(autoIncrementName);
if (!bindingAutoIncrement) return;
bindingArray.path.remove();
bindingAutoIncrement.path.remove();
```

去除基于 switch 语句的控制流平坦化部分的代码如下，完整代码见`src/switch_cff_demo.ts`。相比于上一节的代码，换了一种方式获取控制流平坦化的数组的值：

```
function switchCFF (ast: Node) {
  traverse(ast, {
    WhileStatement (path) {
      const wNode = path.node;
      if (!isBlockStatement(wNode.body) || !wNode.body.body.length) return;
      const switchNode = wNode.body.body[0];
      if (!isSwitchStatement(switchNode)) return;
      const { discriminant, cases } = switchNode;
      if (!isMemberExpression(discriminant) ||
          !isIdentifier(discriminant.object)) return;
      // switch语句内的控制流平坦化数组名，本例中是 _0x31ce85
      const arrayName = discriminant.object.name;
      // 获取控制流数组绑定的节点
      const bindingArray = path.scope.getBinding(arrayName);
      if (!bindingArray) return;
      // 经过restoreStringLiteral，我们认为它已经恢复为'v1|v2...'['split']('|')
      if (!isVariableDeclarator(bindingArray.path.node) ||
          !isCallExpression(bindingArray.path.node.init)) return;
      const varInit = bindingArray.path.node.init;
      if (!isMemberExpression(varInit.callee) ||
          !isStringLiteral(varInit.callee.object) ||
          varInit.arguments.length !== 1 ||
          !isStringLiteral(varInit.arguments[0])) return;
      const object = varInit.callee.object.value;
      const propty = varInit.callee.property;
      if (!isStringLiteral(propty) && !isIdentifier(propty)) return;
      const propertyName = isStringLiteral(propty) ? propty.value : propty.name;
      const splitArg = varInit.arguments[0].value;
      // 目前只支持'v1|v2...'.split('|')的解析
      if (propertyName !== 'split') {
        console.warn('switchCFF(ast)：目前只支持\'v1|v2...\'.split(\'|\')的解析');
        return;
      }
      const indexArr = object[propertyName](splitArg);

      const replaceBody = indexArr.reduce((replaceBody, index) => {
        const caseBody = cases[+index].consequent;
        if (isContinueStatement(caseBody[caseBody.length - 1])) {
          caseBody.pop();
        }
        return replaceBody.concat(caseBody);
      }, [] as Statement[]);
      path.replaceInline(replaceBody);

      // 可选择的操作：删除控制流平坦化数组绑定的节点、自增变量名绑定的节点
      if (!isUpdateExpression(discriminant.property) ||
          !isIdentifier(discriminant.property.argument)) return;
      const autoIncrementName = discriminant.property.argument.name;
      const bindingAutoIncrement = path.scope.getBinding(autoIncrementName);
      if (!bindingAutoIncrement) return;
      bindingArray.path.remove();
      bindingAutoIncrement.path.remove();
    }
  });
}
switchCFF(ast);
```

### 表达式级别的控制流平坦化

OB 提供的控制流平坦化至少有两种。第一种是语句级别的，基于 switch 语句，用于打乱语序。第二种是表达式级别的，用于替换双目运算符、逻辑运算符和常量等。

准备一段代码（来自参考链接 4）：

```
function check_pass(passwd) {
    var i=0;
    var sum=0;
    for(i=0;;i++)
    {
        if(i==passwd.length)
        {
            break;
        }
        sum=sum+passwd.charCodeAt(i);
    }
    if(i==4)
    {
        if(sum==0x1a1 && passwd.charAt(3) > 'c' && passwd.charAt(3) < 'e' && passwd.charAt(0)=='b')
        {
            if((passwd.charCodeAt(3)^0xd)==passwd.charCodeAt(1))
            {
                return 1;
            }
            console.log("Orz..");
        }
    }
    else
    {
        console.log("len error")
    }

    return 0;
}

function test()
{
    if(check_pass("bird"))
    {
        alert( "congratulation!");
    }
    else
    {
        alert( "error!");
    }
}
test();
```

在 [OB 网站](https://obfuscator.io/) 中使用如下选项加密：`Control Flow Flattening`，`Control Flow Flattening Threshold`选择 1，注意不要让网站隐藏常量串，因为我们这个版本的脚本还不支持。得到的代码如`src/inputs/check_pass_demo_easy.js`所示：

```
function check_pass (_0x57a7be) {
  var _0x252e28 = {
    'tPlEX': function (_0x52a315, _0x59fdfd) {
      return _0x52a315 == _0x59fdfd;
    },
    'TcjYB': function (_0x300e56, _0x2fe857) {
      return _0x300e56 + _0x2fe857;
    },
    'ZtFYf': function (_0x53b823, _0x136f17) {
      return _0x53b823 == _0x136f17;
    },
    'tPstu': function (_0x1607f2, _0x4a18be) {
      return _0x1607f2 > _0x4a18be;
    },
    'Vhxzy': function (_0x248a47, _0x5a2ca2) {
      return _0x248a47 < _0x5a2ca2;
    },
    'uuFIS': function (_0x3718bc, _0x3081f9) {
      return _0x3718bc == _0x3081f9;
    },
    'cRvgS': function (_0x56fd75, _0x1d2164) {
      return _0x56fd75 ^ _0x1d2164;
    },
    'GsTse': 'Orz..',
    'ykyBq': 'len\x20error'
  };
  var _0x537fc8 = 0x0;
  var _0x3df4b0 = 0x0;
  for (_0x537fc8 = 0x0;; _0x537fc8++) {
    if (_0x252e28['tPlEX'](_0x537fc8, _0x57a7be['length'])) {
      break;
    }
    _0x3df4b0 = _0x252e28['TcjYB'](_0x3df4b0, _0x57a7be['charCodeAt'](_0x537fc8));
  }
  if (_0x252e28['ZtFYf'](_0x537fc8, 0x4)) {
    if (_0x252e28['ZtFYf'](_0x3df4b0, 0x1a1) && _0x252e28['tPstu'](_0x57a7be['charAt'](0x3), 'c') && _0x252e28['Vhxzy'](_0x57a7be['charAt'](0x3), 'e') && _0x252e28['uuFIS'](_0x57a7be['charAt'](0x0), 'b')) {
      if (_0x252e28['uuFIS'](_0x252e28['cRvgS'](_0x57a7be['charCodeAt'](0x3), 0xd), _0x57a7be['charCodeAt'](0x1))) {
        return 0x1;
      }
      console['log'](_0x252e28['GsTse']);
    }
  } else {
    console['log'](_0x252e28['ykyBq']);
  }
  return 0x0;
}
function test () {
  var _0x288152 = {
    'eOZRR': function (_0x3f5c8e, _0x24ced8) {
      return _0x3f5c8e(_0x24ced8);
    },
    'alzHn': 'bird',
    'GyIol': function (_0x5ddbd5, _0x5cc507) {
      return _0x5ddbd5(_0x5cc507);
    },
    'FWSbx': 'congratulation!',
    'tYizA': 'error!'
  };
  if (_0x288152['eOZRR'](check_pass, _0x288152['alzHn'])) {
    _0x288152['GyIol'](alert, _0x288152['FWSbx']);
  } else {
    _0x288152['GyIol'](alert, _0x288152['tYizA']);
  }
}
test();
```

`_0x288152`和`_0x252e28`就是控制流平坦化的哈希表，我们看哈希表的值的几种形式：

*   `function(x, y){return x + y}`，对应`BinaryExpression`
*   `function(x, y){return x > y}`，对应`LogicalExpression`
*   `function(f, ...args){return f(...args)}`
*   `function(x){return x}`（在此没出现）
*   非函数（这个例子中，只有`StringLiteral`）

对于函数的情况，调用必定形如`tbl['xxx'](...args)`。对于非函数的情况，调用则形如`tbl['xxx']`。

我们依旧需要不断地观看 [https://astexplorer.net/](https://astexplorer.net/) 给出的 AST，做到：

*   哈希表的值是函数的情况，把函数体的`ReturnStatement`抠出来，再拿到函数体的参数，最后才进行替换。
*   哈希表的值不是函数的情况，进行一般意义的替换（参考链接 4 是直接替换为`StringLiteral`了，我们用 TS 写，可以有更具一般性的写法：`path.replaceWith<Node>(cffTableValue)`）。

#### 算法时间复杂度优化

参考链接 4 先遍历了控制流平坦化的哈希表的每一个键值对，然后对每个键值对都完整遍历一遍树。这个时间复杂度不太好。我们可以进行预处理（相关的数据结构`cffTables`，类型为`{[key: string]: {[key: string]: Node}}`），然后通过`cffTables[tableName][keyName]`来访问所需的`Node`。具体见`src/check_pass_demo_easy.ts`。这样我们就只需要遍历树两次了。

#### 代码

由于水平有限（鶸），这段代码：

*   不能识别作用域。如果存在多个层的作用域的变量同名，则无法正确去掉控制流平坦化。
*   控制流平坦化的哈希表的方括号只能识别常量串。需要**先去除常量串隐藏**，再调用该函数。

完整代码见`src/check_pass_demo_easy.ts`：

```
function cff (ast: Node) {
  type ASTNodeMap = {[key: string]: Node}
  const cffTables: {[key: string]: ASTNodeMap} = {};
  traverse(ast, {
    VariableDeclarator (path) {
      const node = path.node;
      if (!node.id || !isIdentifier(node.id)) return;
      const tableName = node.id.name;
      if (!isObjectExpression(node.init)) return;
      const tableProperties = node.init.properties;
      cffTables[tableName] = tableProperties.reduce((cffTable, tableProperty) => {
        if (!isObjectProperty(tableProperty) ||
           !isStringLiteral(tableProperty.key)) return cffTable;
        cffTable[tableProperty.key.value] = tableProperty.value;
        return cffTable;
      }, {} as ASTNodeMap);
    }
  });

  traverse(ast, {
    CallExpression (path) {
      const cNode = path.node;
      if (isMemberExpression(cNode.callee)) {
        if (!isIdentifier(cNode.callee.object)) return;
        const callParams = cNode.arguments;
        const tableName = cNode.callee.object.name;
        if (!isStringLiteral(cNode.callee.property)) return;
        const keyName = cNode.callee.property.value;
        if (!(tableName in cffTables) ||
            !(keyName in cffTables[tableName])) return;
        const shouldBeFuncValue = cffTables[tableName][keyName];
        if (!isFunctionExpression(shouldBeFuncValue) ||
            !shouldBeFuncValue.body.body.length ||
            !isReturnStatement(shouldBeFuncValue.body.body[0])) return;
        // 拿到返回值
        const callArgument = shouldBeFuncValue.body.body[0].argument;
        if (isBinaryExpression(callArgument) && callParams.length === 2) {
          if (!isExpression(callParams[0]) || !isExpression(callParams[1])) {
            throw '二元运算符中，两个参数都应为表达式';
          }
          // 处理function(x, y){return x + y}这种形式
          path.replaceWith(binaryExpression(callArgument.operator, callParams[0], callParams[1]));
        } else if (isLogicalExpression(callArgument) && callParams.length === 2) {
          if (!isExpression(callParams[0]) || !isExpression(callParams[1])) {
            throw '逻辑运算符中，两个参数都应为表达式';
          }
          // 处理function(x, y){return x > y}这种形式
          path.replaceWith(logicalExpression(callArgument.operator, callParams[0], callParams[1]));
        } else if (isCallExpression(callArgument) && isIdentifier(callArgument.callee)) {
          // 处理function(f, ...args){return f(...args)}这种形式
          if (callParams.length == 1) {
            path.replaceWith(callParams[0]);
          } else {
            if (!isExpression(callParams[0])) {
              throw '仅支持第一个参数为函数的形式，如：function(f, ...args){return f(...args)}';
            }
            path.replaceWith(callExpression(callParams[0], callParams.slice(1)));
          }
        }
      }
    },
    MemberExpression (path) {
      const mNode = path.node;
      if (!isIdentifier(mNode.object)) return;
      const tableName = mNode.object.name;
      if (!isStringLiteral(mNode.property)) return;
      const keyName = mNode.property.value;
      if (!(tableName in cffTables) ||
          !(keyName in cffTables[tableName])) return;
      const cffTableValue = cffTables[tableName][keyName];
      path.replaceWith<Node>(cffTableValue);
    }
  });
}

cff(ast);
```

效果（`src/outputs/check_pass_demo_easy_out.js`，可直接运行，弹框`'congratulation!'`）：

```
function check_pass (password) {
  var v1 = {
    'tPlEX': function (v2, v3) {
      return v2 == v3;
    },
    'TcjYB': function (v4, v5) {
      return v4 + v5;
    },
    'ZtFYf': function (v6, v7) {
      return v6 == v7;
    },
    'tPstu': function (v8, v9) {
      return v8 > v9;
    },
    'Vhxzy': function (v10, v11) {
      return v10 < v11;
    },
    'uuFIS': function (v12, v13) {
      return v12 == v13;
    },
    'cRvgS': function (v14, v15) {
      return v14 ^ v15;
    },
    'GsTse': 'Orz..',
    'ykyBq': 'len error'
  };
  var i = 0;
  var sum = 0;
  for (i = 0;; i++) {
    if (i == password.length) {
      break;
    }
    sum = sum + password.charCodeAt(i);
  }
  if (i == 4) {
    if (sum == 417 && password.charAt(3) > 'c' && password.charAt(3) < 'e' && password.charAt(0) == 'b') {
      if ((password.charCodeAt(3) ^ 13) == password.charCodeAt(1)) {
        return 1;
      }
      console.log('Orz..');
    }
  } else {
    console.log('len error');
  }
  return 0;
}
function test () {
  var v16 = {
    'eOZRR': function (v17, v18) {
      return v17(v18);
    },
    'alzHn': 'bird',
    'GyIol': function (v19, v20) {
      return v19(v20);
    },
    'FWSbx': 'congratulation!',
    'tYizA': 'error!'
  };
  if (check_pass('bird')) {
    alert('congratulation!');
  } else {
    alert('error!');
  }
}
test();
```

### 最后提供一个比较完整的 demo

相关的流程：

1.  恢复被隐藏的常量串
2.  去除`Strings Transformations`（常量串隐藏）
3.  识别无用代码并删除（本文没涉及）
4.  去除控制流平坦化
5.  清理常量串隐藏和控制流平坦化带来的无用变量
6.  MemberExpression Array Notation 转 Dot Notation
7.  重命名变量
8.  还原不直观的编码字符串或数值
9.  ……

`src/switch_cff_demo.ts`的骨架基本上和`src/check_pass_demo.ts`类似，只不过更完善。这表明我的代码有一定的通用性。`src/switch_cff_demo.ts`

```
import * as parser from '@babel/parser';
import { renameVars } from './rename_vars';
import generator from '@babel/generator';
import { getFile, writeOutputToFile } from './file_utils';
import { memberExpComputedToFalse } from './member_exp_computed_to_false';
import { translateLiteral } from './translate_literal';
import traverse from '@babel/traverse';
import {
  Node,
  isIdentifier,
  isMemberExpression,
  isObjectExpression,
  isObjectProperty,
  isStringLiteral,
  isFunctionExpression,
  isReturnStatement,
  isBinaryExpression,
  binaryExpression,
  isLogicalExpression,
  logicalExpression,
  isCallExpression,
  callExpression,
  isExpression,
  isNumericLiteral,
  stringLiteral,
  isBlockStatement,
  isSwitchStatement,
  isVariableDeclarator,
  isContinueStatement,
  Statement,
  isUpdateExpression
} from '@babel/types';

const jsCode = getFile('src/inputs/switch_cff_demo.js');
const ast = parser.parse(jsCode);

// 如果常量表不止1处，则此代码不正确
function restoreStringLiteral (ast: Node, stringLiteralFuncs: string[], getStringArr: (idx: number) => string) {
  // 收集与常量串隐藏有关的变量
  traverse(ast, {
    VariableDeclarator (path) {
      const vaNode = path.node;
      if (!isIdentifier(vaNode.init) || !isIdentifier(vaNode.id)) return;
      if (stringLiteralFuncs.includes(vaNode.init.name)) {
        stringLiteralFuncs.push(vaNode.id.name);
      }
    }
  });
  traverse(ast, {
    CallExpression (path) {
      const cNode = path.node;
      if (!isIdentifier(cNode.callee)) return;
      const varName = cNode.callee.name;
      if (!stringLiteralFuncs.includes(varName)) return;
      if (cNode.arguments.length !== 1 || !isNumericLiteral(cNode.arguments[0])) return;
      const idx = cNode.arguments[0].value;
      path.replaceWith(stringLiteral(getStringArr(idx)));
    }
  });
}
restoreStringLiteral(ast, ['_0x27c4'], (idx: number) => {
  return ['661835nuUXrL', 'dKifE', 'try again', 'log', '7aEbwep', 'awvtQ', '2804302XtaWgC', 'rmnID', 'flag{hans}', '21393471OyFTzd', 'lXUhG', '1914456NQDFwp', '1xRwaZJ', '36ZbcbZP', '3gJgrjU', '8162226GwaJpl', '3|4|2|0|5|1', 'split', 'charCodeAt', 'pass', '6278120IHpVNF', 'W_PTJ[P]BN', 'length', 'fromCharCode', '939280gOLaZV'][idx - 0x89];
});

function cff (ast: Node) {
  type ASTNodeMap = {[key: string]: Node}
  const cffTables: {[key: string]: ASTNodeMap} = {};
  traverse(ast, {
    VariableDeclarator (path) {
      const node = path.node;
      if (!node.id || !isIdentifier(node.id)) return;
      const tableName = node.id.name;
      if (!isObjectExpression(node.init)) return;
      const tableProperties = node.init.properties;
      cffTables[tableName] = tableProperties.reduce((cffTable, tableProperty) => {
        if (!isObjectProperty(tableProperty) ||
           !isStringLiteral(tableProperty.key)) return cffTable;
        cffTable[tableProperty.key.value] = tableProperty.value;
        return cffTable;
      }, {} as ASTNodeMap);
    }
  });

  traverse(ast, {
    CallExpression (path) {
      const cNode = path.node;
      if (isMemberExpression(cNode.callee)) {
        if (!isIdentifier(cNode.callee.object)) return;
        const callParams = cNode.arguments;
        const tableName = cNode.callee.object.name;
        if (!isStringLiteral(cNode.callee.property)) return;
        const keyName = cNode.callee.property.value;
        if (!(tableName in cffTables) ||
            !(keyName in cffTables[tableName])) return;
        const shouldBeFuncValue = cffTables[tableName][keyName];
        if (!isFunctionExpression(shouldBeFuncValue) ||
            !shouldBeFuncValue.body.body.length ||
            !isReturnStatement(shouldBeFuncValue.body.body[0])) return;
        // 拿到返回值
        const callArgument = shouldBeFuncValue.body.body[0].argument;
        if (isBinaryExpression(callArgument) && callParams.length === 2) {
          if (!isExpression(callParams[0]) || !isExpression(callParams[1])) {
            throw '二元运算符中，两个参数都应为表达式';
          }
          // 处理function(x, y){return x + y}这种形式
          path.replaceWith(binaryExpression(callArgument.operator, callParams[0], callParams[1]));
        } else if (isLogicalExpression(callArgument) && callParams.length === 2) {
          if (!isExpression(callParams[0]) || !isExpression(callParams[1])) {
            throw '逻辑运算符中，两个参数都应为表达式';
          }
          // 处理function(x, y){return x > y}这种形式
          path.replaceWith(logicalExpression(callArgument.operator, callParams[0], callParams[1]));
        } else if (isCallExpression(callArgument) && isIdentifier(callArgument.callee)) {
          // 处理function(f, ...args){return f(...args)}这种形式
          if (callParams.length == 1) {
            path.replaceWith(callParams[0]);
          } else {
            if (!isExpression(callParams[0])) {
              throw '仅支持第一个参数为函数的形式，如：function(f, ...args){return f(...args)}';
            }
            path.replaceWith(callExpression(callParams[0], callParams.slice(1)));
          }
        }
      }
    },
    MemberExpression (path) {
      const mNode = path.node;
      if (!isIdentifier(mNode.object)) return;
      const tableName = mNode.object.name;
      if (!isStringLiteral(mNode.property)) return;
      const keyName = mNode.property.value;
      if (!(tableName in cffTables) ||
          !(keyName in cffTables[tableName])) return;
      const cffTableValue = cffTables[tableName][keyName];
      path.replaceWith<Node>(cffTableValue);
    }
  });
}
cff(ast);

function switchCFF (ast: Node) {
  traverse(ast, {
    WhileStatement (path) {
      const wNode = path.node;
      if (!isBlockStatement(wNode.body) || !wNode.body.body.length) return;
      const switchNode = wNode.body.body[0];
      if (!isSwitchStatement(switchNode)) return;
      const { discriminant, cases } = switchNode;
      if (!isMemberExpression(discriminant) ||
          !isIdentifier(discriminant.object)) return;
      // switch语句内的控制流平坦化数组名，本例中是 _0x31ce85
      const arrayName = discriminant.object.name;
      // 获取控制流平坦化数组绑定的节点
      const bindingArray = path.scope.getBinding(arrayName);
      if (!bindingArray) return;
      // 经过restoreStringLiteral，我们认为它已经恢复为'v1|v2...'['split']('|')
      if (!isVariableDeclarator(bindingArray.path.node) ||
          !isCallExpression(bindingArray.path.node.init)) return;
      const varInit = bindingArray.path.node.init;
      if (!isMemberExpression(varInit.callee) ||
          !isStringLiteral(varInit.callee.object) ||
          varInit.arguments.length !== 1 ||
          !isStringLiteral(varInit.arguments[0])) return;
      const object = varInit.callee.object.value;
      const propty = varInit.callee.property;
      if (!isStringLiteral(propty) && !isIdentifier(propty)) return;
      const propertyName = isStringLiteral(propty) ? propty.value : propty.name;
      const splitArg = varInit.arguments[0].value;
      // 目前只支持'v1|v2...'.split('|')的解析
      if (propertyName !== 'split') {
        console.warn('switchCFF(ast)：目前只支持\'v1|v2...\'.split(\'|\')的解析');
        return;
      }
      const indexArr = object[propertyName](splitArg);

      const replaceBody = indexArr.reduce((replaceBody, index) => {
        const caseBody = cases[+index].consequent;
        if (isContinueStatement(caseBody[caseBody.length - 1])) {
          caseBody.pop();
        }
        return replaceBody.concat(caseBody);
      }, [] as Statement[]);
      path.replaceInline(replaceBody);

      // 可选择的操作：删除控制流平坦化数组绑定的节点、自增变量名绑定的节点
      if (!isUpdateExpression(discriminant.property) ||
          !isIdentifier(discriminant.property.argument)) return;
      const autoIncrementName = discriminant.property.argument.name;
      const bindingAutoIncrement = path.scope.getBinding(autoIncrementName);
      if (!bindingAutoIncrement) return;
      bindingArray.path.remove();
      bindingAutoIncrement.path.remove();
    }
  });
}
switchCFF(ast);

function removeStringTransCodes (ast: Node) {
  traverse(ast, {
    // 去除给string数组进行随机移位的自执行函数
    CallExpression (path) {
      if (!isFunctionExpression(path.node.callee)) return;
      if (path.node.arguments.length !== 2 ||
          !isNumericLiteral(path.node.arguments[1]) ||
          path.node.arguments[1].value !== 0xdbab3) return;
      path.remove();
    },
    // 去除给string数组进行随机移位的函数
    FunctionDeclaration (path) {
      if (!isIdentifier(path.node.id)) return;
      const funcName = path.node.id.name;
      if (!['_0x27c4', '_0x379e'].includes(funcName)) return;
      path.remove();
    },
    // 去除控制流平坦化的哈希表和用于隐藏常量串的变量
    VariableDeclarator (path) {
      if (!isIdentifier(path.node.id)) return;
      const varName = path.node.id.name;
      // 控制流平坦化的哈希表和用于隐藏常量串的变量
      if (!['_0x550d17', '_0x55bea2', '_0x47f9f1'].includes(varName)) return;
      path.remove();
    }
  });
}
removeStringTransCodes(ast);

memberExpComputedToFalse(ast);
renameVars(
  ast,
  (name:string) => name.substring(0, 3) === '_0x',
  {
    enc: 'enc', _0x263396: 'i', _0x13adf6: 'out'
  }
);
translateLiteral(ast);

const { code } = generator(ast);
writeOutputToFile('switch_cff_demo_out.js', code);
```

解混淆前：

```
var _0x47f9f1 = _0x27c4;
(function (_0x47124a, _0x19f73e) {
  var _0x3b6574 = _0x27c4,
    _0x2c307d = _0x47124a();
  while ([]) {
    try {
      var _0x585cd6 = parseInt(_0x3b6574(0x95)) / 0x1 * (parseInt(_0x3b6574(0x8f)) / 0x2) + -parseInt(_0x3b6574(0x97)) / 0x3 * (parseInt(_0x3b6574(0x9d)) / 0x4) + -parseInt(_0x3b6574(0x89)) / 0x5 + -parseInt(_0x3b6574(0x98)) / 0x6 + -parseInt(_0x3b6574(0x8d)) / 0x7 * (-parseInt(_0x3b6574(0x94)) / 0x8) + parseInt(_0x3b6574(0x96)) / 0x9 * (parseInt(_0x3b6574(0xa1)) / 0xa) + parseInt(_0x3b6574(0x92)) / 0xb;
      if (_0x585cd6 === _0x19f73e) break;
      else _0x2c307d['push'](_0x2c307d['shift']());
    } catch (_0x28b17f) {
      _0x2c307d['push'](_0x2c307d['shift']());
    }
  }
}(_0x379e, 0xdbab3));

function _0x27c4 (_0x122105, _0x24f040) {
  var _0x379e52 = _0x379e();
  return _0x27c4 = function (_0x27c4d4, _0x569919) {
    _0x27c4d4 = _0x27c4d4 - 0x89;
    var _0x5dfb85 = _0x379e52[_0x27c4d4];
    return _0x5dfb85;
  }, _0x27c4(_0x122105, _0x24f040);
}

function _0x379e () {
  var _0x3ed6e2 = ['1914456NQDFwp', '1xRwaZJ', '36ZbcbZP', '3gJgrjU', '8162226GwaJpl', '3|4|2|0|5|1', 'split', 'charCodeAt', 'pass', '6278120IHpVNF', 'W_PTJ[P]BN', 'length', 'fromCharCode', '939280gOLaZV', '661835nuUXrL', 'dKifE', 'try\x20again', 'log', '7aEbwep', 'awvtQ', '2804302XtaWgC', 'rmnID', 'flag{hans}', '21393471OyFTzd', 'lXUhG'];
  _0x379e = function () {
    return _0x3ed6e2;
  };
  return _0x379e();
}

function enc (_0x3bf54e) {
  var _0x55bea2 = _0x27c4,
    _0x550d17 = {
      'dKifE': _0x55bea2(0x99),
      'lXUhG': function (_0x7a78d6, _0x13ee42) {
        return _0x7a78d6 < _0x13ee42;
      },
      'rmnID': function (_0x28f0fb, _0x77896d) {
        return _0x28f0fb & _0x77896d;
      },
      'awvtQ': function (_0x26b565, _0x3ffc0b) {
        return _0x26b565 ^ _0x3ffc0b;
      }
    },
    _0x31ce85 = _0x550d17[_0x55bea2(0x8a)][_0x55bea2(0x9a)]('|'),
    _0x1ffdde = 0x0;
  while ([]) {
    switch (_0x31ce85[_0x1ffdde++]) {
      case '0':
        _0x263396 += 0x1;
        continue;
      case '1':
        return _0x13adf6;
      case '2':
        var _0x13adf6 = '';
        continue;
      case '3':
        var _0x263396 = 0x0;
        continue;
      case '4':
        _0x263396 += -0x1;
        continue;
      case '5':
        for (; _0x550d17[_0x55bea2(0x93)](_0x263396, _0x3bf54e[_0x55bea2(0x9f)]); ++_0x263396) {
          var _0x494484 = 0x0;
          if (_0x550d17[_0x55bea2(0x90)](_0x263396, 0x1)) _0x494484 = 0x33;
          else _0x494484 = 0x31;
          _0x13adf6 += String[_0x55bea2(0xa0)](_0x550d17[_0x55bea2(0x8e)](_0x3bf54e[_0x263396][_0x55bea2(0x9b)](), _0x494484));
        }
        continue;
    }
    break;
  }
}
if (enc(_0x47f9f1(0x91)) === _0x47f9f1(0x9e)) console[_0x47f9f1(0x8c)](_0x47f9f1(0x9c));
else console[_0x47f9f1(0x8c)](_0x47f9f1(0x8b));
```

解混淆后：

```
function enc (v1) {
  var i = 0;
  i += -1;
  var out = '';
  i += 1;
  for (; i < v1.length; ++i) {
    var v2 = 0;
    if (i & 1) v2 = 51;else v2 = 49;
    out += String.fromCharCode(v1[i].charCodeAt() ^ v2);
  }
  return out;
}
if (enc('flag{hans}') === 'W_PTJ[P]BN') console.log('pass');else console.log('try again');
```

完美还原！

### 参考资料

1.  npm package.json scripts 传递参数的解决方案：[https://juejin.cn/post/7032919800662016031](https://juejin.cn/post/7032919800662016031)
2.  node 执行 shell 命令：[https://www.jianshu.com/p/c0d31513953a](https://www.jianshu.com/p/c0d31513953a)
3.  IDEA 配置 eslint：[https://blog.csdn.net/weixin_33850015/article/details/91369049](https://blog.csdn.net/weixin_33850015/article/details/91369049)
4.  利用 AST 对抗 js 混淆 (三) 控制流平坦化(Control Flow Flattening) 的处理：[https://blog.csdn.net/lacoucou/article/details/113665767](https://blog.csdn.net/lacoucou/article/details/113665767)
5.  Babel AST 节点介绍：[https://www.jianshu.com/p/4f27f4aa576f](https://www.jianshu.com/p/4f27f4aa576f)
6.  Babel 还原不直观的编码字符串或数值：[https://lzc6244.github.io/2021/07/28/Babel%E8%BF%98%E5%8E%9F%E4%B8%8D%E7%9B%B4%E8%A7%82%E7%9A%84%E7%BC%96%E7%A0%81%E5%AD%97%E7%AC%A6%E4%B8%B2%E6%88%96%E6%95%B0%E5%80%BC.html](https://lzc6244.github.io/2021/07/28/Babel%E8%BF%98%E5%8E%9F%E4%B8%8D%E7%9B%B4%E8%A7%82%E7%9A%84%E7%BC%96%E7%A0%81%E5%AD%97%E7%AC%A6%E4%B8%B2%E6%88%96%E6%95%B0%E5%80%BC.html)
7.  AST 在 js 逆向中 switch-case 反控制流平坦化：[https://blog.csdn.net/Python_DJ/article/details/126882432](https://blog.csdn.net/Python_DJ/article/details/126882432)![](https://avatar.52pojie.cn/data/avatar/000/93/53/52_avatar_middle.jpg)xixicoco 牛逼啊，看不懂 ![](https://avatar.52pojie.cn/data/avatar/000/00/00/01_avatar_middle.jpg) Hmily 这四篇主题都很短，合并到一起吧，方便阅读。![](https://avatar.52pojie.cn/data/avatar/001/90/61/77_avatar_middle.jpg)

> [Hmily 发表于 2022-10-17 00:36](https://www.52pojie.cn/forum.php?mod=redirect&goto=findpost&pid=44349476&ptid=1700036)  
> 这四篇主题都很短，合并到一起吧，方便阅读。

已经合并啦 ![](https://avatar.52pojie.cn/images/noavatar_middle.gif) hans7 谢谢分享 ![](https://avatar.52pojie.cn/images/noavatar_middle.gif) xzhtx "用 Babel 解析 AST 处理 OB 混淆 JS 代码（三）：处理 Strings Transformations（全网首创）"  
你这个字符串解密，2019 年人家就给出方案了，现在都快 2023 年了，还首创。。。。![](https://avatar.52pojie.cn/images/noavatar_middle.gif)悦来客栈的老板 ![](https://static.52pojie.cn/static/image/smiley/default/17.gif)牛皮牛皮 学习了 ![](https://avatar.52pojie.cn/images/noavatar_middle.gif) acfv5222 怕是楼主村里刚通网，没见识过蔡老板的 AST![](https://avatar.52pojie.cn/images/noavatar_middle.gif)JunLee123 谢谢了，学习一下 ![](https://avatar.52pojie.cn/images/noavatar_middle.gif) homyang 感谢分享