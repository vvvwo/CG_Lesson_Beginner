# OpenGL环境配置

OpenGL实际上只是一个图形程序开发接口标准, 实现是由操作系统和驱动程序来实现的。如果我们希望开发OpenGL程序，需要下载对应的工具库，如freeglut, glut, glfw等，以建立接口到实现的关联。

## 1. 相关库下载

按照Learnopengl的教程，我们基于GLFW和GLAD来开发OpenGL程序。GLFW是配合OpenGL使用的轻量级工具程序库，缩写自 Graphics Library Framework（图形库框架）。GLFW 的主要功能是创建并管理窗口和 OpenGL上下文，同时还提供了处理手柄、键盘、鼠标输入的功能。GLAD代表 OpenGL Loading Library, 是一个轻量级的C/C++库，用于跨平台地加载和管理OpenGL函数指针。它的主要目的是简化在不同操作系统和图形库之间使用OpenGL的过程。由于OpenGL支持的函数和扩展在不同的操作系统和图形库中可能有所不同，因此在编写跨平台OpenGL应用程序时，需要一种方法来加载正确版本的OpenGL函数。这就是GLAD发挥作用的地方。

GLFW, GLAD和GLM可以从以下链接处下载：

GLFW: https://www.glfw.org/download.html

GLAD: https://glad.dav1d.de/

GLM: https://github.com/g-truc/glm

在课程根目录中，我们提供了已经下载好的版本，方便大家使用。

## 2. 基于VS的环境配置

基于VS2022, 我们实现对OpenGL的配置

![20201204174222828](https://github.com/vvvwo/CG_Lesson/assets/65271555/e58267be-c986-4e8d-bd29-47814ff7cedc)

包含目录：

../glfw-3.3.bin.WIN64/include

../glad/include

../glm

库目录：

../glfw-3.3.bin.WIN64/lib-vc2019

链接库：

glfw3dll.lib
glfw3.lib

注意，要把glad.c文件添加在你的项目中：

![20201204182125850](https://github.com/vvvwo/CG_Lesson/assets/65271555/a889ad4e-8b7b-4d79-bd1a-3cbcc04a2e89)

DLL链接有两种方法，一种是在环境变量里，将../glfw-3.3.bin.WIN64/lib-vc2019配置Path中，还有一种就是将glfw.dll拷贝到项目根目录文件。为了方便起见，我选择的是第二种：

<img width="216" alt="image" src="https://github.com/vvvwo/CG_Lesson/assets/65271555/1b1b590d-325f-4249-8091-5b736c28da8a">

## 3. Hello World！

运行lesson1.cpp，可以看到如下现实窗口，则OpenGL配置成功：

<img width="470" alt="image" src="https://github.com/vvvwo/CG_Lesson/assets/65271555/d5862aac-8a81-4f6c-88ae-f29804e6e512">

上面的程序仅仅是显示了一个由glfwCreateWindow创建的窗口。在后续的课程中，我们将介绍实例代码的基本信息，以及如何在窗口中实现绘制以及显示。
