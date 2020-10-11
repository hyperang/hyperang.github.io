---
layout: post
title:  "Ray Marching Basic"
date:   2020-10-11 18:00:00 +0800
tags: ray marching
color: rgb(255,90,90)
cover: '../assets/First Ball.jpg'
subtitle: 'First Ball'
---
Ray Marching入门，主要参考了 tutorial：https://www.youtube.com/watch?v=PGtv-dBi2wE&t=273s



## 主函数

```glsl
void mainImage(out vec4 fragColor, in vec2 fragCoord)
{
    // Normalized pixel coordinates (from 0 to 1)
	vec2 uv = (fragCoord - 0.5 * iResolution.xy) / min(iResolution.x, iResolution.y);
    
    // simple camera system
    // ray origion(camera position)
    vec3 ro = vec3(0.0, 1.0, -0.5);
    // ray direction
    vec3 rd = normalize(vec3(uv, 1));

    float d = RayMarch(ro, rd);
    d /= 8.;
    vec3 col = vec3(d);

    // Output to screen
    fragColor = vec4(col,1.0);
}
```



主函数 mainImage 输入为一个类型为二维向量的片元坐标，输出一个类型为四维向量的片元颜色。函数每一次计算一个像素的颜色值。

第 4 行用输入的片元坐标除以屏幕的分辨率得到归一化的坐标 uv。

第 4 行的代码可以等同于以下代码：

```glsl
// 将坐标转换到[0, 1]之间
vec2 uv = fragCoord.xy / iResolution.xy;
// 将坐标转换到[-0.5, 0.5]之间，中央为原点(0，0)
uv = uv - 0.5;
// 保持短边为[-0.5, 0.5]，长的那条边坐标相应放大
if (iResolution.x > iResolution.y) {
    uv.x *= iResolution.x / iResolution.y;
} else {
    uv.y *= iResolution.y / iResolution.x;
}
```

第 8 行定义相机的位置。第 10 行定义本次计算的像素的射线的方向，由于屏幕中心的坐标已经对应了世界坐标中的原点，故相机的视锥如下图，rd 的 z 值相当于相机的焦距。

第 12 行调用 RayMarch 函数计算射线碰到的第一个物体的距离。第 14 行将输出颜色的 rgb 都设为距离 d，得到一张灰度图，越靠近相机的物体越黑，反之月白。第 13 行通过调整 d 的值来调节物体的明暗。



## RayMarch

RayMarch 函数计算射线碰到的第一个物体的距离。计算过程如下图。



<img src="..\assets\Ray Marching Basic\ray marching 1 sphere tracing.jpg" />



```glsl
float RayMarch(vec3 ro, vec3 rd) {
    float dO = 0.;
    
    // marching loop
    for(int i = 0; i < MAX_STEPS; i++) {
    	vec3 p = ro + rd * dO;
        float dS = GetDist(p);
        dO += dS;
        if(dO > MAX_DIST || dS < SURF_DIST) break;
    }
    
    return dO;
}
```



第 2 行定义了初始距离（distance origin）为 0。在 marching loop 中，定义位置变量 p，其初始值为 ro，即上图中相机所在的蓝色标记处。通过 GetDist 函数得到 p 到任以物体的最小距离 dS 并更新总距离 dO 和位置 p（上图中射线上的蓝色标记）。如果总距离超出范围或者最小距离小于判定则返回。

在 GetDist 函数中，我们定义出球和地面的位置并返回与某个位置最近的物体的距离。



![1-3](..\assets\Ray Marching Basic\ray marching 2 distance.jpg)



## 定义空间中的物体

```glsl
float GetDist(vec3 p) {
    vec4 s = vec4(0, 1, 6, 1);
    
    float sphereDist = length(p - s.xyz) - s.w;
    float planeDist = p.y;
    
    float d = min(sphereDist, planeDist);
    return d;
}    
```



第 2 行定义一个球，前三维是球心的坐标，第四维是球的半径。地面设定为 xOz，故 p 到地面的距离就是 p.y。p 到球面的距离为 p 到球心的距离减去球的半径。由于只有地面和球两个物体，于是返回到两者的距离中小的那个。

至此，fragment shader 的输出如下图所示。



![1-4](..\assets\Ray Marching Basic\out_put_1.PNG)



## 光照

定义一个简单的光照系统，用光照的值来替代距离灰度。光照系统的原理如下图所示。



![1-5](..\assets\Ray Marching Basic\ray marching 4 lighting.jpg)



假设从光源发出的每一束光的能量相同，以 6 束为一组，带有角度照射远处的地面照亮的范围比垂直照射地面的范围更大，故此时光线的亮度更低。



