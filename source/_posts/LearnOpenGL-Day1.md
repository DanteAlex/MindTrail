---
title: OpenGL入门第一课：它的核心思维，不是函数，而是状态
date: 2021-06-10 14:34:22
cover: /img/01/01/opengl1.png
categories:
  - 实用教程
tags:
  - OpenGL
keywords:
  - 博客
  - blog
  - OpenGL
  - 渲染
---
---
# OpenGL是什么

凡人的误解：一般它被认为是一个API(Application Programming Interface)，即应用程序编程接口，包含了一系列可以操作图形、图像的函数。
真实的理解：他仅仅是一个由Khronos组织制定并维护的规范(Specification)。
OpenGL严格规范了每个函数该如何执行，以及他们的输出值。至于内部的实现，是OpenGL开发者自由发挥的。因此OpenGL给开发者的自由度实际上是很高的，只要遵循他的龟范，即函数的功能和返回值，剩下的FreeStyle。
实际上的OpenGL库基本都是生产显卡的厂商开发的，普及一个常识，当显示产生BUG的时候通常都会通过升级显卡驱动来解决，这些驱动会让你的显卡支持最新版本的OpenGL，这也是为什么总是建议更新显卡驱动。

---
# 核心模式与立即渲染模式

早期的OpenGL采用立即渲染模式（Immediate mode，也就是固定渲染管线），这个模式下绘制图形很方便。但是提供方便的同时肯定是包装过度，牺牲了一部分的自由度。随着时间的推移，规范越来越灵活，开发者对绘图细节有了更多的掌握。因此从**OpenGL3.2**开始，规范文档开始废弃立即渲染模式，并鼓励开发者在OpenGL的核心模式模式（Core-profile）下开发，这个分支的规范完全移除了旧的特性。
使用OpenGL核心模式，该规范迫使我们使用现代的函数。当试图使用一个已经废弃的函数时，OpenGL会抛出一个错误并终止绘图。然后有舍就有得，当提供更高的灵活性和效率得同时，必定会更难以理解和学习，使用Core-profile会让很难去把握OpenGL具体是如何运作的。

---
# 拓展

OpenGL的一大特性就是对拓展（Extension）的支持，当一个显卡公司提出一个新特性或者渲染上的大优化，通常会以拓展的方式在驱动中实现。通常，当一个扩展非常流行或者非常有用的时候，它将最终成为未来的OpenGL规范的一部分。拓展就像发布正式版本前的补丁一样。
使用扩展的代码大多看上去如下：
```c++
if (GL_ARB_extension_name)
{
    // 使用硬件支持的全新的现代特性
}
else
{
    // 不支持此扩展: 用旧的方式去做
}
```

---
# 状态机

当使用OpenGL的时候，会遇到一些状态设置函数(State-changing Function)，这类函数将会改变上下文。以及状态使用函数(State-using Function)，这类函数会根据当前OpenGL的状态执行一些操作。
1. 例如设置颜色状态
```c++
glColor3f(1.0f, 0.0f, 0.0f); // 设置当前颜色为红色
glBegin(GL_TRIANGLES);
    glVertex3f(...); // 这个顶点就是红色的
    ...
glEnd();
```
注意 `glVertex3f` 没有带颜色参数，是因为“当前颜色”已经被设置进 OpenGL 的状态了。
2. 例如设置绑定纹理状态
```c++
glBindTexture(GL_TEXTURE_2D, textureA);
// 接下来的绘制就用 textureA

glBindTexture(GL_TEXTURE_2D, textureB);
// 接下来改成用 textureB
```

---
# 对象

OpenGL库是用C语言写的，同时也支持多种语言的派生，但其内核仍是一个C库。由于C的一些语言结构不易被翻译到其它的高级语言，因此OpenGL开发的时候引入了一些抽象层。“对象(Object)”就是其中一个。
在OpenGL中一个对象是指一些选项的集合，它代表OpenGL状态的一个子集。比如，我们可以用一个对象来代表绘图窗口的设置，之后我们就可以设置它的大小、支持的颜色位数等等。可以把对象看做一个C风格的结构体(Struct)：
```c
struct object_name {
    float  option1;
    int    option2;
    char[] name;
};
```

---
# OpenGL工作流
```c++
// OpenGL的状态
struct OpenGL_Context {
  	...
  	object* object_Window_Target;
  	...  	
};
```

```c++
// 创建对象
unsigned int objectId = 0;
glGenObject(1, &objectId);
// 绑定对象至上下文
glBindObject(GL_WINDOW_TARGET, objectId);
// 设置当前绑定到 GL_WINDOW_TARGET 的对象的一些选项
glSetObjectOption(GL_WINDOW_TARGET, GL_OPTION_WINDOW_WIDTH, 800);
glSetObjectOption(GL_WINDOW_TARGET, GL_OPTION_WINDOW_HEIGHT, 600);
// 将上下文对象设回默认
glBindObject(GL_WINDOW_TARGET, 0);
```
这一小段代码展现了你以后使用OpenGL时常见的工作流。我们首先创建一个对象，然后用一个id保存它的引用（实际数据被储存在后台）。然后我们将对象绑定至上下文的目标位置（例子中窗口对象目标的位置被定义成<var>GL_WINDOW_TARGET</var>）。接下来我们设置窗口的选项。最后我们将目标位置的对象id设回0，解绑这个对象。设置的选项将被保存在<var>objectId</var>所引用的对象中，一旦我们重新绑定这个对象到<var>GL_WINDOW_TARGET</var>位置，这些选项就会重新生效。