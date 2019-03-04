

原地址: <https://github.com/glium/glium/blob/master/book/tuto-06-texture.md>

# 上传纹理

纹理是由显卡内存加载的一张图片或者图片的集合.  

为了加载纹理, 我们首先要将图片解码为字节序列以储存. 因此, 我们打算使用 `image` 库. 现在我们将其添加到 Cargo.toml 文件中:  

```toml
[dependencies]
image = "*"
```

然后在 crate根 下添加:  

```rust
extern crate image;
```

为了加载图片, 我们需要使用 `image::load` 这个函数:  

```rust
use std::io::Cursor;
let image = image::load(Cursor::new(&include_bytes!("/path/to/image.png")[..]),
                        image::PNG).unwrap().to_rgba();
let image_dimensions = image.dimensions();
let image = glium::texture::RawImage2d::from_raw_rgba_reversed(&image.into_raw(), image_dimensions);
```

之后, 将图片作为纹理上传:  

```rust
let texture = glium::texture::Texture2d::new(&display, image).unwrap();
```

# 使用纹理

和其他渲染技术一样, OpenGL中并没有在图形上自动显示纹理的方法, 而是需要你手动来做. 这也意味着我们必须手动从纹理中加载颜色值然后将其作为片段着色器的返回值.  

为了做到这一点, 我们首先修改以下我们的形状, 以指示每个顶点附着在纹理的什么位置上:  

```rust
#[derive(Copy, Clone)]
struct Vertex {
    position: [f32; 2],
    tex_coords: [f32; 2],       // <- 这里是新加的代码
}

implement_vertex!(Vertex, position, tex_coords);        // 别忘了在这添加 `tex_coords`

let vertex1 = Vertex { position: [-0.5, -0.5], tex_coords: [0.0, 0.0] };
let vertex2 = Vertex { position: [ 0.0,  0.5], tex_coords: [0.0, 1.0] };
let vertex3 = Vertex { position: [ 0.5, -0.25], tex_coords: [1.0, 0.0] };
let shape = vec![vertex1, vertex2, vertex3];
```

纹理坐标的范围在 `0.0` 到 `1.0` 之间. 坐标 `(0.0, 0.0)` 指的是纹理的左下角, 坐标 `(1.0, 1.0)` 指的是纹理的右上角.  

新的 `tex_coords` 属性将像 `position` 一样传递给顶点着色器. 我们不对其进行修改, 直接将其传递给片段着色器:  

```glsl
#version 140

in vec2 position;
in vec2 tex_coords;
out vec2 v_tex_coords;

uniform mat4 matrix;

void main() {
    v_tex_coords = tex_coords;
    gl_Position = matrix * vec4(position, 0.0, 1.0);
}
```

和 `my_attr` 类似, `v_tex_coords` 的值将会被插值(interpolate), 这样每个像素都能找到其在纹理上的坐标. `v_tex_coords` 的值与此像素附着到纹理上的坐标相对应.  

接下来在片段着色器中, 我们调用OpenGL的 `texture()` 函数来获取纹理上该坐标点的颜色值.  

```glsl
#version 140

in vec2 v_tex_coords;
out vec4 color;

uniform sampler2D tex;

void main() {
    color = texture(tex, v_tex_coords);
}
```

正如你看见的, 纹理是一个 `sampler2D` 类型的 `uniform` 变量. GLSL提供许多种供纹理对象使用的数据类型, `sampler2D` 指的是简单的二维纹理.  

因为纹理是一个uniform变量. 因此在绘制时, 我们必须将之前创建的纹理对象的引用传递给这个uniform变量:  

```rust
let uniforms = uniform! {
    matrix: [
        [1.0, 0.0, 0.0, 0.0],
        [0.0, 1.0, 0.0, 0.0],
        [0.0, 0.0, 1.0, 0.0],
        [ t , 0.0, 0.0, 1.0f32],
    ],
    tex: &texture,
};
```

这里是最终运行结果:  

![结果](tuto-06-texture.png)

**[完整代码在此](https://github.com/glium/glium/blob/master/examples/tutorial-06.rs).**
