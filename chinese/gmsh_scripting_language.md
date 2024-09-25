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
x_1 = 1;
x_2 = 2;
x_3 = 3;
```

方括号`[]`允许从列表中提取一项（也可以使用圆括号代替方括号）。井号`#`允许获取列表的长度。运算符*一元左运算符*、*一元右运算符*、*二元运算符*、*三元左运算符*和*三元右运算符*在[运算符](https://gmsh.info/doc/texinfo/gmsh.html#Operators)中定义。有关内置函数的定义，参见[内置函数](https://gmsh.info/doc/texinfo/gmsh.html#Built_002din-functions)。各种*数字选项*列在[Gmsh选项](https://gmsh.info/doc/texinfo/gmsh.html#Gmsh-options)。

- `Find` 搜索第二个表达式中出现的第一个表达式（两者都可以是列表）。

- `StrFind`搜索第一个字符串表达式中出现的第二个字符串表达式。

- `StrCmp`比较两个字符串（根据第一个字符串大于、等于或小于第二个字符串返回大于、等于或小于0的整数）。

- `StrLen`返回字符串长度。

- `TextAttributes`为文本字符串创建属性。

- `Exists`检查具有给定的名称的变量是否存在（即，之前是否定义过）。

- `FileExists`检查具有给定的名称的文件是否存在。

- `StringToName`根据提供的字符串创建一个名称。

- `GetNumber`允许获取ONELAB变量的值（第二个可选参数是在变量不存在时返回的默认值）。

