---
layout: post
title:  "Blender基础"
date:   2020-11-04 21:35:00 +0800
tags: Blender
color: rgb(255,90,90)
cover: '../assets/blender-guru-donut-part1/d13.jpg'
subtitle: '捏一个甜甜圈'
---
Blender入门，内容为 [Blender Guru][link-1] 的 [tutorial][link-2] 的第一部分。

[link-1]: https://www.youtube.com/channel/UCOKHwx1VCdgnxwbjyb9Iu1g
[link-2]: https://www.youtube.com/watch?v=NyJWoyVx_XI&list=PLjEaoINr3zgEq0u2MzVgAaHEBt--xLB6U&index=1&ab_channel=BlenderGuru



## 内容梗概

基础操作（Basic Operatio）

建模（Modelling）

雕刻（Sculpting）

光线（Lighting）

渲染（Rendering）

材质（Materials）



## 基础操作（Basic Operation）

移动 

- *G*（Grab）

旋转

- *R*（Rotate）

缩放

- *S*（Scale）

拉伸

- *E*（Extrude）

确认

- *Left Mouse Button*

取消

- *Right Mouse Button*

在指定坐标轴移动

- *G + X / Y / Z* 
- *G + Middle Mouse Button*

旋转视图（Orbiting）

- *MMB*

平移视图 

- *Shift + MMB*

复制物体（Objects） 

- *Shift + D*（Duplicate）

选中物体的所有顶点

- *Ctrl + L*
- *A*

选中一条边上的所有顶点（Edge Select）

- *Alt + LMB*

聚焦物体（Zoom）

- *Num Pad "."*

视角功能键  

- *Tilde* （`）

切换到左/中/上视图

-  *Num Pad "3"/"1"/"7"*
- *Alt + MMB + direction*
- *Tilde (`)*

删除物体 

- *X*

添加物体 

- *Shift + A*（Add）

查找 

- *F3*





**一个着火的猴子脑袋**

添加物体 $\longrightarrow$ 选择猴子脑袋 $\longrightarrow$ 查找烟雾特效 $\longrightarrow$ 选中猴子脑袋 $\longrightarrow$ 在物理属性面板的 Flow Type 一项选择 Fire + Smoke

播放动画得到效果：



![mkh]({{site.url}}\assets\blender-guru-donut-part1\mkh.jpg)



## 建模（Modelling）

### 捏一个甜甜圈！

新建一个圆环体（Torus），在其基础属性面板（选中 Object 后按 *F9* 呼出）进行初步设置。



![add_setting]({{site.url}}\assets\blender-guru-donut-part1\add_setting.jpg)



建模时最好将物体的尺寸设置为现实中的实际大小。**面**（Segments）的数量应在可表现物体形状的前提下尽可能少。

得到一个圆环体。



![d1]({{site.url}}\assets\blender-guru-donut-part1\d1.png)



当然现实中的甜甜圈是不可能这么完美的，需要将圆环体变成一个不那么规则的甜甜圈。

选中物体，切换至**编辑模式**（Edit Mode）（热键 *Tab* ）。

在编辑模式下按 *O* 开启**依比例编辑**（Proportional Edit）（热键 *O* ）。

选中可编辑的点，按 *G* 进行拖拽，使用鼠标滚轮调整比例大小，得到想要的效果。



![d2]({{site.url}}\assets\blender-guru-donut-part1\d2.jpg)



为甜甜圈进行细分，使其表面更加光滑。

在 **Properties** 一栏中选择 **Modifier Properties**（图标是一个蓝色扳手）$\longrightarrow$ 在 **Add Modifier** 栏中选择 **Subdivision Surface**



![sdmp]({{site.url}}\assets\blender-guru-donut-part1\sdmp.jpg)



**Levels Viewport** 属性为细分程度。

得到效果如下。



![d3]({{site.url}}\assets\blender-guru-donut-part1\d3.jpg)



在编辑模式中可以直观地看到 Subdivision Modifier 是如何插入新的面进行细分的。



![subd_detail]({{site.url}}\assets\blender-guru-donut-part1\subd_detail.jpg)



