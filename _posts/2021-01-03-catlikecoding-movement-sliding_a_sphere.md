---
layout: post
title:  "滑动一个球体"
date:   2021-01-03 10:24:36 +0800
tags: Unity
color: rgb(255,90,90)
cover: '../assets/catlikecoding-movement-sliding_a_sphere/title.jpg'
subtitle: '玩家控制的运动'
---
翻译作品，原文为[Catlike Coding][link-1]的[教程][link-2]。本文使用 Unity 2020.1.12f1c1 Personal 对文章内容进行复现。

[link-1]: https://catlikecoding.com/
[link-2]: https://catlikecoding.com/unity/tutorials/movement/sliding-a-sphere/

> 目标：
>
> - 将带有轨迹的球放在平面上
> - 根据玩家输入来定位球体
> - 控制速度和加速度
> - 限制球的位置，使其在平面的边缘反弹

# 1 控制位置

许多游戏都是通过角色四处移动来完成游戏中的目标的，玩家的任务是引导角色移动。动作游戏通常通过按下键盘或者转动摇杆来直接控制角色；在通过框选及点击操作的游戏中玩家选中一个目的地后角色会自动移动到该位置；编程游戏需要编写指令使角色执行，等等。

本教程系列将重点介绍如何在3D动作游戏中控制角色。我们从在一个小的平面矩形上滑动一个球体开始简单。熟练掌握之后，我们就可以完成更加复杂的场景。

![A sphere stuck on a plane]({{site.url}}\assets\catlikecoding-movement-sliding_a_sphere\title.jpg)

<center>限定在平面上运动的球体</center>

## 1.1 设置场景

新建一个默认的 3D 项目，不需要任何插件包，渲染管线也随意。

使用线性色彩空间，可以在 *Edit / Project Settings / Player / Other Settings* 中进行设置。

![Linear color space]({{site.url}}\assets\catlikecoding-movement-sliding_a_sphere\1.jpg)

<center>线性颜色空间</center>

保留默认样例场景中的相机和平行光光源。添加一个平面作为地面，再添加一个球体，两者都摆在初始位置。球体的默认半径为 0.5，故将球体 Y 坐标的值设为 0.5 使其看上去落在地面上。

![Scene hierarchy]({{site.url}}\assets\catlikecoding-movement-sliding_a_sphere\2.jpg)

<center>场景层次</center>

将球体的运动限制为地面上的 2D 运动。将相机朝向向下放置在平面的正上方，投影方式设置为正交。正交投影可以使玩家看到没有形变的 2D 运动。

![Orthographic camera looking down]({{site.url}}\assets\catlikecoding-movement-sliding_a_sphere\3.jpg)

<center>方向向下的正交投影相机</center>

将灯光的阴影类型设置为无或者无阴影（取决于Unity版本）来消除球体的阴影。

![Light casts no shadows]({{site.url}}\assets\catlikecoding-movement-sliding_a_sphere\4.jpg)

<center>光投无影</center>

为地面和球体添加材质并根据需要进行配置。原教程将球体设置为黑色；将地面设置为浅灰色，读者可自行设置。原教程为轨迹也创建了一个材质，本文使用默认的 `default-line` 材质。最后，创建一个 `MovingSphere` 脚本来实现球体的移动。

脚本可以从从一个空的 `MonoBehaviour` 开始拓展。

```c#
using UnityEngine;
public class MovingSphere : MonoBehaviour { }
```

![Game View]({{site.url}}\assets\catlikecoding-movement-sliding_a_sphere\5.jpg)

<center>游戏视角</center>

将 `TrailRenderer` 组件和 `MovingSphere` 脚本添加到球体中，其他保持不变。

![Sphere object with components]({{site.url}}\assets\catlikecoding-movement-sliding_a_sphere\6.jpg)

<center>球体的组件</center>

