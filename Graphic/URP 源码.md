1.入口

UniversalRenderPiplineAsset.cs在该文件中，进行了很多的操作，其中最重要的就是CreatePipeline函数，该函数实例化了一个RenderPipeline对象。

```c#
 public partial class UniversalRenderPipelineAsset : RenderPipelineAsset, ISerializationCallbackReceiver
{
        protected override RenderPipeline CreatePipeline()
        {
            .....
            CreateRenderers();
            return new UniversalRenderPipeline(this);
        }
}
```

CreateRenderers()函数里面主要是创建一组ScriptableRenderer，ScriptableRenderer和ScriptableRendererData相对应。

![image-20220425204705816](E:\Codes\Doc\Graphic\URPImg\image-20220425204705816.png)

从通过ScriptableRendererData创建的

![image-20220425204843946](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220425204843946.png)

2.UniversalRenderPipeline

![image-20220425205417602](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220425205417602.png)

在UniversalRenderPipeline()的构造函数首先是进行了一些全局设置，这些全局设置暂时不知道在干啥

3.Tick

Render()函数就是渲染主体，每帧都会调用。

