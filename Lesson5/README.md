# 三维模型读取与显示

在前面的课程中我们已经学过了利用OpenGL渲染管线实现三角形以及正方体的绘制方法。实际的应用当然不会止步于此。我们希望能够从文件中读取三维模型数据，随后进行绘制。如果你已经掌握了渲染管线的执行原理，那么读取三维模型文件并进行绘制是一个非常简单的任务，只是我们一次需要绘制的三角面片数量增加了而已！

## 1. 三维模型读取

三维模型文件包含多种格式，如obj, ply, stl, off等。这些文件数据存储形式不同，但是对应的信息是一致的，这里我们放两个例子：

<img height = "200" alt="fig1" src="https://github.com/user-attachments/assets/6b0e4ff9-9599-4291-abf0-3093da9f0fd2">
<img height = "200" alt="fig2" src="https://github.com/user-attachments/assets/e214436e-3414-413e-afe1-6a3484c98451">

如果我们希望绘制上述模型文件，我们只要把顶点和面片信息读取出来就可以了。关于模型读取，LearnOpenGL课程建议使用Assimp来执行。我使用的[VCG库]([https://markdown.com.cn](https://github.com/cnr-isti-vclab/vcglib))提供的模型读取程序，代码如下：

<pre class="prettyprint"><code class=" hljs xml">char* pfile = new char[strlen(pfileS.c_str()) + 1];
strcpy(pfile, pfileS.c_str());
if (tri::io::Importer<MyMesh>::Open(m, pfile) != 0){
   printf("Error reading file  %s\n", pfile);
   exit(0);
}</code></pre>

这里的MyMesh是基于VCG库的一个存储模型的数据结构。我们可以在数据载入到MyMesh后，从中读取顶点和面片信息：

<pre class="prettyprint"><code class=" hljs xml">std::vector&#60int&#62VertexId(m.vert.size());
int index_R = 0;
for (auto vi = m.vert.begin(); vi != m.vert.end(); ++vi) if (!(*vi).IsD()){
    VertexId[index_R] = index_R;
    index_R++;
    Point3f p_i;
    p_i[0] = (*vi).P()[0];
    p_i[1] = (*vi).P()[1];
    p_i[2] = (*vi).P()[2];
    pointOriginal.push_back(p_i);
}
for (auto fi = m.face.begin(); fi != m.face.end(); ++fi) if (!(*fi).IsD()) {
    vector<int> face_i(3);
    face_i[0] = VertexId[tri::Index(m, (*fi).V(0))];//index of vertex per face   
    face_i[1] = VertexId[tri::Index(m, (*fi).V(1))];//index of vertex per face  
    face_i[2] = VertexId[tri::Index(m, (*fi).V(2))];//index of vertex per face  
    faceOriginal.push_back(face_i); 
}</code></pre>

此时模型数据被载入到pointOriginal和faceOriginal两个vector里。之后我们只需要从这两个vector中提取数据并进行面片绘制就可以了。

## 2. 三维模型显示

模型的显示过程就是三角形的绘制过程。不同的是，模型的三角形数量很多，通过数组向着色器传输数据就变得不方便了。这里我使用vector来传输数据：

<pre class="prettyprint"><code class=" hljs xml">vector&#60float&#62 vertices;   
unsigned int VBO, VAO;
glGenVertexArrays(1, &VAO);
glGenBuffers(1, &VBO);

glBindVertexArray(VAO);
glBindBuffer(GL_ARRAY_BUFFER, VBO);   

glBufferData(GL_ARRAY_BUFFER, vertices.size()*sizeof(float), &vertices[0], GL_STATIC_DRAW);

// position attribute
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);
glEnableVertexAttribArray(0);</code></pre>

这里glBufferData指定的是vector&#60float&#62类型的变量，而不是我们之前经常使用的动态数组。它的首地址位置是&vertices。
glVertexAttribPointer函数说明数据流就是按照三角形三个顶点的坐标输入。vertices自然也是按照顺序存储三角形的顶点数据。这里其实可以通过EBO来指定index，与原始模型的数据存储格式保持一致的。我这里选择了更直接的方式来做数据传入，更方便理解。

剩下的和我们之前学习的内容就完全一样了，通过VAO来实现绘制就可以：

<pre class="prettyprint"><code class=" hljs xml">glBindVertexArray(VAO);
glDrawArrays(GL_TRIANGLES, 0, vertices.size()/3);
</code></pre>

vertices.size()/3的意思是三角面片的数量，正好是顶点数量的1/3。我们载入一个实验模型，渲染的效果如下：

<img height = "600" alt="fig1" src="https://github.com/user-attachments/assets/88c0190a-2751-4b5c-b50d-fe1169ce6254">

如果我们使用线绘制模型，就能够画出一个线框效果的三维模型。我把整个项目，连带VCG库上传到这里，供大家参考。

<pre class="prettyprint"><code class=" hljs xml">glPolygonMode(GL_FRONT_AND_BACK, GL_LINE);
</code></pre>

<img height = "600" alt="fig1" src="https://github.com/user-attachments/assets/76dd2063-6821-48f2-8f77-c43a9f4d8651">

可能你会说，这个模型不能看出三维效果！原因很简单，我们还没有使用光照信息，不用急，这部分内容我们很快就会讲到。



