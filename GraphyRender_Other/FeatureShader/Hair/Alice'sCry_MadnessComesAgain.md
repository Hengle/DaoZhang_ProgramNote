##  [爱丽丝的发丝──《爱丽丝惊魂记：疯狂再临》制作点滴](https://www.cnblogs.com/miloyip/archive/2011/06/14/alice_madness_returns_hair.html) 

 

今天(2011年6月14日)是《[爱丽丝惊魂记：疯狂再临 (Alice: Madness Returns)](http://www.ea.com/alice) Xbox360/PlayStation3/PC》(下简称《爱》)正式发售日，身为其开发程序员之一，特撰此文以作纪念。

## 简介

《爱》(图1a)是一款由上海独立游戏工作室[麻辣马(Spicy Horse)](http://www.spicyhorse.com/)制作、[美商电艺(Electronic Arts)](http://www.ea.com/)发行的惊悚动作冒险游戏。此全乃2000年发行的《爱丽丝惊魂记(American McGee’s Alice) PC》(图1b)的续作。

![img](Alice'sCry_MadnessComesAgain.assets/o_image01.jpg) ![img](Alice'sCry_MadnessComesAgain.assets/o_image06.png)

图1(a): 《Alice: Madness Returns》Xbox360封面 (b): 《American McGee’s Alice》PC封面

在为期超过两年的制作期间，《爱》的制作团队最高达75人，另外有50人左右的美术外包团队。《爱》的制作团队有许多不同国籍的成员，但当中主要为华人。从制作地点及人员来说，《爱》可以说是一个国产游戏。但从目前的环境来说，《爱》应该不会在国内发行。

《爱》使用[Unreal Engine 3](http://www.unrealengine.com/)开发，并使用了[Scaleform](http://www.scaleform.com/)、[Kynapse](http://usa.autodesk.com/adsk/servlet/pc/index?id=11390544&siteID=123112)和[Bink](http://www.radgametools.com/bnkmain.htm)中间件。在PC平台上，合作伙伴nVidia加入了使用GPU加速[PhysX](http://www.nvidia.com/object/physx_new.html)效果。但游戏主角爱丽丝的头发和衣饰模拟，并非使用PhysX，而是一个自定义解决方案，这也是本文将谈及的主要内容。

有时候，需求和技术，就像是鸡和蛋的关系──因某需求而开发新技术，或因某技术而产生新的需求。本人在《爱》的开发过程，清楚体会到这个关系。让我细细回想当天的事……

## 研究之始

2009年8月23日(星期日)，刚入职满三个星期了。这段时间的工作，主要是按游戏策画的需求，做了一些简单的游戏性编程(gameplay programming)，例如是一些关卡内的机关，这正好让我学习一下[UnrealScript](http://udn.epicgames.com/Three/UnrealScriptReference.html)(Unreal引擎的脚本语言)。

上周看到新版本的主角模型，虽然比旧版本更精细，但我看上去觉得还有改善空间，于是分别和美术总监和动画总监讨论，大家也认为现时对头发和衣饰以手工关键帧动画(keyframe)方法表现，效果不够理想，而且用Phong反射模型来渲染头发，有点像塑料玩偶的感觉。从动画的工作来说，头发和衣饰的关键帧动画要做得自然，并不容易；尤其动画间的混合(blending)更为困难，不是动画不自然加减速，就是会穿过身体。

传统上，许多游戏会避免把角色设计为长发，也会避免穿着长裙。但爱丽丝无可避免要触犯此二禁忌。既然如此，何不尝试进行突破，并以此为游戏特色呢？

当天虽是周日，我在上海孤单一人，在炎夏就不外出了。脑里不断思考着上周工作上的事情。不过单单在想也没有用，就直接敲键盘实验一些方案。最先想到的，是常用于模拟绳子和布的弹簧质点系统(mass-spring  system)，记得以前看过相关的入门文章[1]，就以该文的基础。

## 弹簧质点系统

所谓弹簧质点系统，其实就是仿真一些有质量的粒子(质点)，再在粒子之间加入一些无质量的虚拟弹簧。例如要模拟一条绳子，最简单的方法是建立n个粒子，再在每两个连续的粒子之间加入弹簧，即有n-1个弹簧，如图2。

![img](Alice'sCry_MadnessComesAgain.assets/r_image10.png)

图2: 用5个粒子和4个弹簧模拟的绳子

要模拟粒子运动，可使用《[用JavaScript玩转游戏物理(一)运动学模拟与粒子系统](http://www.cnblogs.com/miloyip/archive/2010/06/14/Kinematics_ParticleSystem.html)》一文中谈及的欧拉方法(Euler method)，但[1]里介绍的Verlet数值积分在很多情况下是更好的选择。我使用了含简单阻尼效果的Verlet数值积分方程:

![img](Alice'sCry_MadnessComesAgain.assets/png.png) + \mathbf{a}(t) \Delta t^2)

当中，![img](http://latex.codecogs.com/png.latex?\mathbf{x}(t))是时间在t时粒子的位置，![img](http://latex.codecogs.com/png.latex?\Delta t)为时步(timestep)，![img](http://latex.codecogs.com/png.latex?d \in [0,1])的阻尼系数，![img](http://latex.codecogs.com/png.latex?\mathbf{a}(t))为时间在![img](Alice'sCry_MadnessComesAgain.assets/png.png)时作用于粒子的加速度(即当时作用于粒子的力除以其质量，例如引力加速度![img](http://latex.codecogs.com/png.latex?[0, 0, -9.8]))。Verlet方法分的计算简单，不需保留或计算速度(velocity)，也比欧拉稳定，但缺点是时步(![img](http://latex.codecogs.com/png.latex?\Delta t))必须是固定的。

Verlet积分的另一特点，是可以简单地加入各种约束(constraint)，例如某粒子在仿真之后，其位置位于地面以下，只需把粒子移至最近地面的点。对于绳子，另一约束就是相邻粒子的距离，在Verlet积分下，此距离约束可以模拟弹簧。假设两个相邻粒子的位置为![img](Alice'sCry_MadnessComesAgain.assets/png-1565854739738.png)、![img](Alice'sCry_MadnessComesAgain.assets/png-1565854739769.png)，两者间的止动长度(rest length)为![img](Alice'sCry_MadnessComesAgain.assets/png-1565854740030.png)，则可以这样调节两粒子的位置：

![img](Alice'sCry_MadnessComesAgain.assets/png-1565854740077.png)

此外，要模拟头发，必须避免头发移动至头颅及其他身体部分之内，此仍碰撞检测(collision  detection)和碰撞决议(collision  resolution)。如前所述，这部分也可以用约束来表示。由于头颅较接近球体，在此简单测试中，只加入一个球体去进行检测。此约束把球体内的粒子推至最近的球面上。

要同时满足多个约束，最简单的方法是松弛法(relaxation method)，即进行多个迭代，每次执行所有约束一次，那么其结果就会就趋近合乎所有约束的解。当天写的测试程序，其数据结构和伪代码表示如下:

```
`// 节点(粒子)``struct` `Node {``        ``Vector3 p0, p1; ``// 前帧/本帧的位置``        ``float` `length; ``// 和上一节点的止动长度``}` `// 发束``struct` `Strand {``        ``size_t` `nodeStart, nodeEnd; ``// 此发束中，节点数组的起始和结束索引``        ``Vector3 rootP; ``// 发根的局部坐标(相对于头的变换)``};` `SimulateHair(nodes, strands, sphere, damping, dt, headToWorld)``    ``// 对每个节点进行Verlet积分``    ``for` `each n in nodes``    ``a = Accumulating force ``for` `n, divided by mass ``// 现时只是引力加速度常量``    ``p2 = Verlet(n.p0, n.p1, damping, a, dt)``    ``n.p0 = n.p1, n.p1 = p2 ``// 以新状态取代旧状态`  `    ``// 对每束发丝以松弛法进行约束求解``    ``for` `each s in strands``        ``for` `a number of iterations``            ``for` `index = s.nodeStart to s.nodeEnd - 1``                ``na = nodes[index]``                ``nb = nodes[index + 1]``                ``// 碰撞检测和决议``                ``nb.p1 = collideSphere(sphere, nb.p1)``                ``// 长度约束``                ``na.p1, nb.p1 = lengthConstraint(na.p1, nb.p1, nb.length)` `             ``// 固定发根``             ``nodes[s.nodeStart].p1 = transform(headToWorld, s.rootP)`
```

用程序产生一些发束，并把模疑结果用直线线段渲染出来，就做成图3的效果：

![img](Alice'sCry_MadnessComesAgain.assets/r_image04.jpg)

图3: 最初的头发实验

程序中，能使用鼠标旋转头颅，表现暮然回首的飘逸；也可改变引力方向，表现风吹秀发的感觉。这个花了一天时间写的程序实验，其实并不复杂，至少比写这篇博文容易。

## 把研究放进日程

次日，把成果带到公司，向程序同事和动画同事演示，决定把这个初步构思带到周三的定期技术会议。除了继续完成之前的工作，也花了些时间搜集、阅读关于实时头发模拟及渲染的文献，并把一些想法写到项目的wiki里。

终于到了周三的定期技术会议，把程序向项目组的主要决策者(包括制作人、创意总监、美术总监、技术总监等)演示，并展示一些文献里的最终渲染效果。基本上反馈是正面的，一些主要讨论重点大约如下:

>  问：使用程序化的头发，相比手工动画有何好处？
>  答：节省工作量，而且效果应该会比手工更好。另外，《爱》中有海底场景，可使用阻尼等参数模拟水里的头发飘动效果；在室外、天空上的场景，也可以加入风的效果。
>  

>  问：此技术会否很耗CPU/GPU时间？
>  答：具体开销暂时未能确定。由于我们游戏在Xbox360/PS3上，GPU应该会成为瓶颈，所以可以考虑在Xbox360使用空闲的CPU核，在PS3上使用SPU，去进行头发模拟。若假设发束之间无互动关系，还可以使用这种并行性作多核/多SPU并行加速。
>  

>  问：三维美术方面如何去设定发型？
>  答：最简单的方法，是使用额外的骨头(bone)去设定发型，那么就不用更改导出工具或编写特别的工具。模拟中可以加入额外的约束，使发束自然回复至预设的发型。
>  

>  问：为甚么其他市面上的游戏不用这种技术？
>  答：……(当时真答不出来，但也许这个问题也是我的重要得着，详见后文)
>  

然后也讨论了预计所需的研发时间。最后决定可以继续第一阶段的研发，以成果决定是否继续下一阶段。

## 头发渲染

获准研究，除了我感到兴奋，动画组同事也非常希望此技术能成功应用到游戏，因为此技术会大大节省他们的工作量。因此，他们也热心准备爱丽丝的测试用发型数据。为了更容易设定发型，只要求建立一个头皮(sculp)的三角形网格，并在网格上每个顶点上加入一串骨架，以代表引导发束(guided  strands)，程序中按LOD细分(subdivide)头皮网格，并以插值方式产生引导发束之间的发束。

我更新了测试程序，使用D3DX库去导入爱丽丝的白模及发型，再把发型数据转化为仿真用的节点和发束数据结构。接着是先尝试渲染部分，然后再改进模拟部分。

我尝试过几种渲染方法。[2]中以线表(line list)去渲染发丝，要表现稠密发丝所需的像素数目(primitive  count)很大，在目标平台上需要占用很多GPU时间。而另一种方法，是把发束线段向屏幕空间展开，形成固定宽度的三角形表(triangle  list)或四边形表(quad list)。

为了使用较少的节点而得到圆滑的发束，我采用了均匀三次B样条(uniform cubic B-splines)，把原来的发束插值为曲线(图4)，之后才把该曲线展开为三角形表。插值的数量可以成为运行期动态LOD的参数。

图4: 绿色线段为模拟结果，橙色为三次B样条

  ![img](Alice'sCry_MadnessComesAgain.assets/r_image08.jpg)

至于着色方面，采用了较简单的Kajiya-Kay模型[3]。此模型基于切线(tangent)而非法线(normal)，能表现出头发的高光(图5)。

  ![img](Alice'sCry_MadnessComesAgain.assets/r_image02.jpg)

图5: 早期基于Kajiya-Kay反射模型的着色

## 改进模拟

之前的做法，虽然能模拟出一条绳子，但它的行为更像一条锁链，因为它是完全柔软的，而真实的绳子在止动时通常是直线的，弯曲绳子需要施力。一个简单的实现方式是再加入弹簧(长度约束)，连接相隔一个粒子的每对粒子(图6)。

  ![img](Alice'sCry_MadnessComesAgain.assets/r_image00.png)

图6: 加入防止弯曲的长度约束(红色)

要把发束回复至原来的发型，方法是把目前节点位置向该节点的引导动位置(guided  position，即发型中设置的位置)施以归还力(restitution  force)。经过实验测试，发觉可以把接近发根的节点设定较大的归还力，越接近发梢则归还力越弱。我简单地使用一个衰变的关系![img](Alice'sCry_MadnessComesAgain.assets/png-1565854740114.png)，当中![img](Alice'sCry_MadnessComesAgain.assets/png-1565854740463.png)为发根的归还力，![img](Alice'sCry_MadnessComesAgain.assets/png-1565854740738.png)为自发根起计的节点索引，![img](Alice'sCry_MadnessComesAgain.assets/png-1565854740798.png)是衰变的速度。

在碰撞方面，只是从一个球体扩展至多个球体，模拟更准确的头形，以及对脖子、胸、肩、手臂的碰撞。Verlet方法使碰撞计算简单之时又真实，可表现出发丝在肩上顺滑地流动。

研发通常都不是一帆风顺的。当在程序中加入了移动模型的操控后，发现在少量迭代的情况下，发丝像弹簧般弹来弹去，换句话说，长度约束的收敛不够快。此问题是技术关键，当时没找到好的现成办法，苦恼多时。我试过不同的方法，例如把约束的执行乱序化，或是以不同分组方法进行长度约束，但效果都不如理想。最后灵机一触，想到既然碰撞这么简单、效果又好，可以想象每个节点都被限制在一个球体之内，球体中心为发根，半径则是发根至该节点的止动长度之和，如图7a所示。

  ![img](Alice'sCry_MadnessComesAgain.assets/r_image03.png)

图7(a): 每个粒子限制在一个球体之内 (b): 不能满足长度约束的情形，但仍然保持每个节点和发根的直线距离

此法能有效地避免头发超出半径范围，但不能控制如图7(b)的情况。从实验得知，后者其实不太显眼，只要不做成弹簧伸缩的感觉，视觉上很难察觉出问题。

## 效能测试

至2009年8月30日(星期日)，在Windows上实现的基本头发模拟和渲染实验已经完成，代码亦使用了XNA Math做SIMD矢量优化。很兴奋地把屏幕截图发给团队成员，分享成果。

在往后的技术会议基本上满意这个研发进度及结果，但还有一点忧虑，就是此技术在游戏机平台的性能。

因此，之后赶紧把代码移植至Xbox 360上测试(图8)。实验证明此技术的性能是可以的，瓶颈主要出现在特写镜头时的GPU填充率。

  ![img](Alice'sCry_MadnessComesAgain.assets/r_image09.jpg)

图8: Xbox360上的测试截屏

除了效能，另一让我纠结的问题是发丝排序。使用半透明alpha混合(alpha blending)的渲染效果，比alpha测试(alpha  testing)好得多。因为前者能表现出很柔滑的感觉，而后者则较粗糙。然而，实验里的排序是基于一个启发(heuristic)──按每根发束的第i个节点在观察空间(view   space)的深度进行排序。当设i=0就是用发根来排序，以实验测试找出各种情况下较好的值。但以发束为单位的排序不能完美解决问题，只是一个折衷方案。

当时还考虑过一些次序无关透明(order independent transparency,  OIT)技术，例如实现过screen-door  transparency，但效果都不如理想。或许用硬件的OIT方案，如alpha-to-coverage(需要较高的MSAA)、Direct3D  11中在渲染目标的缓冲区做OIT。

## 整合至Unreal引擎

效能测试大约花了一星期，之后就开始把此头发方案整合至Unreal引擎。

这个整合比我预期中困难许多，主要原因可能是我不熟悉Unreal渲染架构，而Unreal也缺乏这类文档，只能靠阅读原代码。由于所需的顶点信息和Unreal内置的不一样，所以要加入新的顶点格式、顶点着色器代码等。为了渲染第一个三角形，记得大概要增加、修改数十个源代码文件，并且由于变换矩阵的一些特别设置，以及多线程的问题，最终花了一星期时间才能渲染一个用自定顶点格式的三角形。

然而，之后的代码整合就变得容易，很快就可以为爱丽丝加上飘逸的长发。在战斗中猛烈摇头的效果，尤其令我鼓舞。之后再配合Unreal的工具，设置碰撞用的球体，最基本的整合就完成了。接着是要移植代码至Xbox360和PS3，我的同事Jake帮忙做了PS3的部分，把代码改写成SPU的方式。而专门做图形方面的杨同学也帮忙解决了不少问题，例如是景深效果(depth  of field,  DoF)时的问题(因为半透明的关系，头发本来并没有写入深度)。接下来一年的时间，也不断作出调整和新功能，例如爱丽丝缩小、跳跃、滑翔和风吹效果等等。

有一次EA来访，我们展示这个新技术时，对我印象最深刻的评语是：

> 不如让爱丽丝做洗发水广告吧！

## 新的需求

因为头发系统研发的成果，便希望爱丽丝的衣饰都能舍弃手工动画，采用程序式的动画。之前我们尝试过用现成的物理引擎的布料仿真，但效果不如理想。主要原因是，爱丽丝的裙子并不是轻薄柔滑的，而是里面有许多层布，形成一个较固定的形状。而且，设定中爱丽丝往下滑翔时，希望把裙子变成像降落伞的形状(图9)。

  ![img](Alice'sCry_MadnessComesAgain.assets/r_image05.jpg)

图9: 爱丽丝滑翔时，裙子变成像降落伞 (游戏截屏)

我采用和头发相同的弹簧质点系统代码，来做实验模拟爱丽丝的裙子。不同之处在于那些长度约束的拓朴(topology)，以及采用节点来驱动裙子骨架，用标准的蒙皮(skinning)方式渲染。

后来，这个衣服仿真系统被应用到其他衣饰，例如爱丽丝背上的蝴蝶结、其他装饰，甚至应用到最终boss的钢丝手。当中也花了许多时间做调整，例如采用类似连续式碰撞检测(continuous collision detection, CCD)来尽量避免膝盖穿出裙子。

## 结语

爱丽丝的发丝只是《爱》中的小插曲，后续还做了不少用到及未用到的技术和游戏实现。但是，这一缕发丝将成为一段美好的回忆。

再次回想「为甚么其他市面上的游戏不用这种技术？」这个问题。也许，有人尝试过但失败了；也许，有人做了实验的效果或效能未达期望；也许，有人考虑到技术困难在设计时规避了长发；也许，有人因为答不出「为甚么其他市面上的游戏不用这种技术？」这个问题而从没尝试过。

这次经历告诉我，主动思考如何以技术改善项目，不要过早否定可能性，有想法时尽量动手尝试。

希望日后能继续从技术方面，开拓游戏中的应用。最后也希望大家喜爱、享受《爱》这个游戏。

  ![img](Alice'sCry_MadnessComesAgain.assets/r_image07.jpg)