在 Object Mode 下右键选中物体选择 shade smooth 可使物体表面光滑渲染。



![d4]({{site.url}}\assets\blender-guru-donut-part1\d4.jpg)



### 给甜甜圈加上糖霜

进入编辑模式，打开 **X-Ray Mode**（热键 *Alt + Z*）。调整至正视图。

由于**剪裁**（clipping）的初始值为 0.01m，而物体的尺寸很小（主半径 0.05m），造成 X-Ray Mode 下物体表面很容易被切割。解决方法为调出场景属性（热键 *N*），选择 View 一栏，将 Clipping Start 值调小。



![X-Ray]({{site.url}}\assets\blender-guru-donut-part1\X-Ray.jpg)



选中上半部分的所有 vertex，复制顶点（热键 *Shift + D*），右键取消移动。

将复制的顶点分离到新对象（热键 *P*）。



![d5]({{site.url}}\assets\blender-guru-donut-part1\d5.jpg)



给糖霜添加厚度。

选中新建的物体，在 Properties 一栏中选择 Modifier Properties $\longrightarrow$ 在 Add Modifier 栏中选择 **Solidify** $\longrightarrow$ 将 offset 值调为 1（此时厚度（Thickness）的值直接作用与物体） $\longrightarrow$ 将 Thickness 值调整为合适的值。

此时糖霜的边缘会很锐利，因为 Modifier Properties 的加载顺序为从上至下（先渲染 Subdivision Surface 然后再渲染 Solidify）。将 Solidify 拖至 Subdivision Surface 上方。



![d6]({{site.url}}\assets\blender-guru-donut-part1\d6.jpg)



制造糖霜从液体状态凝固的效果。

在 Modifier Properties 一栏中取消选中 Edit Mode，这样在 Edit Mode 中就不会渲染 Solidify 的效果导致顶点被遮蔽。

为了更好的增加细节，在 Edit Mode 中选择全部的顶点，右键选择 Subdivision 对其进行细分。

选中最下方边的边上的所有顶点（*Alt* + *MLB*点击要选中的边），反选中其他所有顶点（热键 *Ctrl + I*），隐藏这些顶点（热键 *H*，显示隐藏顶点 *Alt + H*）。



![hidden]({{site.url}}\assets\blender-guru-donut-part1\hidden.jpg)



开启 **Snap**（热键 *Shift + Tab*），在 snap 的属性面板的 Snap to 一栏选择 Face，Snap with 一栏选择 Project onto Self 以及 Project Individual Elements（所有顶点在 Grabbing 时都附着在表面上）。



![snap]({{site.url}}\assets\blender-guru-donut-part1\snap.jpg)



为边缘做出为液体缓慢流下并凝固的样子。



![d7]({{site.url}}\assets\blender-guru-donut-part1\d7.jpg)



选中边缘顶点，向下进行**拉伸**（Extrude）（热键 *E*），做出液体快速流下并凝固的样子。



![d8]({{site.url}}\assets\blender-guru-donut-part1\d8.jpg)



糖霜的边缘和甜甜圈是分离的。



![separate]({{site.url}}\assets\blender-guru-donut-part1\separate.jpg)



选中糖霜，在 Properties 一栏中选择 Modifier Properties $\longrightarrow$ 在 Solidify Pad 中打开 Edge Data 一栏 $\longrightarrow$ 将 Crease Inner 值调为 1。



![Inner]({{site.url}}\assets\blender-guru-donut-part1\Inner.jpg)



## 雕刻（Sculpting）

在雕刻之前，对目前的甜甜圈进行备份，复制后将其放在一个新的 Collection 下（热键 *M*，选择 +New Collection）。

将备份隐藏（操作与 ps 图层相似）。

在雕刻之前，将物体的 Subdivision Modifier 中 Levels View 值提高到 2 来增加细节。

将要雕刻的物体的 Modifier Properties 应用（Apply）（热键 *Ctrl + A* 呼出 Apply 菜单选择 Visual geometry to mesh）之后进行雕刻。



还原由于油炸不均匀导致的上下两面比中间更蓬松的效果。

