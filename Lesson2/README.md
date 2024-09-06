# Shader编程入门

我在开始学习OpenGL的时候，一直希望找到上手难度不是太大的demo，能够把可编程渲染管线的一些技术细节串起来，这样能够更方便理解。所以，我在接下来的几个课程中，利用一个OpenGL程序实例，来讲解可编程渲染管线的技术细节，尤其是局部坐标系和全局坐标系的变换，以及交互部分。我相信基于实例的课程，能够最大程度帮助新手掌握可编程渲染管线。

## 1. 实例介绍

我们这里使用LearnOpenGL坐标系统章节提供的一个实例代码：

<img width="400" alt="屏幕截图 2024-09-06 165624" src="https://github.com/user-attachments/assets/344212ba-1098-4787-b045-797246cf9eaf">

基于该代码，我实现了一个有趣的功能，即类似于FPS游戏那样，可以利用键盘鼠标自由移动视角和位置，开枪射击子弹，子弹与固定的物体产生碰撞检测后，完成被击中目标的删除。当子弹飞行到一定距离，会自动消失。这些功能几乎支持了一个FPS游戏最基本实现，类似最早期的三维游戏DOOM (图1为初始界面，图2为移动后的效果，图3为射击后带有子弹飞行轨迹的界面)。

<img width="200" alt="屏幕截图 2024-09-06 165355" src="https://github.com/user-attachments/assets/531f930d-6437-43fc-b1ca-4ba7440081ec">

<img width="200" alt="屏幕截图 2024-09-06 165431" src="https://github.com/user-attachments/assets/cbc030c2-075e-4ebc-883b-150ce8e841a3">

<img width="200" alt="屏幕截图 2024-09-06 165501" src="https://github.com/user-attachments/assets/bb99bc18-2b62-47cc-966a-1913a1950b8d">

这里我没有使用任何新的模型，完全是基于原始代码的数据来实现的对应功能。我认为这样更方便大家理解OpenGL可编程渲染管线的执行逻辑。这节课我们重点来梳理一下Shader的基本知识。

## 2. 为什么使用Shader

如果你是从OpenGL红宝书到Nehe来开始学习的OpenGL，并且在LearnOpenGL课程体系中配置过相关代码，那么我相信你会对shader有更深入的了解。简而言之，shader就是帮助我们与GPU沟通的机制。在没有shader之前，我们的渲染过程是非常低效的，即要把所有需要绘制的数据，在内存做好计算，然后打包给GPU。这样在内存与显存之间就会发生频繁的数据交互，大大降低了对大规模数据的处理效率。不过这个过程也有一个好处，就是在应用端的代码是比较容易理解的。基于shader的OpenGL实现逻辑就完全不同了。内存上的数据会按照其属性和长度，做好严格的声明和标注后，提前传输到显存中。shader提供了一个模板形式的C代码理解机制，可以预先定义相关程序，以通过GPU如何执行诸如矩阵变换和光照渲染等计算。这个过程是在GPU上完成的，这显然比传统的实现要高效。但同时，由于这里涉及到应用端对GPU渲染过程的控制以及数据的交互，这使得相关代码要比传统固定渲染管线的代码要难懂。

## 3. Shader在OpenGL中的使用

我们已经提到，shader是联系GPU的一种机制。shader的实现即为了建立CPU与GPU，显存与内存的联系。因此，我们在看shader时，应该考虑这种联系，以理解相关实现的逻辑。

### VAO和VBO (该部分摘录自LearnOpenGL)

顶点缓冲对象(Vertex Buffer Objects, VBO)和顶点数组对象(Vertex Array Object, VAO)就是shader用于在应用端传递使用显存数据的两个重要对象。 VBO对应了显存的一组顶点数据，对应一个独立的ID：

<pre class="prettyprint"><code class=" hljs xml">unsigned int VBO;
glGenBuffers(1, &VBO);</code></pre>

OpenGL有很多缓冲对象类型，顶点缓冲对象的缓冲类型是GL_ARRAY_BUFFER,使用glBindBuffer函数把新创建的缓冲绑定到GL_ARRAY_BUFFER目标上：

<pre class="prettyprint"><code class=" hljs xml">glBindBuffer(GL_ARRAY_BUFFER, VBO);</code></pre>  

这样我们就利用VBO搭建了到显存的一个“通路”，“通路”有了，就要传输数据。使用glBufferData，就可以把我们在应用端的数据，传输的GL_ARRAY_BUFFER，也就是VBO指定的显存区域。

<pre class="prettyprint"><code class=" hljs xml">glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);</code></pre>  

当我们把数据按照顺序传输后，我们还需要告诉OpenGL怎样解析上述数据：

<pre class="prettyprint"><code class=" hljs xml">glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);
glEnableVertexAttribArray(0);</code></pre>  

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

当我们进行绘制时，只需要绑定VAO即可
<pre class="prettyprint"><code class=" hljs xml">glBindVertexArray(VAO);
someOpenGLFunctionThatDrawsOurTriangle();</code></pre>  

### Shader(着色器)的表现形式

Shader是沟通GPU的一种机制。基于VAO和VBO，我们能够建立对显存数据的访问。除此之外，我们还希望告诉GPU，我们希望怎么绘制这些数据，包括坐标变换、纹理映射和光照渲染的具体方式。这就要求Shader能够满足渲染程序的相关需求。
简言之，shader的表现形式可以被认为是在GPU解释渲染过程的一段代码，与普通程序没有明显的区别，但对其命名规则有严格的要求，示例如下：

<pre class="prettyprint"><code class=" hljs xml">#version 330 core
layout (location = 0) in vec3 aPos;

void main()
{
    gl_Position = vec4(aPos.x, aPos.y, aPos.z, 1.0);
}</code></pre>  

上述代码就是完成了一个坐标的传递，没有添加任何变换。在形式上，就是一段普通的C代码。请注意，这里的 (location = 0)需要对应上面我们讲过的glVertexAttribPointer的第一个参数，即对数据的解析需要直接体现在shader代码的处理流程上。可以看出，对数据的传输，shader的要求是非常严格的，差之毫厘，谬之千里。关于shader的绑定，此处不再赘述了，有兴趣的同学可以学习LearnOpenGL的“你好，三角形”。

## 3. Hello World

当你对上述内容有了基本的认识后，接下来就是在你的程序中，绘制一个三角形了。如果你使用LearnOpenGL的原始代码不能完成绘制，那我把自己已经调好的源程序放在该课程的文件夹下，可以比照参考。
