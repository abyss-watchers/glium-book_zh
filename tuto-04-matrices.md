

原地址: <https://github.com/glium/glium/blob/master/book/tuto-04-matrices.md>

# 矩阵

在上一节, 我们改写代码来让三角形从屏幕左边移动到右边. 如果我们想旋转, 拉伸或者缩放我们的三角形时, 我们该怎么做呢?  

我们想要的所有几何变换都能通过一些数学公式来完成:  

 - `position *= factor;` 可以缩放三角形.  
 - `new_position = vec2(pos.x * cos(angle) - pos.y * sin(angle), pos.x * sin(angle) + pos.y * cos(angle));` 可以旋转三角形.  
 - `position.x += position.y * factor;` 可以拉伸三角形

但是如果我们想做一些复合变换, 例如先旋转再移动再缩放, 或者先拉伸再旋转, 这时我们该如何做? 虽然你也可以用上面几个数学公式来解决这个问题, 但是事情会变得很复杂.  

因此, 程序员们通常使用 **matrices**(矩阵) 来解决复杂的变换. 矩阵是一个二维的数表, 可以用来表示*几何变换*. 在计算机图形学中, 我们通常使用4x4的矩阵.  

回到上一节我们写的移动的三角形. 现在我们打算用矩阵来修改顶点着色器的代码. 我们会用一个矩阵变量乘每个顶点的坐标, 来替代上一节使用的变量 `t` . 这会将矩阵所描述的几何变换应用到顶点坐标上.  

```rust
let vertex_shader_src = r#"
    #version 140

    in vec2 position;

    uniform mat4 matrix;

    void main() {
        gl_Position = matrix * vec4(position, 0.0, 1.0);
    }
"#;
```

注意: 矩阵的乘法并不满足交换律, 因此 `matrix * vertex` 和 `vertex * matrix` 的结果是不一样的.  

我们需要在调用 `draw` 函数时将矩阵的值传递给GPU.  

```rust
let uniforms = uniform! {
    matrix: [
        [1.0, 0.0, 0.0, 0.0],
        [0.0, 1.0, 0.0, 0.0],
        [0.0, 0.0, 1.0, 0.0],
        [ t , 0.0, 0.0, 1.0f32],
    ]
};

target.draw(&vertex_buffer, &indices, &program, &uniforms,
            &Default::default()).unwrap();
```

修改后的代码应该和之前的表现一致, 不过现在的写法更加灵活. 例如, 如果我们想旋转三角形, 我们只需修改矩阵的值:  

```rust
let uniforms = uniform! {
    matrix: [
        [ t.cos(), t.sin(), 0.0, 0.0],
        [-t.sin(), t.cos(), 0.0, 0.0],
        [0.0, 0.0, 1.0, 0.0],
        [0.0, 0.0, 0.0, 1.0f32],
    ]
};
```

**[完整的代码在此](https://github.com/glium/glium/blob/master/examples/tutorial-04.rs).**
