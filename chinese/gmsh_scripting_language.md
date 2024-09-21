# 5 Gmsh脚本语言

脚本用Gmsh 脚本语言在运行时由 Gmsh 分析器解释。脚本以 ASCII 文件格式编写，通常使用 .geo 扩展名，但也可以使用任何扩展名（或根本不使用扩展名）。例如，Gmsh 通常使用 .pos 扩展名来表示包含后处理命令的脚本，特别是已解析的后处理视图（参见[后处理脚本命令](https://gmsh.info/doc/texinfo/gmsh.html#Post_002dprocessing-scripting-commands)）。

一直以来，.geo脚本都是使用Gmsh执行复杂任务的主要方法，而且它们也确实相当强大：它们可以处理（列表的）浮点（参见[浮点表达式](https://gmsh.info/doc/texinfo/gmsh.html#Floating-point-expressions)）和字符串（参见[字符串表达式]([Gmsh 4.13.1](https://gmsh.info/doc/texinfo/gmsh.html#String-expressions))）变量，循环和测试（参见[循环和条件](https://gmsh.info/doc/texinfo/gmsh.html#Loops-and-conditionals)），宏（参见[用户自定义宏](https://gmsh.info/doc/texinfo/gmsh.html#User_002ddefined-macros)）等等。不过，与实际的编程语言相比，Gmsh脚本语言仍然具有很大的局限性：例如，没有私有变量，宏不能带参数并且解析器的运行时解释会影响大模型的性能。因此根据工作流程和应用程序的不同，使用Gmsh API（参见[Gmsh应用程序接口](./gmsh_application_programming_interface.md)）有时会更好。API的缺点在于，虽然脚本语言已经嵌入到Gmsh中，因此可以直接在Gmsh应用程序中独立运行，但API仍需要外部依赖（一个C++，C或Fortran编译器；或一个Python或Julia解释器）。

本章介绍脚本语言时，首先详细介绍常规命令（参见[常规脚本命令](https://gmsh.info/doc/texinfo/gmsh.html#General-scripting-commands)），然后详细介绍几何体（参见[几何脚本命令](https://gmsh.info/doc/texinfo/gmsh.html#Geometry-scripting-commands)）、网格（参见[网格脚本命令](https://gmsh.info/doc/texinfo/gmsh.html#Mesh-scripting-commands)）和后处理（参见[后处理脚本命令](https://gmsh.info/doc/texinfo/gmsh.html#Post_002dprocessing-scripting-commands)）模块的特定脚本语言。

本章的剩余部分描述脚本语言时，将使用以下的规则（请注意，元句法变量定义在整章中都有效，不仅在其出现定义的章节中有效）：

1. 关键字和字面符号会写成`this`。

2. 元句法变量（即，不属于语法的文本位，但代表其他文本位）会写成这样 _this_。

3. 元句法变量后的冒号（:）将变量和其定义分开。

4. 可选的规则用“<>”对包起来。

5. 多种选择用“|”分开。

6. 三个点（...）表明前一条规则可能（多次）重复。
- 常规脚本语言

- 几何脚本语言

- 网格脚本语言

- 后处理脚本语言

## 5.1 常规脚本语言

- 注释

- 浮点表达式

- 字符串表达式

- 颜色表达式

- 操作符

- 内置函数

- 用户自定义宏

- 循环和条件

- 其他常规命令

### 5.1.1 注释

Gmsh脚本文件支持C和C++风格注释

1. 在/\*和\*/文本对直接的文本将被忽略；

2. 双斜线//之后的其余文本将被忽略。

这些命令在双引号内或关键字内不会产生所述效果。另外请注意，所有表达式中的 “空白”（空格、制表符、换行符）都将被忽略。

### 5.1.2 浮点表达式

Gmsh脚本中使用的两种常量类型是*实数*和*字符串*（没有整数类型）。这些类型的含义和语法与C和C++编程语言相同。

浮点表达式（或更简单的，”表达式“）通过元句法变量*表达式*表示，并且在解析脚本文件时进行评估：

```GmshScriptingLanguage
expression:
  real |
  string |
  string ~ { expression }
  string [ expression ] |
  # string [ ] |
  ( expression ) |
  operator-unary-left expression |
  expression operator-unary-right |
  expression operator-binary expression |
  expression operator-ternary-left expression
    operator-ternary-right expression |
  built-in-function |
  number-option |
  Find(expression-list-item, expression-list-item) |
  StrFind(string-expression, string-expression) |
  StrCmp(string-expression, string-expression) |
  StrLen(string-expression) |
  TextAttributes(string-expression<,string-expression…>) |
  Exists(string) | Exists(string~{ expression }) |
  FileExists(string-expression) |
  StringToName(string-expression) | S2N(string-expression) |
  GetNumber(string-expression <,expression>) |
  GetValue("string", expression) |
  DefineNumber(expression, onelab-options)
```

这些*表达式*构成Gmsh脚本命令的大部分。当`~{expression}`被添加到字符串之后，结果将是一个由*字符串*，\_(下划线)和*表达式*的值构成的新字符串。这在循环（参见[循环和条件]([Gmsh 4.13.1](https://gmsh.info/doc/texinfo/gmsh.html#Loops-and-conditionals))）中最有用，因为它运行自动定义唯一的字符串。例如，

```GmshScriptingLanguage
For i In {1:3}
  x~{i} = i;
EndFor
```

等同于

```GmshScriptingLanguage

```