- `GetValue`允许互动地向用户请求输入值（第二个参数是非互动模式下返回的默认值）。例如，在输入文件中插入`GetValue("Value of parameter alpha?", 5.76)`将向用户查询某个参数`alpha`的值，假设默认值为`5.76`。如果已经设置选项`General.NoPopup`（参见[Gmsh选项](https://gmsh.info/doc/texinfo/gmsh.html#General-options)），将不会询问并且自动使用默认值。

- `DefinNumber`允许定义ONELAB变量（append： in-line），作为第一个参数给出的*表达式*是默认值；后面是各种ONELAB 选项。有关更多信息，请参阅 ONELAB 教程 wiki。

表达式列表经常被使用，并且如下定义：

```GmshScriptingLanguage
expression-list:
  expression-list-item <, expression-list-item> …
```

其中

```GmshScriptingLanguage
expression-list-item:
  expression |
  expression : expression |
  expression : expression : expression |
  string [ ] |  string ( ) |
  List [ string ] |
  List [ expression-list-item ] |
  List [ { expression-list } ] |
  Unique [ expression-list-item ] |
  Abs [ expression-list-item ] |
  ListFromFile [ expression-char ] |
  LinSpace[ expression, expression, expression ] |
  LogSpace[ expression, expression, expression ] |
  string [ { expression-list } ] |
  Point { expression } |
  transform |
  extrude |
  boolean |
  Point|Curve|Surface|Volume In BoundingBox { expression-list } |
  BoundingBox Point|Curve|Surface|Volume { expression-list } |
  Mass Curve|Surface|Volume { expression } |
  CenterOfMass Curve|Surface|Volume { expression } |
  MatrixOfInertia Curve|Surface|Volume { expression } |
  Point { expression } |
  Physical Point|Curve|Surface|Volume { expression-list } |
  <Physical> Point|Curve|Surface|Volume { : } |
```

最后一个定义中第二种情况允许创建一个列表，包含两个表达式之间的以单位递增步长的数字范围。第三种情况也允许创建一个列表，包含两个表达式之间的数字范围，但正负递增步长等于第三个表达式。第四种、五种和六种情况允许引用表达式列表（也可以使用圆括号代替方括号）。

- `Unique` 对列表中的条目进行排序并删除所有重复项。

- `Abs` 取列表中所有条目的绝对值。

- `ListFromFile` 从文件读取数字列表

- `ListSpace` 和 `LogSpace` 使用线性或对数间距构建列表。

接下来的两种情况允许引用表达式子列表（其元素对应于*表达式列表*提供的索引）。接下来的情况允许检索通过几何变换、挤压和布尔运算创建的实体的索引（参见[变换](https://gmsh.info/doc/texinfo/gmsh.html#Transformations)、[挤压](https://gmsh.info/doc/texinfo/gmsh.html#Extrusions)和[布尔运算](https://gmsh.info/doc/texinfo/gmsh.html#Boolean-operations)）。

接下来的两种情况允许检索给定边框中的实体，或得到给定实体的边框（边框被指定为（X最小值，Y最小值，Z最小值，X最大值，Y最大值，Z最大值））。请注意，坐标的顺序和场景的`BoundingBox`命令中的不同：参见[其他常规命令](https://gmsh.info/doc/texinfo/gmsh.html#Other-general-commands)。最后一种情况允许检索实体的质量、质心或惯性矩阵、给定集合点的坐标（参见[Points](https://gmsh.info/doc/texinfo/gmsh.html#Points)）、组成物理组的基本实体和模型中所有（物理或基本）点、曲线、表面或体积。这些操作都会出发CAD模型与内部Gmsh模型的同步。

要了解此类表达式的实际用途，请查看[Gmsh教程](./gmsh_tutoral.md)中的前几个示例。请注意，为了简化语法，如果*表达式列表*仅包含单个项目，您可以省略包括住*表达式列表*的花括号`{}`。另外请注意，可以在花括号*表达式列表*前添加一个负号，以改变所有*表达式列表项*的符号。

对于某些命令，在列表中指定所有可能的表达式是有意义的。可以通过*表达式列表或全部*，定义如下：

```GmshScriptingLanguage
expression-list-or-all:
  expression-list | :
```

”全部“的意思依赖于上下文。例如，`Curve {:}`将会得到模型中所有存在的曲线的id，而`Surface {:}`会得到所有表面的id。

### 5.1.3 字符串表达式

字符串表达式定义如下：

```GmshScriptingLanguage
string-expression:
  "string" |
  string | string[ expression ] |
  Today | OnelabAction | GmshExecutableName |
  CurrentDirectory | CurrentDir | CurrentFileName
  StrPrefix ( string-expression ) |
  StrRelative ( string-expression ) |
  StrCat ( string-expression <,…> ) |
  Str ( string-expression <,…> ) |
  StrChoice ( expression, string-expression, string-expression ) |
  StrSub( string-expression, expression, expression ) |
  StrSub( string-expression, expression ) |
  UpperCase ( string-expression ) |
  AbsolutePath ( string-expression ) |
  DirName ( string-expression ) |
  Sprintf ( string-expression , expression-list ) |
  Sprintf ( string-expression ) |
  Sprintf ( string-option ) |
  GetEnv ( string-expression ) |
  GetString ( string-expression <,string-expression>) |
  GetStringValue ( string-expression , string-expression ) |
  StrReplace ( string-expression , string-expression , string-expression )
  NameToString ( string ) | N2S ( string ) |
  <Physical> Point|Curve|Surface|Volume { expression } |
  DefineString(string-expression, onelab-options)
```

- `Today`返回当前日期。

- `OnelabAction`返回当前ONELAB动作（例如，`check` 或 `compute`）。

- `GmshExecutableName` 返回Gmsh可执行文件的完整路径。

- `CurrentDirectory` 或 `CurrentDir` 或 `CurrentFileName` 返回文件夹和当前在解释的脚本文件名。

- `StrPrefix`和 `StrRelative` 取出给定文件名的前缀（例如，去除拓展名）或相对路径。

- `StrCat` 和 `Str` 连接字符串表达式（`Str` 在除去最后的每个字符串末尾添加换行字符）。

- `StrChoice` 根据*表达式*的值返回第一个或第二个*字符串表达式*。

- `StrSub` 返回从第一个*表达式*给出的字符位置开始，跨过第二个*表达式*给出的字符数或直到字符串末尾（先出现者为准；如果未提供第二个表达式，则始终返回后者）的部分字符串。

- `UpperCase` 转换*字符串表达式*到大写。

- `AbsolutePath` 返回文件的绝对路径。

- `DirName` 返回文件的目录。

- `Sprintf` 等价于C语言的 `Sprintf` 函数（其中*字符串表达式*是一个可以包含浮点格式字符的格式字符串：%e, %g, etc.）。各种*字符串选项*列在[Gmsh选项](https://gmsh.info/doc/texinfo/gmsh.html#Gmsh-options)。

- `GetEnvThe` 取得操作系统的环境变量的值。

- `GetString` 允许得到ONELAB字符串的值（第二个可选参数是变量不存在时候返回的默认值）。

- `GetStringValue` 互动式地请求用户输入一个值（第二个参数是非互动模式下将会用到的值）。

- `StrReplace` 的参数是：输入字符串，旧子字符串，新子字符串（方括号可以用来代替`Str` 和 `Sprintf` 中的圆括号）。

- `Physical Point`，等，或 `Point`， 检索物理或基本实体的名称（如果有）。

- `NameToString` 转化一个变量名到字符串。

- `DefineString` 允许定义ONELAB变量（in-line）。作为第一个参数给出的*字符串表达式*是默认；后面是各种ONELAB选项。参见[ONELAB教程百科](https://gitlab.onelab.info/doc/tutorials/wikis/ONELAB-syntax-for-Gmsh-and-GetDP)以获取更多信息。

字符串表达式常常被用来指定非数字选项和输入/输出文件名。参见[t8](https://gmsh.info/doc/texinfo/gmsh.html#t8)，内含动画脚本中对*字符串表达式*的一种有趣的用法。

字符串列表定义如下：

```GmshScriptingLanguage
string-expression-list:
  string-expression <,…>
```

### 5.1.4 颜色表达式

颜色表达式是固定长度的括号表达式列表和字符串的混合：

```GmshScriptingLanguage
color-expression:
  string-expression |
  { expression, expression, expression } |
  { expression, expression, expression, expression } |
  color-option
```

第一种情况允许使用X Windows名称来指代颜色，例如，`Red`, `SpringGreen`, `LavenderBlush3`, ... （参见[src/common/Colors.h](https://gitlab.onelab.info/gmsh/gmsh/blob/gmsh_4_13_1/src/common/Colors.h)的源代码的完整列表）。

第二种情况允许通过三个表达式指定红绿蓝组分来定义颜色。

第三种情况允许通过红绿蓝组分和alpha通道来定义颜色。

最后一种情况允许将*颜色选项*作为*颜色表达式*来使用。各种*颜色选项*列在[Gmsh选项](https://gmsh.info/doc/texinfo/gmsh.html#Gmsh-options)中。

参见[t3](https://gmsh.info/doc/texinfo/gmsh.html#t3)，内含使用颜色表达的示例。

- 操作符

- 内置函数

- 用户自定义宏

- 循环和条件

- 其他常规命令

### 5.1.5 操作符

Gmsh操作符与C和C++中的相应运算符类似。这里是可用的一元，二元和三元操作符的列表。

*一元左操作符*

- \-

> 一元负号

- !

> 逻辑非

*一元右操作符*

- ++ 

> 后自增

- -- 

> 后自减

*二元操作符*

- ^

> 指数运算

- *

> 乘法运算

- /

> 除法运算

- %

> 取模

- \+

> 加法运算

- \-

> 减法运算

- ==

> 等于

- !=

> 不等

- \>

> 大于

- \>=

> 大于或等于

- < 

> 小于

- <=

> 小于或等于

- &&

> 逻辑与

- ||

> 逻辑或。（注意：逻辑或总是隐含两个操作数的评估。不同于C或C++，第一个操作数为真时第二个`||的操作数将不被评估）。

*三元左运算符*

- ?

*三元右运算符*

- : 唯一的三元运算符，由*三元左运算符*和*三元右运算符*构成，第一个参数非零时，返回第二个参数，否则返回其第三个参数的值。

评估优先级总结如下12条（从强到弱，即`*`的评估优先级高于`+`）。圆括号`()`可在任何位置使用以评估顺序。

1. `()`, `[]`, `.`, `#`
2. `^`
3. `!`, `++`, `--`, `-` (unary)
4. `*`, `/`, `%`
5. `+`, `-`
6. `<`, `>`, `<=`, `>=`
7. `==`, `!=`
8. `&&`
9. `||`
10. `?:`
11. `=`, `+=`, `-=`, `*=`, `/=`

### 5.1.6 内置函数

内置函数由一个标识符和一对括号组成，括号内是*表达式列表*，即函数的参数列表。此参数列表也可以放在方括号内，而不是圆括号内。以下是当前实现的内置函数列表：

*内置函数*：

- `Acos (expression)`

> *表达式*的反余弦（逆余弦）在 [-1,1] 范围内。返回 [0,Pi] 范围内的值。

- `Asin (expression)`

> *表达式*的反正弦（逆正弦）在[-1, 1]范围内。返回[-Pi/2, Pi/2]范围内的值。

- `Atan (expression)`

> *表达式*的反正切（逆正切）。返回[-Pi/2, Pi/2]范围的值。

- `Atan2 (expression, expression)`

> 第一个表达式除以第二个表达式的反正切（逆正切），返回[-Pi, Pi]之间的值。

- `Ceil (expression)`

> *表达式*向上取整得到最近的整数。

- `Cos (expression)`

> *表达式*的余弦。

- `Cosh (expression)`

> *表达式*的双曲余弦。

- `Exp (expression)`

> 返回e（自然对数的底数）的*表达式*的值的次方。

- `Fabs (expression)`

> *表达式*的绝对值

- `Fmod (expression)`

> 第一个*表达式*除以第二个*表达式*的余数，并保留第一个表达式的符号

- `Floor  (expression)`

> *表达式*向下取整得到最近的整数

- `Hypot (expression, expression)`

> 返回两个参数的平方和的平方根

- `Log (expression)`

> *表达式*的自然对数（*表达式* > 0）

- `Log10 (expression)`

> 表达式的以10为底的对数（*表达式*  > 0）

- `Max (expression, expression)`

> 两个参数的最大值

- `Min (expression, expression)`

> 两个参数的最小值

- `Modulu (expression, expression)`

> 参见 `Fmod (expression, expression)`

- `Rand (expression)`

> 0到*表达式*之间的随机数

- `Round (expression)`

> 四舍五入*表达式*到最近的整数

- `Sqrt (expression)`

> *表达式*的平方根（表达式 >= 0）

- `Sin (expression)`

> *表达式*的正弦

- `Sinh (expression)`

> *表达式*的双曲正弦

- `Tan (expression)`

> *表达式*的正切

- `Tanh (expression)`

> *表达式*的双曲正切

### 5.1.7 用户自定义宏

用户自定义宏不接受参数，并且被评估为就像包含宏主体的文件被包含在`Call`语句的位置。

- `Marco String | string-expression`

> 开始声明名称为`string`的用户自定义宏。宏的内容从“宏字符串”后的行开始，可以包含任何Gmsh命令。宏的同义词是函数。

- `Return`

> 结束当前用户自定义宏的内容。宏声明不能嵌套。

- Call string | string-expression

> 调用名称为`string`的宏的内容。

参见[t5](https://gmsh.info/doc/texinfo/gmsh.html#t5)，有一个用户自定义宏的例子。Gmsh脚本语言有一个缺点是所有的变量都是公有的(public)，因此宏内部定义的变量在外部也可以被访问！

### 5.1.8 循环和条件

循环和条件定义如下，并且可以嵌套：

- `For (expression: expression)`

> 以一个单位自增步长从第一个*表达式*的值迭代到第二个*表达式*的值。在每一步迭代，`For (expression: expression)`和配对的`EndFor`之间的命令将会被执行。

- `For (expression: expression: expression)`

> 以等于第三个*表达式*的正或负的自增步长从第一个*表达式*的值迭代到第二个*表达式*的值。在每一步迭代，`For (expression: expression)`和配对的`EndFor`之间的命令将会被执行。

- `For string In {expression: expression}`

> 以一个单位自增步长从第一个*表达式*的值迭代到第二个*表达式*的值。在每一步迭代，迭代的值受到名为`string`的表达式的影响。`For (expression: expression)`和配对的`EndFor`之间的命令将会被执行。

- `For string In {expression: expression: expression}`

> 以等于第三个*表达式*的正或负的自增步长从第一个*表达式*的值迭代到第二个*表达式*的值。在每一步迭代，迭代的值受到名为`string`的表达式的影响。`For (expression: expression)`和配对的`EndFor`之间的命令将会被执行。

- EndFor

> 结束匹配的`For`命令

- `If (expression)`

> 如果*表达式*为非零，`If (expression)`和匹配的`ElseIf, Else`或`EndIf`之间包含的内容将会被执行(evaluated)。

- `Else (expression)`

> 如果*表达式*为非零并且之前匹配的代码`If`和`ElseIf`的*表达式*都不为非零，则`If (expression)`和匹配的`ElseIf, Else`或`EndIf`之间包含的内容将会被执行(evaluated)。

- `Else`

> 如果之前匹配的代码`If`和`ElseIf`的*表达式*都不为非零，`Else`和匹配的`EndIf`之间的内容将会被执行。

- EndIf

> 结束匹配的`If`命令

### 5.1.9 其他常规命令

下面的命令可以在Gmsh脚本的任何地方使用：

- `string = expression`

> 创建一个新的表达式的标识符`string`，或将*表达式*赋值给一个已经存在的表达式标识符。下面的表达式标识符被预定义了（Gmsh解释器中被硬编码）：
> 
> - `Pi`
> 
> > 返回  3.1415926535897932。
> 
> - `GMSH_MAJOR_VERSION`
> 
> > 返回 Gmsh的大版本号。
> 
> - `GMSH_MINOR_VERSION`
> 
> > 返回Gmsh的小版本号。
> 
> - `GMSH_PATCH_VERSION`
> 
> > 返回Gmsh的补丁版本号。
> 
> - `MPI_Size`
> 
> > 返回Gmsh正在运行的处理器数量。除非您使用`ENABLE_MPI`编译Gmsh，否则它始终是1（参见[编译源代码](./appendix _A_compiling_the_source_code.md)）。
> 
> - `MPI_Rank`
> 
> > 返回当前处理器的排名
> 
> - `Cpu`
> 
> > 返回当前CPU时间（以秒计）
> 
> - `Memory`
> 
> > 返回当前内存占用（以Mb即）
> 
> - `TotalMemory`
> 
> > 返回总可用内存（以Mb计）
> 
> - `newp`
> 
> > 返回下一个可用的点标签。就如[几何模块](https://gmsh.info/doc/texinfo/gmsh.html#Geometry-module)中解释的那样，每个几何点必须关联一个唯一标签：`newp`允许知道已经分发的最高的标签（通过加一）。在写用户自定义宏（参见[用户自定义宏](https://gmsh.info/doc/texinfo/gmsh.html#User_002ddefined-macros)）或常规几何原语，在不知道哪些标签已经被分发和那些标签还可用，这将会很有用。
> 
> - `newc`
> 
> > 返回下一个可用的曲线标签
> 
> - `news`
> 
> > 返回下一个可用的表面标签
> 
> - `newv`
> 
> > 返回下一个可用的体积标签
> 
> - `newcl`
> 
> > 返回下一个可用的曲线环(curve loop)标签
> 
> - `newsl`
> 
> > 返回下一个可用的表面环(volume loop)标签
> 
> - `newreg`
> 
> > 返回下一个可用的区域标签。即，`newreg`返回`newp`,`newl`,`news`,`newv`,`newcl`(此处原版gmsh文档写成`newll`，经鉴定，视为笔误，并将通过邮件形式联系gmsh作者团队进行修改), `newsl`和所有物理组的标签的最大值。



- `string = {}`

> 创建一个空的新的表达式列表的标识符`string`

- `string[] = {expression-list}`