将轨迹材质分配给 `TrailRenderer` 组件的 `Materials` 数组的第一个也是唯一的元素（本文中使用 `Default-Line` 材质）。轨迹不需要阴影，所以将阴影禁用。另外，将宽度从 1.0 减小到更合理的值（如 0.1），这样就会生成细的轨迹。

![Trail renderer]({{site.url}}\assets\catlikecoding-movement-sliding_a_sphere\7.jpg)

<center>轨迹渲染组件</center>

尽管当前还没有进行任何针对运动的编程，但可以通过进入播放模式并在场景窗口中移动球体来预览效果。

![Movement trail]({{site.url}}\assets\catlikecoding-movement-sliding_a_sphere\8.jpg)

<center>运动轨迹</center>

## 1.2 读取玩家输入

要移动球体，就要读取玩家输入的命令。在 `MovingSphere` 的 `Update` 方法中执行读取操作。玩家输入的信息是 2D 的，故将其存储在 `Vector2` 类型的变量中。最初，将变量的 X 和 Y 分量都设置为 0，然后用这一组坐标将球体放置在 XZ 平面中。因此，输入的 Y 分量成为小球位置的 Z 分量。Y 分量保持为 0。

```c#
using UnityEngine;

public class MovingSphere : MonoBehaviour {

	void Update () {
		Vector2 playerInput;
		playerInput.x = 0f;
		playerInput.y = 0f;
		transform.localPosition = new Vector3(playerInput.x, 0.5f, playerInput.y);
	}
}
```

从玩家输入获取移动方向最简单的方法是调用带有轴名称的 `Input.GetAxis` 方法。Unity 定义了 `Horizontal` 和 `Vertical` 的映射，该定义可在项目设置的输入部分（*Edit / Project Settings / Input Manager*）进行检查和更改。将 `Horizontal` 映射为 `X axis`，`Vertical` 映射为 `Y axis`。

```c#
		playerInput.x = Input.GetAxis("Horizontal");
		playerInput.y = Input.GetAxis("Vertical");
```

默认设置将轴连接到方向键和 WASD 键。输入值会被调整所以使用键盘控制的感觉与摇杆相似。这些设置都可以根据需要进行调整，本文保留默认设置。

![Using arrow or WASD keys]({{site.url}}\assets\catlikecoding-movement-sliding_a_sphere\9.gif)

<center>使用键盘输入</center>

两个轴都拥有第二个定义，允许它们读取摇杆的输入。摇杆的输入比键盘更加流畅。不过除了下一个动画以外本文都使用键盘输入。

![Using controller stick]({{site.url}}\assets\catlikecoding-movement-sliding_a_sphere\10.gif)

<center>使用摇杆输入</center>

>为什么不使用输入系统插件包
>
>可以这样做，且原理与上述方法的原理相同。所需的只是检索两个轴的值。

## 1.3 输入向量归一化

轴的值在没有输入时返回 0，在极限时返回 -1 或 1。当使用键盘进行输入时，球体的运动范围被限制在一个正方形内。如果是使用摇杆输入，则通常在任何方向上都被限制为距离原点的最大距离为 1，球体的运动范围被限制在一个圆内（笔者复现的时候并非为标准的圆）。

使用摇杆输入的优点是无论方向，输入向量的最大长度始终为 1。因此，各个方向上的移动速度相同。键盘与摇杆不同，单个键的最大值为 1，而同时按下两个键的最大值为$\sqrt 2$，即沿对角线方向移动的速度最快。

由勾股定理可得键盘输入的最大值为$\sqrt 2$。轴的值定义了直角三角形两个直角边的长度，斜边即为输入的大小。输入向量的大小为$\sqrt{x^2+y^2}$。

通过把输入矢量除以其大小即可确保矢量为单位向量（除非其初始长度为零）。可以通过对向量调用方法 `Normalize` 来实现此目的，该方法会将向量缩放并在结果不确定时变为零向量。

```c#
		playerInput.x = Input.GetAxis("Horizontal");
		playerInput.y = Input.GetAxis("Vertical");
		playerInput.Normalize();
```

