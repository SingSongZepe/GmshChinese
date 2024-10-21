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

浮点表达式（或更简单的，”表达式“）通过元句法变量*表达式*表示，在解析脚本文件时进行评估：

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

接下来的两种情况允许检索给定边框中的实体，或得到给定实体的边框（边框被指定为（X最小值，Y最小值，Z最小值，X最大值，Y最大值，Z最大值））。请注意，坐标的顺序和场景的`BoundingBox`命令中的不同：参见[其他常规命令](https://gmsh.info/doc/texinfo/gmsh.html#Other-general-commands)。最后一种情况允许检索实体的质量、质心或惯性矩阵、给定集合点的坐标（参见[Points](https://gmsh.info/doc/texinfo/gmsh.html#Points)）、组成物理组的基本实体和模型中所有（物理或基本）点、曲线、表面或体积。这些操作都会触发CAD模型与内部Gmsh模型的同步。

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

- `string[] = { expression-list }`

> 用列表`expression-list`创建一个新的表达式列表标识符`string`，或给一个已经存在的表达式列表标识符赋值。可用用圆括号代替方括号；虽然不建议，但是圆括号和方括号都可以被省略。

- `string [{ expression-list }] = { expression-list }`

> 将右侧表达式列表中的每一项影响到现有表达式列表标识符的元素（根据左侧表达式列表索引）。两个*表达式列表*必须包含相同数量的项。圆括号也可以用来代替方括号。

- `string += expression`

> 添加并赋值`expression`到一个存在的表达式标识符。

- `string -= expression`

> 减去并赋值`expression`到一个存在的表达式标识符。

- `string *= expression`

> 乘以并赋值`expression`到一个存在的表达式标识符。

- `string /= expression`

> 除以并赋值`expression`到一个存在的表达式标识符。

- `string += { expression-list }`

> 将表达式列表附加到现有的表达式列表或使用表达式列表创建一个新的表达式列表。

- `string -= { expression-list }`

> 从已经存在的表达式列表移除`expression-list`中的项。

- `string [{ expression-list }] += { expression-list }`

> 项对项得添加并赋值右边`expression-list`到一个已经存在的表达式列表标识符。圆括号也可以用来代替方括号。

- `string [{ expression-list }] -= { expression-list }`

> 项对项得减去并赋值右边`expression-list`到一个已经存在的表达式列表标识符。圆括号也可以用来代替方括号。

- `string [{ expression-list }] *= { expression-list }`

> 项对项得乘以并赋值右边`expression-list`到一个已经存在的表达式列表标识符。圆括号也可以用来代替方括号。

- `string [{ expression-list }] /= { expression-list }`

> 项对项得除以并赋值右边`expression-list`到一个已经存在的表达式列表标识符。圆括号也可以用来代替方括号。

- `string = string-expression`

> 用给定的*字符串表达式*创建一个新的字符串表达式标识符`string`

- `string[] = Str( string-expression-list )`

> 用给定的*字符串表达式*列表创建一个新的字符串表达式列表标识符`string`。圆括号也可以用来代替方括号。

- `string[] += Str( string-expression-list )`

> 添加字符串表达式列表到一个已经存在的表达式。圆括号也可以用来代替方括号。

- `DefineConstant[ string = expression|string-expression<, ...> ] `

> 仅当字符串表达式标识符`string`未被定义时，用*表达式*的值创建新的字符串表达式标识符`string`。

- `DefineConstant[ string = {expression|string-expression, onelab-options} <, ...> ]`

> 和前一个情况一样，但如果之前未被定义，还会和ONELAB数据库交互该变量。参见[ONELAB教程wiki](https://gitlab.onelab.info/doc/tutorials/wikis/ONELAB-syntax-for-Gmsh-and-GetDP)以获取更多信息。

- `SetNumber( string-expression, expression )`

> 设置数字ONELAB变量`string-expression`的值。

- `SetString( string-expression, string-expression )`

> 设置字符串ONELAB变量`string-expression`的值。

- `number-option = expression`

> 将*表达式*赋值给一个实选项(real option)。

- `string-option = expression`

> 将*字符串表达式*赋值给一个字符串选项(string option)。

- `color-option = color-expression`

> 将*颜色表达式*赋值给一个颜色选项(color option)。

- `number-option += expression`

> 添加并赋值*表达式*到一个实选项。

- `number-option -= expression`

> 减去并赋值*表达式*到一个实选项。

- `number-option /= expression`

> 乘以并赋值*表达式*到一个实选项。

- `number-option *= expression`

> 除以并赋值*表达式*到一个实选项。

- `Abort`

> 退出当前脚本。

- `Exit < expression >`

> 退出Gmsh（可用使用级别`expression`代替0）。

- `Printf ( string-expression <, expression-list> )`

> 在信息窗口和/或终端打印字符串表达式。`Printf`等价于C语言的`printf`函数：其中*字符串表达式*是一个可以包含浮点格式字符的格式字符串（%e, %g, etc.）。注意所有表达式都会作为Gmsh中的浮点值（参见[浮点表达式](https://gmsh.info/doc/texinfo/gmsh.html#Floating-point-expressions)）来评估，所以*字符串表达式*中只有有效的浮点格式字符才有作用。参见[t5](https://gmsh.info/doc/texinfo/gmsh.html#t5)，一个使用`Printf`的例子。

- `Printf ( string-expression, expression-list ) > string-expression`

> 和上面的`Printf`相同，但是输出表达式到文件内。

- `Printf ( string-expression, expression-list ) >> string-expression`

> 和上面的`Printf`相同，但是追加表达式到文件末尾。

- `Warning|Error ( string-expression <, expression-list > )`

> 和`Printf`一样，但是抛出一个警告或错误。

- `Merge string-expression`

> 合并名为`string-expression`的文件。这个命令等价于GUI中的"文件(File) -> 合并(Merge)"菜单。如果*字符串表达式*中的路径不是绝对的，*字符串表达式*将会添加到当前文件的路径。这一操作触发CAD模型和内置Gmsh模型的同步。

- `ShapeFromFile ( string-expression )`

> 合并一个BREP，STEP或IGES文件并且返回最高维实体的标签。仅在使用OpenCASCADE几何内核的时候可用。

- `Draw`

> 重绘场景

- `SplitCurrentWindowHorizontal expression`

> 按`expression`的给定比例来水平分割当前窗口。

- `SplitCurrentWindowVertical expression`

> 按`expression`的给定比例来竖直分割当前窗口。

- `SetCurrentWindow expression`

> 通过在所有窗口列表中指定当前窗口的索引（从0开始）来设置当前窗口。当新窗口被创建，它将会被添加到列表的末尾。

- `UnsplitWindow`

> 恢复到单独窗口。

- `SetChanged`

> 强制再生成网格和后处理的顶点数组。例如，对于创建具有变化剪切平面的动画时很有用。

- `BoundingBox`

> 重新计算场景的边界框（正常情况下，它只在新模型实体被添加或文件被包含或合并之后计算）。边界框通过以下方式计算：
> 
> 1. 如果经过网格划分（例如，至少一个网格节点），边界框将会取为包含所有网格点的框(box)。
> 
> 2. 如果没有网格划分，但是有几何体（例如，至少一个几何点），边界框将会取为包含所有几何点的框。
> 
> 3. 如果没有网格划分和几何体，但是有一些后处理视图，边界框将会取为包含所有视图中原语(primitives)的框。
> 
> 这一操作会触发CAD模型和内置Gmsh模型的同步。

- `BoundingBox { expression, expression, expression, expression, expression, expression}`

> 强制设置场景的边界框为指定表达式（X最小值，X最大值，Y最小值，Y最大值，Z最小值，Z最大值）。注意到坐标的顺序不同于模型实体的`BoundingBox`命令：参见[浮点表达式](https://gmsh.info/doc/texinfo/gmsh.html#Floating-point-expressions)。

- `Delete Model`

> 删除当前模型（所有模型实体和它们相关的网格）。

- `Delete Meshes`

> 删除当前模型的全部网格。

- `Delete Physicals`

> 删除全部物理组

- `Delete Variables`

> 删除全部表达式

- `Delete Options`

> 删除当前选项并恢复到默认值。

- `Delete string`

> 删除表达式`string`

- `Print string-expression`

> 使用当前`Print.Format`（参见[Gmsh选项](https://gmsh.info/doc/texinfo/gmsh.html#General-options)）在名为string-expression的文件中打印图像窗口。如果*字符串表达式*中的路径不是绝对的，*字符串表达式*将会添加到当前文件的路径。这个操作触发CAD模型和内置Gmsh模型的同步。

- `Sleep expression`

> 暂停Gmsh运行`expression`秒

- `SystemCall string-expression`

> 执行（阻塞）系统调用

- `NonBlockingSystemCall string-expression`

> 执行（非阻塞）系统调用

- `OnelabRun ( string-expression <, string-expression> )`

> 运行ONELAB客户端（第一个参数是客户端名称，第二个可选才是是命令行）

- `SetName string-expression`

> 改变当面模型的名称

- `SetFactory ( string-expression )`

> 改变当前几何内核（即，确定用于后续所有几何命令的CAD内核）。当前可用内核”内置“和”OpenCASCADE“

- `SyncModel`

> 强制将老几何模型数据库立即转移到新的几何模型（正常情况，此传输在文件阅读之后立即发生）。

- `NewModel`

> 创建一个新的当前模型。

- `Include string-expression`

> 在输入文件的当前位置包含名为`string-expression`的文件。包含命令需要单独占一行。如果*字符串表达式*中路径不是绝对的，*字符串表达式*会被添加到当前文件路径之后。

## 5.2 几何脚本命令

内置内核和OpenCASCADE的CAD内核都可以在脚本语言中在几何脚本命令之前分别通过指定`SetFactory("build-in")`或`SetFactory("OpenCASCADE")`来使用。如果`SetFactory`未被指定，将会使用内置内核。

可以用自下而上的边界表示法，首先定义点（使用`Point`命令），然后是曲线（使用例如`Line`,`Circle`,`Spline`, ..., 命令或通过挤压点），然后是表面（使用例如`Plane`, `Surface`或`Surface`命令，或通过挤压曲线），最后是体积（使用`Volume`命令或通过挤压表面）。然后实体可以通过多种方式操作，例如使用`Translate`, `Rotate`, `Scale`或`Symmetry`命令。它们可以用`Delete`命令删除，前提是没有高阶实体引用它们。在OpenCASCADE内核下，可以使用额外的布尔运算：`BooleanIntersection`, `BooleanUnion`, `BooleanDifference`和`BooleanFragments`。

接下来的小节描述所有脚本语言中可用的几何命令。请注意，定义模型实体需要遵循以下一般规则：如果*表达式*定义了新实体，则将其括在圆括号中。如果*表达式*引用先前定义的实体，则将其括在方括号中。

- 点

- 曲线

- 表面

- 体积

- 挤压

- 布尔运算

- 变换

- 其他几何命令

### 5.2.1 点

- `Point ( expression ) = { expression, expression, expression <, expression> }`

> 创建一个点，圆括号内的*表达式*是点的标签；右边的花括号内前三个*表达式*给出欧几里和空间内三维中的点的X，Y和Z坐标；可选的最后一个*表达式*设置该点规定的网格元素尺寸。参见[指定网格元素尺寸](https://gmsh.info/doc/texinfo/gmsh.html#Specifying-mesh-element-sizes)，以获取关于如何在网格划分过程中使用改值的更多信息。

- `Physical Point ( expression|string-expression <, expression> ) <+|->= { expression-list }`

> 创建一个物理点。圆括号内部的*表达式*是物理点的标签；右边的*表达式列表*应该包含所有物理点内需要被分组的基本点的标签。如果圆括号内给定的是*字符串表达式*而非*表达式*，则字符标签与物理标签相关联，可以显式提供（在逗号后）或不提供（这种情况下，将会自动创建唯一标签）。

### 5.2.2 曲线

- `Line ( expression ) = { expression, expression }`

> 创建一个直线段。圆括号内的*表达式*是线段的标签；右边的花括号内的两个*表达式*给出线段起点和终点的标签。

- `Bezier ( expression ) = { expression-list }`

> 创建一个贝塞尔曲线。*表达式列表*包含控制点的标签。

- `BSpline ( expression ) = { expression-list }`

> 创建三次B样条曲线（cubic BSpline）。*表达式列表*包含控制点的标签。如果第一个和最后一个点相同，则创建一个周期曲线。

- `Spline ( expression ) = { expression-list }`

> 创建一个穿过表达式列表中点的样条曲线。使用内置几何内核时它构建一个Catmull-Rom样条曲线。使用OpenCASCADE内核时它构建一个C2 B样条曲线。如果第一个和最后一个点相同，则创建一个周期曲线。

- `Circle ( expression ) = { expression, expression, expression <, ...> }`

> 创建一个圆弧。如果右边提供三个*表达式*，则它们定义圆弧的起点、圆心和终点。使用内置内核时，圆弧应该严格小于Pi。使用OpenCASCADE内核，如果提供了第4到第6之间*表达式*，前三项提供了圆心的坐标，接下来的一个定义了半径，接下来两个可选参数提供了起点角度和终点角度。

- `Ellipse ( expression ) = { expression, expression, expression <, ...> }`

> 创建一个椭圆弧，如果右边提供四个*表达式*，则它们定义起点、中心，任意一个一个主轴上的点和终点。如果第一个点是主轴点，第三个表达式可以被省略。使用OpenCADCASE内核时，如果提供了第5到第7个*表达式*，前三个定义了中心，接下来的两个定义长半径（沿着x轴）和短半径（染着y轴），再接下来的两个提供了起点角度和终点角度。请注意，OpenCASCADE不允许创建椭圆弧时长半径小于短半径。

- `Compound Spline | BSpline ( expression ) = { expression-list } Using expression`

> 从`expression-list`中的曲线上的取样控制点创建样条曲线或B样条曲线。`Using`*表达式*指定每条曲线上计算采样点的区间数。复合样条曲线和B样条曲线仅适用于内置几何内核。

- `Cruve Loop ( expression ) = { expression-list }`

> 创建一个有向曲线环，即一条闭合的路径。圆括号内的*表达式*是曲线环的标签；右边的*表达式列表*应该包含组成曲线环的全部曲线的标签。曲线环必须是闭环。使用内置几何内核时，曲线必须有序并且有向，使用负数标签可以指定相反方向。（如果朝向是正确的，但是顺序有误，Gmsh实际上会在内部重新排序列表以建立一致的环；内置内核也支持在`Curve Loop`中使用多曲线环（或子环），但是并不建立这样做）。使用OpenCASCADE内核时，曲线环始终朝向第一个曲线的方向；可以指定负数标签来兼容内置内核，但会被忽略。曲线环可以用来创建曲面：参见[表面](https://gmsh.info/doc/texinfo/gmsh.html#Surfaces)。

- `Wire ( expression ) = { expression-list }`

> 创建一条由曲线构成的路径。`Wire`仅适用于OpenCASCADE内核。它们用来创建`ThruSections`和沿着路径挤压。

- `Physical Curve ( expression|string-expression <, expression> ) <+|-> = { expression-list }`

> 创建一条物理曲线。圆括号内的*表达式*是物理曲线的标签；右边的*表达式列表*应该包含需要在物理曲线内分组的所有基本曲线的标签。如果圆括号内给出*字符串表达式*而不是*表达式*，字符串标签会和物理标签（可以显式给出（在逗号后）或不给出（这种情况下会自动创建一个为标签））相关联。在某些网格划分文件格式中（例如，MSH2），在表达式列表内指定负数标签会反转保存的网格划分文件内该网格元素所属的相应基本曲线的方向。

### 5.2.3 表面

- `Plane Surface ( expression ) = { expression-list }`

> 创建一个平面表面。圆括号内的*表达式*是平面表面的标签；右边的*表达式列表*应该包含所有定义该表面的曲线环的标签。第一个曲线环定义表面的外部边界；所有其他曲线环定义表面内的孔。曲线环定义孔不应有任何一条曲线和外曲线环公用（这种情况下，它不是孔，并且两个表面应该分开定义）。同样地，曲线环定义孔不应有任何一条曲线和同表面内的另外一个定义孔的曲线环公用（这种情况下，两个曲线环应该结合）。

- `Surface ( expression ) = { expression-list } < In Sphere { expression }, Using Point { expression-list } >`

> 创建一个表面填充。使用内部内核时，第一个曲线环应该由三或四条曲线组成，表面采用超限插值构建(transfinite interpolation)，并且可选的`In Sphere`参数强制强制表面成为球面（额外的参数给出球心的标签）。使用OpenCASCADE内核时，通过优化构造 B 样条曲面来匹配边界曲线，以及使用`Using Point`后提供的（可选的）点。

- `BSpline Surface ( expression ) = { expression-list }`

> 创建一个B样条表面填充。仅可提供由 2、3 或 4 条 BSpline 曲线构成的单个曲线环。B样条曲面也适用于OpenCASCADE内核。

- `Bezier Surface ( expression ) = { expression-list }`

> 创建一个贝塞尔曲面填充。仅可提供由 2、3 或 4 条 BSpline 曲线构成的单个曲线环。贝塞尔曲面也适用于OpenCASCADE内核。

- `Disk ( expression ) = { expression-list }`

> 创建一个圆盘。当右侧提供四个表达式（中心和半径的 3 个坐标）时，圆盘为圆形。第五个表达式定义沿 Y 方向的半径，从而形成椭圆形。圆盘仅适用于 OpenCASCADE 内核。

- `Rectangle ( expression ) = { expression-list }`

> 创建一个矩形。前三个表达式定义左下角；接下来的两个定义宽和高。如果提供了第6个表达式，则它定义了矩形的圆角半径。`Rectangle` 仅适用于OpenCASCADE内核。

- `Surface Loop ( expression ) = { expression-list } < Using Sewing >`

> 创建一个表面环路(surface loop)（一个壳(shell)）。圆括号内的*表达式*是表面环路的标签；右边的*表达式列表*应该包含组成表面环路的全部表面的标签。表面环路总是表示闭合的壳，并且表面应该方向一致（使用负数标签来指定反方向）。（表面环路常用来创建体积：参见[体积](https://gmsh.info/doc/texinfo/gmsh.html#Volumes)。）使用OpenCASCADE内核时，可选的`Using Sewing`参数允许创建构成壳的表面共用几何同一（但是拓扑不同）的曲线。

- `Physical Surface ( expression | string-expression <, expression> <+|-> = { expression-list } )`

> 创建一个物理表面。圆括号内的*表达式*是物理表面的标签；右边的表达式列表应该包含所有需要分组到物理表面的基本表面的标签。如果圆括号内给出的是*字符串表达式*而不是*表达式*，则字符串标签会和物理标签相关联，后者可以显式提供（在逗号后）或不提供（这种情况下会自动创建唯一标签）。在一些网格划分文件格式（例如MSH2）中，在表达式列表中指定负数标签会反转保存在网格划分文件内属于相应基本表面的网格元素的方向。

### 5.2.4 体积

- `Volume ( expression ) = { expression-list }`

> 创建一个体积。圆括号内的*表达式*是体积的标签；右边的*表达式列表*应该包含定义该体积的全部表面环路的标签。第一个表面环路定义体积的外边界；所有其他的表面环路定义体积内的孔。表面环路定义孔时，不应与外部表面环路公用表面（这种情况下不是孔，并且两个体积需要分开定义）。同样，表面环路定义孔时，不应与相同体积内的其他定义孔的表面环路公用表面（这种情况下两个表面环路可以组合）。

- `Sphere ( expression ) = { expression-list }`

> 创建一个体积，由其中心的三维坐标和半径定义。额外的参数定义3个角度极限。前两个可选参数定义极角张开大小（从-Pi/2到Pi/2）。可选的“角3”定义方位角张开大小（从0到2*Pi）。`Sphere`仅适用于OpenCASCADE内核。

- `Box ( expression ) = { expression-list }`

> 创建一个方框，由一个点的三维坐标和3个伸长量定义。`Box`仅适用于OpenCASCADE内核。

- `Cylinder ( expression ) = { expression-list }`

> 创建一个圆柱，由第一个圆面的圆心的三维坐标，定义轴的向量的三个分量及其半径定义。额外一个表达式定义张角(angular opening)。`Cylinder`仅适用于OpenCASCADE内核。

- `Torus ( expression ) = { expression-list }`

> 创建一个圆环，由其中心的三维坐标和两个半径定义。额外一个表达式定义张角。`Torus`仅适用于OpenCASCADE内核。

- `Cone ( expression ) = { expression-list }`

> 创建一个锥体，由其第一个圆面的圆心的三维坐标，定义轴的向量的三个分量及其两个面的半径（半径可以为0）。额外一个表达式定义张角。`Cone`仅适用于OpenCASCADE内核。

- `Wedge ( expression ) = { expression-list }`

> 创建一个直角楔，由其直角点的三维坐标和3个伸长量定义。额外一个参数定义顶部X伸长量（默认为0）。`Wedge`仅适用于OpenCASCADE内核。

- `ThrusSections ( expression ) = { expression-list }`

> 创建一个由曲线环定义的体积（截面）。`ThruSections`仅适用于OpenCASCADE内核。

- `Ruled ThruSections ( expression ) = { expression-list }`

> 和`ThruSections`相同，但是边界上创建的表面强制被控制。`Ruled ThruSections`仅适用于OpenCASCADE内核。

- `Physical Volume ( expression|string-expression <, expression> ) <+|->= { expression-list }`

> 创建一个物理体积。圆括号内的*表达式*是物理体积的标签；右边的*表达式列表*一个包含所有需要分组到物理体积的基本体积的标签。如果圆括号内给出的是*字符串表达式*而不是*表达式*，则字符串标签会和物理标签相关联，后者可以显式提供（在逗号后）或不提供（这种情况下会自动创建唯一标签）。

### 挤压

曲线，表面和体积可以分别通过点，曲线和表面的挤压创建。这是几何挤压命令的语法（去往[结构网格](https://gmsh.info/doc/texinfo/gmsh.html#Structured-grids)，看看如何扩展这些命令以便挤压网格）。

***挤压***

- `Extrude { expression-list } { extrude-list }`

> 使用平移挤压所有*待挤压列表*的基本实体。*表达式列表*应该包含三个*表达式*给出平移向量的X，Y和Z分量。

- `Extrude { { expression-list }, { expression-list }, expression } { extrude-list }`

> 使用旋转挤压所有*待挤压列表*的基本实体。第一个*表达式列表*应该包含三个*表达式*给出旋转轴的X，Y和Z方向；第二个*表达式列表*应该包含三个*表达式*给出轴上任意一点的X，Y和Z分量；最后一个*表达式*应该包含旋转角度（弧度制）。使用内置内核时，角度必须严格小于Pi。

- `Extrude { { expression-list }, { expression-list }, { expression-list }, expression } { extrude-list }`

> 使用平移和旋转（作用效果为“扭曲”）挤压所有*待挤压列表*的基本实体。第一个*表达式列表*包含三个*表达式*给出平移向量的X，Y和Z分量；第二个*表达式列表*应该包含三个*表达式*给出旋转轴的X，Y和Z方向；第三个*表达式列表*应该包含三个*表达式*给出轴上任意一点的X，Y和Z分量；最后一个*表达式*应该包含旋转角度（弧度制）。使用内置内核时，角度必须严格小于Pi。

- `Extrude { extrude-list }`

> 使用平移沿着他们的法线挤压*待挤压列表*的实体。仅使用于内置几何内核。

- `Extrude { extrude-list } Using Wire { expression-list }`

> 沿着给出的路径挤压*待挤压列表*的实体。仅使用于OpenCASCADE几何内核。

- `ThruSections { expression-list }`

> 通过给定的曲线环或路径创建表面。`ThruSections`仅适用于OpenCASCADE内核。

- `Ruled ThruSections { expression-list }`

> 通过给定的曲线环或路径创建控制表面。`Ruled ThruSections`仅适用于OpenCASCADE内核。

- `Fillet { expression-list } { expression-list } { expression-list }`

> 使用提供的半径（第三个列表）在某些曲线（第二个列表）上对体积进行圆角处理（第一个列表）。半径列表要么包含一个半径，要么包含与曲线一样多的半径或两倍于曲线数量的半径（这种情况下曲线的起点和终点的半径将会不同）。

- `Chamfer { expression-list } { expression-list } { expression-list } { expression-list }`

> 使用在给定表面（第三个列表）上测量提供的距离（第四个列表）在某些（第二个列表）上对体积（第一个列表）进行倒角。距离列表要么包含一个距离，要么包含与曲线一样多的半径或两倍于曲线数量的距离（这种情况下每对的第一个在给定的相应表面上进行测量）。`Chamfer`仅适用于OpenCASCADE内核。

使用

```GMSH
extrude-list:
<Physical> Point | Curve | Surface { expression-list-or-all }; ...
```

就如[浮点表达式](https://gmsh.info/doc/texinfo/gmsh.html#Floating-point-expressions)中说明的那样，挤压可以在表达式中使用，这种情况下它返回一个标签的列表。默认情况下，列表包含索引 0 处的挤压实体的“顶部”和索引 1 处的挤压实体，然后是索引 2、3 等处的挤压实体的“侧面”。例如：

```GMSH
Point(1) = {0, 0, 0};

Point(2) = {1, 0, 0};

Line(1) = {1, 2};

out[] = Extrude {0, 1, 0} { Curve {1}; };

Printf("top curve = %g", out[0]);

Printf("surface = %g", out[1]);

Printf("side curves = %g and %g", out[2], out[3]);
```

这种行为可以通过Geometry.ExtrudeReturnLateralEntities选项改变（参见[几何选项](https://gmsh.info/doc/texinfo/gmsh.html#Geometry-options)）。

### 5.2.6 布尔操作

布尔操作可以在曲线，表面和体积上应用。所有布尔操作作用在两个基本实体的列表上。第一个列表表示物体(object)，第二个表示工具(tool)。对于布尔操作的基本语法如下：

布尔：

- `BooleanIntersection { boolean-list } { boolean-list }`

> 计算物体和工具的交集。

- `BooleanUnion { boolean-list } { boolean-list }`

> 计算物体和工具的集合。

- `BooleanDifference { boolean-list } { boolean-list }`

> 物体减去工具。

- `BooleanFragments { boolean-list } { boolean-list }`

> 计算对象和工具中实体交集产生的所有片段，使所有接口都具有共形性。当作用于不同维度的实体时，如果低维度实体不在高纬度实体的边界上，低维度实体自动嵌入到高维实体内。

使用

```GMSH
boolean-list:
<Physical> Curve | Surface | Volume { expression-list-or-all }; ... |
Delete
```

如果*布尔列表*里指定`Delete`，工具和/或物体会被删除。

[浮点表达式](https://gmsh.info/doc/texinfo/gmsh.html#Floating-point-expressions)中说明，*boolean*可以用于表达式，这种情况下它返回一个通过布尔操作创建的最高维实体的标签的列表。参见[示例/布尔](https://gitlab.onelab.info/gmsh/gmsh/blob/gmsh_4_13_1/examples/boolean/)。

如果预先知道作用在单个（最高维）实体上，存在一个替代的布尔操作的语法。

***boolean-explicit***

- `BooleanIntersection ( expression ) = { boolean-list } { boolean-list }`

> 计算物体和工具的交集并且分配其结果给*expression*对应的标签。

- `BooleanUnion ( expression ) = { boolean-list } { boolean-list }`

> 计算物体和工具的并集并且分配其结果给*expression*对应的标签。

- `BooleanDifference ( expression ) = { boolean-list } { boolean-list }`

> 从物体减去工具并且分配其结果给*expression*对应的标签。

同样，参见[示例/布尔](https://gitlab.onelab.info/gmsh/gmsh/blob/gmsh_4_13_1/examples/boolean/)以获取示例。

布尔操作仅适用于OpenCASCADE几何内核。

### 5.2.7 变换

几何变换可以应用于基本实体，或基本实体的复制（使用`Duplicate` 命令：请看下面）。变换命令的语法是：
***transform:***

- `Dilate { { expression-list }, expression } { transform-list }`

> 按*expression*为值的因子缩放*变换列表*内所有基本实体。*表达式列表*应该包含同类变换中心的给定的X，Y和Z坐标的三个*表达式*。

- `Dilate { { expression-list }, { expression, expression, expression } } { transform-list }`

> 按不同的因子（这三个*表达式*）分别沿着X，Y和Z缩放*变换列表*内所有基本实体。*表达式列表*应该包含同类变换中心的给定的X，Y和Z坐标的三个*表达式*。

- `Rotate { { expression-list }, { expression-list }, expression } { transform-list }`

> 按*expression*为值的弧度的角度旋转*变换列表*内所有的基本实体。第一个*表达式列表*应该包含旋转轴的给定的X，Y和Z方向的三个*表达式*；第二个*表达式列表*应该包含轴上任意一点的X，Y和Z分量的三个表达式。

- `Symmetry { expression-list } { transform-list }`

> 将所有基本实体对称变换至平面。*表达式列表*应该包含给出这个平面方程的参数的四个表达式。

- `Affine { expression-list } { transform-list }`

> 将4x4仿射变换矩阵（每行提供 16 个条目；为方便起见，只能提供 12 个）应用在所有基本实体上。现在只能OpenCASCADE内核上使用。

- `Translate { expression-list } { transform-list }`

> 变换*变换列表*中所有的基本实体。表达式列表应该包含变换向量的给定的X，Y和Z轴的三个*表达式*。

- `Boundary { transform-list }`

> 本身(pre-se)不是一种变换类型。返回在*变换列表*中的基本实体的边界上的实体，并附带符号，用以表示他们在边界上的方向。如果要获取不带符号的标签（例如，在其他命令中使用该命令输出结果），对返回的列表使用`Abs`函数。这一操作会触发CAD模型与内部Gmsh模型的同步。

- `CombinedBoundary { transform-list }`

> 本身(pre-se)不是一种变换类型。返回*变换列表*中基本实体的边界，组合起来就像一个实体。在计算一个复杂部分的边界时有用。这一操作会触发CAD模型与内部Gmsh模型的同步。

- `PointsOf { transform-list }`

> 本身(pre-se)不是一种变换类型。返回基本实体边界上的所有几何点。在计算一个复杂部分的边界时有用。这一操作会触发CAD模型与内部Gmsh模型的同步。

- `Intersect Curve { expression-list } Surface { expression }`

> 本身(pre-se)不是一种变换类型。返回*表达式列表*中给出的曲线和指定表面的交点。现在仅适用于内置内核。

- `Split Curve { expression } Point { expression-list }`

> 本身(pre-se)不是一种变换类型。用指定的控制点分割*expression*对应的曲线。仅适用于内置内核，对于线，样条曲线，B样条曲线。

使用：

```GMSH
transform-list:
  <Physical> Point | Curve | Surface | Volume
    { expression-list-or-all }; … |
  Duplicata { <Physical> Point | Curve | Surface | Volume
    { expression-list-or-all }; … } |
  transform
```

### 5.2.8 其他几何命令

如下是现在可用的所有其他几何命令的列表：

- `Coherence;`

> 移除所有重复的基本实体（例如，有相同坐标的点）。注意，使用内置内核时，在每个几何变换之后，Gmsh自动执行`Coherence`命令，除非`Geometry.AutoCoherence`被设置为零（参见[几何选项](https://gmsh.info/doc/texinfo/gmsh.html#Geometry-options)）。使用OpenCASCADE内核时，`Coherence `只是对所有实体执行 `BooleanFragments `操作的快捷方式，其中 `Delete `运算符应用于所有操作数。

- `HealShapes;`

> 依据`Geometry.OCCFixDegenerated`, `Geometry.OCCFixSmallEdges`,`Geometry.OCCFixSmallFaces`, `Geometry.OCCSewFaces`, `Geometry.OCCMakeSolids`应用形状修复程序。仅适用于OpenCASCADE几何内核。

- `< Recursive > Delete { <Physical> Point | Curve | Surface | Volume { expression-list-or-all }; … }`

> 删除所有在*expression-list-or-all*中给出标签的基本实体。如果一个实体链接到另一个实体（例如，如果一个点被用作曲线的控制点），则`Delete`无效（必须先删除曲线，然后才能删除点）。`Recursive`变体会删除实体及其所有较低维度的子实体。这一操作会触发 CAD 模型与内部 Gmsh 模型的同步。

- `Delete Embedded { <Physical> Point | Curve | Surface | Volume { expression-list-or-all }; … }`

> 删除在*expression-list-or-all*中给出标签的基本实体中的所有嵌入实体。这一操作会触发 CAD 模型与内部 Gmsh 模型的同步。

- `SetMaxTag Point | Curve | Surface | Volume ( expression )`

> 强制设定一种类别的实体的标签最大值为给定值，之后创建的相同类别的实体将不会有大于（原文此处为smaller，结合上下文认定为文本错误）给定值的标签。

- `< Recursive > Hide { <Physical> Point | Curve | Surface | Volume { expression-list-or-all }; … }`

> 隐藏*expression-list-or-all*中列出的实体。

- `Hide { : }`

> 隐藏所有实体。

- `< Recursive > Show { <Physical> Point | Curve | Surface | Volume { expression-list-or-all }; … }`

> 显示*expression-list-or-all*中列出的实体。

- `Show { : }`

> 显示全部实体。

- `Sphere | PolarSphere ( expression ) = {expression, expression};`

> 将内置几何内核使用的当前（表面）几何体更改为（极）球体，由右侧指定的两个点标签定义。左边圆括号内的*表达式*为这个新几何体指定一个新的唯一标签。

- `Parametric Surface ( expression ) = "string" "string" "string";`

> 将内置几何内核使用的当前（表面）几何体更改为一个参数表面，由三个字符串表达式定义，分别计算 X，Y和Z坐标。

- `Coordinates Surface expression;`

> 将内置几何内核使用的当前（表面）几何体更改为标识为给定*表达式*的几何体。

- `Euclidian Coordinates;`

> 存储给内置几何内核的默认平面几何。

## 5.3 网格划分脚本命令

网格划分模块脚本语言允许修改网格划分元素尺寸和指定结构化网格参数。特定的网格划分行为（例如，“划分所有表面”）也可以在脚本文件中指定，但是通常在GUI或命令行中运行。（参见[Gmsh图形用户界面](https://gmsh.info/doc/texinfo/gmsh.html#Gmsh-graphical-user-interface/)和[Gmsh命令行界面](https://gmsh.info/doc/texinfo/gmsh.html#Gmsh-command_002dline-interface)）。

- 网格元素尺寸

- 结构化网格

- 其他网格划分命令

### 5.3.1 网格元素尺寸

如下这些网格划分命令和网格元素尺寸的指定有关：

- `MeshSize { expression-list } = expression;`

> 修改标签在*表达式列表*中的点的预定网格元素尺寸。新值由*表达式*给出。

- `Field[expression] = string;`

> 创建一个新的字段（其标签为*表达式*的值），类型为*字符串*

- `Field[expression].string = string-expression | expression | expression-list;`

> 设定第*expression*个选项*string*的字段。

- `Background Field = expression;`

> 选择第*expression*个字段用作计算元素尺寸。只能给出一个背景字段；如果你想组合几个字段，使用`Min`或`Max`字段（参见下面的内容）。

### 5.3.2 结构化网格

- `Extrude { expression-list } { extrude-list layers }`

> 同时使用变换挤压几何体和网格划分（参见[挤压](https://gmsh.info/doc/texinfo/gmsh.html#Extrusions)）。`layers`选项决定网格是如何挤压的，该命令具有如下的语法：
> 
> ```GMSH
> layers:
> ```
> 
> ```GMSH
>   Layers { expression } |
>   Layers { { expression-list }, { expression-list } } |
>   Recombine < expression >; …
>   QuadTriNoNewVerts <RecombLaterals>; |
>   QuadTriAddVerts <RecombLaterals>; ...
> ```
> 
> 在第一个`Layers`形式中，*expression*给出在该（单）层创建的元素的数量。在第二个形式中，第一个*表达式列表*定义在每个挤压层应该创建多少个元素，第二个*表达式列表*给出每一层的归一化高度（列表应该包含n个数字的序列 0 < h1 < h2 < ... <= 1）. 参见[t3](https://gmsh.info/doc/texinfo/gmsh.html#t3)的一个示例。
> 
> 对于曲线挤压，可能时，`Recombine`选项会重新组合三角形为四边形。对于表面挤压，`Recombine`选项会重新组合四面体为三棱柱，六面体或金字塔形。
> 
> 请注意，从Gmsh2.0开始，区域标签不能再*Layers*命令中显示指定了，取而代之的是，和其他几何命令一样，你必须使用由挤压命令自动创建的实体标识符。例如，下面的挤压命令返回`num[0]`中最”上层”的表面的标签和`num[1]`中新体积的标签：
> 
> ```GMSH
> num[] = Extrude {0,0,1} { Surface{1}; Layers{10}; };
> ```
> 
> 
