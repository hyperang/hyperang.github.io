---
layout: post
title:  "Unity Shader Graph基础"
date:   2020-11-11 19:30:00 +0800
tags: Unity
color: rgb(255,90,90)
cover: '../assets/shader-graph-basic/shader_graph.gif'
subtitle: 'Dissolved & Hologram'
---
Unity Shader Graph入门，复现了 [Brackeys][link-1] 的 [DISSOLVE using Unity Shader Graph][link-2] 和 [HOLOGRAM using Unity Shader Graph][link-3] 中的内容。

[link-1]: https://www.youtube.com/channel/UCYbK_tjZ2OrIZFBvU6CCMiA
[link-2]: https://www.youtube.com/watch?v=taMp1g1pBeE&t=62s&ab_channel=Brackeys
[link-3]: https://www.youtube.com/watch?v=KGGB5LFEejg&ab_channel=Brackeys

复现环境为 Unity 2020.1.12f1c1 Personal，使用 HDRP 渲染管线。



## PBR Graph 基础

在 Assets 窗口右键 $\longrightarrow$ Create $\longrightarrow$ Shader $\longrightarrow$ PBR Graph 新建一个PBR Graph Shader。

双击新建的 shader 文件进入 Shader Graph 的编辑界面。



![sgwindow]({{site.url}}\assets\shader-graph-basic\shader_grap_window.jpg)



左边是 shader 的属性和关键词界面，中间为 Master Node，右边是主预览窗口。

PBR Master Node详细信息如下：

**Ports**

| Name                           | Direction | Type     | Stage    | Binding               | Description                                                  |
| :----------------------------- | :-------- | :------- | :------- | :-------------------- | :----------------------------------------------------------- |
| Vertex Position                | Input     | Vector 3 | Vertex   | Object Space Position | Defines the absolute object space vertex position per vertex. |
| Vertex Normal                  | Input     | Vector 3 | Vertex   | Object Space Normal   | Defines the absolute object space vertex normal per vertex.  |
| Vertex Tangent                 | Input     | Vector 3 | Vertex   | Object Space Tangent  | Defines the absolute object space vertex tangent per vertex. |
| Albedo[反射率]                 | Input     | Vector 3 | Fragment | None                  | Defines material's albedo value. Expected range 0 - 1.       |
| Normal                         | Input     | Vector 3 | Fragment | Tangent Space Normal  | Defines material's normal value. Expects normals in tangent space. |
| Emission[自发光]               | Input     | Vector 3 | Fragment | None                  | Defines material's emission color value. Expects positive values. |
| Metallic                       | Input     | Vector 1 | Fragment | None                  | Defines material's metallic value where 0 is non-metallic and 1 is metallic. Only available in Metallic **Workflow** mode. |
| Specular[高光]                 | Input     | Vector 3 | Fragment | None                  | Defines material's specular color value. Expected range 0 - 1. Only available in Specular **Workflow** mode. |
| Smoothness                     | Input     | Vector 1 | Fragment | None                  | Defines material's smoothness value. Expected range 0 - 1.   |
| Occlusion[遮蔽]                | Input     | Vector 1 | Fragment | None                  | Defines material's ambient[环境光] occlusion value. Expected range 0 - 1. |
| Alpha                          | Input     | Vector 1 | Fragment | None                  | Defines material's alpha value. Used for transparency and/or alpha clip. Expected range 0 - 1. |
| Alpha Clip Threshold[剪裁阈值] | Input     | Vector 1 | Fragment | None                  | Fragments with an alpha below this value will be discarded. Requires a node connection. Expected range 0 - 1. |

**Material Options**



![m_o]({{site.url}}\assets\shader-graph-basic\pbr_m_o.png)



| Name                  | Type     | Options                                | Description                                                  |
| :-------------------- | :------- | :------------------------------------- | :----------------------------------------------------------- |
| Workflow              | Dropdown | Metallic, Specular                     | Defines workflow mode for the material.                      |
| Surface               | Dropdown | Opaque[不透明], Transparent[透明]      | Defines if the material is transparent.                      |
| Blend                 | Dropdown | Alpha, Premultiply, Additive, Multiply | Defines blend mode of a transparent material.                |
| Fragment Normal Space | Dropdown | Tangent, Object, World                 | Defines the coordinate space of the value supplied to the **Normal** slot. |
| Two Sided             | Toggle   | True, False                            | If `true`, both front and back faces of the mesh are rendered. |
| Override ShaderGUI    | Toggle   | True, False                            | Lets you override the ShaderGUI that this Shader Graph uses. If `true`, the **ShaderGUI** property appears, which lets you specify the ShaderGUI to use. |





## 消融

### 基础效果

#### 思路