![Normalized key input]({{site.url}}\assets\catlikecoding-movement-sliding_a_sphere\11.gif)

<center>键盘输入的归一化</center>

## 1.4 约束输入向量

始终对输入向量进行归一化会将球体的位置限制在圆和水平与垂直的直径上，除非输入是中性的。在这种情况下，球体在没有输入时会回到原点。 从圆心到圆之间仅有一帧，即球体在一帧内从圆心跳到圆上（或相反）。

这种二元的输入可能在某些情况下是理想的，但是更多情况下需要球体在圆内的所有位置都可以进行移动。可以通过调整大小大于 1 的输入向量的大小来达到效果。一种简便的方法是用静态方法 `Vector2.ClampMagnitude` 来代替 `Normalize`，并使用输入向量和其最大值 1 作为参数。该方法返回一个大小被限定在所提供的最大值之内的向量。

```c#
		//playerInput.Normalize();
		playerInput = Vector2.ClampMagnitude(playerInput, 1f);
```

![Constrained key input]({{site.url}}\assets\catlikecoding-movement-sliding_a_sphere\12.gif)

<center>约束后的键盘输入</center>

# 2 控制速度

目前为止我们完成了通过输入直接设定球体的位置，即当输入向量 $\bf{i}$ 改变时，球体位置 $\bf{p}$ 也立刻改变为同样的值。也即 $\bf p = \bf i$。这并非真正的运动，而是传送。更加自然的控制球体的方式是通过在原位置 $\bf{p_0}$ 加上位移向量 $\bf d$ 得到下一个位置 $\bf p_1$，即 $\bf p_1 = \bf p_0 + \bf d$。

## 2.1 相对运动

通过用 $\bf d=\bf i$ 代替 $\bf p=\bf i$ 来改变输入和位置直接对应的关系。同时也可以消除位置上的约束，现在每次更新都是相对于上一次的位置而非初始位置。现在球体的位置由无限的迭代序列 $\bf p_{n+1}=\bf p_n + \bf d$ 描述，$\bf p_0$ 是初始位置。

```c#
		Vector3 displacement = new Vector3(playerInput.x, 0f, playerInput.y);
		transform.localPosition += displacement;
```

![Relative movement]({{site.url}}\assets\catlikecoding-movement-sliding_a_sphere\13.gif)

<center>相对运动</center>

## 2.2 速度（矢量）

现在球体可以随意移动了，但是它的速度太快以至于玩家几乎无法操控。这是因为每一次调用 `update` 都将输入向量添加到相对位置。帧率越高速度就越快。显然，我们希望速度和帧率无关并且是一个稳定的值，无论帧率是否波动，在相同输入的情况下球体的位移应该是不变的。

为了实现这个目标，每一帧代表的应为这一帧持续的时间 $t$。可以通过 `Time.deltaTime` 来得到这个值。于是，之前的位移向量实际为 $\bf {d} = \bf {i} t$，且我们错误的假设 $t$ 的值恒定。

位移以 Unity 单位测量，假定 Unity 单位为一米。位移向量为输入向量乘以持续时间，持续时间单位为秒。为了使位移向量的单位为米，则输入向量必须以米/秒为单位。 因此，输入向量表示速度：$ \bf {v} = \bf {i}$，$\bf {d} = \bf {v}t$。

```c#
		Vector3 velocity = new Vector3(playerInput.x, 0f, playerInput.y);
		Vector3 displacement = velocity * Time.deltaTime;
		transform.localPosition += displacement;
```

![Controlling velocity independent of frame rate]({{site.url}}\assets\catlikecoding-movement-sliding_a_sphere\14.gif)

<center>独立于帧率的速度</center>

## 2.3 速度大小

最大输入向量的大小为 1，即 1 米每秒（等于 3.6 公里每小时或 2.24 英里每小时）。这个速度并不快。

