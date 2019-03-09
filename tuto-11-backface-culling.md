

原地址: <https://github.com/glium/glium/blob/master/book/tuto-11-backface-culling.md>

# 背面剔除

还有一件事.  

顶点着色器在屏幕上输出顶点坐标之后, 每个三角形可能处在以下两种状态之一:  

 - 三角形的三个顶点是按照顺时针的方向在屏幕上绘制的.  
 - 三角形的三个顶点是按照逆时针的方向在屏幕上绘制的.  

假设某个三角形是按顺时针方向绘制的, 当你把视角转到它的背面, 这个三角形三个顶点的绘制方向就变成逆时针的了.  

因此你所看到的三角形的面就与其三个顶点的绘制方向联系起来. 例如. 如果三角形是顺时针的, 那么你看到的就是三角形的A面, 如果三角形是逆时针的, 那么你看到的就是三角形的B面.  

在绘制3D模型时, 有些面是不用绘制的: 那些位于模型背部的面. 人们通常只会看到模型朝向摄像机的那部分, 因此模型背朝摄像机的那些面存不存在都无所谓.  

如果你可以确保模型中朝向摄像机的那部分(即模型在摄像机视角内可见的部分)的三角形都是逆时针的, 而背朝摄像机的那部分(即模型在摄像机视角内不可见的部分)的三角形都是顺时针的, 你就能要求显卡自动丢弃那些顺时针的三角形. 这项技术被称为 *背面剔除(backface culling)*. 你所使用的3D模型软件通常可以确定这项技术是否可以正确地应用到你的模型上.  

这项技术多数时候是为了优化. 通过在顶点着色器步骤之后舍弃掉半数三角形, 将片段着色器的调用次数减少了一半. 可以大大提高OpenGL的程序性能.  

## glium中的背面剔除

只需要在 `DrawParameters` 中增加一个变量.  

把:  

```rust
let params = glium::DrawParameters {
    depth: glium::Depth {
        test: glium::DepthTest::IfLess,
        write: true,
        .. Default::default()
    },
    .. Default::default()
};
```

换成:  

```rust
let params = glium::DrawParameters {
    depth: glium::Depth {
        test: glium::DepthTest::IfLess,
        write: true,
        .. Default::default()
    },
    backface_culling: glium::draw_parameters::BackfaceCullingMode::CullClockwise,
    .. Default::default()
};
```

然而, 我们并不打算对茶壶模型开启这项功能, 因为茶壶模型不是封闭的. 当你从壶口向里看时, 你将什么都看不见. 3D模型通常都是完全封闭的, 不过我们的茶壶模型不是.  