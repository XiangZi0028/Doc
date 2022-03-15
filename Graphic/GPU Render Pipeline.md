### GPU流水线和软光栅化器模拟

#### GPU渲染流水线	

​		主要说一下GPU硬件渲染管线，准确来说是GPU渲染流水线，再不回忆一下就忘了，顺便写文章练练手。主要是说明一下CPU数据到屏幕上呈现的结果，都经历了哪些步骤，他们分别又有什么意义。本篇文章主要是回忆半年前所学的东西，仅仅是自己的一些理解，不保证完全正确，仅供参考。（以OpenGL为例）

​		action！不用说了吧，数据肯定是先加载到内存中，也就是这段时间数据都在应用层中，之后我们要通过图形API，把数据送到GPU中，下面一OpenGL为例。

<img src="E:\Document\Graphic\pic\image-20220314173249836.png" alt="image-20220314173249836" style="zoom: 80%;" />

```c++
glGen*() //从GPU中申请一个或多个未使用的名称 ---> 申请标识
glBind*() //设置当前使用的标识的类型
glBufferData() //在标识的数据空间写入数据
```

​		OpenGL它是隐式图形API，我们不知道这些API内部到底干了些什么，或许经过这3步操作之后数据也并没有写到GPU缓存中去。或许底层驱动的实现会根据 `glBufferData(GLenum target, GLsizeiptr size, const void *data, GLenum usage) `的最后一个参数usage来决定数据存储的位置，以及数据什么时候被写如到GPU缓存中去。

![image-20220314173933026](E:\Document\Graphic\pic\image-20220314173933026.png)

​		在应用程序中我们还要负责对数据进行解释。举个例子吧，如下图所示，每一行代表一个顶点，也就是一共有6个顶点，每个顶点包含了它的pos、uv、normal三个数据，总共n个浮点数，如果我们不对这些数据进行说明，那你的流水线输入头也不知道该如何来定义这些数据，他就只能按照它自己的理解来进行解释了。但是这不一定是我们想要的结果，所以我们需要在应用程序中自己组织这些数据，比如那几个数表示的是pos，哪几个点表示的是uv，哪几个点表示的normal。明确这些意义之后我们就可以很好的解释vertexBuffer中的数据了。如下所示：

![image-20220314174538005](E:\Document\Graphic\pic\image-20220314174538005.png)

​		当我们需要的所有数据都已经准备完毕之后，我们的渲染流水线也就可以开始了。大致流程如下：

![image-20220314183245030](E:\Document\Graphic\pic\image-20220314183245030.png)

​		首先我们的数据依次的进入到我们的顶点着色器中，一般情况下我们在顶点着色器中会进行三种变换，也就是模型变换、视图变换和投影变换。这是为了模型的坐标转移到GL的世界坐标系中去。以前的固定管线中，顶点着色器和片段着色器以及几何着色器是没有被开放出来的，到了可编程管线的时候，顶点着色器才被开放出来，固定管线我也没有用过。

​		问题来了，这些经过顶点着色器冲洗的顶点就是一个点呀，最终我们要在屏幕上看到的肯定不能是这么一个一个的点呀，所以下一步我们就要把这些数据按照我们的在应用程序中的设置给组装起来，在应用程序中我们会调用`glDrawArrays(GLenum mode, GLint first, GLsizei count)`或者`glDrawElements(GLenum mode, GLsizei count, GLenum type, const void *indices)`来发起绘制命令，其中第一个参数就是mode就是指定图元的类型。可以是点、线、三角形等。

​	以图元是三角形为例，那么现在我们就拿到了三个点Vertex[0]、Vertex[1]、Vertex[2]，这三个点会组成一个平面，之后我们会进行平面进行裁剪，裁剪的主要目的也是说减少计算吧，对于哪些不在GL [-1,1]^3空间中的部分，或者处于物体背面看不见的部分(`glEnable(GL_CULL_FACE)`)，我们的流水线也没有必要对它进行加工，如果不进行裁剪，那比如所有的三角形都在可视空间之外，那我们的GPU就可以闲置休息，没必要做无用功。平面经过裁剪剩下的部分是我们最终需要渲染的部分，那么我们如何把这个平面转化成我们屏幕上的点呢。大家经常所说的光栅化就来了。

​	首先是屏幕映射，也就是把每个点映射到屏幕上去。这里后面我会用我写的软渲染器的代码来进行描述。经过屏幕映射之后又进行了三角形的设置。也许你会感到好奇，为什么这里又要进行图元的设置。