可以通过缩放输入向量的方式来提高最大速度。缩放的系数表示最大速度，是没有方向的标量，仅表示速度的大小。用一个带有 `SerializeField` 属性的 `maxSpeed` 字段（默认值为10）表示这个系数，并为其指定 `Range` 属性（例如1–100）。

```c#
	[SerializeField, Range(0f, 100f)]
	float maxSpeed = 10f;
```

>`SerializeField` 的作用
>
>它告诉 Unity 对字段进行序列化，这意味着字段已被保存并在 Unity 编辑器中公开，因此可以通过检查器进行调整。我们也可以公开该字段，但是通过这种方式，该字段仍然不受 `MovingSphere` 类之外的代码的影响。

将输入向量和最大速度相乘得到我们想要的速度。

```c#
		Vector3 velocity = new Vector3(playerInput.x, 0f, playerInput.y) * maxSpeed;
```

![Max speed set to 10]({{site.url}}\assets\catlikecoding-movement-sliding_a_sphere\15.jpg)

![Max speed set to 10]({{site.url}}\assets\catlikecoding-movement-sliding_a_sphere\16.gif)

<center>最大速度设置为10</center>

## 2.4 加速度

由于上面的方法将输入向量当作速度，球体运动的速度可以瞬间改变。只有输入系统本身的过滤会稍微减慢速度更改的速度。现实中速度不能立刻改变，速度的改变和位置的改变一样需要对物体施加力和时间。速度改变的速率称为加速度，记为$\bf a$，有$\bf v_{n+1}=\bf v_n + \bf a t$，其中$\bf v_0$为零向量。减速是与当前速度方向相反的加速度，因此不需要特殊处理。

使用输入矢量控制加速度而非控制速度。需要追踪当前的速度，故将速度存储在一个字段中。

```c#
	Vector3 velocity;
```

现在输入向量在 `Update` 中定义加速度，但目前先继续使其与 `maxSpeed` 相乘，暂时将其解释为最大加速度。然后将得到的速度的增量加到原速度上并计算位移。

```c#
		Vector3 acceleration = new Vector3(playerInput.x, 0f, playerInput.y) * maxSpeed;
		velocity += acceleration * Time.deltaTime;
		Vector3 displacement = velocity * Time.deltaTime;
```

![Smooth velocity change]({{site.url}}\assets\catlikecoding-movement-sliding_a_sphere\17.gif)

<center>平滑的速度变化</center>

## 2.5 目标速度

控制加速度会比控制速度产生更平滑的运动，但同时也会削弱我们对球体的控制。前者就像在开车而后者是步行。在大多数游戏中，需要对速度进行更直接的控制，因此让我们回到这种方法。但是，施加加速度确实会产生更平滑的运动。

![Acceleration changes velocity which in turn changes position]({{site.url}}\assets\catlikecoding-movement-sliding_a_sphere\18.png)

<center>加速度改变速度，从而改变位置</center>

我们可以通过直接控制球体的速度并将加速度施加到实际速度上，直到与目标速度相匹配，来将此两种方法相结合。然后，我们可以通过调整球的最大加速度来调整球的响应速度。为最大加速度添加一个可序列化的字段。

```c#
	[SerializeField, Range(0f, 100f)]
	float maxAcceleration = 10f;
```

在 `Update` 方法中使用输入向量来定义目标速度，而不是使用之前的方法调整速度。

```c#
		Vector3 desiredVelocity = new Vector3(playerInput.x, 0f, playerInput.y) * maxSpeed;
		//velocity += acceleration * Time.deltaTime;
```

首先通过将最大加速度乘以 $t$ 得到最大速度的变化。这个值就是更新速度的最大程度。

```c#
		Vector3 desiredVelocity = new Vector3(playerInput.x, 0f, playerInput.y) * maxSpeed;
		float maxSpeedChange = maxAcceleration * Time.deltaTime;
```

先只考虑速度的 X 分量。如果当前的速度比目标速度小，则加上速度更新的最大值。

```c#
		float maxSpeedChange = maxAcceleration * Time.deltaTime;
		if (velocity.x < desiredVelocity.x) {
			velocity.x += maxSpeedChange;
		}
```

