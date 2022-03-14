### 软光栅化器

![image-20220314215050680](E:\Doc\Graphic\pic\image-20220314215050680.png)

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

​		1.创建VertexBuffer、indexBuffer、uniformBuffer，并设置数据

```c++
int vbo=tranglePipeline.createDataBuffer();
tranglePipeline.setVertexBufferData(vbo,vertices,sizeof(vertices));
    
int ebo=tranglePipeline.createDataBuffer();
tranglePipeline.setIndexBufferData(ebo,indices,sizeof(indices));

int ubo_vertex=tranglePipeline.createDataBuffer();
tranglePipeline.setBuffer(ubo_vertex,&mvp,sizeof(MVPBuffer));
```

​		createDataBuffer()主要是创建一个DataBuffer数据结构并初始化，然后将它插入到bufferMapIndexTop中去，在把bufferMapIndexTop++。setVertexBufferData主要是拷贝vertices到DataBuffer中的data中去。

​	2.shader

​	shader是最不好处理的一个东西，但又是核心，这里我采用了一个不太好的办法，使用lambda表达式，虽然没能实现shader和代码分离，但是也能实现功能。这里我只允许每个着色器有一个uniformBuffer块，这样相当于写死了，但是只能说是能用吧。其实也可以实现多uniformBuffer块，但是当时偷懒了。

​	对与顶点着色器的输入我定义有两块 VS_IN_DEFAULT，该数据结构是每个顶点的描述，VS_IN_UNIFORMBUFFER是顶点着色器的uniformBuffer。顶点着色器的输出也包括两部分，一个是VS_OUT_DEFAULT数据块，比如什么gl_Position等，另一个是用户定义的VS_OUT数据块，不如uv等数据。

​	片段着色器的输入除了接收uniformBuffer之外还要接收顶点着色器的输出。它的输出就是一个vec4像素点。

3.光栅化核心

​	软光栅化器的核心drawElements()函数，该函数具体实现了这些点集是如何一步一步的变成屏幕上的像素点。

​	(1).我们根据index获取所有的顶点数据，让他们经过顶点着色器的变换，每个顶点变换之后的值保存到gl_Position的vector中。

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

​	(2).三个点凑成一组，也就是构成一个图元，然会对它进行光栅化。

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

​	(3).cullFace 裁剪面

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

​	(4)光栅化

​		对平面进行光栅化，将它映射到frameTexture上。这里的主要方法是找出3个点的最大x、最小x、最大y和最小y，然后遍历这个矩形中的每个元素是否是在三角形内，判断方法可以使用重心法，刚好后面插值的时候也能用的上。如果该像素不在三角形内部，那么就跳过这个点，如果该点在三角形的内部，就根据它的权重计算出该点的uv，深度等信息。

![image-20220314230928775](E:\Doc\Graphic\pic\image-20220314230928775.png)

​		如果计算出的深度比depthTexture中对应点记录的深度值大，那么就跳过对该点的着色，如果深度值比该点的值小，那么就把这个点送到fragmentShader中去进行着色。当所有的三角形都绘制完成之后，把colorTex输出到屏幕上。

![image-20220314230954778](E:\Doc\Graphic\pic\image-20220314230954778.png)

