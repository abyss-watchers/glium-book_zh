

原地址: <https://github.com/glium/glium/blob/master/book/tuto-03-animated-triangle.md>

# 移动我们的三角形

上一节我们画了个三角形, 这一节我们打算试着让它动起来. 记住OpenGL不像绘画软件. 如果我们想改变屏幕上显示的内容, 我们必须再绘制一次, 然后用新绘制的内容替换掉已经存在的内容. 还好我们已经写好了一个 `loop` 循环, 它正不断地在窗口中重复绘制, 因此我们所写的代码几乎会立即反映到窗口上.    

# 比较naive的方法

第一个方法是创建一个名为 `t` 的变量并用它来表示移动中的每一步. 我们将在每次循环时更新 `t` 的值, 然后把它加到三角形的坐标值上:  

```rust
let mut t: f32 = -0.5;
let mut closed = false;
while !closed {
    // 更新 `t`
    t += 0.0002;
    if t > 0.5 {
        t = -0.5;
    }

    // 创建形状然后把 `t` 加到每个顶点的x坐标上
    let vertex1 = Vertex { position: [-0.5 + t, -0.5] };
    let vertex2 = Vertex { position: [ 0.0 + t,  0.5] };
    let vertex3 = Vertex { position: [ 0.5 + t, -0.25] };
    let shape = vec![vertex1, vertex2, vertex3];
    let vertex_buffer = glium::VertexBuffer::new(&display, &shape).unwrap();

    // 绘制
    let mut target = display.draw();
    target.clear_color(0.0, 0.0, 1.0, 1.0);
    target.draw(&vertex_buffer, &indices, &program, &glium::uniforms::EmptyUniforms,
                &Default::default()).unwrap();
    target.finish().unwrap();

    events_loop.poll_events(|event| {
        match event {
            glutin::Event::WindowEvent { event, .. } => match event {
                glutin::WindowEvent::CloseRequested => closed = true,
                _ => ()
            },
            _ => (),
        }
    });
}
```

如果你运行这段代码, 你就能看见我们的三角形从左边移动到右边, 然后又跳回左边.  

在上个世纪90年代, 很多游戏开发者都是这样做的. 在你的图形比较简单时(例如一个三角形), 这个方法确实好用. 但是一旦你的图形变得复杂, 例如某些有几千个多边形组成的3D模型, 这个方法的效率会变得特别低. 原因如下:  

 - 每次绘制时, CPU都会花大量时间来计算坐标(每个模型的每个顶点都进行一次计算的话, 每次绘制你将进行成百上千次计算).  

 - 将我们的形状的顶点数据从内存上传到显卡内存也要花费时间. GPU会等待所有数据上传完毕再开始绘制工作, 而等待的这段时间都被浪费掉了.  

# Uniform变量

还记得顶点着色器吗? 顶点着色器输入每个顶点的属性, 输出顶点在窗口中的位置. 之前, 我们在程序中让三角形顶点坐标值增加然后将计算的结果上传到GPU中, 现在我们把这件事交给GPU来做.  

现在把程序改回上一节结束时的样子, 不过依然保留 `t` :  

```rust
let vertex1 = Vertex { position: [-0.5, -0.5] };
let vertex2 = Vertex { position: [ 0.0,  0.5] };
let vertex3 = Vertex { position: [ 0.5, -0.25] };
let shape = vec![vertex1, vertex2, vertex3];

let vertex_buffer = glium::VertexBuffer::new(&display, &shape).unwrap();

let mut t: f32 = -0.5;
let mut closed = false;
while !closed {
    // 更新 `t`
    t += 0.0002;
    if t > 0.5 {
        t = -0.5;
    }

    // 绘制
    let mut target = display.draw();
    target.clear_color(0.0, 0.0, 1.0, 1.0);
    target.draw(&vertex_buffer, &indices, &program, &glium::uniforms::EmptyUniforms,
                &Default::default()).unwrap();
    target.finish().unwrap();

    events_loop.poll_events(|event| {
        match event {
            glutin::Event::WindowEvent { event, .. } => match event {
                glutin::WindowEvent::CloseRequested => closed = true,
                _ => ()
            },
            _ => (),
        }
    });
}
```

然后, 我们稍微修改一下顶点着色器的代码:  

```rust
let vertex_shader_src = r#"
    #version 140

    in vec2 position;

    uniform float t;

    void main() {
        vec2 pos = position;
        pos.x += t;
        gl_Position = vec4(pos, 0.0, 1.0);
    }
"#;
```

你也许注意到了, 这不就是我们之前在rust代码里写的操作吗, 只不过这次这段代码是在GPU上执行的. 我们在着色器的代码中添加了一个用 **uniform** 关键字声明的变量. uniform变量是一个全局变量, 它的值是在绘制时, 由 `draw` 函数传递给GPU. 现在, 我们可以用 `uniform!` 宏来实现:  

```rust
target.draw(&vertex_buffer, &indices, &program, &uniform! { t: t },
            &Default::default()).unwrap();
```

使用uniform变量解决了之前第一种方法的两个问题. CPU不必进行任何计算, 而且不必上传整个形状的顶点的数据给GPU, 只需上传 `t` 的值(一个单精度浮点数)就行了.  
