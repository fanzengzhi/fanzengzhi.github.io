---
title: Perspective Devide
teaser: 
category: graphics
tags: [graphics, rendering]
---

以OpenGL为例：

OpenGL的Projection矩阵为：

[

n/r,	0,	0,	0,

0,	n/t,	0,	0,

0,	0,	-(f+n)/(f-n),	-2fn/(f-n),

0,	0, 	-1, 0

]

其中f为far，n为near。着重看z和w分量，xy就不解释了。

设z为view 变换后，camera space的深度值。那么乘以projection矩阵以后，

**z' = -(f+n)/(f-n) * z - 2fn/(f-n)**

**w' = -z**

这个时候，z‘的值并不在[-1,1]的范围内（x,y也不在这个范围）。因此，clip space的值域并非为[-1,-1]的一个cube。但是距离这个已经很接近了。 Perspective devide的意思是，clip space的x‘y'z'w‘同时除以w’，此时得到的x'',y'',z''才在[-1,1]的范围内。

这时：

**z'' = (f+n)/(f-n)  + 2fn/z*(f-n)**

**w'' = 1**

此时的坐标称为NDC（Normalized Device Coordinates）。

所以，我们在VS中把坐标乘以MVP以后得到的值，是位于clip space中，而不是NDC，因此此时的w分量还是有意义的，并不能略去。glPosition也是一个vec4的类型，而不是vec3。clip space转换到NDC的这个perspective devision过程是由硬件做的，在裁剪之前完成。

顺便解释一下如何将depth texture中的z值转换成线性的z值。

首先，depth texture中的z值，值域为[0,1]，是NDC，而不是在clip space中。首先应该z = z*2 - 1，将其转换到[-1,1]的范围中。

通常Linear01Depth = 1.0 / (_ZBufferParams.x * z + _ZBufferParams.y);

其中：

_ZBufferParams.x = (1.0 - far/ near) / 2.0;
_ZBufferParams.x = (1.0 + far/ near) / 2.0;

将带入上述公式：

Linear01Depth = z/f, 即为[0,1]范围内的线性深度。上述算法和Unity中的Linear01Depth函数是一致的。