使用 Draw工具，*Ctrl + LMB* 进行雕刻，*F* 改变雕刻工具的半径，*Shift + F* 改变雕刻的强度。



![d9]({{site.url}}\assets\blender-guru-donut-part1\d9.jpg)



雕刻时物体的材质可以在编辑面板右上角的圆球标志更改。



![c_m]({{site.url}}\assets\blender-guru-donut-part1\change_m.jpg)



为糖霜增加液体流下时最低处由于重力作用而形成的类似水珠的细节。

涂抹糖霜表面使其富有层次。

将糖霜的 Subdivision Modifier 中 Levels View 提高到 3。

使用 Inflate 工具雕出水滴形状。

在糖霜上表面用 Draw 工具增加细节，使用 Smooth 工具对过度雕刻的地方进行平滑。



![d10]({{site.url}}\assets\blender-guru-donut-part1\d10.jpg)



## 光线（Lighting）&  渲染（Rendering）

3D 坐标（3D Cursor）是新建物体的位置。

*Shift + LMB* 移动 3D 坐标的位置。

*Shift + C* 将 3D 坐标设置为初始位置。



新建一个平面（plane）。

*Shift + A* $\longrightarrow$ Mesh $\longrightarrow$ Plane。



将甜甜圈移动到平面上。

把糖霜和甜甜圈做一个移动时的绑定（parenting）。

首先选中糖霜（先选中的物体为 child），*Shift + LMB* 选中甜甜圈（后选中的物体为 parent）$\longrightarrow$ *Ctrl + P* $\longrightarrow$ 选择 Object (Keep Transform)。



将光源（Light）移动到合适的位置，选中 Light，在属性面板可以调整光源的参数（图标为绿色的小灯泡）。



在属性面板找到渲染属性（Render Properties）（图标为微波炉），选择 Cycles 作为渲染引擎并使用 GPU 计算。



![d11]({{site.url}}\assets\blender-guru-donut-part1\d11.jpg)



点击编辑界面右上角的摄像机按钮或者使用热键 *Num Pad 0* 切换至摄像机视角。



![c](C:\Users\Hyper\Desktop\Notes\blender\assets\camera.png)



在摄像机视角下，选择摄像机，使用热键 *G* 进行摄像机位置的移动，在移动的状态下单击 *MMB* 进入 Zoom 状态移动鼠标进行放缩。

进入编辑界面属性面板（热键 *N*），在 View 一栏中选中 Camera to View 选项，则在 Camera 视角下通过调整相对位置来改变相机位置（与编辑界面操作逻辑相同）。

或者在编辑视角确定画面后使用热键 *Ctrl + Alt + Num Pad 0* 进行捕捉。

确定好 Camera 中的画面后，在左上方的菜单栏中选择 Render $\longrightarrow$ Render Image（热键 *F12*）渲染得到图片。

在生成图片的窗口的菜单栏中选择 Image $\longrightarrow$ Save 对图片进行保存。



![d12]({{site.url}}\assets\blender-guru-donut-part1\d12.jpg)



## 材质（Materials）

选中糖霜，在 Material Properties（图标为红色的材质球）中添加一个新的材质。

将 Base Color 设置为粉红色。

**Specular**（高光）为物体会反射多少射入的光线。虽然调整 Specular 的值可以改变物体的反光效果（0 为亚光，1 为镜面），但是由于现实中物体都会反射光线，故不建议通过更改 Specular 的值来调整物体的反光效果。

**Roughness** （粗糙度）为物体表面的粗糙程度。表面越粗糙，物体反射光的方式更偏向于漫反射，表面越光滑，物体反射光的方式更偏向于镜面反射。

**Subsurface** 为透光，类似于强光透过耳廓使耳廓变红的效果。

**Subsurface Radius** 有三个值分别对应 RGB 的透光率。

**Subsurface Color** 为透光部分的本色。



再分别给甜甜圈和背景上色，得到一个粉色桌面上的草莓味甜甜圈。

调整光线和摄像机，渲染出图片。



![d13]({{site.url}}\assets\blender-guru-donut-part1\d13.jpg)