给 Alpha 通道加一个噪声，使物体表面的 alpha 值在 $(0,1)$ 内随机分布。

然后通过改变 Alpha Clip Threshold 的大小，将 alpha 小于某个值的 fragment 丢弃（discard）做出消融的效果。



#### 结点

**Simple Noise Node**

*Noise $\rightarrow$ Simple Noise*

基于 UV 坐标生成一个 Simplex noise

**Ports**

| Name  | Direction | Type     | Binding | Description    |
| :---- | :-------- | :------- | :------ | :------------- |
| UV    | Input     | Vector 2 | UV      | Input UV value |
| Scale | Input     | Vector 1 | None    | Noise scale    |
| Out   | Output    | Vector 1 | None    | Output value   |



**Time Node**

*Basic $\rightarrow$ Time*

提供对着色器中各种**时间**参数的访问。

**Ports**

| Name         | Direction | Type     | Binding | Description                 |
| :----------- | :-------- | :------- | :------ | :-------------------------- |
| Time         | Output    | Vector 1 | None    | Time value                  |
| Sine Time    | Output    | Vector 1 | None    | Sine of Time value          |
| Cosine Time  | Output    | Vector 1 | None    | Cosine of Time value        |
| Delta Time   | Output    | Vector 1 | None    | Current frame time          |
| Smooth Delta | Output    | Vector 1 | None    | Current frame time smoothed |



**Remap Node**

*Math $\rightarrow$ Range $\rightarrow$ Remap*

根据输入 **In** 的值在输入 **In Min Max** 的 x 和 y 分量之间的线性插值（linear interpolation），返回输入 **Out Min Max** 的 x 和 y 分量之间的值。

即通过线性插值将 $\text{In}\in\text{In Min Max}$ 映射到 $\text{out}\in\text{Out Min Max}$。

**Ports**

| Name        | Direction | Type           | Description                                         |
| :---------- | :-------- | :------------- | :-------------------------------------------------- |
| In          | Input     | Dynamic Vector | Input value                                         |
| In Min Max  | Input     | Vector 2       | Minimum and Maximum values for input interpolation  |
| Out Min Max | Input     | Vector 2       | Minimum and Maximum values for output interpolation |
| Out         | Output    | Dynamic Vector | Output value                                        |



#### 实现



![d_p1]({{site.url}}\assets\shader-graph-basic\dissolve_p1.jpg)



- 使用 Simple Noise 为 Alpha 赋值

- 使用 Time 中的 Cosine Time 得到一个随时间变化的值，使用 Remap 将其映射到 $(0,1)$ 后赋值给 Alpha Clip Threshold



### 边缘的高亮效果

#### 思路

将消融的过程复制并延迟，类似于二重唱，得到物体消融的边缘。



#### 结点

**Step Node**

*Math $\rightarrow$ Range $\rightarrow$ Step*

对于每个分量，如果输入 **In** 的值大于或等于输入 **Edge** 的值，则返回 1，否则返回 0。

**Ports**

| Name | Direction | Type           | Description  |
| :--- | :-------- | :------------- | :----------- |
| Edge | Input     | Dynamic Vector | Step value   |
| In   | Input     | Dynamic Vector | Input value  |
| Out  | Output    | Dynamic Vector | Output value |



**Add Node**

*Math $\rightarrow$ Basic $\rightarrow$ Add*

返回两个输入值之和。

**Ports**

| Name | Direction | Type           | Description        |
| :--- | :-------- | :------------- | :----------------- |
| A    | Input     | Dynamic Vector | First input value  |
| B    | Input     | Dynamic Vector | Second input value |
| Out  | Output    | Dynamic Vector | Output value       |



**Multiply Node**

*Math $\rightarrow$ Basic $\rightarrow$ Multiply*

返回两个输入值的乘积。

如果两个输入皆为向量，则输出为向量，维度由结点评估（evaluate）得到。

如果两个输入分别为向量和矩阵，则输出为与向量输入维度相同的向量。

如果两个输入皆为矩阵，则输出为矩阵，维度由结点评估（evaluate）得到。

**Ports**

| Name | Direction | Type    | Description        |
| :--- | :-------- | :------ | :----------------- |
| A    | Input     | Dynamic | First input value  |
| B    | Input     | Dynamic | Second input value |
| Out  | Output    | Dynamic | Output value       |



**Color**

*Input $\rightarrow$ Basic $\rightarrow$ Color*

使用色域（color field）在着色器中定义一个恒定的（constant ）四维向量。

颜色的属性类型（Color Property Type）根据结点的上下文进行转换。

参数 Mode 也会影响颜色的属性类型。

**Ports**

| Name | Direction | Type     | Binding | Description  |
| :--- | :-------- | :------- | :------ | :----------- |
| Out  | Output    | Vector 4 | None    | Output value |

