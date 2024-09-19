# 4 Gmsh命令行界面

Gmsh定义了许多命令行开关，可用于从命令行以“批处理”模式控制Gmsh，并传递选项，而无需使用脚本（参见[Gmsh脚本语言](./gmsh_scripting_language.md)）或API（参见[Gmsh应用程序接口](./gmsh_application_programming_interface.md)）。

例如，在批处理模式下划分第一篇教程的网格可以在终端中通过传递“-2”命令行开关执行该命令：

> gmsh t1.geo -2

同样的效果也可以通过在“t1.geo”后添加添加“Mesh 2”命令实现，然后运行：

> gmsh t1.geo -parse_and_exit

或者在脚本末尾添加 Exit; 命令，然后直接打开这个新文件：

> gmsh t1.geo

请注意，所有数字和字符串选项（参见 [Gmsh选项](./gmsh_options.md)）都可以通过 -setnumber 和 -setstring 开关在命令行中设置

> gmsh t1.geo -setnumber Mesh.Nodes 1 -setnumber Geometry.SurfaceLabels 1

接下来给出所有命令行开关的列表

（括号内为相关选项名称（如果有））



#### 几何

- -0

> 输出模式，然后退出

- -tol value

> 设置几何公差(Geometry.Tolerance)

- -match

> 匹配几何图形和网格

#### 网格

- -1, -2, -3

> 运行1D, 2D 或 3D网格生成，然后退出

- -format string

> 选择输出网格格式：auto、msh1、msh2、msh22、msh3、msh4、msh40、msh41、msh、unv、vtk、wrl、mail、stl、p3d、mesh、bdf、cgns、med、diff、ir3、inp、ply2、celum、su2、x3d、dat、neu、m、key、off、rad (Mesh.Format)

- -bin

> 当可能时创建二进制文件（Mesh.Binary）

- -refine

> 运行均匀网格细化，然后退出