![1-6](..\assets\Ray Marching Basic\ray marching 5 lighting's code.jpg)



当光线照射到物体一点时，这一点到光线的方向记为 light vector，这一点的法线方向记为 normal vector，若两者方向一致则光线强度为 1，如果两者垂直则光线强度为 0。计算光线强度可以点积两个向量。



定义 GetLight 函数，输入一个三维向量的位置 p，输出 float 类型的在该位置的光线强度。



```glsl
float GetLight(vec3 p) {
	vec3 lightPos = vec3(1, 5, 6);
    lightPos.xz += vec2(sin(iTime), cos(iTime)) * 2.;
    vec3 l = normalize(lightPos - p);
    vec3 n = GetNormal(p);
    
    float diffuse = dot(n, l);
    return diffuse;
}
```



第 2 行定义光源的位置 lightPos。

第 3 行使光源旋转。

第 4 行定义 light vector，即光源位置减去目标位置并归一化。

第 5 行调用 GetNormal 函数定义 normal vector。



定义 GetNormal 函数，输入一个三维向量的位置 p，输出一个三维向量类型的 p 位置的法向量。



```glsl
vec3 GetNormal(vec3 p) {
    float d = GetDist(p);
    vec2 e = vec2(.01, 0);
    
    vec3 n = d - vec3(GetDist(p-e.xyy),
                      GetDist(p-e.yxy),
                      GetDist(p-e.yyx));
    
    return normalize(n);
}
```



第 2 行调用 GetDist 函数得到位置 p 到物体的最小距离 d。

第 5 行计算 p 周围的点到物体表面的距离。通过分别计算三个轴方向上附近（固定距离）的点到表面的距离，用 p 点到表面的距离减去这三个距离作为三维向量对应轴的值，再进行归一化得到法向量。



```glsl
float d = RayMarch(ro, rd);

vec3 p = ro + rd * d;
float l = GetLight(p);
vec3 col = vec3(l);

fragColor = vec4(col,1.0);
```



在主函数中用光线模型代替以距离为值的灰度，得到效果如下图。



![1-8](..\assets\Ray Marching Basic\out_put_2.jpg)



## 阴影

添加阴影效果，原理如下图所示。



![1-9](..\assets\Ray Marching Basic\ray marching 6 shadow.jpg)



修改 GetLight 函数



```glsl
float GetLight(vec3 p) {
	vec3 lightPos = vec3(1, 5, 6);
    lightPos.xz += vec2(sin(iTime), cos(iTime)) * 2.;
    vec3 l = normalize(lightPos - p);
    vec3 n = GetNormal(p);
    
    float dif = clamp(dot(n, l), 0.0, 1.0);
    float d = RayMarch(p + n * SURF_DIST * 2., l);
    if(d < length(lightPos - p)) dif *= 0.1;
    return dif;
}
```



第 7 行将光线的值限定在 0 到 1 之间。

第 8 行调用 RayMarch 函数计算 p 位置沿着 light vector 方向到任何物体的距离。

第 9 行判断：如果位于阴影中则距离 d 小于位置 p 和光源位置 lightPos 的距离，将光线的值调整为黑色。

最后的效果如下图。



![1-10](..\assets\Ray Marching Basic\First Ball.jpg)



完整代码如下。



```glsl
/* 着色器输入
uniform vec3      iResolution;           // viewport resolution (in pixels)
uniform float     iTime;                 // shader playback time (in seconds)
uniform float     iTimeDelta;            // render time (in seconds)
uniform int       iFrame;                // shader playback frame
uniform float     iChannelTime[4];       // channel playback time (in seconds)
uniform vec3      iChannelResolution[4]; // channel resolution (in pixels)
uniform vec4      iMouse;                // mouse pixel coords. xy: current (if MLB down), zw: click
uniform samplerXX iChannel0..3;          // input channel. XX = 2D/Cube
uniform vec4      iDate;                 // (year, month, day, time in seconds)
uniform float     iSampleRate;           // sound sample rate (i.e., 44100)
**/

#define MAX_STEPS 100
#define MAX_DIST 100.
#define SURF_DIST .01

float GetDist(vec3 p) {
    vec4 s = vec4(0, 1, 6, 1);
    
    float sphereDist = length(p - s.xyz) - s.w;
    float planeDist = p.y;
    
    float d = min(sphereDist, planeDist);
    return d;
}    

float RayMarch(vec3 ro, vec3 rd) {
    float dO = 0.;
    
    // marching loop
    for(int i = 0; i < MAX_STEPS; i++) {
    	vec3 p = ro + rd * dO;
        float dS = GetDist(p);
        dO += dS;
        if(dO > MAX_DIST || dS < SURF_DIST) break;
    }
    
    return dO;
}

vec3 GetNormal(vec3 p) {
    float d = GetDist(p);
    vec2 e = vec2(.01, 0);
    
    vec3 n = d - vec3(GetDist(p-e.xyy),
                      GetDist(p-e.yxy),
                      GetDist(p-e.yyx));
    
    return normalize(n);
}
    
float GetLight(vec3 p) {
	vec3 lightPos = vec3(1, 5, 6);
    lightPos.xz += vec2(sin(iTime), cos(iTime)) * 2.;
    vec3 l = normalize(lightPos - p);
    vec3 n = GetNormal(p);
    
    float dif = clamp(dot(n, l), 0.0, 1.0);
    float d = RayMarch(p + n * SURF_DIST * 2., l);
    if(d < length(lightPos - p)) dif *= 0.1;
    return dif;
}

void mainImage(out vec4 fragColor, in vec2 fragCoord)
{
    // Normalized pixel coordinates (from 0 to 1)
    vec2 uv = (fragCoord - 0.5 * iResolution.xy) / iResolution.y;
    
    // simple camera system
    // ray origion(camera position)
    vec3 ro = vec3(0.0, 1.0, -0.5);
    // ray direction
    vec3 rd = normalize(vec3(uv, 1));

    float d = RayMarch(ro, rd);
    //d /= 8.;
    vec3 p = ro + rd * d;
    float l = GetLight(p);
    vec3 col = vec3(l);

    // Output to screen
    fragColor = vec4(col,1.0);
}
```