**Controls**

| Name | Type     | Options      | Description                        |
| :--- | :------- | :----------- | :--------------------------------- |
|      | Color    |              | Defines the output value.          |
| Mode | Dropdown | Default, HDR | Sets properties of the Color field |



#### 实现



![d_p2]({{site.url}}\assets\shader-graph-basic\dissolve_p2.jpg)



- 使即将隐藏的 fragment 自发光，故在基础效果中作为阈值的重映射后的 Cosine Time 变为 Step 中的输入而 Simple Noise 的值为 Step 中的 Edge
- 线性增加重映射后的 Cosine Time 的值使得自发光的效果比消融的效果提前出现 



### 最终效果

将 Color Node 设置为 Property。

添加两个 Vector 1 类型的 Property 分别作为 Simple Noise Node 的 Scale 以及 Add Node 的输入。

在 PBR Master 的 Emission 输入前添加 Emission Node 结点。



![dissolve]({{site.url}}\assets\shader-graph-basic\dissolve.jpg)



效果



![dball]({{site.url}}\assets\shader-graph-basic\dball.jpg)





## 全息影像

### 基础效果

#### 思路

将 PBR Master 中的 Surface 属性设置为 Transparent 使物体透明。

使用一张模拟显像管扫描线效果的贴图来产生扫描成像的效果。

将 UV 坐标由物体 UV 改为 屏幕 UV 使扫描线对应屏幕而非物体表面。

给 Texture 增加一个偏移量



#### 结点

**Sample Texture 2D Node**

*Input $\rightarrow$ Texture $\rightarrow$ Sample Texture 2D*

对一个 Texture 2D 采样并返回一个 Vector 4 类型的颜色值。

可以使用 UV 输入来重写 UV 坐标。

使用 Sampler 输入自定义采样器状态。

进行法线贴图采样时，确保类型参数设置为法线。

此结点只能用于 Fragment Shader Stage。

若要在 Vertex Shader Stage 对 Texture 2D 采样，使用 Sample Texture 2D LOD Node。

**Ports**

| Name    | Direction | Type          | Binding               | Description                        |
| :------ | :-------- | :------------ | :-------------------- | :--------------------------------- |
| Texture | Input     | Texture 2D    | None                  | Texture 2D to sample               |
| UV      | Input     | Vector 2      | UV                    | Mesh's normal vector               |
| Sampler | Input     | Sampler State | Default sampler state | Sampler for the texture            |
| RGBA    | Output    | Vector 4      | None                  | Output value as RGBA               |
| R       | Output    | Vector 1      | None                  | red (x) component of RGBA output   |
| G       | Output    | Vector 1      | None                  | green (y) component of RGBA output |
| B       | Output    | Vector 1      | None                  | blue (z) component of RGBA output  |
| A       | Output    | Vector 1      | None                  | alpha (w) component of RGBA output |

**Controls**

| Name | Type     | Options         | Description              |
| :--- | :------- | :-------------- | :----------------------- |
| Type | Dropdown | Default, Normal | Selects the texture type |



**Tiling And Offset Node**

*UV $\rightarrow$ Tiling And Offset*

分别通过平铺（Tiling）和偏移（Offset）输入对输入的 UV 进行平铺与偏移操作。

通常用于细节贴图和随时间滚动纹理。

**Ports**

| Name   | Direction | Type     | Binding | Description                           |
| :----- | :-------- | :------- | :------ | :------------------------------------ |
| UV     | Input     | Vector 2 | UV      | Input UV value                        |
| Tiling | Input     | Vector 2 | None    | Amount of tiling to apply per channel |
| Offset | Input     | Vector 2 | None    | Amount of offset to apply per channel |
| Out    | Output    | Vector 2 | None    | Output UV value                       |



**Screen Position Node**

*Input $\rightarrow$ Geometry $\rightarrow$ Screen Position Node*

访问网格顶点（mesh vertex）或片段的屏幕位置。

通过 Mode 参数选择输出值的模式。

- 默认：返回屏幕位置。 此模式将屏幕位置除以剪辑空间位置 W 分量。

- 原始：返回屏幕位置。 此模式不会将“屏幕位置”除以剪辑空间位置W分量。 这对于投影很有用。

- 中央：返回屏幕位置偏移，以便位置 float2(0,0) 在屏幕的中心。

- 平铺：返回屏幕位置偏移量，以便位置 float2(0,0) 位于屏幕中心并使用 frac 进行平铺。

**Port**

| Name | Direction | Type     | Binding | Description                 |
| :--- | :-------- | :------- | :------ | :-------------------------- |
| Out  | Output    | Vector 4 | None    | Mesh's **Screen Position**. |

**Controls**

