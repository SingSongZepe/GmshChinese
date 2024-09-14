# 1 Gmsh概述

Gmsh是一个内置CAD引擎和后处理器的三维有限元网格生成器。它的设计目标是提供一个快速、轻量并且用户友好式的网格划分工具，并且参数化输入和具有灵活的可视化功能。

Gmsh围绕四个模块（几何、网格、求解器和后处理）构建，可以通过用户图形界面（GUI，参见[Gmsh图形用户界面](./gmsh_graphical_user_interface.md)）、命令行（参见[Gmsh命令行界面](gmsh_commandline_interface.md)）、使用Gmsh自己的脚本语言编写的文本文件（.geo文件，参见[Gmsh脚本语言](gmsh_scripting_language.md)）或通过C++、C、Python、Julia和Fortran应用程序接口（API，参见[Gmsh应用程序接口](gmsh_application_programming_interface.md)）。

下面将对这四个模块进行简要介绍，然后概述Gmsh最擅长的功能（...以及不太擅长的功能），并提供一些如何在您的计算机上安装和运行Gmsh的实践信息。

- 几何模块
- 网格模块
- 求解器模块
- 后处理模块
- Gmsh很擅长的方面
- Gmsh不擅长的方面
- 在你的计算机上安装并运行Gmsh

## 1.1 几何模块

Gmsh中的**模型**使用其**边界表示**(BRep)进行定义：**体积**用一组**表面**界定，**表面**由一组**曲线**界定，**曲线**由两个**端点**界定。模型实体是拓扑实体，即仅处理模型中的邻接关系，并作为一组抽象拓扑类实体实现。BRep通过嵌入或内部模型实体的定义进行拓展：内部点、曲线和表面可以嵌入体积中，并且内部点和曲线也可以嵌入表面中。