这有可能导致结果比所需的速度更快。可以通过将目标速度设定为上限来防止这种情况。可以调用 `Mathf.Min` 方法来实现。

```c#
		if (velocity.x < desiredVelocity.x) {
			//velocity.x += maxSpeedChange;
			velocity.x = Mathf.Min(velocity.x + maxSpeedChange, desiredVelocity.x);
		}
```

或者当前的速度比需要的速度快，此时加速度与速度方向相反，则可能导致结果比目标速度慢。通过调用 `Mathf.Max` 方法将所需的速度设定为下限。

```c#
		if (velocity.x < desiredVelocity.x) {
			velocity.x = Mathf.Min(velocity.x + maxSpeedChange, desiredVelocity.x);
		}
		else if (velocity.x > desiredVelocity.x) {
			velocity.x = Mathf.Max(velocity.x - maxSpeedChange, desiredVelocity.x);
		}
```

也可以通过调用 `Mathf.MoveTowards` 方法更加方便的完成所有操作。将当前速度，目标速度和速度变化的最大值传递给该方法并分别对速度的 X 轴和 Z 轴分量执行此操作。

```c#
		float maxSpeedChange = maxAcceleration * Time.deltaTime;
		//if (velocity.x < desiredVelocity.x) {
		//	velocity.x = Mathf.Min(velocity.x + maxSpeedChange, desiredVelocity.x);
		//}
		//else if (velocity.x > desiredVelocity.x) {
		//	velocity.x = Mathf.Max(velocity.x - maxSpeedChange, desiredVelocity.x);
		//}
		velocity.x = Mathf.MoveTowards(velocity.x, desiredVelocity.x, maxSpeedChange);
		velocity.z = Mathf.MoveTowards(velocity.z, desiredVelocity.z, maxSpeedChange);
```

![Both max speed and acceleration set to 10]({{site.url}}\assets\catlikecoding-movement-sliding_a_sphere\19.jpg)

![Both max speed and acceleration set to 10]({{site.url}}\assets\catlikecoding-movement-sliding_a_sphere\20.gif)

<center>最大速度和最大加速度均设为10</center>

现在，我们可以通过调整最大加速度来达到运动的平滑程度和响应性之间的平衡。

# 3 约束位置

除了控制角色的速度，游戏设计中另一个重要的部分是限定角色可以到达的位置。我们搭设的场景中包含一个代表地面的平面。接下来限制球体移动的范围使之必须停留在地面上。

## 3.1 限定在某区域

我们并不直接使用平面本身为约束，而是使用一个序列化字段作为球体允许移动的区域。可以使用 `Rect` 结构体定义此区域。通过调用其构造函数方法（其前两个参数为-5和后两个参数为10），为其提供与默认平面匹配的默认值。前两个参数定义了矩形的左下角，后两个参数定义了矩形的大小。

```c#
	[SerializeField]
	Rect allowedArea = new Rect(-5f, -5f, 10f, 10f);
```

在将新位置赋值给 `transform.localPosition` 之前，我们先通过约束新位置的值来约束球体。因此，首先将其存储在变量中。

```c#
		//transform.localPosition += displacement;
		Vector3 newPosition = transform.localPosition + displacement;
		transform.localPosition = newPosition;
```

我们可以在允许移动的区域上调用 `Contains` 方法检查一个点是否位于其内部或边缘。如果返回值为否，那么我们将新位置设置为当前位置来取消此更新期间内的运动。

```c#
		Vector3 newPosition = transform.localPosition + displacement;
		if (!allowedArea.Contains(newPosition)) {
			newPosition = transform.localPosition;
		}
		transform.localPosition = newPosition;
```

当我们将 `Vector3` 类型传递给 `Contains` 时，它会检查 XY 坐标，这在我们的场景中是错误的。因此，将新位置的值传递给带有其 XZ 坐标的 `Vector2` 类型的变量。