| Name | Type     | Options                     | Description                                         |
| :--- | :------- | :-------------------------- | :-------------------------------------------------- |
| Mode | Dropdown | Default, Raw, Center, Tiled | Selects coordinate space of **Position** to output. |



#### 实现



![holo_p1]({{site.url}}\assets\shader-graph-basic\holo_p1.jpg)



- 使用 Sample Texture 2D 给 PBR Master 的 Alpha 赋值
- 使用 Screen Position 得到屏幕位置并重写 UV



### 立体感

#### 思路

利用菲涅尔效应使得物体具有立体感。



#### 结点

**Fresnel Effect Node**

*Math $\rightarrow$ Vector $\rightarrow$ Fresnel Effect*

菲涅耳效应为在不同的视角下物体表面的反射率不同，当接近掠射角时，物体表面会反射更多的光。

通过计算表面法线（surface normal）和视图方向（view direction）之间的角度来对此进行近似，该角度与返回值成正比。

**Ports**

| Name     | Direction | Type     | Description                                                  |
| :------- | :-------- | :------- | :----------------------------------------------------------- |
| Normal   | Input     | Vector 3 | Normal direction. By default bound to World Space Normal     |
| View Dir | Input     | Vector 3 | View direction. By default bound to World Space View Direction |
| Power    | Input     | Vector 1 | Exponent of the power calculation                            |
| Out      | Output    | Vector 1 | Output value                                                 |



**One Minus Node**

*Math $\rightarrow$ Range $\rightarrow$ One Minus*

返回 1 - 输入的值。

**Ports**

| Name | Direction | Type           | Description  |
| :--- | :-------- | :------------- | :----------- |
| In   | Input     | Dynamic Vector | Input value  |
| Out  | Output    | Dynamic Vector | Output value |



#### 实现



![holo_p2]({{site.url}}\assets\shader-graph-basic\holo_p2.jpg)



- 将菲涅尔效果设置为自发光营造立体感
- 将 Texture 通过一个 One Minus 反转灰度并加入自发光效果



### 闪烁

#### 思路





#### 结点

**Random Range Node**

*Math $\rightarrow$ Range $\rightarrow$ Random Range*

返回一个根据输入的 Seed 产生的伪随机数，该值介于分别由输入 Min 和 Max 定义的最小值和最大值之间。

虽然输入相同的 Seed 将总是产生相同的输出值，但输出值本身将显示为随机值。Input Seed 是 Vector 2 类型的值，以方便基于 UV 输入生成随机数，但是在大多数情况下，Vector 1类型的输入就足够了。

**Ports**

| Name | Direction | Type     | Description                    |
| :--- | :-------- | :------- | :----------------------------- |
| Seed | Input     | Vector 2 | Seed value used for generation |
| Min  | Input     | Vector 1 | Minimum value                  |
| Max  | Input     | Vector 1 | Maximum value                  |
| Out  | Output    | Vector 1 | Output value                   |



**Comparison Node**

*Utility $\rightarrow$ Logic $\rightarrow$ Comparison*

比较两个输入值，输出一个布尔类型的比较结果。

通常为 Branch Node 的输入。

**Ports**

| Name | Direction | Type     | Binding | Description        |
| :--- | :-------- | :------- | :------ | :----------------- |
| A    | Input     | Vector 1 | None    | First input value  |
| B    | Input     | Vector 1 | None    | Second input value |
| Out  | Output    | Boolean  | None    | Output value       |



**Branch Node**

*Utility $\rightarrow$ Logic $\rightarrow$ Branch*

为着色器提供一个动态分支。

若输入谓词为真，输出与 True 输入相同，否则与 False 输入相同。

根据所处的着色器阶段确定顶点或像素。

Branch 两侧都将在 shader 中评估，未使用的分支将被丢弃。

**Ports**

| Name      | Direction | Type           | Binding | Description                        |
| :-------- | :-------- | :------------- | :------ | :--------------------------------- |
| Predicate | Input     | Boolean        | None    | Determines which input to returned |
| True      | Input     | Dynamic Vector | None    | Returned if **Predicate** is true  |
| False     | Input     | Dynamic Vector | None    | Returned if **Predicate** is false |
| Out       | Output    | Boolean        | None    | Output value                       |



#### 实现



![holo_p3]({{site.url}}\assets\shader-graph-basic\holo_p3.jpg)



- 用时间作为随机种子生成 $(0,1)$ 之间的随机数
- 为产生的随机数设置阈值降低闪烁的频率
- 得到的闪烁频率与自发光颜色相乘



### 最终效果



![holo]({{site.url}}\assets\shader-graph-basic\holo.jpg)



效果展示（消融与全息）：



![sg]({{site.url}}\assets\shader-graph-basic\shader_graph.gif)