模型实体的几何形状可以被不能的CAD内核提供。Gmsh使用的两个默认内核是**内置内核**和**OpenCASCADE内核**。Gmsh不会将几何表示从一个内核转化到另外一个内核，也不会将这下内核的集合表示转化成中性表示。相反，Gmsh直接查询每个CAD内核的原始数据，从而避免数据丢失，这对于复杂模型来说至关重要，因为转化会不可避免地引入与略有不同的表示相关的问题。在.geo脚本中选择CAD内核是由SetFactory命令完成的（参见[几何脚本命令](https://gmsh.info/doc/texinfo/gmsh.html#Geometry-scripting-commands)），而在Gmsh API中，内核明确出现在gmsh/model命名空间的所有相关函数中，内置内核和OpenCASCADE内核分别带有geo或occ前缀（参见[命名空间gmsh/model](https://gmsh.info/doc/texinfo/gmsh.html#Namespace-gmsh_002fmodel)）。

实体既可以使用内置内核和OpenCASCADE内核以自下而上的方式（先是点，然后曲线、表面和体积）构建，也可以使用OPenCASCADE内核以自上而下的方式构造实体几何（在其上执行布尔运算的实体）。这两种方法还可以结合着使用。最后，根据基本几何实体定义模型实体组（称为“物理组”）。（获取更多关于物理组如何影响网格保存方式的信息，参见[基本实体和物理组](https://gmsh.info/doc/texinfo/gmsh.html#Elementary-entities-vs-physical-groups)）。

模型实体（也称为“基本实体”）和物理组都被一对整数唯一定义：它们的维度（0是点，1是曲线，2是表面，3是体积）和标签（严格为正的全局标识号）。实体和组的标签在每个维度上都是唯一的：

1. 每个点必须具有一个唯一标签。
2. 每条曲线必须具有一个唯一标签。
3. 每个表面必须具有一个唯一标签。
4. 每个体积必须具有一个唯一标签。

零和负数标签被Gmsh保留用于内部使用。

模型实体可以在几何模块中以多种方式操作和转化，但是操作始终直接在各自的CAD内核中执行。如上所述，没有通用的几何表示：Gmsh直接使用内核各自的API对原生几何表示执行操作（平移、旋转、交集、联合、片段等）。按照相同的理念，可以通过每个 CAD 内核自己的导入机制将模型导入几何模块。例如，默认情况下，Gmsh通过OpenCASCADE导入STEP和IGES文件，这将导致创建具有内部OpenCASCADE表示的模型实体。使用内置CAD内核表示的模型可以通过将其到处为.geo_unrolled文件序列化到磁盘，而使用OpenCASCADE内核构建的模型可以序列化为.brep或.xao文件。

Gmsh教程从[t1](https://gmsh.info/doc/texinfo/gmsh.html#t1)开始，是学习如何使用几何模块的最佳场所：它包含基于内置和OpenCASCADE内核不断提升复杂性的示例。请注意，几何模块的许多功能可以在GUI中交互使用（参见[Gmsh图形用户界面](./gmsh_graphical_user_interface.md)），这也是了解Gmsh脚本语言和API的好途径，因为几何模块中的操作会自动附加相关命令到输入输入脚本文件中，并且还可以选择为API支持的脚本语言生成输入（参见General.ScriptingLanguages 项；从Gmsh 4.12开始仍在进行中。）

除了CAD类型的几何实体（其几何形状由CAD内核提供）外，Gmsh还支持由网格（例如STL）定义的**离散**模型实体。Gmsh不会对此类离散实体几何操作，但可以通过所谓的[“重新参数化”程序](https://gmsh.info/doc/texinfo/gmsh.html#FOOT1)$^1$为它们配备几何形状。然后参数化将用于网格划分，其方式与CAD实体完全相同。有关示例，参见[t13](https://gmsh.info/doc/texinfo/gmsh.html#t13)

## 1.2 网格模块

模型的有限元网格是其几何形状的细分，由各种简单的几何元素组成（Gmsh中，线，三角形，四边形，四面体，棱柱，六面体和金字塔），这些元素的排列方式是，如果其中两个元素相交，它们会沿着面、边或节点相交，而不会是其他情况。这定义了所谓的保形网格。网格模块实现了几种算法来自动生成这种网格。默认情况下，Gmsh生成的网格被视为非结构化的，即使它们以结构化方法生成（例如通过挤压）。这意味着网格元素完全由其节点的有序列表定义，并且任何两个元素之间不存在预定义的排序关系。

为了确保网格的一致性，网格生成采用自下而上的流程：首先将曲线离散化；然后使用曲线对表面进行划分；然后使用表面对体积进行划分。在此过程中，一个实体的网格仅受其边界网格的限制，除非低维实体明确嵌入高维实体中。例如，在三维中，只有当表面是体积的边界的一部分或表面以及明确嵌入体积中，该离散化表面的三角形才会在最终的3D网格中被强制成为四面体的面。

这样可以自动确保网格的一致性，例如，两个体积共用一个表面，网格元素根据底层实体的几何方向进行定向。每个网格划分步骤都受**网格尺寸场**限制，它规定了网格中元素的所需尺寸。此尺寸场可以是均匀的，由与几何中的点相关的值指定，或一般网格尺寸场定义（例如到某个边界的距离、到另外一个网格上定义的任意标量场的距离相关）：参见[Gmsh网格尺寸场](./gmsh_mesh_size_fields.md)。对于每个网格划分步骤，首先执行所有结构化网格指令，并且作为非结构化部分的附加约束。（保形网格的生成和处理对网格在Gmsh中的内部存储方式以及如何通过API访问网格具有重要影响：参见[Gmsh应用程序接口](./gmsh_application_programming_interface.md)）。

Gmsh的网格模块重组了几种一维、二维和三维网格划分算法：

- 二维**非结构化**算法生成三角形和/或四边形（当使用了重组命令或选项时）。三维**非结构化**算法生成四面体或四面体和金字塔形（当边界网格存在四边形时）。
- 二维**结构化**算法（超限和挤压）默认生成三角形，但可以通过重组命令或选项获得四边形。三维**结构化**算法生成四面体、六面体、棱柱和金字塔形，具体取决于它们所基于的表面网格类型。

所有的网格可以使用Mesh.SubdivisionAlgorithm选项进行细分，以生成完全四边形或完全六面体网格（参见[网格选项](https://gmsh.info/doc/texinfo/gmsh.html#Mesh-options)）。

- [选择正确的非结构化算法](https://gmsh.info/doc/texinfo/gmsh.html#Choosing-the-right-unstructured-algorithm)
- [指定网格划分元素的大小](https://gmsh.info/doc/texinfo/gmsh.html#Specifying-mesh-element-sizes)
- [基本实体与物理组](https://gmsh.info/doc/texinfo/gmsh.html#Elementary-entities-vs-physical-groups)

### 1.2.1 选择正确的非结构化算法

Gmsh提供了几种二维和三维非结构化算法可供选择。每种算法都有自己的优缺点。

对于所有的二维非结构化算法，首先使用[分治算法](https://gmsh.info/doc/texinfo/gmsh.html#FOOT2)$^2$构建包含一维网格的所有点的Delaunay网格。使用[边交换](https://gmsh.info/doc/texinfo/gmsh.html#FOOT3)$^3$恢复缺失边，初始化步骤之后，可以使用多种算法来生成最终网格：

- [&#34;MeshAdapt&#34;（网格适应）算法](https://gmsh.info/doc/texinfo/gmsh.html#FOOT4)$^4$基于局部网格修改。该技术利用了边交换、分割和折叠：长边被分割，短边被折叠，如果获得更好的几何配置，则边交换。
- “Delaunay”算法的灵感来源于[INRIA](https://gmsh.info/doc/texinfo/gmsh.html#FOOT5)$^5$的GAMMA团队的工作。新点按顺序插入到具有最大无量纲外接圆半径的元素的外接圆心。然后使用各向异性的Delaunay标准来重新连接网格。
- “Frontal-Delaunay”算法的灵感来源于[S. Rebay](https://gmsh.info/doc/texinfo/gmsh.html#FOOT6)$^6$的工作。
- 还有其他具有特定功能的实验算法可用，特别是[“Frontal-Delaunay for Quads”算法](https://gmsh.info/doc/texinfo/gmsh.html#FOOT7)$^7$是"Frontal-Delaunary"算法的变体，旨在生成重组的直角三角形；而[“BAMG”算法](https://gmsh.info/doc/texinfo/gmsh.html#FOOT8)$^8$允许生成各向异性的三角剖分。
