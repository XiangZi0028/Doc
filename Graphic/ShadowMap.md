### ShadowMap

​		在光线追踪的技术中不需要单独的计算阴影，通过光线不停的弹射就计算出了各个点接收到的光照强度，从而就达到了阴影的效果，基于光栅化渲染中关ShadowMap是常见的实时阴影技术。

​		笔者主要是说一下自己学习到阴影技术的思路以及实现。什么是阴影，很简单，光照看不到的地方就是应该是阴影。好了有了这句话我们就知道我们第一步应该干什么了，当然是确定哪些东西能够被光照到，那么自然而然就知道哪些地方不能被光照到了。

![image-20220315170132007](E:\Document\Graphic\pic\image-20220315170132007.png)

#### 最简单的shadowmap--平行光

###### （1）pass_1——shadow

​		求光空间深度图就是我们的第一步，我们把光源当作一个摄像机，按照光照的方向作为look at的方向，这样我们就能记录下光源所能看到的每个点的深度最小的地方。如上图所示蓝色区域就是我们在光空间中能够看到的部分。

###### （2）pass_2——map

​		着色的过程中，我们计算一个像素点的时候，首先通过光空间的视图矩阵，把它转换到逛空间中去，计算出该点的深度值，然后与光空间深度图记录的该点的深度值进行比较，如果深度值比记录的深度值大，就表示该点处于阴影当中，否则就能够被光照到。

​	根据结果我们可以看到，有些时候阴影出现了失真，本来完全暴露在光照范围内的地面也出现了阴影条纹。那么出现这样的情况的原因是什么呢？

![image-20220315172151422](E:\Document\Graphic\pic\image-20220315172151422.png)



#### Point ShadowMap

![image-20220315173833466](E:\Document\Graphic\pic\image-20220315173833466.png)

​		点光源阴影，我们知道点光源是朝四面八方去发射光纤的，那么我们如何去给它做阴影贴图呢。方法有很多，六面体、八面体等，核心是通过记录光源对各个方向的光照的情况，这里主要是说cube map的方式。

​	（1）以光源点作为摄像机的位置，朝着立方体的六个面去分别构建六个不同的视图变换矩阵。

```C++
 glm::lookAt(glm::vec3(lightPos),glm::vec3(lightPos) + glm::vec3(1.0f,  0.0f,  0.0f), glm::vec3(0.0f, -1.0f,  0.0f)),
            glm::lookAt(glm::vec3(lightPos),glm::vec3(lightPos) + glm::vec3(-1.0f, 0.0f,  0.0f), glm::vec3(0.0f, -1.0f,  0.0f)),
            glm::lookAt(glm::vec3(lightPos),glm::vec3(lightPos) + glm::vec3(0.0f,  1.0f,  0.0f), glm::vec3(0.0f,  0.0f,  1.0f)),
            glm::lookAt(glm::vec3(lightPos),glm::vec3(lightPos) + glm::vec3(0.0f, -1.0f,  0.0f), glm::vec3(0.0f,  0.0f, -1.0f)),
            glm::lookAt(glm::vec3(lightPos),glm::vec3(lightPos) + glm::vec3(0.0f,  0.0f,  1.0f), glm::vec3(0.0f, -1.0f,  0.0f)),
            glm::lookAt(glm::vec3(lightPos),glm::vec3(lightPos) + glm::vec3(0.0f,  0.0f, -1.0f), glm::vec3(0.0f, -1.0f,  0.0f))
```

​	 (2)以这六个矩阵分别生成一个shadow

![image-20220315174222510](E:\Document\Graphic\pic\image-20220315174222510.png)

​		这里我把数据存到了一个cube中，这样的话方便使用阴影贴图的时候采样。使用cubemap的话我就可以根据当前位置和光的方向直接进行采样了。

​	当然这里我们可以发现，对于采用cubemap的方式比较的浪费空间，一个点光源我们就需要花费6张纹理去保存，所以也有一些压缩的方法来保存cubemap。

​	比如我们可以再生成阴影贴图的时候采用球面映射的方法，把所有的像素映射到一个球面，再保存到一个纹理中。但是球面映射的效果比较差(想象一下地球仪，两极会损失很多的细节)，我之前在看论文的时候看到有人把阴影映射到一个八面体上，把八面体展开之后平成一个张纹理，差不多只需要用cubemap中四张纹理的大小，就能得到也还不错的效果。

#### CSM

级联阴影。它的中心思想就是根据相机目标点离相机的位置远近来采样不同精度的阴影图。有时候我们会发现，离光源比较远的物体它的阴影细节就丢失了很多

​	它最重要的就是切割视锥和重新计算新的视锥。

​	常见的分割方法有均匀分配、对数分配、综合分配，均匀分配就是说我按照near和far平面之间的距离把平面均匀分成3份或者其它份数。但是每个平截锥的near和far之间的距离都是相等的。对数分配的公式如下：
$$
Z_i=n(f/n)^{i/n}
$$
[级联阴影贴图（CSM） - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/388459633)对数分配的原理这篇文章讲的比较清除了，主要是分析精度对锯齿的影响来进行推导。综合分配就是综合上述两种分配来进行计算。



参考：

[阴影映射 - LearnOpenGL CN (learnopengl-cn.github.io)](https://learnopengl-cn.github.io/05 Advanced Lighting/03 Shadows/01 Shadow Mapping/)