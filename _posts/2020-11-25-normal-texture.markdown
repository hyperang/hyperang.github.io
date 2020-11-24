---
layout: post
title:  "法线纹理"
date:   2020-11-25 02:35:00 +0800
tags: 图形学
color: rgb(255,90,90)
cover: '../assets/normal_map/normal_mapping_normal_map.png'
subtitle: '原理浅析和实践'
---
光照模型部分主要参考了教程[LearnOpenGL CN][link-1]中基础光照和高级光照两章中的内容。
法线纹理部分主要参考了[Unity Shader入门精要][link-2](冯乐乐著)中凹凸映射一节中的内容。

[link-1]: https://learnopengl-cn.github.io/
[link-2]: https://github.com/candycat1992/Unity_Shaders_Book

## 光照模型

基础光照模型的主要结构由3个分量组成：环境(Ambient)、漫反射(Diffuse)和镜面(Specular)光照。

![phong]({{site.url}}\assets\normal_map\basic_lighting_phong.png)

- 环境光照(Ambient Lighting)：即使在黑暗的情况下，世界上通常也仍然有一些光亮（月亮、远处的光），所以物体几乎永远不会是完全黑暗的。为了模拟这种光照，我们会使用一个环境光照常量，它永远会给物体一些颜色。
- 漫反射光照(Diffuse Lighting)：模拟光源对物体的方向性影响(Directional Impact)。它是冯氏光照模型中视觉上最显著的分量。物体的某一部分越是正对着光源，它就会越亮。
- 镜面光照(Specular Lighting)：模拟有光泽物体上面出现的亮点。镜面光照的颜色相比于物体的颜色会更倾向于光的颜色。

### 环境光照

环境光照是一个常量。

### 漫反射光照

假设环境中只有一个光源。

漫反射的特点为，不管从哪个方向看，漫反射的颜色都相同。即漫反射与观测方向无关。

漫反射的强度仅与光源和法向量有关。

![diffuse]({{site.url}}\assets\normal_map\diffuse_light.png)

对物体的一个微分面（一个片元）的法向量
$\bf{n}$
与光源方向向量（指向光源）
$\bf{l}$
之间的夹角
$\theta$
有：

- 越接近
$90^{\circ}$
，漫反射亮度越接近
$0$
。
- 越接近
$0^{\circ}$
，漫反射亮度越接近最大值。
- 大于等于
$90^{\circ}$
，漫反射亮度为
$0$
。

### 镜面光照

镜面光照依据光的方向向量、物体的法向量和观察方向向量来决定。

![specular]({{site.url}}\assets\normal_map\basic_lighting_specular_theory.png)

向量
$\bf{r}$
是光线的反射方向向量，
$\bf{r}=2(\bf{n}\cdot\bf{l})\bf{n}-\bf{l}$
。

对反射方向向量
$\bf{r}$
与观察方向向量（指向观察者）
$\bf{v}$
之间的夹角
$\theta$
有：

- 越接近
$90^{\circ}$
，漫反射亮度越接近
$0$
。
- 越接近
$0^{\circ}$
，漫反射亮度越接近最大值。
- 大于等于
$90^{\circ}$
，漫反射亮度为
$0$
。

上述的计算方法存在一定的问题。

![problem]({{site.url}}\assets\normal_map\advanced_lighting_over_90.png)

如图，当视线与反射方向之间的夹角大于90度，下镜面光分量会变为
$0$
。当物体的反光度非常小时，它产生的镜面高光半径足以让这些相反方向的光线对亮度产生足够大的影响。在这种情况下就不能忽略它们对镜面光分量的贡献了。

#### Blinn-Phong着色模型

Blinn-Phong模型不再依赖于反射向量，而是采用了所谓的半程向量(Halfway Vector)，即光线与视线夹角一半方向上的一个单位向量。当半程向量与法线向量越接近时，镜面光分量就越大。

![blinn]({{site.url}}\assets\normal_map\advanced_lighting_halfway_vector.png)

半程向量
$\bf{h}=\frac{\bf{l}+\bf{v}}{|\bf{l}+\bf{v}|}$
。

镜面反射的强度由半程向量
$\bf{h}$
和法向量
$\bf{n}$
之间的夹角
$\theta$
决定，此时不论观察者向哪个方向看，
$\theta$
都不会超过
$90^{\circ}$
（除非光源在表面以下）。

## 法线纹理原理

由光照模型，漫反射光照和镜面光照强度都需要法向量
$\bf{n}$
才能决定。

法线纹理的原理就是改变每一个片元的法向量
$\bf{n}$
的值来给平面增加细节，使得平面看上去是凹凸起伏而非平坦光滑。

![normal]({{site.url}}\assets\normal_map\normal_mapping_surfaces.png)

法线纹理中存储的就是表面的法线方向。由于法向量的分量范围是
$[-1,1]$
而像素的分量范围是
$[0,1]$
，因此需要将法向量的分量做如下映射。
$$
pixel=\frac{normal+1}{2}
$$
如果法线纹理中存储的法线方向定义在模型空间中（模型顶点自带的法线信息即定义在模型空间中），则这种法线纹理称为**模型空间的法线纹理**。由于模型空间中每个片元的法线方向是各异的，故模型空间的法线纹理看起来是五彩斑斓的。

