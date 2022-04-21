RenderPipelineAssets   -> Graphics Pipeline 资产

RenderPipeline -> 真正的管线   其中Render()函数是渲染的入口 

​		URenderPipeline->Render()->:

​		BeginContextRendering(context,camera):调用RenderPipelineManager. beginCamera?ContextRendering委托

​		//设置各种GraphicsSettings里面的各种参数  比如光线使用gamma还是liner

​		U_SetupPerFrameShaderConstants():设置帧着色器常量Shader.setGlobalVector...

​		SortCameras(cameras) 根据摄像机的深度进行了一个排序

​		遍历camera

​				是否是GameCamera， Game、SceneView、preview、VR、Reflection。只有VR 和Game才是GameCamera

​						如果是URenderCameraStack

​						不是的话就BeginCameraRendering:调用RenderPipelineManager. beginCameraRendering委托;然后更新体积框架UUpdateVolumeFrameWork，然后再RUenderSingleCamera(context,camera),然后EndCameraRendering 然后再EndContextRendering

​		