![image-20220314181742049](E:\Document\Graphic\pic\image-20220314181742049.png)

​		根据上图所示有两个经过各种变换三角形平面，从图2中我们可以看到上面的三角形有一个顶点会落到屏幕之外，之前我们说到，流水线中会对三角平面进行裁剪，裁剪之后如果我们补重新设置三角形的话，那么将会如图4所示，所有这也是我认为此次图元设置的意义是什么。时候我们要进行三角形遍历。

​		遍历的内容主要是我们映射到屏幕上的像素点，每个像素点都会经过片段着色器，片段着色器和顶点着色器一样，需要我们自己来编写shader，在片段着色器中我们可以计算每个点的光照，也就是最终呈现到我们平屏幕上的颜色。这些像素会经过各种测试，比如深度测试(`glEnable(GL_DEPTH_TEST)`)，只有通过深度测试的点才会经过片段着色器的洗礼。片段着色器进行光照结束之后，这些值会写入到我们的帧缓冲区或者我们的RenderTarget渲染目标上。这基本上就是GPU渲染流水线的整个流程。

​		其实除了上面所展示的这些过程之外，还有一些其它的不是必须的部分，比如在顶点着色器之后还可能有细分着色器，几何着色器。细分着色器可以根据一定的算法细化很多的细节，生成更多的顶点，几何着色器可以生成各种形状的图元。

#### 软光栅化器

​	之前写这个软光栅化器的目的主要是为了模拟GPU的渲染流水线，加深自己的学习吧。

![image-20220314215050680](E:\Document\Graphic\pic\image-20220314215050680.png)

​		软光栅化器的窗口使用的SDL库，数学库没有自己造了使用的是glm，对于数学库这种东西，一定要搞清楚原理，不然调试都调不了。主要是模仿OpenGL的方式，定义了一个GraphicPipeline类来作为GPU的渲染管线。因为只是作为自己回忆光栅化流程的产物，一切从简，没有单独抽象frame或者render target这些东西。函数的接口名字都很直白，基本上都能看懂。后面主要从main函数开始一步一步的到最终的画面！！

```c++
class GraphicPipeline
{
    //......
    int bufferMapIndexTop;
    int textureMapIndexTop;
     //all data buffer  uniformbuffer vertexbuffer indexbuffer
    std::map<int, DataBuffer*> bufferMap;
    //all texture buffer colorbuffer depth sencial 
    std::map<int, Texture*> textureMap;
    //......
    
    //开放的接口函数
    void init(); 
    int createDataBuffer();
    int createTexture();
    void setCullFace(stateSwitch ON_OFF);
    void setDepthTest(stateSwitch ON_OFF);
    void setTextureData(int identifier,int with,int height,int channel,void *data,int length);
    void setVertexBufferData(int identifier,void *data,int length);
    void setIndexBufferData(int identifier,void *data,int length);
    void setRendertargetSize(int with,int height);
    void setBuffer(int identifier,void *data,int length);
    void bindVertexBuffer(int identifier);
    void bindIndexBuffer(int identifier);
    void bindTexture(int identifier);
    void bindVertexShader(std::function <std::pair<void*,void*(GraphicPipeline *,void*,int )>);
    void bindFragmentShader(std::function<colorPixel(GraphicPipeline*,     								std::vector<std::pair<void*,void*>>, 										std::vector<float>,int)>);
    void bindVertexUniformBuffer(int identifier);
    void bindFragmentUniformBuffer(int identifier);
    void drawElements(int count,int begin);
    void drawArray();
    void swapBuffer();
    bool quit();
    void clearColor(colorPixel color);
    void clearDepth(float depth);
}
```

​		在GraphicPipeline类中有两个map，用来保存所有的Buffer，textureMap用来保存所有的纹理相关的数据，dataBufferMap用来保存vertexBuffer、indexBuffer、uniformBuffer的数据。bufferMapIndexTop和textureMapIndexTop用来表示下次创建Buffer时分配的对应map里面的key。

##### 		1.创建VertexBuffer、indexBuffer、uniformBuffer，并设置数据

```c++
int vbo=tranglePipeline.createDataBuffer();
tranglePipeline.setVertexBufferData(vbo,vertices,sizeof(vertices));
    
int ebo=tranglePipeline.createDataBuffer();
tranglePipeline.setIndexBufferData(ebo,indices,sizeof(indices));

int ubo_vertex=tranglePipeline.createDataBuffer();
tranglePipeline.setBuffer(ubo_vertex,&mvp,sizeof(MVPBuffer));
```

