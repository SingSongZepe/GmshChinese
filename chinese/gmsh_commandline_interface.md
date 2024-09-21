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

- -barycentric_refine

> 执行重心网格细化，然后退出

- -reclassify angle

> 重新分类表面网格，然后退出

- -reparam angle

> 重新参数化表面网格，然后退出

- -hybrid

> 生成带有三面体的混合六面体网格以进行过渡

- -part int

> 批量网格生成后的分区（Mesh.NbPartitions）

- -part_weight [tri, quad, tet, hex, pri, pyr, trih] int

> 分割过程中三角形、四边形等的权重（Mesh.Partition[Tri,Quad,...]Weight）

- -part_split

> 将网格分区保存在单独的文件中

- -part_[no_]topo

> 创建分区拓扑（Mesh.PartitionCreateTopology）

- -part_[no_]ghosts

> 创建幽灵区块（Mesh.PartitionCreateGhostsCells）

- -part_[no]physicals

> 为分区创建物理组（Mesh.PartitionCreatePhysicals）

- -part_topo_pro

> 保存分区拓扑.pro文件（Mesh.PartitionTopologyFile）

- -preserve_numbering_msh2

> 保留MSH2格式的元素编号（Mesh.PreserveNumberingMsh2）

- -save_all

> 保存所有元素（Mesh.SaveAll）

- -save_parametric

> 保存节点和其参数坐标（Mesh.SaveParametric）

- -save_topology

> 保存模型拓扑（Mesh.SaveTopology）

- -algo string

> 选择网格划分算法：自动（auto），网格自适应（meshadapt），del2d, front2d, delquad, quadqs, initial2d, del3d, front3d, mmg3d, hxt, inital3d（Mesh.Algorithm 和 Mesh.Algorithm3D）

- -smooth int

> 设置网格平滑步骤数（Mesh.Smoothing）

- -order int

> 设置网格阶数（Mesh.ElementOrder）

- -optimize[\_netgen]

> 优化四面体质量（Mesh.Optimize[Netgen]）

- -optimize_threshold

> 优化质量小于阈值的四面体元素

- -optimize_ho

> 优化高阶网格（Mesh.HighOrderOptimize）

- -ho_[min, max, nlayers]

> 高阶优化参数（Mesh.HighOrderThreshold[Min, Max], Mesh.HighOrderNumLayers）

- -clscale value

> 设置网格元素尺寸因子（Mesh.MeshSizeFactor）

- -clmin value

> 设置最小网格元素尺寸（Mesh.MeshSizeMin）

- -clmax value

> 设置最大网格元素尺寸（Mesh.MeshSizeMax）

- -clextend value

> 从边界拓展网格元素大小（Mesh.MeshSizeExtendFromBounday）

- -clcurv value

> 根据曲率计算网格元素大小，值为2*pi弧度的目标元素数（Mesh.MeshSizeFromCurvature）

- -aniso_max value

> 设置bamg的最大各向异性（Mesh.AnisoMax）

- -smooth_ratio value

> 为 bamg 设置同一边缘节点的网格大小之间的平滑比率 (Mesh.SmoothRatio)

- -epslcld value

> 设置一维网格的网格尺寸字段评估精度（Mesh.LcIntegrationPrecision）

- -swapangle value

> 设置两个相邻面之间的阈值角度（以度为单位），低于该角度时允许交换（Mesh.AllowSwapAngle）

- -rand value

> 设置随机扰动因子（Mesh.RandomFactor）

- -bgm file

> 从文件加载背景网格

- -check

> 对网格进行各种一致性检查

- -ignore_periocity

> 忽略周期性边界 (Mesh.IgnorePeriodicity)

#### 后处理

- -link int

> 选择视图之间的链接模式（PostProcessing.Link）

- -combine

> 将名称相同的视图合并为多时间步骤视图

#### 求解器

- -listen string

> 持续监听来自给定的接口（uses Solver.SocketName）的连接请求（Solver.AlwaysListen）

- -minterpreter string

> Octave 解释器名称（Solver.OctaveInterpreter）

- -pyinterpreter string

> Python 解释器名称（Solver.OctaveInterpreter）

- -run

> 运行 ONELAB 求解器

#### 展示

- -n

> 启动时隐藏全部网格和后处理视图（View.Visible, Mesh.[Points, Lines, SufaceEdges]）

- -nodb

> 禁用双缓存（General.DoubleBuffer）

- -numsubedges

> 设置高阶元素显示的子网个数（Mesh.NumSubEdges）

- -fontsize int

> 指定图形用户界面的字体大小（General.FontSize）

- -theme string

> 指定FLTK图形用户界面主题（General.FltkTheme）

- -display string

> 指定展示（General.Display）

- -camera

> 使用相机模式视图（General.CameraMode）

- -stereo

> OpenGL 四缓冲立体渲染（General.Stereo）

- -gamepad

> 使用游戏手柄控制器（如果有）

#### 其他

- -parse_and_exit

> 解析输入文件，然后退出

- -save

> 保存输出文件，然后退出

- -o file

> 指定输出文件名称

- -new

> 在合并下一个文件前创建新模型

- -merge

> 合并接下来的文件

- -open

> 打开接下来的文件

- -log filename

> 以日志方式打印所有消息到文件名

- -a, -g, -m, -s, -p

> 以自动、几何、网格、求解器或后处理模式启动（General.InitialModule）

- -pid

> 打印进程id到标准输出

- -watch pattern

> 可用(available)时要合并的文件模式（General.WatchFilePattern）

- -bg file

> 加载背景（图像或PDF）文件（General.BackgroundImageFileName）

- -v int

> 设置冗长程度（General.Verbosity）

- -string "string"

> 在启动时解析命令字符串

- -setnumber name value

> 设置常量，ONELAB 或数字选项 name = value

- -setstring name value

> 设置常量，ONELAB或字符串选项 name = value

- -nopopup

> 不在脚本中弹出对话窗口（General.NoPopup）

- -noenv

> 启动时不要修改环境

- -nolocale

> 启动时不要修改本地语言

- option file

> 启动时解析选项文件

- -convert files

> 将文件转换为最新的二进制格式，然后退出

- -nt int

> 设置线程数（General.NumThreads）

- -cpu

> 为任何操作报告CPU时间

- -version

> 展示版本数

- -info

> 展示更多细节版本信息

- -help

> 展示命令行用法

- -help_options

> 展示所有选项
