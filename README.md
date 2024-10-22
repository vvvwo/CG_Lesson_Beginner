# 计算机图形学初级课程

This is a computer graphics course for entry-level learner. Course designed by [Assistant Prof. Chenlei Lv](https://aliexken.github.io/),
Shenzhen University, with assistance from [VCCLab](https://vcc.tech/index.html) and [CSSE](https://csse.szu.edu.cn/).

## 前言

本课程面向计算机图形学初学者，以帮助他们尽快了解图形学渲染管线，几何变换，图形接口编程等基础知识。学习该课程最好需要具备C++编程、数据结构、线性代数等前置知识，这会大大提升学习效率。当然，我会尽可能在课程中增加相关概念的解释。

针对计算机图形学教学，大部分教材与课程设置，默认以OpenGL作为学习对象，以完成对图形学内容的解释和实践。遵从传统，我们使用OpenGL3.0+作为图形编程接口，IDE默认为VS平台，以方便C++程序开发和后期扩展。这样一方面能够兼容目前主流的教学案例，另外一方面能够方便初学者了解Shader、体渲染、GPU粒子动画等新特性。包括Vulkan, DirectX等图形接口，在基础架构上是和OpenGL保持一致的。因此掌握OpenGL编程，可以非常平顺的迁移到其他接口平台。一些关于OpenGL配置以及开源库使用的说明可参看如下博客：

 1. [VS2019+GLFW+GLAD环境配置](https://blog.csdn.net/aliexken/article/details/110656551)
 2. [VCGLib环境配置](https://blog.csdn.net/aliexken/article/details/119331112)
 3. [Eigen环境配置](https://blog.csdn.net/aliexken/article/details/125539709)

如果你对上述内容不甚了解，不用担心，我会在正式的课程中融入对各种库配置的说明，用以辅助学习。

## 课程内容

根据计算机图形学初级课程的要求，以上机实验为主，课程内容包括：

 1. [OpenGL环境配置](https://github.com/vvvwo/CG_Lesson/tree/main/Lesson1)
 2. [Shader编程入门](https://github.com/vvvwo/CG_Lesson/tree/main/Lesson2)
 3. [键盘与鼠标交互](https://github.com/vvvwo/CG_Lesson/tree/main/Lesson3)
 4. [三维模型变换](https://github.com/vvvwo/CG_Lesson/tree/main/Lesson4)
 5. [三维模型读取与显示]()
 6. [相机定位与投影]()
 7. [光照模型]()
 8. [纹理映射]()
 9. [课程大作业]()

该课程参考了[learnopengl-CN](https://learnopengl-cn.github.io/)的部分内容，有兴趣的同学也可以浏览该站点学习OpenGL编程知识。我希望在经过这门课的学习之后，学生可以建立对渲染管线的基本认知，能够利用渲染管线进行基础的图形学编程，能够理解光照模型以及纹理映射的实现原理，能够通过程序设计完成对一个三维模型的数据提取与简单编辑任务。我的主要研究方向是几何处理，犹豫再三，还是把我比较熟悉的网格和点云的处理内容移除了该课程。主要原因是该课程的重点还是围绕对渲染管线的理解来展开的。有兴趣的同学可以follow我即将开设的计算机图形学中级课程，我会介绍更多的相关内容，包括网格点云处理，GPU并行加速以及CG与深度网络技术的融合。

## 写在最后

我自2012年开始从事科学研究以来，就专注在CG这个领域。我为CG中那些酷炫的效果和优雅的算法所着迷！我认为CG是计算机应用技术领域最酷炫的研究反向，没有之一。不过，受限于我的个人能力，我对CG的认知与学习依然停留在比较初级的阶段，但这不会影响我对这门学问的喜爱。
CG的学习路径是辛苦的，需要许多前置知识，学习曲线陡峭。但我相信，只要持之以恒，坚持学习，我敢担保你会爱上这门学问。欢迎那些对CG充满热情的学生、研究者以及工程师，加入到CG队伍中，共享CG的魅力！
