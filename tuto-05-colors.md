

原地址: <https://github.com/glium/glium/blob/master/book/tuto-05-colors.md>

# 属性

在我们的图形管线中, 三角形内每个像素的颜色就是片段着色器的输出. 如果片段着色器返回的值为 `(1.0, 0.0, 0.0, 1.0)`, 那么每个像素的颜色都是不透明的红色(四个值分别对应: 红, 绿, 蓝, alpha).  

为了输出正确的颜色, 我们需要一些有关我们将要绘制的像素点的信息. 幸运的是, 我们可以在顶点着色器和片段着色器之间传递信息.  

为此, 我们需要在顶点着色器的代码中添加一个用 `out` 关键字修饰的变量...  

```glsl
#version 140

in vec2 position;
out vec2 my_attr;      // 新的属性.

uniform mat4 matrix;

void main() {
    my_attr = position;     // 为每个 `out` 变量赋值.
    gl_Position = matrix * vec4(position, 0.0, 1.0);
}
```

...同时, 在片段着色器器中添加一个同名同类型的, 用 `in` 关键字修饰的变量.  

```glsl
#version 140

in vec2 my_attr;
out vec4 color;

void main() {
    color = vec4(my_attr, 0.0, 1.0);   // 我们用vec2和两个浮点数新建了一个vec4变量
}
```

现在看看发生了什么. 顶点着色器被调用了三次: 每个顶点一次. 每个顶点都会为 `my_attr` 变量赋一个不同的值. 然后在光栅化(Rasterization)阶段, OpenGL决定哪些像素点是位于三角形内的, 然后对每个像素点都调用一次片段着色器. `my_attr` 的值将被传给每一个像素点, **此值的插值(interpolation)取决于像素点的位置**.  

例如, 靠近顶点的像素点所获取的 `my_attr` 的值将接近或者约等于顶点着色器传递给该顶点的 `my_attr` 的值. 在三角形边上, 位于两个顶点之间的像素点的 `my_attr` 的值将是顶点着色器传递给这两个顶点的 `my_attr` 的值的平均. 同理, 位于三角形中心的像素所获取的 `my_attr` 的值将是三个顶点的值的平均.  

*注意: 这是因为变量默认拥有 `smooth` 属性, 这是你多数时候需要用到的属性. 你也可以将其指定为 `flat` 属性.* 

在上面的例子中, 由顶点着色器生成的 `my_attr` 的值取决于顶点的位置. 因此片段着色器获取到的 `my_attr` 的值取决于其正在处理的像素点的位置. 为了示范, 我们将坐标的位置分别赋给颜色的红, 绿部分.  

最终结果:  

![The result](tuto-05-linear.png)

**[完整的代码在此](https://github.com/glium/glium/blob/master/examples/tutorial-05.rs).**