​		createDataBuffer()主要是创建一个DataBuffer数据结构并初始化，然后将它插入到bufferMapIndexTop中去，在把bufferMapIndexTop++。setVertexBufferData主要是拷贝vertices到DataBuffer中的data中去。

##### 	2.shader

​	shader是最不好处理的一个东西，但又是核心，这里我采用了一个不太好的办法，使用lambda表达式，虽然没能实现shader和代码分离，但是也能实现功能。这里我只允许每个着色器有一个uniformBuffer块，这样相当于写死了，但是只能说是能用吧。其实也可以实现多uniformBuffer块，但是当时偷懒了。

​	对与顶点着色器的输入我定义有两块 VS_IN_DEFAULT，该数据结构是每个顶点的描述，VS_IN_UNIFORMBUFFER是顶点着色器的uniformBuffer。顶点着色器的输出也包括两部分，一个是VS_OUT_DEFAULT数据块，比如什么gl_Position等，另一个是用户定义的VS_OUT数据块，不如uv等数据。

​	片段着色器的输入除了接收uniformBuffer之外还要接收顶点着色器的输出。它的输出就是一个vec4像素点。

##### 3.光栅化核心

​	软光栅化器的核心drawElements()函数，该函数具体实现了这些点集是如何一步一步的变成屏幕上的像素点。

（1）我们根据index获取所有的顶点数据，让他们经过顶点着色器的变换，每个顶点变换之后的值保存到gl_Position的vector中。

```C++
  for(int i=begin;i<begin+count;i++)
    {
        int index=all_indice[i];
        VS_IN_DEFAULT *vertex=&all_vertice[index];
        
        std::pair<void*,void*> vs_out=vertexShader(this,vertex,uniformBufferIndex_vertex);
        vs_outs[i-begin]=vs_out;
        glm::vec4 gl_Position = ((VS_OUT_DEFAULT*)vs_out.first)->gl_Position;
        gl_Positions[i-begin]=gl_Position;
    }
```

（2）三个点凑成一组，也就是构成一个图元，然会对它进行光栅化。

​	经过顶点着色器变换的点，可见的范围是在[-1,1]^3的坐标内，但是设备坐标系是在[0,1]^3的坐标范围内，所以我们要先把这些点变换到[-1,1]^3坐标范围内。

```C++
  glm::vec4 gl_Position_0=gl_Positions[i+0];
  gl_Position_0=gl_Position_0/gl_Position_0.w;
  gl_Position_0 = (gl_Position_0 + 1.0f) / 2.0f;

  glm::vec4 gl_Position_1=gl_Positions[i+1];
  gl_Position_1=gl_Position_1/gl_Position_1.w;
  gl_Position_1 = (gl_Position_1 + 1.0f) / 2.0f;

  glm::vec4 gl_Position_2=gl_Positions[i+2];
  gl_Position_2=gl_Position_2/gl_Position_2.w;
  gl_Position_2=(gl_Position_2 + 1.0f) / 2.0f;
```

（3）cullFace 裁剪面

​		通过判断是顺时针还是逆时针来解决。

```
if(this->cullFace==stateSwitch::ON)
{
glm::vec2 v01 = glm::normalize(glm::vec2(gl_Position_1 - gl_Position_0));
glm::vec2 v02 = glm::normalize(glm::vec2(gl_Position_2 - gl_Position_0));
if ((v01.x * v02.y - v01.y * v02.x) < 0.0f)
{
continue;
}
}
```

（4）光栅化

​		对平面进行光栅化，将它映射到frameTexture上。这里的主要方法是找出3个点的最大x、最小x、最大y和最小y，然后遍历这个矩形中的每个元素是否是在三角形内，判断方法可以使用重心法，刚好后面插值的时候也能用的上。如果该像素不在三角形内部，那么就跳过这个点，如果该点在三角形的内部，就根据它的权重计算出该点的uv，深度等信息。

![image-20220314230928775](E:\Document\Graphic\pic\image-20220314230928775.png)

​		如果计算出的深度比depthTexture中对应点记录的深度值大，那么就跳过对该点的着色，如果深度值比该点的值小，那么就把这个点送到fragmentShader中去进行着色。当所有的三角形都绘制完成之后，把colorTex输出到屏幕上。

![image-20220314230954778](E:\Document\Graphic\pic\image-20220314230954778.png)

​	