![object-space]({{site.url}}\assets\normal_map\gargoyle-uv_world_space.jpg)

## 切线空间

对于模型的每一个顶点，都有一个属于自己的切线空间。

![tangent_space]({{site.url}}\assets\normal_map\tangent_space.jpg)

在切线空间中，顶点为原点，顶点的法线永远指着正$z$方向。$x$轴为顶点的切线（Tangent）方向
$\bf{t}$
，
$y$
轴为顶点的副切线（Bitangent）方向
$\bf{b}$
。可以用右手定则来确定顶点的切线空间。

![right_hand]({{site.url}}\assets\normal_map\right_hand.jpg)

中指指向法线方向，则拇指为切线方向，食指为副切线方向。

如果法线纹理中存储的法线方向定义在切线空间中，则这种法线纹理称为**切线空间的法线纹理**。

故若一个点的法线方向在切线空间中并没有改变，则法线在切线空间中为正
$z$
轴，即其值为
$(0,0,1)$
，经过映射后存储在纹理中对应了RGB
$(0.5,0.5,1)$
浅蓝色。这就是法线纹理中有大面积的浅蓝色的原因。而切线空间中的法线在
$z$
上的值总大于
$0$
，映射到纹理的RGB后蓝色通道的值总大于
$0.5$
，故切线空间下的法线纹理整体偏蓝。

![tangent_space]({{site.url}}\assets\normal_map\gargoyle-uv_tangent_space.jpg)



### ShaderLab中切线空间的计算

在Vertex Shader的输入结构体中定义顶点的法线方向和切线方向。输入结构体中的数据皆在模型空间中。

```csharp
struct appdata {
    ...
    float3 normal : NORMAL;
    float4 tangent : TANGENT;
    ...
}
```

使用切线方向和法线方向叉乘即可得到副切线方向。变量`tangent`是`float4`类型的，这是因为`tangent.w`分量决定了副切线的方向性。

```csharp
float3 bitangent = cross(normalize(normal), normalize(tangent.xyz)) * tangent.w;
```

### ShaderLab中在切线空间的光照计算

在Vertex Shader中计算切线空间中的光源方向
$\bf{l}$
和观测方向
$\bf{v}$
。

使用`UnityCG.cginc`中定义的`TANGENT_SPACE_ROTATION`得到变换矩阵`rotation`。

```glsl
// Declares 3x3 matrix 'rotation', filled with tangent space basis
#define TANGENT_SPACE_ROTATION
    float3 binormal = cross( normalize(v.normal), normalize(v.tangent.xyz) ) * v.tangent.w;
    float3x3 rotation = float3x3( v.tangent.xyz, binormal, v.normal )
```

使用`rotation`得到
$\bf{l}$
和
$\bf{v}$
。

```glsl
// Vertex Shader
v2f vert(appdata v) {
    v2f o;
    ...
    TANGENT_SPACE_ROTATION;
    o.lightDir = mul(rotation, ObjSpaceLightDir(v.vertex)).xyz;
    o.viewDir = mul(rotation, ObjSpaceViewDir(v.vertex)).xyz;
    ...
}
```

在Fragment Shader中对切线空间的法线纹理中的法线方向
$\bf{n}$
进行采样并计算得到片元的颜色。

为使
$\bf{n}$
在切线空间中的
$z$
方向大于
$0$
，使用`sqrt()`函数计算得到
$z$
方向的分量。

```glsl
// Fragment Shader
fixed4 frag(v2f i) : SV_Target {
    ...
    fixed4 packedNormal = tex2D(_BumpMap, i.uv);
    // if the texture marked as "Normal Map"
    fixed3 tangentNormal = UnpackNormal(packedNormal);
    tangentNormal.xy *= _BumpScale;
    // else
    // tangentNormal.xy = (packedNormal.xy * 2 - 1) * _BumpScale;
    tangentNormal.z = sqrt(1.0 - saturate(dot(tangentNormal.xy, tangentNormal.xy)));
    
    fixed3 tangentLightDir = normalize(i.lightDir);
    fixed3 tangentViewDir = normalize(i.viewDir);
    ...
}
```

### ShaderLab中在世界空间使用切线空间的法线纹理

在Vertex Shader中将顶点的切线坐标转换为世界坐标。

```glsl
// Vertex Shader
v2f vert(appdata v) {
    v2f o;
    ...
    o.tangent = v.tangent.xyz;
    o.normal = v.normal;
    o.binormal = cross(normalize(v.normal), normalize(v.tangent.xyz)) * v.tangent.w;
    ...
}
```

在Fragment Shader中将切线空间的法线纹理中的法线方向
$\bf{n}$
进行采样并转换为世界坐标。

```glsl
fixed4 frag(v2f i) : SV_Target {
    ...
    float3x3 rotation = float3x3(i.tangent, i.binormal, i.normal);
    fixed4 packedNormal = tex2D(_BumpMap, i.uv);
    fixed3 normal = UnpackNormal(packedNormal);
    normal.xy *= _BumpScale;
    normal.z = sqrt(1.0 - saturate(dot(normal.xy, normal.xy)));
    normal = normalize(mul(normal, rotation));
    ...
}
```
