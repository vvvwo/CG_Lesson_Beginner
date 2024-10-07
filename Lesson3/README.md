# 键盘与鼠标交互

在了解了shader的基本执行逻辑后，接下来我们介绍键盘鼠标的交互逻辑。这有助于我们了解shader渲染管线是如何处理用户交互的。有趣的交互永远都是最酷炫的！

## 1. 渲染管线的交互处理

OpenGL通过回调捕获来处理交互，即当每一帧被绘制完后，渲染管线向回调函数传入相关参数，以实现交互响应，流程图如下：

![image](https://github.com/user-attachments/assets/20b85aa1-5b15-489a-a78b-9ca71081f361)

与固定渲染管线相比，可编程渲染管线支持向GPU直接传递参数，以修改渲染流水线。这里通常是由着色器变量Uniform（全局的）执行数据传输。回想一下我们在Lesson2中介绍的案例，如果我们希望将鼠标移动产生的位移数据，作为控制摄像头的一个参数，那么我们需要将这个参数传入顶点着色器，以改变绘制的角度。着色器代码如下：

<pre class="prettyprint"><code class=" hljs xml">
#version 330 core 
layout (location = 0) in vec3 position;
uniform vec2 translation;
    
void main()
{
    gl_Position = vec4(position.x + translation.x,
                       position.y + translation.y,
                       position.z, 1.0);
}</code></pre>

这里的uniform变量translation实际接收了回调函数传入的鼠标位移，以传入到顶点着色器，进而改变了顶点的坐标信息。


## 2. 交互的种类

如果你是从OpenGL红宝书到Nehe来开始学习的OpenGL，并且在LearnOpenGL课程体系中配置过相关代码，那么我相信你会对shader有更深入的了解。简而言之，shader就是帮助我们与GPU沟通的机制。在没有shader之前，我们的渲染过程是非常低效的，即要把所有需要绘制的数据，在内存做好计算，然后打包给GPU。这样在内存与显存之间就会发生频繁的数据交互，大大降低了对大规模数据的处理效率。不过这个过程也有一个好处，就是在应用端的代码是比较容易理解的。基于shader的OpenGL实现逻辑就完全不同了。内存上的数据会按照其属性和长度，做好严格的声明和标注后，提前传输到显存中。shader提供了一个模板形式的C代码理解机制，可以预先定义相关程序，以通过GPU如何执行诸如矩阵变换和光照渲染等计算。这个过程是在GPU上完成的，这显然比传统的实现要高效。但同时，由于这里涉及到应用端对GPU渲染过程的控制以及数据的交互，这使得相关代码要比传统固定渲染管线的代码要难懂。

## 3. 程序实例

我们已经提到，shader是联系GPU的一种机制。shader的实现即为了建立CPU与GPU，显存与内存的联系。因此，我们在看shader时，应该考虑这种联系，以理解相关实现的逻辑。

### VAO和VBO (该部分摘录自LearnOpenGL)

顶点缓冲对象(Vertex Buffer Objects, VBO)和顶点数组对象(Vertex Array Object, VAO)就是shader用于在应用端传递使用显存数据的两个重要对象。 VBO对应了显存的一组顶点数据，对应一个独立的ID：


按照参数，指定了输入的vertices中哪些是位置，哪些是纹理等。

顶点数组对象(Vertex Array Object, VAO)可以像顶点缓冲对象那样被绑定，任何随后的顶点属性调用都会储存在这个VAO中。这里我的理解是，如果你有很多的VBO对象，你希望一部分用渲染管线1，一部分用2。如果你缺少打包的工具，这个过程就会很乱。VAO帮助你进行打包，以更好的绘制VBO群组。当你绘制时，只要启用VAO对象就可以对应绘制被其绑定的VBO数据。以下是一个完整的绑定。

<pre class="prettyprint"><code class=" hljs xml">unsigned int VBO, VAO;
glGenVertexArrays(1, &VAO);
glGenBuffers(1, &VBO);
glBindVertexArray(VAO);

// 2. 把顶点数组复制到缓冲中供OpenGL使用
glBindBuffer(GL_ARRAY_BUFFER, VBO);
glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
// 3. 设置顶点属性指针
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);
glEnableVertexAttribArray(0);</code></pre>  