```c#
		if (!allowedArea.Contains(new Vector2(newPosition.x, newPosition.z))) {
			newPosition = transform.localPosition;
		}
```

![Stopped when touching the plane's edges]({{site.url}}\assets\catlikecoding-movement-sliding_a_sphere\21.jpg)

![Stopped when touching the plane's edges]({{site.url}}\assets\catlikecoding-movement-sliding_a_sphere\22.gif)

<center>当到达边界时停止运动</center>

我们的球体再也无法离开地面了，当它试移动到边缘时就会停下来。但是过程很生涩，因为在某些帧中运动被忽略了，接下来我们会解决这个问题。不过在此之前我们注意到球体可以一直移动直到地面的边上。这是因为我们在限定球体的移动范围时没有考虑它的半径。如果要使整个球体都保留在地面上，则看起来会更好。我们可以更改代码，但是另一种方法是简单地缩小允许运动的区域。这对于当前的简单场景已经足够。

将控制区域左下角的两个值向上移动 0.5，并将两个边的大小减小 1。

![Stopped when touching the plane's edges]({{site.url}}\assets\catlikecoding-movement-sliding_a_sphere\23.jpg)

![Stopped when touching the plane's edges]({{site.url}}\assets\catlikecoding-movement-sliding_a_sphere\24.gif)

<center>当碰触边界时停止运动</center>

## 3.2 连贯性

我们可以通过保留新位置在允许的区域中的分量而不是全部忽略来摆脱撞墙时卡顿的感觉。通过调用参数为目标值及其允许的最小值和最大值的 `Mathf.Clamp` 方法使将目标值限定在设定的范围中来实现。将 `allowedArea` 的 `xMin` 和 `xMax` 属性作为新位置的 X 分量的范围，将 `yMin` 和 `yMax` 属性作为新位置的 Z 分量的范围。

```c#
		if (!allowedArea.Contains(new Vector2(newPosition.x, newPosition.z))) {
			//newPosition = transform.localPosition;
			newPosition.x = Mathf.Clamp(newPosition.x, allowedArea.xMin, allowedArea.xMax);
			newPosition.z = Mathf.Clamp(newPosition.z, allowedArea.yMin, allowedArea.yMax);
		}
```

![Sticking to edges]({{site.url}}\assets\catlikecoding-movement-sliding_a_sphere\25.gif)

<center>保留在允许区域内的分量</center>

## 3.3 消除碰撞后的速度

现在当球体碰到边缘时似乎粘在了边缘上。到达边缘后，我们沿着它滑动，但需要一段时间才能离开边缘。这是因为在碰到边缘时球体的速度方向没有变化。其指向边缘的速度分量必须通过加速度和时间来消除，需要的取决于最大加速度。

如果我们的球体是一个球，而区域的边缘是一堵墙，那么如果它撞到墙上，它应该停下来。我们已经在场景中将此复现了。但考虑一种情况，当球体撞到墙上后墙消失了，此时球体并不会恢复之前的速度。球体的动量已经消失了，它的能量在碰撞中转移了，这种能量的转移就是现实中碰撞造成损伤的原因。所以我们必须要消除球体撞到边缘时的速度。但是球体应仍可以沿着边缘滑动，因此只应该消除速度指向该边缘方向的分量。

为了将目标速度分量设为零，我们必须检查两个维度的两个方向是否出界。不妨在确定新位置的同时消除速度分量，因为此时的检查条件与 `Mathf.Clamp` 和 `Contains` 相同。

```c#
		//if (!allowedArea.Contains(new Vector2(newPosition.x, newPosition.z))) {
			//newPosition.x =
			//	Mathf.Clamp(newPosition.x, allowedArea.xMin, allowedArea.xMax);
			//newPosition.z =
			//	Mathf.Clamp(newPosition.z, allowedArea.yMin, allowedArea.yMax);
		//}
		if (newPosition.x < allowedArea.xMin) {
			newPosition.x = allowedArea.xMin;
			velocity.x = 0f;
		}
		else if (newPosition.x > allowedArea.xMax) {
			newPosition.x = allowedArea.xMax;
			velocity.x = 0f;
		}
		if (newPosition.z < allowedArea.yMin) {
			newPosition.z = allowedArea.yMin;
			velocity.z = 0f;
		}
		else if (newPosition.z > allowedArea.yMax) {
			newPosition.z = allowedArea.yMax;
			velocity.z = 0f;
		}
```

![No longer sticking to edges]({{site.url}}\assets\catlikecoding-movement-sliding_a_sphere\26.gif)

<center>不再粘连在边缘</center>

## 3.4 弹跳

在碰撞过程中，速度并不总是被消除。假设球体是一个完美的弹球，它就会在相关的维度上反转运动方向。让我们来实现这种理想弹球。

```c#
		if (newPosition.x < allowedArea.xMin) {
			newPosition.x = allowedArea.xMin;
			velocity.x = -velocity.x;
		}
		else if (newPosition.x > allowedArea.xMax) {
			newPosition.x = allowedArea.xMax;
			velocity.x = -velocity.x;
		}
		if (newPosition.z < allowedArea.yMin) {
			newPosition.z = allowedArea.yMin;
			velocity.z = -velocity.z;
		}
		else if (newPosition.z > allowedArea.yMax) {
			newPosition.z = allowedArea.yMax;
			velocity.z = -velocity.z;
		}
```

![Bouncing off edges]({{site.url}}\assets\catlikecoding-movement-sliding_a_sphere\27.gif)

<center>碰触边缘时弹起</center>

现在，球体在碰撞后保持了它的动量，当它碰到墙时只是改变运动方向。其速度确实会放慢一点，因为弹跳后的速度将不再与目标速度匹配。为了获得最佳反弹，玩家必须立即调整输入。

## 3.5 弹跳能力

我们在反转速度的时候不需要将其完全保留。不同的物体回弹的能力不同。所以我们通过添加一个描述弹跳能力的可序列化字段来使其可进行配置，默认设置为 0.5，范围为 [0, 1]。这样我们就可以让球体完美地弹跳或完全不弹跳，或者介于两者之间。

```c#
	[SerializeField, Range(0f, 1f)]
	float bounciness = 0.5f;
```

当与边缘碰撞时，将弹跳能力计入新的速度值。

```c#
		if (newPosition.x < allowedArea.xMin) {
			newPosition.x = allowedArea.xMin;
			velocity.x = -velocity.x * bounciness;
		}
		else if (newPosition.x > allowedArea.xMax) {
			newPosition.x = allowedArea.xMax;
			velocity.x = -velocity.x * bounciness;
		}
		if (newPosition.z < allowedArea.yMin) {
			newPosition.z = allowedArea.yMin;
			velocity.z = -velocity.z * bounciness;
		}
		else if (newPosition.z > allowedArea.yMax) {
			newPosition.z = allowedArea.yMax;
			velocity.z = -velocity.z * bounciness;
		}
```

![Bounciness set to 0.5]({{site.url}}\assets\catlikecoding-movement-sliding_a_sphere\28.jpg)

![Bounciness set to 0.5]({{site.url}}\assets\catlikecoding-movement-sliding_a_sphere\29.gif)

<center>弹跳能力设为0.5</center>

这并是现实的物理学，它要比我们实现的这个简单场景复杂得多。但它开始看起来像了，这对大多数游戏来说已经足够好了。另外，我们的移动不是很精确。我们的计算只有在一帧内运动结束时准确到达边缘时才是正确的。而运行的时候很可能不是这种情况。这意味着我们应该立即将球体移动到离边缘远一点的地方。首先计算剩余的时间，然后在相关的维度上将其与新的速度一起使用。然而，这种方法可能会导致第二次反弹而使得情况变得更加复杂。幸运的是，我们不需要这样的精度来呈现一个令人信服的球体反弹的感觉。

下一个教程是物理。