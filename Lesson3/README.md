# 键盘与鼠标交互

在了解了shader的基本执行逻辑后，接下来我们介绍键盘鼠标的交互逻辑。这有助于我们了解shader渲染管线是如何处理用户交互的。有趣的交互永远都是最酷炫的！

## 1. 渲染管线的交互处理

OpenGL通过回调捕获来处理交互，即当每一帧被绘制完后，渲染管线向回调函数传入相关参数，以实现交互响应，流程图如下：

![image](https://github.com/user-attachments/assets/20b85aa1-5b15-489a-a78b-9ca71081f361)

与固定渲染管线相比，可编程渲染管线支持向GPU直接传递参数，以修改渲染流水线。这里通常是由着色器变量Uniform（全局的）执行数据传输。回想一下我们在Lesson2中介绍的案例，如果我们希望将鼠标移动产生的位移数据，作为控制摄像头的一个参数，那么我们需要将这个参数传入顶点着色器，以改变绘制的角度。着色器代码如下：

<pre class="prettyprint"><code class=" hljs xml">#version 330 core 
layout (location = 0) in vec3 position;
uniform vec2 translation;
    
void main()
{
    gl_Position = vec4(position.x + translation.x,
                       position.y + translation.y,
                       position.z, 1.0);
}</code></pre>

这里的uniform变量translation实际接收了回调函数传入的鼠标位移，以传入到顶点着色器，进而改变了顶点的坐标信息。同样的道理，我们当然可以向片段着色器传入对应参数，来直接改变像素信息。
在应用端或IDE界面，对translation变量的数据传递代码如下：
<pre class="prettyprint"><code class=" hljs xml">int vertexLocation = glGetUniformLocation(shaderProgram,"translation"); 
glUseProgram(shaderProgram); 
glUniform2f(vertexLocation , 0.0f, 1.0f);</code></pre>

我们需要在应用端定义一个指向“translation”的ID，即vertexLocation，然后通过glUniform2f向其传入数据。
这里我们放一个鼠标单击的回调函数实例，当点击后，会传入点击位置的二维坐标，如果将对应的坐标通过glUniform2f替换（0.0f, 1.0f），那么就实现了从鼠标点击数据到顶点着色器传输的一个完整的交互过程。

<pre class="prettyprint"><code class=" hljs xml">void mouseClick_callback(GLFWwindow* window, int button, int action, int mods)
{
    double winX, winY;
    glfwGetCursorPos(window, &winX, &winY);
    
    if ((action == GLFW_PRESS) && (button == GLFW_MOUSE_BUTTON_LEFT))
    {
        x_move = winX;
        y_move = winY;        
    }    
}</code></pre>


## 2. 交互的种类

考虑到鼠标与键盘的各种操作，OpenGL提供了非常丰富的回调函数，这里我们列出Lesson2的实例需要的几个回调函数：

<pre class="prettyprint"><code class=" hljs xml">void framebuffer_size_callback(GLFWwindow* window, int width, int height);
void mouse_callback(GLFWwindow* window, double xpos, double ypos);
void mouse_button_callback(GLFWwindow* window, int button, int action, int mods);
void scroll_callback(GLFWwindow* window, double xoffset, double yoffset);
void processInput(GLFWwindow* window);</code></pre>

### 1）framebuffer_size_callback

用来处理窗口变化；

### 2）mouse_callback

用来处理鼠标的移动；

### 3）mouse_button_callback

用来处理鼠标点击事件，包括触发的是左或右键，以及对应的动作。因为这个回调函数的使用频率较多，我把对应的参数都列出来，以方便查看。
参数包括：int button, int action, int mods

#### button: 被操作的鼠标按钮，可能的值为：

GLFW_MOUSE_BUTTON_1 (左键)

GLFW_MOUSE_BUTTON_2 (右键)

GLFW_MOUSE_BUTTON_3 (中键)

GLFW_MOUSE_BUTTON_4 及以上（额外鼠标按钮）

#### action: 表示事件类型，可能的值为：

GLFW_PRESS（按钮被按下）

GLFW_RELEASE（按钮被释放）

#### mods: 表示修改键的状态：

如 GLFW_MOD_SHIFT、GLFW_MOD_CONTROL 等，指示同时按下的修饰键

### 4）scroll_callback

用来处理鼠标滚轮事件；

### 5）processInput

用来检查键盘输入，配合glfwGetKey函数以检查什么按键被敲击，以及对应的操作是什么；

注意，上述回调函数是按照种类做的划分，其命名是可以自由指定的。当我们需要opengl知道我们设置的回调对应哪一种操作事件，我们需要使用特定的语句进行链接，如下：

 <pre class="prettyprint"><code class=" hljs xml">glfwSetFramebufferSizeCallback(window, framebuffer_size_callback);
 glfwSetCursorPosCallback(window, mouse_callback);
 glfwSetScrollCallback(window, scroll_callback);
 glfwSetMouseButtonCallback(window, mouse_button_callback);</code></pre>

## 3. 程序实例

这里我们介绍一个使用鼠标控制摄像头的实例：

<pre class="prettyprint"><code class=" hljs xml">void mouse_callback(GLFWwindow* window, double xposIn, double yposIn)
{
    float xpos = static_cast<float>(xposIn);
    float ypos = static_cast<float>(yposIn);

    if (firstMouse)
    {
        lastX = xpos;
        lastY = ypos;
        firstMouse = false;
    }

    float xoffset = xpos - lastX;
    float yoffset = lastY - ypos; // reversed since y-coordinates go from bottom to top

    lastX = xpos;
    lastY = ypos;

    camera.ProcessMouseMovement(xoffset, yoffset);
}</code></pre>

这里我们使用mouse_callback回调函数传出一个鼠标的位移（xoffset, yoffset），通过camera.ProcessMouseMovement传出到应用端。

在应用端，camera.ProcessMouseMovement函数对应一个摄像头类。该类直接将这组参数传入到该类中，并转换为角度：

<pre class="prettyprint"><code class=" hljs xml">void ProcessMouseMovement(float xoffset, float yoffset, GLboolean constrainPitch = true)
{
    xoffset *= MouseSensitivity;
    yoffset *= MouseSensitivity;

    Yaw += xoffset;
    Pitch += yoffset;

    // make sure that when pitch is out of bounds, screen doesn't get flipped
    if (constrainPitch)
    {
        if (Pitch > 89.0f)
            Pitch = 89.0f;
        if (Pitch < -89.0f)
            Pitch = -89.0f;
    }

    // update Front, Right and Up Vectors using the updated Euler angles
    updateCameraVectors();
}</code></pre>

此时，摄像头类的角度已经根据鼠标传入的数据产生改变，对应的摄像机局部坐标系统将被更新：

<pre class="prettyprint"><code class=" hljs xml">void updateCameraVectors()
{
    // calculate the new Front vector
    glm::vec3 front;
    front.x = cos(glm::radians(Yaw)) * cos(glm::radians(Pitch));
    front.y = sin(glm::radians(Pitch));
    front.z = sin(glm::radians(Yaw)) * cos(glm::radians(Pitch));
    Front = glm::normalize(front);
    // also re-calculate the Right and Up vector
    Right = glm::normalize(glm::cross(Front, WorldUp));  // normalize the vectors, because their length gets closer to 0 the more you look up or down which results in slower movement.
    Up = glm::normalize(glm::cross(Right, Front));
}</code></pre>

当我们更新了Front，Right和Up三个摄像机方向向量，就能够驱动opengl实现观察矩阵更新：

<pre class="prettyprint"><code class=" hljs xml">glm::mat4 view = camera.GetViewMatrix(); 
ourShader.setMat4("view", view);</code></pre>

其中 camera.GetViewMatrix()传入的mat4，就是通过glm::lookAt函数建立的摄像机局部坐标。

实例如下：

![image](https://github.com/user-attachments/assets/14217d50-9fdd-49c5-a059-58b2882fa9de)![image](https://github.com/user-attachments/assets/09128dfb-48f0-432e-8181-015e7131e14a)![image](https://github.com/user-attachments/assets/05be058a-bd69-4b28-bebe-3fb781b188eb)

可以参考本次课程的内容，结合https://learnopengl-cn.github.io/01%20Getting%20started/09%20Camera/#_7的样例代码，尝试实现一个鼠标交互。       
