

原地址: <https://github.com/glium/glium/blob/master/book/tuto-01-getting-started.md>

# 创建项目

首先, 创建一个新项目:  

```sh
cargo new --bin my_project
cd my_project
```

在你刚创建的目录下的 `Cargo.toml` 文件中含有项目的元数据, `src/main.rs` 文件包含了Rust的源代码. 如果在目录下的文件不是 `src/main.rs` 而是 `src/lib.rs` , 说明你忘了输入 `--bin` 选项; 只要将 `src/lib.rs` 重命名为 `src/main.rs` 就可以了.  

在 `Cargo.toml` 文件中添加如下依赖, 以在项目中导入glium库:  

```toml
[dependencies]
glium = "*"
```

在使用依赖库之前, 需要在 `src/main.rs` 中添加如下代码以导入库:  

```rust
#[macro_use]
extern crate glium;

fn main() {
}
```

是时候开始填满 `main` 函数了!

# 创建窗口

创建图形应用的第一步就是创建一个窗口. 如果你熟悉OpenGL那一套, 你应该了解其中的复杂程度. 无论是创建窗口还是创建上下文(Context), 在不同平台下的你要写的代码是不一样的, 唯一不变的是: 它非常无聊. 幸运的是, 这正是 **glutin** 库所擅长的东西.  

使用glutin初始化OpenGL窗口需要经过如下几步:  

1. 创建 `EventsLoop` 用于处理窗口和设备的事件.  
2. 使用 `glium::glutin::WindowBuilder::new()` 指定窗口的参数. 这些参数与OpenGL无关, 而是窗口特有的属性.   
3. 使用`glium::glutin::ContextBuilder::new()`指定上下文(Context)参数. 我们可以在这里设置OpenGL特定的属性, 例如垂直同步与多重采样.    
4. 创建OpenGL窗口(在glium中称为 `Display`):  
    `glium::Display::new(window, context, &events_loop).unwrap()`  
    这句代码使用给定的窗口和上下文创建了一个Dispaly, 并且使用所给的events_loop注册窗口.  

```rust
fn main() {
    use glium::glutin;

    let mut events_loop = glutin::event_loop::EventsLoop::new();
    let window = glutin::WindowBuilder::new();
    let context = glutin::ContextBuilder::new();
    let display = glium::Display::new(window, context, &events_loop).unwrap();
}
```

但是有一个问题: 在窗口创建完成时, 我们的main函数就退出了, 导致 `display` 的析构函数关闭了窗口. 为了避免这个问题, 我们需要创建一个无限循环, 直到收到 `CloseRequested` 事件为止:  

```rust
events_loop.run(move |ev, _, control_flow| {
        let next_frame_time = std::time::Instant::now() + std::time::Duration::from_nanos(16_666_667);
        *control_flow = glutin::event_loop::ControlFlow::WaitUntil(next_frame_time);

        match ev {
            glutin::event::Event::WindowEvent { event, .. } => match event {
                glutin::event::WindowEvent::CloseRequested => {
                    *control_flow = glutin::event_loop::ControlFlow::Exit;
                    return;
                }
                _ => return,
            },
            _ => (),
        }
    });
```

虽然, 这段代码将导致CPU占用率达到100%, 但是已经解决了上面所说的问题. 在实际的应用程序中, 你应当使用垂直同步或者在每次循环结束时sleep几毫秒, 不过这是之后要考虑的事了.  

你现在可以运行 `cargo run`. 在Cargo下载完glium及其依赖并编译好之后, 你就能看见一个还过得去的小窗口了.  

### 清除颜色

然而, 窗口里的内容并不怎么吸引人. 它也许是一片空白, 或者是一张随机的图片, 也可能是一些雪花, 这取决于你使用什么系统. 我们需要在窗口内绘制图形, 因此系统并不需要将窗口内的颜色初始化为某个特定的值.  

Glium和OpenGL的API就像Windows下的画笔工具. 首先我们有一幅空白的画布, 我们可以在上面画一个又一个图形. 到你觉得满意为止. 和绘图软件不一样的是, 你不想你的用户看见绘画的过程. 他们能看见的应该只有最终的结果.  

glium使用 `Frame` 对象来实现这一机制. 如果你打算在窗口中绘制图形, 首先应该调用 `display.draw()` 方法来创建一个 `Frame`:   

```rust
let mut target = display.draw();
```

之后, 我们就能将 `target` 作为画布(drawing surface). OpenGL和glium提供的一个操作就是使用给定的颜色填充画布. 这正是我们要做的.  

```rust
target.clear_color(0.0, 0.0, 1.0, 1.0);
```

注意: 我们需要先导入 `Surface` trait, 然后才能使用这个函数:  

```rust
use glium::Surface;
```

我们传递给 `clear_color` 的四个值表示颜色的四个组成部分: 红, 绿, 蓝和透明度. 取值范围为[0.0, 1.0]. 这里我们选择的是不透明的蓝色.  

正如我之前解释的, 现在用户没法在屏幕上看见蓝色. 如果我们是在写一个真正的应用的话, 我们也许会画上角色, 武器, 地面, 天空等等. 但是在这个指南中, 我们做到这一步就足够了:  

```rust
target.finish().unwrap();
```

`finish()` 函数意味着我们结束了绘制. 它将销毁 `Frame` 对象, 并将画布复制到窗口上. 现在, 我们的窗口被一片蓝色填满了.  

以下是所有的代码:  

```rust
use glium::{glutin, Surface};

fn main() {
    // 1. The **winit::EventsLoop** for handling events.
    let events_loop = glutin::event_loop::EventLoop::new();
    // 2. Parameters for building the Window.
    let wb = glium::glutin::window::WindowBuilder::new()
        .with_inner_size(glium::glutin::dpi::LogicalSize::new(960.0, 640.0))
        .with_title("Hello world");
    // 3. Parameters for building the OpenGL context.
    let cb = glium::glutin::ContextBuilder::new().with_vsync(true);
    // 4. Build the Display with the given window and OpenGL context parameters and register the
    //    window with the events_loop.
    let display = glium::Display::new(wb, cb, &events_loop).unwrap();

    let gl_version = display.get_opengl_version();
    println!("{:?}", gl_version);

    events_loop.run(move |ev, _, control_flow| {
        let mut target = display.draw();
        target.clear_color(0.4, 0.5, 0.8, 0.8);
        target.finish().unwrap();

        let next_frame_time =
            std::time::Instant::now() + std::time::Duration::from_nanos(16_666_667);
        *control_flow = glutin::event_loop::ControlFlow::WaitUntil(next_frame_time);

        match ev {
            glutin::event::Event::WindowEvent { event, .. } => match event {
                glutin::event::WindowEvent::CloseRequested => {
                    *control_flow = glutin::event_loop::ControlFlow::Exit;
                    return;
                }
                _ => return,
            },
            _ => (),
        }
    });
}

```
